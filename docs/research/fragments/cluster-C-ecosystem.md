# Cluster C : Ecosystem, Performance & Ops

> Phase 2 deep research fragment. Verified against approved sources on 2026-05-20.
> Target version : CesiumJS 1.124+.

## 1. Cesium ion platform

Cesium ion is an open platform for tiling, hosting, and serving geospatial data as 3D Tiles. It accepts 3D buildings, AEC models, imagery, photogrammetry, point clouds, and terrain, then runs them through a tiling pipeline that produces streamable 3D Tiles or quantized-mesh terrain. ion ships in two deployment forms : ion SaaS (free signup, hosted by Cesium) and ion Self-Hosted (enterprise, by request). The REST API and most platform features behave identically across both.

Authentication uses ion access tokens. In CesiumJS, `Ion.defaultAccessToken` sets a process-wide token; individual services accept a per-call `accessToken` and `server` (default `Ion.defaultServer`). The bridge class is `IonResource` : a `Resource` subclass not normally constructed directly. The factory `IonResource.fromAssetId(assetId, options)` returns a `Promise<IonResource>` that resolves to a streamable resource handle. Typical flow : `const resource = await IonResource.fromAssetId(123); const tileset = await Cesium3DTileset.fromUrl(resource);`. `IonImageryProvider.fromAssetId()` covers imagery assets. `Cesium3DTileset.fromIonAssetId()` is a direct shortcut that wraps the IonResource step.

The ion REST API exposes asset listing, creation, and upload (the docs reference "Upload your 3D data with REST" and OAuth2). The upload flow involves creating an asset record, pushing source data to ion storage, and triggering tiling, then polling asset status. Free tier covers signup plus a quota of stored/streamed data and access to Cesium-hosted base layers (Cesium World Terrain, Bing/Cesium imagery, OSM Buildings); paid tiers unlock higher quotas, private assets, and self-hosting. Self-hosting alternatives outside ion : serve quantized-mesh terrain via `cesium-terrain-server`, host 3D Tiles on any static CDN with correct CORS, or use `CesiumTerrainProvider.fromUrl()` against a custom endpoint. The exact REST endpoint paths could not be verified from the public ion docs page and must be confirmed against the dedicated REST API reference before any skill quotes them.

## 2. Geocoding and routing

CesiumJS geocoding is the `GeocoderService` interface. `IonGeocoderService` implements it, backed by ion's geocoding infrastructure. Constructor options : `scene` (required), `accessToken` (defaults to `Ion.defaultAccessToken`), `server`, and `geocodeProviderType` (defaults to `IonGeocodeProviderType.DEFAULT`). Its `geocode(query, type)` method takes a query string plus optional `GeocodeType` (default `GeocodeType.SEARCH`) and returns `Promise<Array<GeocoderService.Result>>`. A readonly `credit` member supplies attribution.

The `Viewer` constructor `geocoder` option accepts `boolean | IonGeocodeProviderType | Array<GeocoderService>` (default `IonGeocodeProviderType.DEFAULT`). Passing `false` suppresses the Geocoder widget; passing an array of custom `GeocoderService` implementations replaces the backend. The read-only `viewer.geocoder` property exposes the widget. Custom geocoders are built by implementing `geocode()` on the `GeocoderService` interface. CRITICAL : Cesium ion and CesiumJS provide NO routing/directions capability. Geocoding is forward and reverse address-to-coordinate only. Any routing skill claim would be a hallucination; routing requires an external service (e.g. OSRM, a Mapbox/HERE API) feeding a `Polyline`/`Entity`.

## 3. Resium : React wrapper

Resium (`resium.reearth.dev`, repo `reearth/resium`) is a declarative React component library wrapping CesiumJS. It supports hot module replacement and ships full TypeScript types. Components map to Cesium objects : `Viewer` (container, e.g. `<Viewer full>`), `CesiumWidget`, `Entity` (with `position`, `point`, `name`, `description` props), `Cesium3DTileset`, plus imagery/terrain/datasource components. CesiumJS is a peer dependency, so the app pins its own CesiumJS version independently of Resium. The `useCesium` hook reads context (viewer, scene, camera) from inside child components. Native object access uses the `ref` pattern : a `CesiumComponentRef<T>` exposes `.cesiumElement`, giving the underlying Cesium instance for imperative calls. NOTE : the reearth-hosted docs site was unreachable during research (ECONNREFUSED); component/hook names above are confirmed from the GitHub repo but exact peer-dependency version ranges must be re-verified from `package.json` before a skill pins them.

## 4. AEC georeferencing

The core AEC problem is placing BIM/city models into Earth-Centered-Earth-Fixed (ECEF) space. CityGML is not directly loadable; the verified workflow is to convert CityGML to 3D Tiles via ion's tiler (or FME/external tooling) and stream the result. IFC has no native loader either : the practical path is exporting IFC to glTF (via ThatOpen/IfcOpenShell), then georeferencing the glTF.

Model orientation is official API. `Transforms.eastNorthUpToFixedFrame(origin, ellipsoid, result)` builds a `Matrix4` from a local east-north-up frame to ECEF. `Transforms.headingPitchRollToFixedFrame(origin, headingPitchRoll, ellipsoid, fixedFrameTransform, result)` adds rotation from a `HeadingPitchRoll`. `Transforms.localFrameToFixedFrameGenerator(firstAxis, secondAxis)` produces custom frame functions. The resulting `Matrix4` is assigned to `Model.modelMatrix` (or `Cesium3DTileset.modelMatrix`) to place and orient the asset. Geo-BIM and digital-twin patterns combine a georeferenced tileset, clipping planes/`ClippingPolygon`, and feature styling. FLAG : the conversion pipelines (CityGML/IFC to 3D Tiles) are workflow, not CesiumJS API; only `Transforms`, `Matrix4`, `HeadingPitchRoll`, and `modelMatrix` are official API.

## 5. Performance

`Scene.requestRenderMode` renders frames only when scene state changes, paired with `maximumRenderTimeChange` (max simulation-time delta before a render is forced) and explicit `scene.requestRender()` calls. `Scene.fog` blends distant geometry and reduces terrain requests. `Scene.logarithmicDepthBuffer` cuts multi-frustum count. `Scene.msaaSamples` (1 disables MSAA; MSAA defaults to 4 samples since 1.121). `Scene.debugShowFramesPerSecond` overlays FPS and frame time. For tilesets, `maximumScreenSpaceError` (default 16) drives LOD : higher means coarser and faster. `foveatedScreenSpaceError`, `skipLevelOfDetail`, `preferLeaves`, and `preloadWhenHidden` tune traversal. `RequestScheduler` throttles concurrent network requests (per-server limits). `TaskProcessor` wraps Web Workers : `scheduleTask(parameters, transferableObjects)` returns a `Promise` or `undefined` when `maximumActiveTasks` is exceeded; the worker is lazily created on first task.

## 6. Memory management

WebGL resources are not garbage-collected; explicit teardown is required. `viewer.destroy()` and `viewer.isDestroyed()` tear down the widget; `Cesium3DTileset.destroy()` / `isDestroyed()` free tile GPU memory. ALWAYS call `destroy()` on primitives, tilesets, data sources, and the viewer before dropping references; after `destroy()` the object must not be used. Tileset memory budget is `cacheBytes` (default 512 MiB) plus `maximumCacheOverflowBytes`. Known leak patterns from GitHub issues : large tilesets where tile shells persist even after content is freed (issue #12049), CPU memory not released after repeated add/remove (often a lost reference, not a Cesium bug, since Cesium pools memory), XHR responses retained by `Request` (#8843), and historical `CesiumWidget`/CLAMP_TO_GROUND entity-removal leaks (since fixed). WebGL context loss : `Scene` exposes a `renderError` event and `rethrowRenderErrors`; `contextOptions` configures WebGL attributes. Context-loss recovery is not richly documented and should be flagged as a known limitation.

## 7. Build and deployment

CesiumJS ships as the `cesium` npm package. It requires four static directories served separately : `Build/Cesium/Workers`, `Build/Cesium/ThirdParty`, `Build/Cesium/Assets`, `Build/Cesium/Widgets`. `window.CESIUM_BASE_URL` MUST be set before importing CesiumJS and point to where those directories are served. Vite : use `vite-plugin-cesium`. Webpack : use `CopyWebpackPlugin` to copy the four directories plus a `DefinePlugin` for `CESIUM_BASE_URL`; the official `cesium-webpack-example` repo is the reference. ES6 named imports are the production pattern (`import { Viewer, Cartesian3 } from 'cesium'`). TypeScript types ship with the package.

## 8. Sandcastle

Sandcastle (`sandcastle.cesium.com`) is the official live code-example gallery. Each example is a JS snippet plus an optional HTML pane; Sandcastle auto-provides the `Cesium` global, the HTML scaffold, and a demo ion token. Converting to production : replace the implicit global with ES6 named imports, supply your own `Ion.defaultAccessToken`, set `CESIUM_BASE_URL`, and provide real HTML structure. The Sandcastle source lives in `cesium/packages/sandcastle`.

## Newly Discovered Sub-Topics

Topics surfaced in research but absent from the raw masterplan inventory :

- `IonGeocodeProviderType` enum and the `geocoder` option accepting `Array<GeocoderService>` (custom geocoder injection).
- `RequestScheduler` per-server throttling as a distinct performance lever.
- `TaskProcessor` Web Worker API (`scheduleTask`, `maximumActiveTasks`) for custom worker offloading.
- `cacheBytes` / `maximumCacheOverflowBytes` GPU memory budgeting (replaced the older `maximumMemoryUsage`).
- `ClippingPolygon` / `ClippingPolygonCollection` (added 1.117) for arbitrary-shape clipping.
- `Ellipsoid.default` global default (1.119) and `Scene.verticalExaggeration` migration (1.113).
- `EdgeDisplayMode` / `EXT_mesh_primitive_edge_visibility` for CAD-style edge rendering (1.135+) : relevant to AEC.
- Voxel rendering : `VoxelProvider`, `EXT_primitive_voxels` (1.127+).
- `renderError` event + `rethrowRenderErrors` for render-loop error handling.
- The `defaultValue()` removal (1.134) affecting any copied legacy code.

## Version Differences

| Version | Change | Impact |
|---------|--------|--------|
| 1.104 | Async factory constructors : `Cesium3DTileset.fromUrl/fromIonAssetId`, `IonImageryProvider.fromAssetId`, `Model.fromGltfAsync`, `createOsmBuildingsAsync`. `readyPromise` deprecated. | Synchronous `new` + `readyPromise` patterns break. |
| 1.107 | `createWorldTerrainAsync` replaces `createWorldTerrain`; `readyPromise` removed; old Model patterns removed. | Hard break for pre-1.104 code. |
| 1.113 | Vertical exaggeration moved to `Scene.verticalExaggeration` / `verticalExaggerationRelativeHeight`. | `Globe`-based exaggeration code breaks. |
| 1.114-1.115 | `HeightReference` gains `CLAMP_TO_TERRAIN` / `CLAMP_TO_3D_TILE`; tileset collision on by default; `disableCollision` to `enableCollision`. | Clamping/collision behavior shifts. |
| 1.117 | `ClippingPolygon` / `ClippingPolygonCollection` added. | New clipping capability. |
| 1.119 | `Ellipsoid.default` introduced; constructor defaults shift. | Implicit ellipsoid behavior changes. |
| 1.121 | MSAA on by default (4 samples); PBR Neutral tonemap default; higher fog density. | Visual output differs from older versions. |
| 1.127 | `VoxelProvider.requestData()` returns `Promise<VoxelContent>`; `EXT_primitive_voxels`. | Voxel API break. |
| 1.134 | `defaultValue()` utility removed (use `??`). | Legacy copied code breaks. |
| 1.135-1.142 | `EXT_mesh_primitive_edge_visibility`, `EdgeDisplayMode` enum for CAD edges. | New AEC-relevant capability. |
| 1.139 | `Cartesian2/3/4` converted to ES6 classes; static factories cannot use `new`. | Subtle break for code calling `new` on factories. |

## Anti-patterns and cross-tech boundaries

Real-world mistakes (from CesiumGS/cesium issues and docs) :

1. **`CESIUM_BASE_URL` not set / wrong path** : Workers, Assets, Widgets 404; globe is blank or terrain never loads. Must be set before import.
2. **Bundler asset-copy omitted** : forgetting `CopyWebpackPlugin` / `vite-plugin-cesium` ships JS without the four static directories : runtime worker failures.
3. **Missing or invalid ion token** : `Ion.defaultAccessToken` unset causes 401 on World Terrain, OSM Buildings, ion assets.
4. **Not calling `destroy()`** : dropping a tileset/viewer reference without `destroy()` leaks GPU memory (issues #7858, #12049).
5. **Lost references during add/remove** : repeatedly adding/removing tilesets while losing the handle prevents cleanup; appears as a leak but is a usage bug.
6. **Using deprecated sync constructors** : `new Cesium3DTileset({url})` + `readyPromise` copied from old Sandcastle examples fails on 1.124+.

Cross-tech boundaries : QGIS is the geospatial-data source/sink (export GeoJSON, imagery, terrain inputs to the ion tiler); ThatOpen/IfcOpenShell handles BIM-to-glTF export feeding the georeferencing workflow; Speckle federates multi-discipline data, exported to glTF/3D Tiles for streaming; Three.js is the general-purpose alternative renderer (CesiumJS is geo-first with a true WGS84 globe, ECEF math, and streaming 3D Tiles, which Three.js lacks). These remain README companion notes, not separate skills.

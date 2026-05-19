# Vooronderzoek : CesiumJS

> Status : Phase 2 complete
> Generated : 2026-05-19 | Researched : 2026-05-20
> Target version : CesiumJS 1.124+ (current release line reaches ~1.142, ref-doc at 1.141)
> Verification : all API names verified via WebFetch against SOURCES.md approved URLs
> Source fragments : `docs/research/fragments/cluster-A-rendering.md`, `cluster-B-data.md`, `cluster-C-ecosystem.md`

This document consolidates three parallel research clusters into the single
vooronderzoek that drives Phase 3 masterplan refinement.

## 1. Architecture and Runtime Model

CesiumJS layers a WebGL2 renderer beneath an application widget. `CesiumWidget` is the
foundational rendering component: it owns an `HTMLCanvasElement`, a `Scene`, a `Camera`,
a `Clock`, and a `ScreenSpaceEventHandler`, and runs the render loop. `Viewer` is a
higher-level composite that wraps a `CesiumWidget` and adds UI widgets (Animation,
Timeline, BaseLayerPicker, Geocoder, InfoBox, SelectionIndicator, etc.).

`Scene` is the container for all 3D state: it holds the `Globe` (the depth-test
ellipsoid surface rendering imagery and terrain), `skyAtmosphere`, `skyBox`, `sun`,
`moon`, `light`, `fog`, `postProcessStages`, the `Camera`, the primitive collections,
and the `ScreenSpaceCameraController`. `Globe` is a member of `Scene`, NOT a sibling.
This containment hierarchy is the single most important architectural fact for skills:
`viewer.scene.globe`, `viewer.scene.camera`, `viewer.scene.primitives`.

The render loop is driven by `Scene.render(time)`, which fires `preUpdate`,
`postUpdate`, `preRender`, and `postRender` events each frame. By default the loop runs
continuously via `requestAnimationFrame`. With `requestRenderMode: true`, frames render
only on scene change and the app must call `scene.requestRender()`. `maximumRenderTimeChange`
bounds how much simulation time may pass before an auto-render. `SceneMode` has three
values: `SCENE3D`, `SCENE2D`, `COLUMBUS_VIEW` (2.5D), with `morphTo*()` transitions.

CRITICAL CORRECTION: CesiumJS renders EXCLUSIVELY on WebGL2 through the current 1.142
line. WebGPU is a long-term roadmap item, NOT shipped, and there is no WebGPU backend
toggle. The original project brief said "WebGL2 + WebGPU backend (in transition)";
research (Cesium Community, issue #4989) proves this is wrong. Skills MUST treat WebGL2
as the only backend and MUST NOT reference a WebGPU path. This is recorded as L-001.

### Design philosophy

CesiumJS is geo-first: a true WGS84 globe, ECEF math, streaming 3D Tiles, and
time-dynamic data are first-class. This distinguishes it from general 3D engines
(Three.js) that have no geospatial coordinate model. Two API tiers coexist: the
high-level retained-mode Entity API (declarative, time-dynamic, easy) and the
low-level Primitive API (batched, fast, static).

## 2. API Surface

### Viewer and rendering
`new Cesium.Viewer(container, options)`. Boolean widget toggles: `animation`,
`baseLayerPicker`, `fullscreenButton`, `vrButton`, `geocoder`, `homeButton`, `infoBox`,
`sceneModePicker`, `selectionIndicator`, `timeline`, `navigationHelpButton`. Rendering
options: `scene3DOnly`, `requestRenderMode`, `maximumRenderTimeChange`, `contextOptions`
(raw WebGL flags incl. `requestWebgl2`, `powerPreference`), `sceneMode`, `msaaSamples`,
`useBrowserRecommendedResolution`, `targetFrameRate`. Minimal UI-free setup uses
`new Cesium.CesiumWidget(container, options)`.

### Camera
`setView({destination, orientation})`, `flyTo({destination, orientation, duration,
complete, cancel, easingFunction})`, `lookAt(target, offset)`, `lookAtTransform`,
`flyToBoundingSphere`, `viewBoundingSphere`. `destination` is a `Cartesian3` or
`Rectangle`; `orientation` is direction/up vectors or `HeadingPitchRoll`. Frustums:
`PerspectiveFrustum` (default), `OrthographicFrustum`, `PerspectiveOffCenterFrustum`.

### Coordinate systems
`Cartesian3` is an ECEF position in meters. Factories: `Cartesian3.fromDegrees(lon,
lat, height?, ellipsoid?)`, `fromRadians`, `fromDegreesArray`, `fromDegreesArrayHeights`.
`Cartographic` stores longitude/latitude in RADIANS plus height in meters;
`Cartographic.fromCartesian`, `fromDegrees`, `fromRadians`. Default ellipsoid is WGS84
(`Ellipsoid.WGS84`, configurable `Ellipsoid.default`). `Transforms.eastNorthUpToFixedFrame(origin)`
and `Transforms.headingPitchRollToFixedFrame(origin, hpr)` build local-frame `Matrix4`s.

### Imagery and terrain providers
Imagery: `ImageryLayer` + provider subclasses `BingMapsImageryProvider`,
`ArcGisMapServerImageryProvider`, `IonImageryProvider`, `OpenStreetMapImageryProvider`,
`TileMapServiceImageryProvider`, `UrlTemplateImageryProvider`, `WebMapServiceImageryProvider`,
`WebMapTileServiceImageryProvider`, `SingleTileImageryProvider`. MapTiler is consumed via
`UrlTemplateImageryProvider` or WMTS (no dedicated class). Terrain:
`CesiumTerrainProvider.fromUrl` / `.fromIonAssetId`, `createWorldTerrainAsync`,
`ArcGISTiledElevationTerrainProvider.fromUrl`, `EllipsoidTerrainProvider` (sync fallback).
Helper classes `Terrain` and `ImageryLayer.fromProviderAsync` / `.fromWorldImagery` wrap
async loading for the `Viewer` options.

### Entity API
`new Cesium.Entity(options)` with graphics objects `PointGraphics`, `BillboardGraphics`,
`LabelGraphics`, `PolylineGraphics`, `PolygonGraphics`, `ModelGraphics`, `PathGraphics`,
plus shapes (`BoxGraphics`, `CylinderGraphics`, `EllipsoidGraphics`, `WallGraphics`,
`CorridorGraphics`, `RectangleGraphics`, `PolylineVolumeGraphics`). Driven by `Property`
objects: `ConstantProperty`, `SampledProperty`, `CallbackProperty(callback, isConstant)`,
`CallbackPositionProperty`, `TimeIntervalCollectionProperty`. Material properties include
`ColorMaterialProperty`, `ImageMaterialProperty`, `PolylineGlowMaterialProperty`
(`color`, `glowPower` 0.25, `taperPower` 1.0), `PolylineDashMaterialProperty`.

### Primitive API
`new Cesium.Primitive({geometryInstances, appearance, modelMatrix, asynchronous})`.
`GeometryInstance` wraps a `Geometry` + per-instance attributes + pickable `id`.
Appearances: `PerInstanceColorAppearance`, `MaterialAppearance`, `EllipsoidSurfaceAppearance`,
`PolylineColorAppearance`. `GroundPrimitive` drapes geometry over terrain/tiles via
`ClassificationType` (`TERRAIN`, `CESIUM_3D_TILE`, `BOTH`). `ClassificationPrimitive` is
the broader classification primitive. `scene.primitives` is the `PrimitiveCollection`.

### 3D Tiles
`Cesium3DTileset.fromUrl(url, options)` / `.fromIonAssetId(assetId, options)` (async,
mandatory). Properties: `style` (`Cesium3DTileStyle`), `maximumScreenSpaceError`
(default 16), `clippingPlanes` (`ClippingPlaneCollection`), `clippingPolygons`
(`ClippingPolygonCollection`). `Cesium3DTileStyle` uses the 3D Tiles styling expression
language (string expressions, `conditions` arrays, `evaluate` functions; keys `color`,
`show`, `pointSize`). 3D Tiles 1.0 binary formats (`b3dm`, `i3dm`, `pnts`, `cmpt`) are
deprecated in 3D Tiles 1.1 in favor of glTF/glb directly as tile content, with
structured metadata, implicit tiling, and multiple contents per tile. CesiumJS 1.124+
reads both 1.0 and 1.1. `Cesium3DTileFeature` is a picked feature.

### glTF / Model
`Model.fromGltfAsync(options)` is the production async renderer. `ModelExperimental` no
longer exists (merged into `Model`). `modelMatrix` places the model;
`activeAnimations` (`ModelAnimationCollection`) plays glTF animations; articulations via
`setArticulationStage(key, value)` + `applyArticulations()`. glTF 2.0 with 20+ extensions.

### Time-dynamic data
`JulianDate` (TAI-internal, leap-second-safe): `fromDate`, `fromIso8601`, `now`,
`addSeconds`, `compare`. `Clock` + `ClockViewModel`. `SampledPositionProperty`:
`addSample(time, position, derivatives)`, `setInterpolationOptions({interpolationAlgorithm,
interpolationDegree})` with `LinearApproximation`, `LagrangePolynomialApproximation`,
`HermitePolynomialApproximation`. `TimeIntervalCollection` for `Entity.availability`.

### Picking
`scene.pick(windowPosition)` (topmost, `.primitive` or `Cesium3DTileFeature`),
`scene.drillPick` (all), `scene.pickPosition` (depth-buffer `Cartesian3`, needs
`pickPositionSupported`), `globe.pick(ray, scene)`, `scene.sampleHeight`,
`scene.clampToHeight`. `ScreenSpaceEventHandler.setInputAction(action, ScreenSpaceEventType,
modifier)` with types `LEFT_CLICK`, `MOUSE_MOVE`, `WHEEL`, etc.

### DataSources
`viewer.dataSources.add(promise)`. `CzmlDataSource.load` / instance `process` (append)
vs `load` (replace). `GeoJsonDataSource.load` handles GeoJSON + TopoJSON, options
`clampToGround` (default FALSE), `stroke`, `fill`, `markerColor`, simplestyle-spec.
`KmlDataSource` loads KML/KMZ, requires `camera` and `canvas` options.

### Materials, shaders, atmosphere
`Material.fromType('Color', {...})` or `new Material({fabric})`; built-ins include
`Color`, `Image`, `Grid`, `Water`, `PolylineGlow`, elevation ramps. `CustomShader`
(`mode`, `lightingModel` UNLIT/PBR, `uniforms`, `vertexShaderText`, `fragmentShaderText`)
for `Model`/`Cesium3DTileset`. `Scene.postProcessStages` (`PostProcessStageCollection`)
+ custom `PostProcessStage`. `Scene.skyAtmosphere`, `skyBox`, `sun`, `moon`, `fog`,
`light` (`SunLight` default, `DirectionalLight`); `Globe.enableLighting`,
`showGroundAtmosphere`.

## 3. Version Matrix

| Version | Change | Impact |
|---------|--------|--------|
| 1.104 | Async factory constructors; `readyPromise` deprecated | Sync `new` + `readyPromise` patterns break |
| 1.107 | `createWorldTerrainAsync`; `readyPromise` removed | Hard break for pre-1.104 code |
| 1.113 | Vertical exaggeration moved to `Scene.verticalExaggeration` | Globe-based exaggeration breaks |
| 1.114-1.115 | `HeightReference` gains `CLAMP_TO_TERRAIN`/`CLAMP_TO_3D_TILE`; `disableCollision`→`enableCollision` | Clamping/collision shifts |
| 1.117 | `ClippingPolygon` / `ClippingPolygonCollection` added | New clipping capability |
| 1.119 | `Ellipsoid.default` introduced | Implicit ellipsoid behavior changes |
| 1.121 | MSAA on by default (4 samples); PBR Neutral tonemap default | Visual output differs |
| 1.123 | `entities`, `dataSources`, `zoomTo`, `flyTo` moved to `CesiumWidget` | API surface migration |
| 1.127 | Voxel API (`VoxelProvider`, `EXT_primitive_voxels`) | New experimental API |
| 1.134 | `defaultValue()` utility removed (use `??`) | Legacy copied code breaks |
| 1.135-1.142 | `EdgeDisplayMode`, `EXT_mesh_primitive_edge_visibility` | New AEC CAD-edge capability |
| 1.139 | `Cartesian2/3/4` converted to ES6 classes | `new` on static factories breaks |

The async-factory migration (1.104/1.107) is the single biggest cross-cutting break: it
affects `Model`, `Cesium3DTileset`, terrain and imagery providers. Since target is
1.124+, the async pattern is the ONLY valid pattern; skills must never emit `readyPromise`
or `new Cesium3DTileset({url})`.

## 4. Configuration and Security

Cesium ion tokens: `Cesium.Ion.defaultAccessToken = '<token>'` MUST be set before
constructing a `Viewer` that uses any ion asset (World Terrain, OSM Buildings, ion
imagery/geocoder). Missing token = blank/black globe + 401. ion ships as SaaS (free
signup + quota) and Self-Hosted (enterprise). `IonResource.fromAssetId(id)` bridges ion
assets to streamable resources; `Cesium3DTileset.fromIonAssetId` is the shortcut.

Build: CesiumJS ships as the `cesium` npm package and requires four static directories
served separately (`Workers`, `ThirdParty`, `Assets`, `Widgets`). `window.CESIUM_BASE_URL`
MUST be set before import. Vite uses `vite-plugin-cesium`; webpack uses `CopyWebpackPlugin`
+ `DefinePlugin` (reference: `cesium-webpack-example`). ES6 named imports are the
production pattern. TypeScript types ship with the package.

## 5. Common Workflows

Geocoding: `GeocoderService` interface, `IonGeocoderService` implementation, `Viewer`
`geocoder` option accepts `boolean | IonGeocodeProviderType | Array<GeocoderService>`.
CRITICAL: CesiumJS/ion provide NO routing/directions. Geocoding is address-to-coordinate
only. Any routing capability requires an external service.

AEC georeferencing: CityGML and IFC have no native loader. Workflow: convert CityGML to
3D Tiles (ion tiler / FME) and IFC to glTF (ThatOpen/IfcOpenShell), then georeference
via `Transforms.eastNorthUpToFixedFrame` / `headingPitchRollToFixedFrame` producing a
`Matrix4` assigned to `Model.modelMatrix` / `Cesium3DTileset.modelMatrix`. Only the
`Transforms`/`Matrix4`/`HeadingPitchRoll`/`modelMatrix` parts are CesiumJS API; the
conversion pipelines are external workflow.

Resium (React wrapper): declarative components (`Viewer`, `Entity`, `Cesium3DTileset`),
`useCesium` hook, `CesiumComponentRef.cesiumElement` for imperative access. CesiumJS is
a peer dependency. (Resium docs site was unreachable during research; peer-version
ranges need re-verification in Phase 4 from `package.json`.)

Sandcastle (`sandcastle.cesium.com`): live example gallery; auto-provides the `Cesium`
global and a demo token. Production conversion: ES6 named imports, own ion token,
`CESIUM_BASE_URL`, real HTML.

### Entity vs Primitive selection

This is the most common architectural decision in CesiumJS code. The Entity API is
retained-mode and declarative: an `Entity` carries time-dynamic `Property` values, is
visualized automatically by `DataSourceDisplay`, and is picked back as `entity`. It is
the correct default for interactive features, time-dynamic tracks, and datasets below
roughly ten thousand objects. The Primitive API is lower-level: geometry is batched into
a small number of draw calls, built asynchronously on a worker, and is static once
created. It is the correct choice for large static datasets where the per-tick
`EntityCollection` visualizer loop becomes the bottleneck. The verified rule for skills:
ALWAYS prefer Entity for interactive or time-dynamic data under ~10k objects; use
batched `GeometryInstance` Primitives beyond that scale. 3D Tiles supersedes both for
massive streamed datasets (cities, point clouds, photogrammetry).

### Typical bootstrap sequence

A correct CesiumJS application start is: set `window.CESIUM_BASE_URL` before importing
`cesium`; set `Cesium.Ion.defaultAccessToken`; construct the `Viewer` with the desired
widget toggles; `await` async terrain and tileset factories; then add data sources and
fly the camera. Skipping the base-URL or token step is the dominant cause of a blank
globe, which is why both belong in the configuration and errors skills.

## 6. Performance and Memory

Performance levers: `requestRenderMode` + `maximumRenderTimeChange`, `Scene.fog`,
`logarithmicDepthBuffer`, `msaaSamples`, `debugShowFramesPerSecond`. Tilesets:
`maximumScreenSpaceError` (16 default, higher = faster/coarser), `foveatedScreenSpaceError`,
`skipLevelOfDetail`, `preferLeaves`, `preloadWhenHidden`. `RequestScheduler` throttles
network requests. `TaskProcessor` wraps Web Workers (`scheduleTask`, `maximumActiveTasks`).

Memory: WebGL resources are not garbage-collected. ALWAYS call `destroy()` on primitives,
tilesets, data sources, and the viewer before dropping references; check `isDestroyed()`.
Tileset budget: `cacheBytes` (512 MiB default) + `maximumCacheOverflowBytes` (replaced
`maximumMemoryUsage`). Hard WebGL constraint: ~16 contexts per browser tab; `viewer.destroy()`
does not always release the context (issue #11533) — reuse one viewer where possible.

## 7. Error Patterns and Anti-Patterns

Verified from CesiumGS/cesium GitHub issues (>10 real-world failures):

1. Missing ion token → blank/black globe + 401 (issues #8590, #4012).
2. WebGL context not released on `destroy()`; 16-context limit hit (#11533).
3. WebGL context loss on memory-constrained devices, `CONTEXT_LOST_WEBGL` (#10017, #5991).
4. Synchronous tileset construction with `readyPromise` (removed 1.107) — old tutorial code fails (#11195, PR #11059).
5. Tileset CORS failures; `file://` treated as cross-origin (#8584, #10449).
6. 403/401 from Google/ion tilesets on expired tokens (#11605).
7. Entity count at scale: >100k entities crash Chrome OOM; visualizer loops every tick (#6081, #9912).
8. Entities retain primitive memory after removal (~100MB / 5k entities) (#6534, #8767).
9. `pickPosition` without `pickPositionSupported` / depth picking → throws or wrong result.
10. GeoJSON `clampToGround` defaults FALSE → polygons float at ellipsoid height 0.
11. CZML `load` vs `process` confusion → `load` silently wipes prior entities.
12. `CESIUM_BASE_URL` unset / wrong → Workers/Assets/Widgets 404, blank globe.
13. Bundler asset-copy omitted → runtime worker failures.
14. Calling `new` on `Cartesian` static factories after 1.139 ES6-class conversion.

## 8. Newly Discovered Sub-Topics

Topics surfaced during research, absent or thin in the raw masterplan:

- `Terrain` / `ImageryLayer.fromProviderAsync` higher-level async-loading helper classes.
- Viewer→CesiumWidget API migration (1.123) — version-matrix item for core architecture.
- `defaultValue()` removal (1.134) — affects code-pattern examples in every skill.
- `msaaSamples` / `resolutionScale` / `contextOptions` render-quality and WebGL tuning knobs.
- WebGL 16-context budget as a hard architectural constraint (memory + errors skills).
- `ClippingPolygon` / `ClippingPolygonCollection` (1.117) — distinct from plane clipping.
- Voxel rendering (`VoxelProvider`, `scene.pickVoxel`, `EXT_primitive_voxels`, 1.127+) — experimental.
- `ClassificationPrimitive` vs `GroundPrimitive` + `ClassificationType` enum.
- `ModelExperimental` removal — skills must state it no longer exists.
- readyPromise removal as a cross-cutting migration topic.
- 3D Tiles 1.1 metadata system, implicit tiling, multiple contents per tile.
- `CallbackPositionProperty` — position-typed callback, used in interactive editing.
- `RequestScheduler` / `TaskProcessor` Web Worker API as distinct performance levers.
- `cacheBytes` / `maximumCacheOverflowBytes` (replaced `maximumMemoryUsage`).
- `EdgeDisplayMode` / `EXT_mesh_primitive_edge_visibility` (1.135+) — AEC CAD-edge rendering.
- `renderError` event + `rethrowRenderErrors` for render-loop error handling.
- Gaussian splatting 3D Tiles content (emerging, issue #13016).
- WebGPU explicitly NOT shipped — masterplan scope correction (L-001).

## 9. Implications for Phase 3

1. Correct masterplan scope: remove "WebGPU backend in transition" → "WebGL2 only".
2. `cesium-impl-geocoding-routing` is mis-scoped — routing does not exist in Cesium. Rename to `cesium-impl-geocoding`.
3. The async-factory + readyPromise-removal migration is large enough to justify either a dedicated errors/core skill or a strong shared section.
4. `ClippingPolygon`, voxels, edge display are real new API surface — decide depth per skill.
5. 3D Tiles is large enough (1.0 + 1.1, styling, formats, metadata, clipping) to consider splitting `cesium-syntax-3d-tiles` into syntax + a deeper impl skill, or a dedicated `tiles` category.
6. Errors cluster confirmed: rendering, tileset, memory, coordinates all have real, well-documented failure modes.

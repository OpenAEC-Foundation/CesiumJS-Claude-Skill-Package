# Research Fragment : Cluster A : Rendering Core

> Cluster : RENDERING CORE
> Researched : 2026-05-20
> Target version : CesiumJS 1.124+ (current release line reaches 1.142, 2026-06-01)
> Verification : all API names below verified via WebFetch against approved SOURCES.md URLs

## 1. Architecture and Runtime Model

CesiumJS layers a renderer beneath an application widget. `CesiumWidget` is the foundational rendering component: it owns an `HTMLCanvasElement`, a `Scene`, a `Camera`, a `Clock`, and a `ScreenSpaceEventHandler`, and runs the render loop. `Viewer` is a higher-level composite that wraps a `CesiumWidget` and adds UI widgets (Animation, Timeline, BaseLayerPicker, Geocoder, etc.). `Scene` is the container for all 3D state: it holds the `Globe` (the depth-test ellipsoid surface that renders imagery and terrain), `skyAtmosphere`, `skyBox`, `sun`, `moon`, `light`, `fog`, `postProcessStages`, the `Camera`, and the `ScreenSpaceCameraController`. `Globe` is therefore a member of `Scene`, not a sibling of it.

The render loop is driven by `Scene.render(time)`, which fires `preUpdate`, `postUpdate`, `preRender`, and `postRender` events each frame. By default the loop runs continuously via `requestAnimationFrame`. When `requestRenderMode: true` is set on the `Viewer`/`CesiumWidget`/`Scene`, frames render only when the scene changes; the application must call `scene.requestRender()` to force a frame. `maximumRenderTimeChange` defines how much simulation time may pass before a render is auto-requested. `requestRenderMode` is a major performance lever for static scenes.

`scene3DOnly: true` restricts geometry to the 3D `SceneMode` only (no 2D or Columbus View), conserving GPU memory by skipping multi-mode geometry batches. `frameState` is the per-frame state object passed through the render pipeline; it carries camera, frustum-culling, and command-organization data, and is consumed internally by primitive `update` methods.

`SceneMode` has three values: `SCENE3D`, `SCENE2D`, and `COLUMBUS_VIEW` (2.5D). Transitions use `morphTo3D()`, `morphTo2D()`, `morphToColumbusView()`.

WebGPU status (verified 2026-05-20 via Cesium Community and issue #4989): CesiumJS still renders exclusively on WebGL2 as of the 1.142 line. WebGPU is a longer-term roadmap item, not shipped. There is no WebGPU backend toggle in 1.124+. Skills MUST treat WebGL2 as the only backend and MUST NOT reference a WebGPU rendering path for CesiumJS.

## 2. Viewer Construction and the Ion Token

`Viewer` is constructed with `new Cesium.Viewer(container, options)`. Widget toggles (all boolean): `animation`, `baseLayerPicker`, `fullscreenButton`, `vrButton`, `geocoder`, `homeButton`, `infoBox`, `sceneModePicker`, `selectionIndicator`, `timeline`, `navigationHelpButton`. Rendering options: `scene3DOnly`, `requestRenderMode`, `maximumRenderTimeChange`, `contextOptions` (raw WebGL creation flags), `sceneMode`, `showRenderLoopErrors`, `msaaSamples`, `useBrowserRecommendedResolution`, `targetFrameRate`. Data options: `baseLayer`, `terrain`, `terrainProvider`, `imageryProviderViewModels`, `dataSources`, `ellipsoid`, `globe`.

For a minimal, UI-free setup, use `new Cesium.CesiumWidget(container, options)` instead of `Viewer`; it exposes `scene`, `camera`, `clock`, `canvas`, and `screenSpaceEventHandler` but has no toolbar, timeline, or inspector. `CesiumWidget` supports `useDefaultRenderLoop` and `resolutionScale`.

Ion token: `Cesium.Ion.defaultAccessToken = '<token>'` MUST be set before constructing a `Viewer` if any Cesium ion asset is used (World Terrain, Cesium OSM Buildings, ion imagery/geocoder). Version 1.123 moved `entities`, `dataSources`, `zoomTo()`, and `flyTo()` down from `Viewer` onto `CesiumWidget`, so these are now available on both.

## 3. Camera API

`Camera` navigation methods (all verified): `setView({ destination, orientation })` for instant placement, `flyTo({ destination, orientation, duration, complete, cancel, easingFunction, endTransform })` for animated transitions, `lookAt(target, offset)` where offset is a `Cartesian3` or `HeadingPitchRange`, `lookAtTransform(transform, offset)`, `flyToBoundingSphere(boundingSphere, options)`, and `viewBoundingSphere(boundingSphere, offset)`. `destination` accepts a `Cartesian3` or a `Rectangle`. `orientation` accepts either direction/up unit vectors or a `HeadingPitchRoll` (`heading`, `pitch`, `roll` in radians; heading is rotation from north, pitch negative looking down by convention, roll about the view axis).

Frustum types: `PerspectiveFrustum` (default), `PerspectiveOffCenterFrustum`, and `OrthographicFrustum`. The camera switches via `switchToPerspectiveFrustum()` and `switchToOrthographicFrustum()`. `lookAt`/`lookAtTransform` set an `endTransform` that locks the camera into a reference frame; `setView`/`flyTo` with the identity transform release it.

## 4. Coordinate Systems

`Cartesian3` is an ECEF (Earth-centered, Earth-fixed) position in meters. Factory methods: `Cartesian3.fromDegrees(lon, lat, height?, ellipsoid?)`, `Cartesian3.fromRadians(...)`, `Cartesian3.fromDegreesArray([lon,lat,...])`, `Cartesian3.fromDegreesArrayHeights([lon,lat,h,...])`. `Cartographic` stores `longitude` and `latitude` in RADIANS plus `height` in meters; converters are `Cartographic.fromCartesian(cartesian, ellipsoid?)`, `Cartographic.fromDegrees(...)`, `Cartographic.fromRadians(...)`. The default ellipsoid is WGS84, accessed as `Ellipsoid.WGS84` and as the configurable `Ellipsoid.default`.

`Transforms.eastNorthUpToFixedFrame(origin, ellipsoid?)` builds a local ENU `Matrix4` at a position; `Transforms.headingPitchRollToFixedFrame(origin, hpr, ellipsoid?)` builds an oriented frame. Pitfalls: confusing degrees with radians (constructor `new Cartographic()` and the `.longitude`/`.latitude` fields are radians, while `fromDegrees` converts); passing a `Cartographic` where a `Cartesian3` is expected; and assuming Cartesian XYZ corresponds to lon/lat/height (it does not, it is geocentric).

## 5. Imagery Providers

`ImageryLayer` displays one provider's tiles on a `Globe`. Provider subclasses (verified): `BingMapsImageryProvider`, `ArcGisMapServerImageryProvider`, `IonImageryProvider`, `OpenStreetMapImageryProvider`, `TileMapServiceImageryProvider`, `UrlTemplateImageryProvider`, `WebMapServiceImageryProvider`, `WebMapTileServiceImageryProvider`, `SingleTileImageryProvider`, `GoogleEarthEnterpriseImageryProvider`. Since 1.104, providers that fetch metadata use async static factories (`fromUrl`, `fromAssetId`/`fromBasemapType`) instead of direct constructors; the `readyPromise` pattern was fully removed in 1.107. Recommended layer creation: `ImageryLayer.fromProviderAsync(promiseOfProvider, options)` and `ImageryLayer.fromWorldImagery(options)`. MapTiler is consumed through `UrlTemplateImageryProvider` or WMTS, not a dedicated class.

## 6. Terrain Providers

`CesiumTerrainProvider` MUST be created via `CesiumTerrainProvider.fromUrl(url, options)` or `CesiumTerrainProvider.fromIonAssetId(assetId, options)` (both async, return Promises). The direct constructor is documented as not-to-be-called. Options include `requestVertexNormals`, `requestWaterMask`, `requestMetadata`. Cesium World Terrain uses `Cesium.createWorldTerrainAsync(options)` or the higher-level `Terrain.fromWorldTerrain(options)` (the `Terrain` helper class wraps async provider loading for the `Viewer` `terrain` option). `ArcGISTiledElevationTerrainProvider.fromUrl(url, options)` is the async ESRI elevation source. `EllipsoidTerrainProvider` is the flat-ellipsoid fallback and is synchronously constructible.

## 7. Materials and Custom Shaders

`Material` describes surface appearance via Fabric JSON compiled to GLSL. Create with `Material.fromType('Color', { color })` or `new Material({ fabric: { type, uniforms } })`. Built-in types include `Color`, `Image`, `DiffuseMap`, `NormalMap`, `Grid`, `Stripe`, `Checkerboard`, `Dot`, `Water`, `RimLighting`, `Fade`, `PolylineGlow`, `PolylineArrow`, `PolylineDash`, `PolylineOutline`, and elevation materials (`ElevationContour`, `ElevationRamp`, `SlopeRamp`, `AspectRamp`). For `Model` and `Cesium3DTileset`, use `CustomShader` (constructor options: `mode`, `lightingModel`, `uniforms`, `varyings`, `vertexShaderText`, `fragmentShaderText`, `translucencyMode`). `CustomShaderMode` is `MODIFY_MATERIAL` (default) or `REPLACE_MATERIAL`; `LightingModel` is `UNLIT` or `PBR`. Post-processing uses `Scene.postProcessStages` (a `PostProcessStageCollection`) and custom `PostProcessStage` instances with GLSL fragment shaders.

## 8. Atmosphere and Lighting

`Scene.skyAtmosphere` (`SkyAtmosphere`, 3D only) has `hueShift`, `saturationShift`, `brightnessShift`, `atmosphereLightIntensity` (default 50.0), `atmosphereRayleighCoefficient`, `atmosphereMieCoefficient`, scale heights, `atmosphereMieAnisotropy`, and `perFragmentAtmosphere`. `Scene.skyBox` (`SkyBox`) renders the star field. `Scene.sun`, `Scene.moon`, and `Scene.fog` are toggleable celestial/atmospheric elements. `Scene.light` is the shading light, defaulting to a sun-following `SunLight`; it may be set to a `DirectionalLight` for a fixed direction. `Globe` exposes ground-atmosphere and lighting-intensity controls (`showGroundAtmosphere`, `atmosphereLightIntensity`, `enableLighting`).

## 9. Rendering and Coordinate Anti-Patterns (verified from CesiumGS/cesium issues)

1. Missing Ion token causes a blank/black globe when ion-backed terrain or imagery is requested (issues #8590, #4012). Set `Cesium.Ion.defaultAccessToken` first.
2. WebGL context not released on `viewer.destroy()`; after 16 create/destroy cycles Chrome refuses new contexts (issue #11533). Destroy viewers fully; reuse one viewer where possible.
3. WebGL context loss on memory-constrained devices (iPhone SE) after sustained 3D Tiles interaction crashes with `CONTEXT_LOST_WEBGL` (issue #10017). Handle the `webglcontextlost` event.
4. Ungraceful errors thrown on context loss instead of a recoverable path (issue #5991).
5. Calling a provider constructor directly instead of the async `fromUrl`/`fromIonAssetId` factory yields an unusable provider; the removed `readyPromise` (1.107) means old tutorial code fails silently.
6. Treating `flyTo`/`setView` as synchronous: `flyTo` is animated and resolves later; reading camera state immediately after is a frame-timing bug.
7. Confusing `Cartographic` radians with degrees, or passing geographic values where ECEF `Cartesian3` is required, produces NaN positions or off-globe geometry (masterplan `cesium-errors-coordinates`).
8. Transient one-frame black globe after loading a large point cloud (issue #13055).

## Newly Discovered Sub-Topics

Topics not explicit in the raw masterplan, surfaced during verification:

- `Terrain` and `Imagery`/`ImageryLayer.fromProviderAsync` helper classes : a higher-level async-loading abstraction distinct from raw providers; worth a decision-tree in `cesium-syntax-terrain`/`cesium-syntax-imagery`.
- Viewer-to-CesiumWidget API migration (1.123) : `entities`, `dataSources`, `zoomTo`, `flyTo` moved onto `CesiumWidget`; relevant to `cesium-core-architecture` version matrix.
- Removal of the `defaultValue` helper (1.134) in favor of `??` : affects code-pattern examples in every skill.
- `msaaSamples` and `resolutionScale` as render-quality knobs : candidate for `cesium-core-performance`.
- WebGL context budget (16 per tab) as a hard architectural constraint : belongs in `cesium-core-memory` and `cesium-errors-rendering`.
- `CustomShaderTranslucencyMode` and `LightingModel.PBR` interplay : a depth area for `cesium-syntax-materials-shaders`.
- `contextOptions` (powerPreference: high-performance, requestWebgl2) : low-level WebGL tuning, candidate sub-section of `cesium-syntax-viewer`.
- WebGPU explicitly NOT shipped : the masterplan scope line "WebGPU backend in transition" should be corrected to "WebGL2 only; WebGPU is roadmap-only, not available in 1.124+".

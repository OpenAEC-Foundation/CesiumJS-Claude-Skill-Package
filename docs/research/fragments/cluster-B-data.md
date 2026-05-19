# Cluster B Research Fragment : Data and 3D Content

> Phase 2 deep research. Verified against approved sources in SOURCES.md.
> Target version : CesiumJS 1.124+ (reference docs current at 1.141 as of 2026-05-20).
> All API names below were confirmed via WebFetch against cesium.com ref-doc and GitHub.

## 1. Entity API (high-level)

The `Entity` class is the high-level, retained-mode visualization object. Constructor signature is `new Cesium.Entity(options)` taking an `Entity.ConstructorOptions` object. Core options: `id` (auto-generated GUID if omitted), `name`, `availability` (a `TimeIntervalCollection`), `show`, `position` (a `PositionProperty` or `Cartesian3`), `orientation` (defaults to east-north-up), `parent` (hierarchical relationships), `description` (HTML), and `viewFrom` (suggested tracking offset).

Each visualization aspect is a dedicated graphics object: `PointGraphics`, `BillboardGraphics`, `LabelGraphics`, `PolylineGraphics`, `PolygonGraphics`, `ModelGraphics`, `PathGraphics`, plus shape graphics `BoxGraphics`, `CylinderGraphics`, `EllipsoidGraphics`, `EllipseGraphics`, `PlaneGraphics`, `CorridorGraphics`, `RectangleGraphics`, `PolylineVolumeGraphics`, `WallGraphics`, and `Cesium3DTilesetGraphics`. Every graphics field is itself driven by `Property` objects.

The `Property` abstraction is central. `ConstantProperty` holds a fixed value (simplest). `SampledProperty` holds time-indexed values with interpolation between samples. `CallbackProperty(callback, isConstant)` lazily evaluates a callback every frame; `isConstant` must be `false` for animated values or Cesium caches the first result. `CallbackPositionProperty` is the position-typed equivalent. All properties expose `getValue(time, result)` and fire a `definitionChanged` event so the visualizers update reactively.

Material properties for entity graphics include `ColorMaterialProperty`, `ImageMaterialProperty`, `GridMaterialProperty`, `StripeMaterialProperty`, `CheckerboardMaterialProperty`, and the polyline-specific `PolylineGlowMaterialProperty` (options `color` default `Color.WHITE`, `glowPower` default `0.25`, `taperPower` default `1.0`), `PolylineDashMaterialProperty`, `PolylineArrowMaterialProperty`, and `PolylineOutlineMaterialProperty`.

Entity vs Primitive trade-off : Entities are easier (declarative, time-dynamic, picked as `entity`), but the `EntityCollection` visualizer loops every tick and degrades past tens of thousands of objects. Primitives are batched, faster at scale, but static and lower-level. ALWAYS prefer Entity for interactive/time-dynamic data under ~10k objects; use Primitive batching beyond that.

## 2. Primitive API (low-level)

`new Cesium.Primitive(options)` takes `geometryInstances` (one or array of `GeometryInstance`), `appearance`, `modelMatrix` (default `Matrix4.IDENTITY`), `show`, `asynchronous` (default `true`, builds geometry on a web worker), plus optimization flags `compressVertices`, `interleave`, `vertexCacheOptimize`, `allowPicking`, `cull`, `releaseGeometryInstances`, and `debugShowBoundingVolume`. A `GeometryInstance` wraps a `Geometry` with per-instance attributes and an `id`; multiple instances batch into one primitive while staying individually pickable via `Scene#pick` (returns the instance `id`).

Appearances : `PerInstanceColorAppearance` (per-instance color), `MaterialAppearance`, `EllipsoidSurfaceAppearance`, `PolylineColorAppearance`, `PolylineMaterialAppearance`. `PrimitiveCollection` holds primitives and is reachable as `scene.primitives`.

`GroundPrimitive` drapes geometry over terrain or 3D Tiles, using `classificationType` (`ClassificationType.TERRAIN`, `CESIUM_3D_TILE`, or `BOTH`, default `BOTH`). Materials on GroundPrimitive require the `WEBGL_depth_texture` extension; check via the static `GroundPrimitive.supportsMaterials()`. Supported geometries : `CircleGeometry`, `CorridorGeometry`, `EllipseGeometry`, `PolygonGeometry`, `RectangleGeometry`. `ClassificationPrimitive` is the broader classification primitive for rendering geometry directly onto existing surfaces.

## 3. 3D Tiles

`Cesium3DTileset` direct construction is discouraged. The async factory pattern is mandatory since the readyPromise removal (see anti-patterns): `Cesium3DTileset.fromUrl(url, options)` and `Cesium3DTileset.fromIonAssetId(assetId, options)` both return `Promise<Cesium3DTileset>` and throw `RuntimeError` for unsupported versions/extensions. After the promise resolves the tileset is fully ready and properties like `boundingSphere` are immediately valid.

Key properties : `style` (a `Cesium3DTileStyle`, set `undefined` to clear), `maximumScreenSpaceError` (default `16`, higher = faster/lower quality), `clippingPlanes` (`ClippingPlaneCollection`), and `clippingPolygons` (`ClippingPolygonCollection`).

`Cesium3DTileStyle` uses the 3D Tiles Styling expression language. Styles accept string expressions (`'${Temperature} > 90 ? color("red") : color("blue")'`), `conditions` arrays (`[['${height} > 2','color("cyan")'],['true','color("blue")']]`), or `evaluate` functions. Styleable keys include `color`, `show`, `pointSize`, `pointOutlineColor`, `labelColor`, `font`, `labelText`, and `Cartesian4`-valued `scaleByDistance`.

3D Tiles 1.0 (2018 OGC Community Standard) used binary tile formats : `b3dm` (batched 3D models), `i3dm` (instanced), `pnts` (point clouds), `cmpt` (composite), with separate batch tables and feature tables. 3D Tiles 1.1 deprecates all four binary formats in favor of glTF/glb directly as tile content, adds structured metadata (replacing deprecated `tileset.properties`), multiple contents per tile, and implicit tiling. CesiumJS 1.124+ reads both 1.0 and 1.1. `Cesium3DTileFeature` represents a picked feature.

## 4. glTF / Model

The `Model` class is the production glTF/glb renderer; `ModelExperimental` no longer exists as a separate class (it was merged into `Model`, confirmed absent from 1.141 docs). Create via the async static `Model.fromGltfAsync(options)` which resolves to a render-ready instance. `modelMatrix` (default `Matrix4.IDENTITY`) places the model, typically with `Transforms.eastNorthUpToFixedFrame()`. `activeAnimations` is a readonly `ModelAnimationCollection` of currently playing glTF animations. Articulations are driven by `setArticulationStage(key, value)` then `applyArticulations()`, which overwrites node transforms on participating nodes. glTF 2.0 with 20+ extensions is supported.

## 5. Time-dynamic data

`JulianDate` stores days and seconds separately, internally in TAI for leap-second-safe arithmetic. Factory methods : `JulianDate.fromDate(jsDate)`, `JulianDate.fromIso8601(str)` (handles all ISO 8601 forms, superior to `Date.parse`), `JulianDate.now()`. Arithmetic : `addSeconds`, `compare`. `Clock` tracks a current `JulianDate` and controls playback direction/multiplier; `ClockViewModel` is the Knockout UI-binding layer for the Timeline/Animation widgets.

`SampledPositionProperty` combines `SampledProperty` + `PositionProperty`. Add data with `addSample(time, position, derivatives)`, `addSamples(times, positions, derivatives)`, or `addSamplesPackedArray(packed, epoch)`. `setInterpolationOptions({interpolationAlgorithm, interpolationDegree})` selects `LinearApproximation` (default, degree 1), `LagrangePolynomialApproximation`, or `HermitePolynomialApproximation` (derivative-aware). Extrapolation is controlled by `forwardExtrapolationType`/`Duration` and the backward equivalents (default `ExtrapolationType.NONE`). `TimeIntervalCollection` is a non-overlapping sorted set of `TimeInterval`s used for `Entity.availability` and `TimeIntervalCollectionProperty`; build with `fromIso8601` or `fromJulianDateArray`.

## 6. Picking, selection, measurement

`scene.pick(windowPosition)` returns the topmost object with a `primitive` property (or a `Cesium3DTileFeature` for tilesets), `undefined` if nothing hit. `scene.drillPick(windowPosition)` returns all objects front-to-back. `scene.pickPosition(windowPosition)` reconstructs a `Cartesian3` from the depth buffer; it requires `scene.pickPositionSupported` and depth picking enabled, otherwise it throws. For globe-only hits use `globe.pick(ray, scene)`. `scene.sampleHeight` and `scene.clampToHeight` (plus the async `clampToHeightMostDetailed`) sample scene geometry. `scene.pickVoxel` is experimental.

`ScreenSpaceEventHandler` registers input via `setInputAction(action, ScreenSpaceEventType, modifier)`. Event types include `LEFT_CLICK`, `LEFT_DOUBLE_CLICK`, `MOUSE_MOVE`, `WHEEL`, `LEFT_DOWN`/`LEFT_UP`, and pinch events; callbacks receive `position`, or `startPosition`/`endPosition` for motion. `viewer.selectedEntity` controls the InfoBox; `ImageryLayer.pickImageryLayerFeatures`/`scene.pickFeatures` returns imagery features. Measurement tooling is built by combining `MOUSE_MOVE`/`LEFT_CLICK` handlers with `pickPosition` to capture `Cartesian3` vertices, then computing geodesic distances/areas.

## 7. DataSources

`DataSource` implementations are added with `viewer.dataSources.add(dataSourcePromise)`. `CzmlDataSource` processes CZML (a JSON time-dynamic format): static `CzmlDataSource.load(czml, options)` returns a promise; instance `load` replaces data, `process` appends. `LoadOptions` include `sourceUri` and `credit`. `GeoJsonDataSource` handles both GeoJSON and TopoJSON via `GeoJsonDataSource.load(data, options)`; options : `stroke` (default `Color.BLACK`), `strokeWidth` (`2.0`), `fill` (`Color.YELLOW`), `clampToGround` (default `false`), `markerColor` (`Color.ROYALBLUE`), `markerSize` (`48`), `markerSymbol`, `describe`. It also honors simplestyle-spec properties. `KmlDataSource` loads KML/KMZ and requires a `camera` and `canvas` option for network-link/screen-overlay handling.

## 8. Data and 3D Tiles Anti-Patterns

1. **Synchronous tileset construction with readyPromise** : `options.url`, `.ready`, and `.readyPromise` were deprecated in 1.104 and removed in 1.107. Code from older tutorials breaks on 1.124+. ALWAYS `await Cesium3DTileset.fromUrl(url)` (issue #11195, PR #11059).
2. **Tileset CORS failures** : `fromUrl` against a server without `Access-Control-Allow-Origin` floods the console with opaque CORS errors and the tileset never appears; `file://` requests are treated as cross-origin (issues #8584, #10449, OfflineGuide).
3. **403/401 from third-party providers** : Google and ion-hosted tilesets return 403 on expired/missing tokens; IonResource retry does not cover the underlying non-ion Google URL (issue #11605).
4. **Entity count at scale** : >100k SVG-backed entities crash Chrome with out-of-memory while equivalent billboard Primitives render fine; the EntityCollection visualizer loops every entity each tick (issues #6081, #9912).
5. **Entities retain primitive memory after removal** : removing entities leaves ~100MB per 5k entities in primitive collections; removing a 40k-point DataSource can take ~7s (issues #6534, #8767).
6. **pickPosition without depth picking** : calling `scene.pickPosition` without `scene.pickPositionSupported` true throws; results are wrong if `depthTestAgainstTerrain`/`useDepthPicking` is not enabled, giving ellipsoid hits instead of terrain hits.
7. **GeoJSON height handling** : `clampToGround` defaults to `false`, so polygons/lines float at ellipsoid height 0 over terrain; AEC users expect ground-clamped output and must opt in.
8. **CZML vs entity-mutation confusion** : calling `load` instead of `process` silently wipes prior entities; epoch/interval mistakes in CZML break interpolation.

## Newly Discovered Sub-Topics

Topics not explicitly in the raw masterplan that Phase 3 should consider:

- **`ClippingPolygonCollection` / `ClippingPolygon`** : geodesic polygon clipping, distinct from plane-based `ClippingPlaneCollection`. Currently only `clippingPlanes` is named in the masterplan; `clippingPolygons` is a separate, newer API surface.
- **Voxel rendering** (`scene.pickVoxel`, `VoxelCell`, voxel providers) : experimental but present in 1.141.
- **`ClassificationPrimitive` vs `GroundPrimitive`** distinction and `ClassificationType` enum : warrants explicit coverage in `cesium-syntax-primitive`.
- **`ModelExperimental` removal history** : the legacy split is resolved; the skill should explicitly state ModelExperimental no longer exists to stop Claude generating obsolete code.
- **readyPromise removal as a cross-cutting migration topic** : affects `Model`, `Cesium3DTileset`, terrain and imagery providers. Could justify a dedicated migration note in `cesium-core-architecture` or an errors skill.
- **3D Tiles metadata system** (structured metadata, implicit tiling, multiple contents per tile) : 1.1-specific, deeper than the masterplan's "batch/feature tables" line.
- **`CallbackPositionProperty`** : the position-typed CallbackProperty, separate class, used heavily in interactive editing tools.
- **simplestyle-spec support inside GeoJSON** : affects how styling options interact with input data.
- **`Gaussian splatting` 3D Tiles content** : emerging tile content type referenced in recent issues (#13016); may be worth a forward-looking note.

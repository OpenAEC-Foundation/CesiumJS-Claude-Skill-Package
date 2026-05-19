# Masterplan : CesiumJS

> Status : Phase 3 refined : executable
> Generated : 2026-05-19 | Raw inventory : 2026-05-20 | Refined : 2026-05-20
> Research base : docs/research/vooronderzoek-cesium.md

## Scope

- Technology : CesiumJS
- Versions : 1.124+ (WebGL2 only; WebGPU is roadmap-only, NOT shipped : see L-001)
- Languages : JavaScript, TypeScript
- Prefix : cesium
- License : MIT
- Excludes : Cesium Stories, Cesium for Unity / Unreal / Omniverse
- Includes : Cesium ion cloud platform as companion (assets, terrain, imagery, geocoding)

## Refinement Decisions

| ID | Decision | Reason | Source |
|----|----------|--------|--------|
| D-01 | RENAME `cesium-impl-geocoding-routing` to `cesium-impl-geocoding` | Routing does not exist in CesiumJS/ion. A routing claim would be a hallucination | vooronderzoek §5, L-003 |
| D-02 | ADD `cesium-core-versioning` | The async-factory + readyPromise-removal migration is cross-cutting and the biggest source of broken code | vooronderzoek §3, L-002 |
| D-03 | SPLIT 3D Tiles into `cesium-syntax-3d-tiles` (API, formats, loading) + `cesium-impl-3d-tiles-styling` (styling expression language, clipping, optimization) | 3D Tiles 1.0+1.1, formats, styling, metadata is too broad for one skill | vooronderzoek §2 |
| D-04 | MERGE `cesium-impl-postprocess` into `cesium-syntax-materials` | PostProcessStage is GLSL shader API, same domain as Material/CustomShader | vooronderzoek §2 |
| D-05 | MERGE `cesium-impl-sandcastle` into `cesium-impl-build-deploy` | Sandcastle-to-production conversion is a build concern | vooronderzoek §5 |
| D-06 | ADD `cesium-impl-build-deploy` | `CESIUM_BASE_URL` + bundler config is the #1 root cause of a blank globe | vooronderzoek §4/§7 |
| D-07 | KEEP 5 standard categories, no extra `tiles`/`geospatial` category | 30 skills fits the standard core to syntax to impl to errors to agents chain; extra categories are reserved for >60-skill packages | runbook §5.2 |

Final inventory : 30 skills : core 5, syntax 12, impl 7, errors 4, agents 2.

## Skill Inventory

### core/ (5)
| Skill | Scope summary |
|-------|---------------|
| cesium-core-architecture | Viewer/Scene/Globe/Camera/CesiumWidget hierarchy, render loop, requestRenderMode, SceneMode, WebGL2-only |
| cesium-core-coordinates | Cartesian3, Cartographic, Ellipsoid/WGS84, Transforms, ENU frames, conversions |
| cesium-core-versioning | Version matrix 1.124-1.142, async-factory migration, readyPromise/defaultValue removal |
| cesium-core-performance | requestRenderMode, maximumScreenSpaceError, fog, RequestScheduler, TaskProcessor, LOD |
| cesium-core-memory | destroy()/isDestroyed(), cacheBytes, WebGL 16-context budget, context loss |

### syntax/ (12)
| Skill | Scope summary |
|-------|---------------|
| cesium-syntax-viewer | Viewer/CesiumWidget construction, options, widget toggles, contextOptions, ion token |
| cesium-syntax-camera | flyTo/setView/lookAt, HeadingPitchRoll, frustums, ScreenSpaceCameraController |
| cesium-syntax-entity | Entity API, graphics objects, Property types, material properties |
| cesium-syntax-primitive | Primitive/GeometryInstance, appearances, GroundPrimitive, ClassificationPrimitive |
| cesium-syntax-imagery | ImageryLayer + providers (Bing/ESRI/MapTiler/OSM/WMS/WMTS/custom) |
| cesium-syntax-terrain | Terrain providers, World Terrain, ArcGIS, async fromUrl/fromIonAssetId |
| cesium-syntax-gltf-model | Model.fromGltfAsync, modelMatrix, animations, articulations |
| cesium-syntax-3d-tiles | Cesium3DTileset loading, formats b3dm/i3dm/pnts/glb, 1.0 vs 1.1, metadata |
| cesium-syntax-datasources | KML/CZML/GeoJSON/TopoJSON, load vs process, clampToGround |
| cesium-syntax-time | Clock, JulianDate, SampledPositionProperty, TimeIntervalCollection, interpolation |
| cesium-syntax-materials | Material/Fabric, CustomShader, PostProcessStage |
| cesium-syntax-atmosphere | skyAtmosphere, skyBox, sun/moon, fog, lighting |

### impl/ (7)
| Skill | Scope summary |
|-------|---------------|
| cesium-impl-picking-measurement | scene.pick/drillPick/pickPosition, ScreenSpaceEventHandler, measurement tooling |
| cesium-impl-3d-tiles-styling | Cesium3DTileStyle expression language, clipping planes/polygons, optimization |
| cesium-impl-cesium-ion | ion assets upload, tiling, tokens, IonResource, self-hosting |
| cesium-impl-geocoding | GeocoderService, IonGeocoderService, custom geocoders (NO routing) |
| cesium-impl-resium | Resium React wrapper, components, useCesium, refs |
| cesium-impl-aec-georef | CityGML/IFC georeferencing, modelMatrix placement, geo-BIM/digital-twin |
| cesium-impl-build-deploy | npm package, CESIUM_BASE_URL, Vite/webpack, static dirs, Sandcastle conversion |

### errors/ (4)
| Skill | Scope summary |
|-------|---------------|
| cesium-errors-rendering | Blank/black globe, WebGL context lost, missing token, no imagery |
| cesium-errors-tileset | 3D Tiles CORS, 401/403, sync construction, missing tiles |
| cesium-errors-memory | Leaks, destroy ordering, detached primitives, context budget |
| cesium-errors-coordinates | NaN positions, radians/degrees confusion, terrain clamping |

### agents/ (2)
| Skill | Scope summary |
|-------|---------------|
| cesium-agents-scene-architect | Orchestrate viewer/scene/data-source setup decisions |
| cesium-agents-skill-validator | Cross-skill consistency, deprecated-pattern detection, code quality gate |

## Execution Plan : Batches

| Batch | Skills | Count | Dependencies |
|-------|--------|-------|--------------|
| 1 | cesium-core-architecture, cesium-core-coordinates, cesium-core-versioning | 3 | none |
| 2 | cesium-core-performance, cesium-core-memory, cesium-syntax-viewer | 3 | batch 1 |
| 3 | cesium-syntax-camera, cesium-syntax-entity, cesium-syntax-primitive | 3 | batch 2 |
| 4 | cesium-syntax-imagery, cesium-syntax-terrain, cesium-syntax-gltf-model | 3 | batch 2 |
| 5 | cesium-syntax-3d-tiles, cesium-syntax-datasources, cesium-syntax-time | 3 | batch 2 |
| 6 | cesium-syntax-materials, cesium-syntax-atmosphere, cesium-impl-picking-measurement | 3 | batch 3 |
| 7 | cesium-impl-3d-tiles-styling, cesium-impl-cesium-ion, cesium-impl-geocoding | 3 | batch 5 |
| 8 | cesium-impl-resium, cesium-impl-aec-georef, cesium-impl-build-deploy | 3 | batch 3-7 |
| 9 | cesium-errors-rendering, cesium-errors-tileset, cesium-errors-memory | 3 | batch 1-8 |
| 10 | cesium-errors-coordinates, cesium-agents-scene-architect, cesium-agents-skill-validator | 3 | ALL |

Rules : batch-size 3 (tmux-orchestration default), file-scope isolated per skill dir,
dependency chain core to syntax to impl to errors to agents.

## Quality Rules (apply to EVERY skill prompt below)

- Output : `SKILL.md` (< 500 lines) + `references/methods.md` + `references/examples.md` + `references/anti-patterns.md`
- YAML frontmatter : folded block scalar `>`, description starts with "Use when...", includes a `Keywords:` line mixing technical terms + symptom-based phrases ("blank globe", "nothing renders") + plain-language ("how do I"), `license: MIT`, `compatibility: "Designed for Claude Code. Requires CesiumJS 1.124+."`, `metadata.author: OpenAEC-Foundation`, `metadata.version: "1.0"`
- name : kebab-case, max 64 chars, format `cesium-{category}-{topic}`
- English-only, deterministic language : ALWAYS X / NEVER Y. The phrase "you might want to" is BANNED
- Section headings use `:` not em-dashes. NO em-dashes anywhere in user-facing text
- Verify ALL code examples via WebFetch against SOURCES.md approved URLs only
- ALWAYS use async factories (`Cesium3DTileset.fromUrl`, `Model.fromGltfAsync`, `*.fromIonAssetId`). NEVER emit `readyPromise`, `.ready`, `new Cesium3DTileset({url})`, `ModelExperimental`, or `defaultValue()`
- NEVER reference a WebGPU rendering path : CesiumJS is WebGL2 only (L-001)
- NEVER write a README.md inside a skill folder
- Commit per skill : `feat(skill): cesium-{category}-{topic}`
- Quality gate : `validate-frontmatter.js` + `validate-language.js` + `validate-line-count.js` + `validate-structure.js` + `validate-emdash.js` all exit 0

## Agent Prompts

Each prompt below is what a tmux-orchestration `skill-builder` worker receives.
Workspace for all : `/home/freek/GitHub/CesiumJS-Claude-Skill-Package/`.
Research base for all : `docs/research/vooronderzoek-cesium.md` + the per-skill
topic-research file at `docs/research/topic-research/{skill}-research.md` (Phase 4).
All prompts inherit the Quality Rules section above.

### Skill: cesium-core-architecture
Output dir : `skills/source/cesium-core/cesium-core-architecture/`
Research : vooronderzoek §1, §2
Scope :
- Viewer wraps CesiumWidget; CesiumWidget owns canvas/Scene/Camera/Clock/ScreenSpaceEventHandler
- Scene containment : Globe, skyAtmosphere, skyBox, sun, moon, light, fog, postProcessStages, Camera, primitives are members of Scene; Globe is a member of Scene NOT a sibling
- Render loop : Scene.render, preUpdate/postUpdate/preRender/postRender events, requestAnimationFrame
- requestRenderMode + maximumRenderTimeChange + scene.requestRender()
- SceneMode : SCENE3D / SCENE2D / COLUMBUS_VIEW, morphTo* transitions, scene3DOnly
- WebGL2 is the only backend; WebGPU is roadmap-only (L-001)
- Entity (high-level retained-mode) vs Primitive (low-level batched) API tier overview
- Decision tree : when to use Viewer vs CesiumWidget

### Skill: cesium-core-coordinates
Output dir : `skills/source/cesium-core/cesium-core-coordinates/`
Research : vooronderzoek §2 (coordinate systems)
Scope :
- Cartesian3 is ECEF in meters; factories fromDegrees/fromRadians/fromDegreesArray/fromDegreesArrayHeights
- Cartographic stores longitude/latitude in RADIANS + height meters; fromCartesian/fromDegrees/fromRadians
- Ellipsoid.WGS84 and the configurable Ellipsoid.default
- Transforms.eastNorthUpToFixedFrame, headingPitchRollToFixedFrame, localFrameToFixedFrameGenerator
- HeadingPitchRoll semantics (heading from north, pitch sign, roll axis)
- Conversion patterns and the radians-vs-degrees pitfall, Cartographic-vs-Cartesian3 mixups

### Skill: cesium-core-versioning
Output dir : `skills/source/cesium-core/cesium-core-versioning/`
Research : vooronderzoek §3 (version matrix)
Scope :
- Version matrix 1.124 to 1.142 (use the vooronderzoek table as the verified base)
- The async-factory migration (1.104/1.107) : readyPromise deprecated then removed
- defaultValue() removed 1.134 (use `??`); Cartesian2/3/4 became ES6 classes 1.139
- Viewer to CesiumWidget API migration (1.123); MSAA + PBR Neutral defaults (1.121)
- How to read CHANGES.md (raw.githubusercontent.com form) for deprecations
- Decision tree : detecting and migrating legacy code patterns

### Skill: cesium-core-performance
Output dir : `skills/source/cesium-core/cesium-core-performance/`
Research : vooronderzoek §6
Scope :
- requestRenderMode + maximumRenderTimeChange for static scenes
- maximumScreenSpaceError, foveatedScreenSpaceError, skipLevelOfDetail, preferLeaves, preloadWhenHidden
- Scene.fog, logarithmicDepthBuffer, msaaSamples, debugShowFramesPerSecond
- RequestScheduler per-server request throttling
- TaskProcessor Web Worker API (scheduleTask, maximumActiveTasks)
- Frustum culling, LOD strategy decision tree

### Skill: cesium-core-memory
Output dir : `skills/source/cesium-core/cesium-core-memory/`
Research : vooronderzoek §6 (memory), §7
Scope :
- destroy() and isDestroyed() on viewer, tileset, primitive, data source; teardown ordering
- WebGL resources are not garbage-collected : explicit destroy required
- cacheBytes (512 MiB default) + maximumCacheOverflowBytes (replaced maximumMemoryUsage)
- WebGL ~16-context-per-tab budget; viewer.destroy() does not always release the context
- Context loss : renderError event, rethrowRenderErrors, webglcontextlost
- Leak patterns and how to confirm a real leak vs a lost reference

### Skill: cesium-syntax-viewer
Output dir : `skills/source/cesium-syntax/cesium-syntax-viewer/`
Research : vooronderzoek §2 (Viewer), §4
Scope :
- new Cesium.Viewer(container, options); all boolean widget toggles
- Rendering options : scene3DOnly, requestRenderMode, contextOptions, sceneMode, msaaSamples
- contextOptions : requestWebgl2, powerPreference
- CesiumWidget for minimal UI-free setups
- Ion.defaultAccessToken must be set before constructing a Viewer that uses ion assets
- Data options : baseLayer, terrain, dataSources

### Skill: cesium-syntax-camera
Output dir : `skills/source/cesium-syntax/cesium-syntax-camera/`
Research : vooronderzoek §2 (Camera)
Scope :
- setView, flyTo (animated, async-resolving), lookAt, lookAtTransform, flyToBoundingSphere, viewBoundingSphere
- destination : Cartesian3 or Rectangle; orientation : HeadingPitchRoll or direction/up
- Frustum types : PerspectiveFrustum, OrthographicFrustum, PerspectiveOffCenterFrustum
- ScreenSpaceCameraController
- endTransform reference-frame lock from lookAt/lookAtTransform

### Skill: cesium-syntax-entity
Output dir : `skills/source/cesium-syntax/cesium-syntax-entity/`
Research : vooronderzoek §2 (Entity API)
Scope :
- Entity + Entity.ConstructorOptions, parent hierarchy, availability, EntityCollection
- Graphics objects : Point/Billboard/Label/Polyline/Polygon/Model/Path + shape graphics
- Property abstraction : ConstantProperty, SampledProperty, CallbackProperty (isConstant), CallbackPositionProperty
- Material properties : ColorMaterialProperty, PolylineGlowMaterialProperty, PolylineDashMaterialProperty
- Entity vs Primitive selection rule (Entity under ~10k objects)

### Skill: cesium-syntax-primitive
Output dir : `skills/source/cesium-syntax/cesium-syntax-primitive/`
Research : vooronderzoek §2 (Primitive API)
Scope :
- Primitive + geometryInstances + appearance + modelMatrix + asynchronous flag
- GeometryInstance wrapping Geometry + per-instance attributes + pickable id
- Appearances : PerInstanceColorAppearance, MaterialAppearance, EllipsoidSurfaceAppearance, PolylineColorAppearance
- GroundPrimitive + ClassificationType (TERRAIN/CESIUM_3D_TILE/BOTH); GroundPrimitive.supportsMaterials()
- ClassificationPrimitive; PrimitiveCollection / scene.primitives

### Skill: cesium-syntax-imagery
Output dir : `skills/source/cesium-syntax/cesium-syntax-imagery/`
Research : vooronderzoek §2 (imagery providers)
Scope :
- ImageryLayer + provider subclasses : Bing, ArcGis, Ion, OpenStreetMap, TileMapService, UrlTemplate, WMS, WMTS, SingleTile
- Async static factories (fromUrl, fromBasemapType, fromAssetId); readyPromise removed
- ImageryLayer.fromProviderAsync, ImageryLayer.fromWorldImagery
- MapTiler consumed via UrlTemplateImageryProvider or WMTS (no dedicated class)
- Layer ordering, alpha, brightness

### Skill: cesium-syntax-terrain
Output dir : `skills/source/cesium-syntax/cesium-syntax-terrain/`
Research : vooronderzoek §2 (terrain providers)
Scope :
- CesiumTerrainProvider.fromUrl / fromIonAssetId (async, mandatory; direct constructor not to be called)
- createWorldTerrainAsync, the Terrain helper class, viewer.terrain option
- ArcGISTiledElevationTerrainProvider.fromUrl; EllipsoidTerrainProvider (sync fallback)
- Options : requestVertexNormals, requestWaterMask, requestMetadata

### Skill: cesium-syntax-gltf-model
Output dir : `skills/source/cesium-syntax/cesium-syntax-gltf-model/`
Research : vooronderzoek §2 (glTF/Model)
Scope :
- Model.fromGltfAsync : the production async glTF/glb renderer
- ModelExperimental no longer exists (merged into Model) : NEVER emit it
- modelMatrix placement, typically via Transforms.eastNorthUpToFixedFrame
- activeAnimations (ModelAnimationCollection); articulations via setArticulationStage + applyArticulations
- glTF 2.0 with 20+ extensions

### Skill: cesium-syntax-3d-tiles
Output dir : `skills/source/cesium-syntax/cesium-syntax-3d-tiles/`
Research : vooronderzoek §2 (3D Tiles), §7
Scope :
- Cesium3DTileset.fromUrl / fromIonAssetId (async-factory mandatory)
- Tile formats : b3dm/i3dm/pnts/cmpt (1.0) vs glTF/glb tile content (1.1)
- 3D Tiles 1.0 vs 1.1 : structured metadata, implicit tiling, multiple contents per tile
- maximumScreenSpaceError, boundingSphere validity after the promise resolves
- Cesium3DTileFeature
- NEVER readyPromise / new Cesium3DTileset({url})

### Skill: cesium-syntax-datasources
Output dir : `skills/source/cesium-syntax/cesium-syntax-datasources/`
Research : vooronderzoek §2 (DataSources)
Scope :
- viewer.dataSources.add(promise)
- CzmlDataSource : load (replace) vs process (append); CZML format basics
- GeoJsonDataSource : handles GeoJSON + TopoJSON; clampToGround defaults FALSE; stroke/fill/marker; simplestyle-spec
- KmlDataSource : KML/KMZ; requires camera and canvas options

### Skill: cesium-syntax-time
Output dir : `skills/source/cesium-syntax/cesium-syntax-time/`
Research : vooronderzoek §2 (time-dynamic data)
Scope :
- JulianDate : fromDate, fromIso8601, now; TAI-internal, leap-second-safe; addSeconds, compare
- Clock + ClockViewModel
- SampledPositionProperty : addSample/addSamples; interpolation algorithms Linear/Lagrange/Hermite
- TimeIntervalCollection, Entity.availability, extrapolation types

### Skill: cesium-syntax-materials
Output dir : `skills/source/cesium-syntax/cesium-syntax-materials/`
Research : vooronderzoek §2 (materials/shaders)
Scope :
- Material : fromType, Fabric JSON, built-in types (Color, Image, Grid, Water, PolylineGlow, elevation ramps)
- CustomShader for Model and Cesium3DTileset : mode, lightingModel (UNLIT/PBR), uniforms, vertex/fragmentShaderText
- PostProcessStageCollection + custom PostProcessStage (merged here per D-04)

### Skill: cesium-syntax-atmosphere
Output dir : `skills/source/cesium-syntax/cesium-syntax-atmosphere/`
Research : vooronderzoek §2 (atmosphere/lighting)
Scope :
- Scene.skyAtmosphere (SkyAtmosphere : hue/saturation/brightness shift, atmosphereLightIntensity)
- Scene.skyBox (SkyBox star field), Scene.sun, Scene.moon, Scene.fog
- Scene.light : SunLight (default) vs DirectionalLight
- Globe.enableLighting, showGroundAtmosphere, atmosphereLightIntensity

### Skill: cesium-impl-picking-measurement
Output dir : `skills/source/cesium-impl/cesium-impl-picking-measurement/`
Research : vooronderzoek §2 (picking)
Scope :
- scene.pick (topmost), scene.drillPick (all), scene.pickPosition (needs pickPositionSupported)
- globe.pick(ray, scene), scene.sampleHeight, scene.clampToHeight
- ScreenSpaceEventHandler.setInputAction + ScreenSpaceEventType values
- viewer.selectedEntity / InfoBox
- Building distance and area measurement tooling from pickPosition vertices

### Skill: cesium-impl-3d-tiles-styling
Output dir : `skills/source/cesium-impl/cesium-impl-3d-tiles-styling/`
Research : vooronderzoek §2 (3D Tiles styling)
Scope :
- Cesium3DTileStyle : the 3D Tiles styling expression language
- String expressions, conditions arrays, evaluate functions
- Styleable keys : color, show, pointSize, pointOutlineColor, labelColor
- clippingPlanes (ClippingPlaneCollection) and clippingPolygons (ClippingPolygonCollection)
- Tileset optimization workflow

### Skill: cesium-impl-cesium-ion
Output dir : `skills/source/cesium-impl/cesium-impl-cesium-ion/`
Research : vooronderzoek §4 (Cesium ion)
Scope :
- ion platform : SaaS vs Self-Hosted; free tier vs paid
- Ion.defaultAccessToken; IonResource.fromAssetId; Cesium3DTileset.fromIonAssetId; IonImageryProvider.fromAssetId
- Asset upload + tiling pipeline overview
- ion REST API basics (verify exact endpoint paths in Phase 4 topic-research before quoting)
- Self-hosting alternatives outside ion

### Skill: cesium-impl-geocoding
Output dir : `skills/source/cesium-impl/cesium-impl-geocoding/`
Research : vooronderzoek §5 (geocoding)
Scope :
- GeocoderService interface; IonGeocoderService (scene, accessToken, geocodeProviderType)
- Viewer geocoder option : boolean | IonGeocodeProviderType | Array<GeocoderService>
- Custom geocoders by implementing geocode()
- GeocodeType (SEARCH default)
- NEVER claim routing/directions : CesiumJS/ion have NO routing (L-003)

### Skill: cesium-impl-resium
Output dir : `skills/source/cesium-impl/cesium-impl-resium/`
Research : vooronderzoek §5 (Resium)
Scope :
- Resium declarative React component model : Viewer, Entity, Cesium3DTileset, imagery/terrain components
- useCesium hook for context (viewer, scene, camera)
- CesiumComponentRef.cesiumElement for imperative native-object access
- CesiumJS as a peer dependency (verify exact peer-version ranges in Phase 4 from package.json)

### Skill: cesium-impl-aec-georef
Output dir : `skills/source/cesium-impl/cesium-impl-aec-georef/`
Research : vooronderzoek §5 (AEC georeferencing)
Scope :
- The ECEF placement problem for BIM/city models
- CityGML to 3D Tiles workflow (ion tiler / external); IFC to glTF via ThatOpen/IfcOpenShell then georeference
- Transforms.eastNorthUpToFixedFrame / headingPitchRollToFixedFrame; modelMatrix placement
- Geo-BIM and digital-twin patterns; EdgeDisplayMode for CAD-style edges
- Clearly separate official API from external workflow
- Cross-tech companion notes : QGIS, ThatOpen, Speckle

### Skill: cesium-impl-build-deploy
Output dir : `skills/source/cesium-impl/cesium-impl-build-deploy/`
Research : vooronderzoek §4 (build/deployment), §8 (Sandcastle)
Scope :
- cesium npm package; the four static directories Workers/ThirdParty/Assets/Widgets
- window.CESIUM_BASE_URL set before import
- Vite : vite-plugin-cesium; webpack : CopyWebpackPlugin + DefinePlugin (cesium-webpack-example reference)
- ES6 named imports; TypeScript types ship with the package
- Sandcastle to production conversion (merged here per D-05)

### Skill: cesium-errors-rendering
Output dir : `skills/source/cesium-errors/cesium-errors-rendering/`
Research : vooronderzoek §7 (anti-patterns 1-3, 12-13)
Scope :
- Blank/black globe root causes : CESIUM_BASE_URL unset, missing ion token, bundler asset-copy omitted
- WebGL context lost (CONTEXT_LOST_WEBGL); render-loop errors; renderError event
- No imagery / no terrain symptoms
- Each error : symptom, root cause, prevention, recovery

### Skill: cesium-errors-tileset
Output dir : `skills/source/cesium-errors/cesium-errors-tileset/`
Research : vooronderzoek §7 (anti-patterns 4-6)
Scope :
- 3D Tiles CORS failures; file:// treated as cross-origin
- 401/403 from ion and third-party (Google) tilesets on expired/missing tokens
- Synchronous tileset construction with readyPromise (removed 1.107)
- Missing tiles, screen-space-error issues

### Skill: cesium-errors-memory
Output dir : `skills/source/cesium-errors/cesium-errors-memory/`
Research : vooronderzoek §7 (anti-patterns 7-8), §6
Scope :
- Leaks from not calling destroy(); teardown ordering
- Entities retaining primitive memory after removal
- Lost-reference bugs that look like leaks
- WebGL 16-context budget exhaustion

### Skill: cesium-errors-coordinates
Output dir : `skills/source/cesium-errors/cesium-errors-coordinates/`
Research : vooronderzoek §7 (anti-patterns 10), §2
Scope :
- NaN positions and off-globe geometry
- Radians vs degrees confusion; Cartographic vs Cartesian3 mixups
- Terrain clamping failures; GeoJSON clampToGround defaulting false
- Heading/pitch/roll convention mistakes

### Skill: cesium-agents-scene-architect
Output dir : `skills/source/cesium-agents/cesium-agents-scene-architect/`
Research : full vooronderzoek
Scope :
- Orchestration skill : decision trees for viewer/scene/data-source setup
- Choose Entity vs Primitive vs 3D Tiles for a dataset
- Choose imagery and terrain providers; decide requestRenderMode
- Cross-references the core/ and syntax/ skills

### Skill: cesium-agents-skill-validator
Output dir : `skills/source/cesium-agents/cesium-agents-skill-validator/`
Research : full vooronderzoek, all other skills
Scope :
- Validation rules : detect deprecated patterns (readyPromise, ModelExperimental, defaultValue, new on Cartesian factories, WebGPU references)
- API verification checklist against SOURCES.md
- Cross-skill consistency checks
- Code quality gate for generated CesiumJS code

## Next : Phase 4+5

Per batch : dispatch in-process opus topic-research agents writing
`docs/research/topic-research/{skill}-research.md`, then dispatch the 3 skill prompts
to tmux-orchestration workers. Quality gate every worker reply.

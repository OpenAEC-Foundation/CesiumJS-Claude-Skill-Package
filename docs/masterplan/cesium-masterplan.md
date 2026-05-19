# Masterplan : CesiumJS

> Status : Phase 1 raw : pre-research
> Generated : 2026-05-19
> Raw inventory completed : 2026-05-20

## Scope

- Technology : CesiumJS
- Versions : 1.124+ (WebGL2 only; WebGPU is roadmap-only, NOT shipped : see L-001)
- Languages : JavaScript, TypeScript
- Prefix : cesium
- License : MIT
- Excludes : Cesium Stories, Cesium for Unity, Cesium for Unreal, Cesium for Omniverse
- Includes : Cesium ion cloud platform (3D Tiles assets, terrain, imagery, geocoding) as companion

## Identified Topics (raw, pre-research)

Raw inventory derived from user scope brief. NOT researched. Phase 3 refines counts,
merges, splits, and may add/drop based on Phase 2 findings.

### Core (architecture, cross-cutting)

- `cesium-core-architecture` : Viewer + Scene + Camera + Globe + CesiumWidget render model, WebGL2 vs WebGPU backend, render loop, frameState
- `cesium-core-coordinate-systems` : Cartesian3, Cartographic, ECEF, WGS84, Transforms, ENU frames, conversion patterns
- `cesium-core-memory` : tileset.destroy(), viewer.destroy(), primitive lifecycle, leak prevention, context loss
- `cesium-core-performance` : LOD, frustum culling, fog, request scheduler, maximumScreenSpaceError, tile cache

### Syntax (API surface, code patterns)

- `cesium-syntax-viewer` : Viewer constructor options, widget toggles, CesiumWidget-only setups
- `cesium-syntax-camera` : Camera control, flyTo, lookAt, setView, ScreenSpaceCameraController
- `cesium-syntax-entity` : Entity API, ConstantProperty, SampledProperty, PolylineGlowMaterialProperty, EntityCollection
- `cesium-syntax-primitive` : Primitive API, GeometryInstance, GroundPrimitive, ModelPrimitive, appearances
- `cesium-syntax-imagery` : Imagery providers (Bing, ESRI, MapTiler, OSM, TileMapService, custom)
- `cesium-syntax-terrain` : Terrain providers (Cesium World Terrain, ArcGIS, ellipsoid, custom)
- `cesium-syntax-gltf-model` : Model class, ModelMatrix, animations, articulations, glTF loading
- `cesium-syntax-3d-tiles` : Cesium3DTileset, Cesium3DTileStyle, b3dm/i3dm/pnts/cmpt, batch/feature tables, hierarchy
- `cesium-syntax-datasources` : KmlDataSource, CzmlDataSource, GeoJsonDataSource, TopoJSON loading
- `cesium-syntax-time-dynamic` : Clock, JulianDate, SampledPositionProperty, time interval collections
- `cesium-syntax-materials-shaders` : Material, Fabric JSON, custom shaders, CustomShader for 3D Tiles/Model
- `cesium-syntax-atmosphere` : Atmosphere, sun/moon lighting, SkyBox, SkyAtmosphere, ground atmosphere, clouds

### Implementation (workflows, use cases)

- `cesium-impl-picking-measurement` : scene.pick, drillPick, pickPosition, selection, measurement tooling
- `cesium-impl-3d-tiles-workflow` : load + style + optimize tilesets end-to-end, clipping planes
- `cesium-impl-cesium-ion` : ion asset upload, tiling pipeline, access tokens, ion REST API
- `cesium-impl-geocoding-routing` : ion geocoder service, custom geocoders, search workflows
- `cesium-impl-resium` : Resium React wrapper, component model, hooks, companion pattern
- `cesium-impl-aec-georef` : CityGML via 3D Tiles, IFC georeferencing, geo-BIM, digital twin, infra-vis
- `cesium-impl-postprocess` : PostProcessStageCollection, custom post-process stages, effects pipeline
- `cesium-impl-sandcastle` : Sandcastle example structure, converting examples to production code

### Errors (error handling, debugging)

- `cesium-errors-rendering` : blank globe, black screen, WebGL context lost, no imagery
- `cesium-errors-tileset` : 3D Tiles load failures, CORS, token/401, missing tiles, SSE issues
- `cesium-errors-memory` : leaks, destroy ordering, detached primitives, growing heap
- `cesium-errors-coordinates` : NaN positions, terrain clamping failures, height/heading confusion

### Agents (orchestration, validation)

- `cesium-agents-scene-architect` : orchestrate viewer/scene/data-source setup decisions
- `cesium-agents-skill-validator` : cross-skill consistency, API verification, code quality checklist

## Estimated Skill Count

| Category | Estimated |
|----------|-----------|
| core | 4 |
| syntax | 12 |
| impl | 8 |
| errors | 4 |
| agents | 2 |
| **Total** | **~30** |

Phase 3 decision pending : whether to add a dedicated `tiles` or `geospatial` category
for depth, vs keeping 3D Tiles / coordinate topics inside syntax + core.

## Cross-Tech Boundaries

CesiumJS touches : QGIS (geo data input/export), ThatOpen (BIM glTF export), Speckle
(data federation), Three.js (alternative 3D renderer, Cesium is geo-first), IFC
(georeferenced BIM). Companion-skills section in README, no separate skills unless
Phase 2 reveals a real integration surface.

## Next : Phase 2 Deep Research

Dispatch 3 parallel opus research-agents per topic-cluster, each writing a fragment to
`docs/research/fragments/`. Main session consolidates into `docs/research/vooronderzoek-cesium.md`.

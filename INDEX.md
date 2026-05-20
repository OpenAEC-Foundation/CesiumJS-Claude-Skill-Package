# INDEX : CesiumJS 1.124+ Skill Package Catalog

## Overview

30 deterministic Claude skills across 5 categories for CesiumJS 1.124+ (WebGL2).

## Summary

| Category | Skills | Focus |
|----------|:------:|-------|
| **core** | 5 | Architecture, cross-cutting concerns |
| **syntax** | 12 | API syntax, signatures, code patterns |
| **impl** | 7 | Development workflows, end-to-end implementations |
| **errors** | 4 | Error handling, debugging, anti-patterns |
| **agents** | 2 | Validation, orchestration |
| **Total** | **30** | |

## core (5)

| Skill | Use when |
|-------|----------|
| `cesium-core-architecture` | Setting up a CesiumJS app and deciding how Viewer, CesiumWidget, Scene, Globe, and Camera fit together, or when the render loop, requestRenderMode, or SceneMode behaves unexpectedly. |
| `cesium-core-coordinates` | Converting between geographic coordinates and CesiumJS positions, placing entities or models, building local reference frames, or debugging off-globe or NaN positions. |
| `cesium-core-versioning` | Upgrading CesiumJS between releases, when old tutorial code throws, or when porting legacy synchronous patterns to the modern async factories. |
| `cesium-core-performance` | A CesiumJS app runs slowly, drops frames, stutters, drains battery, or streams 3D Tiles too slowly, and the cause must be measured and fixed. |
| `cesium-core-memory` | A CesiumJS app leaks memory, slows down over time, crashes the tab, or shows a blank Viewer after navigation or component remounts. |

## syntax (12)

| Skill | Use when |
|-------|----------|
| `cesium-syntax-viewer` | Creating a CesiumJS app, constructing a Viewer or CesiumWidget, configuring widget toggles, render options, the WebGL context, or the ion token. |
| `cesium-syntax-camera` | Positioning, flying, or constraining the camera, or when a view will not update, the camera ends up underground, or flight completion fires wrong. |
| `cesium-syntax-entity` | Adding points, billboards, labels, polylines, polygons, or models, building time-dynamic tracks, or debugging an entity that never appears. |
| `cesium-syntax-primitive` | Rendering large static geometry sets with the low-level Primitive API, batching shapes, draping geometry over terrain, or choosing Entity versus Primitive. |
| `cesium-syntax-imagery` | Adding, stacking, or styling raster map imagery, or when the globe shows no imagery, tiles return 401 or 403, or a provider fails to construct. |
| `cesium-syntax-terrain` | Adding elevation, choosing between World Terrain, a custom endpoint, an ion asset, or ESRI elevation, or debugging a flat globe. |
| `cesium-syntax-gltf-model` | Loading a glTF or glb model, placing and orienting it on the globe, playing animations, or driving articulations. |
| `cesium-syntax-3d-tiles` | Loading a 3D Tiles tileset, streaming a city, point cloud, or photogrammetry dataset, picking a feature, or tuning a tileset. |
| `cesium-syntax-datasources` | Loading CZML, GeoJSON, TopoJSON, KML, or KMZ files, or when data does not appear or sits flat on the ellipsoid. |
| `cesium-syntax-time` | An entity track does not move, an animation is frozen, a clock shows the wrong time, or a SampledPositionProperty fails to interpolate. |
| `cesium-syntax-materials` | Styling a surface with a Material, writing a CustomShader for a Model or tileset, or adding a screen-space PostProcessStage effect. |
| `cesium-syntax-atmosphere` | Configuring the sky, atmosphere, lighting, sun, moon, star field, or fog, or when the sky renders black or the globe looks too dark. |

## impl (7)

| Skill | Use when |
|-------|----------|
| `cesium-impl-picking-measurement` | A click handler hits nothing or the wrong object, a position reads at the wrong height, or when building distance and area measurement tooling. |
| `cesium-impl-3d-tiles-styling` | Styling tileset features with the declarative styling language, coloring or hiding features by metadata, or clipping a tileset. |
| `cesium-impl-cesium-ion` | Integrating Cesium ion assets, configuring the ion access token, uploading data for tiling, or deciding between ion SaaS and self-hosting. |
| `cesium-impl-geocoding` | A Geocoder widget search returns nothing, an ion geocode call fails, or a place name must become a camera destination. |
| `cesium-impl-resium` | Building a CesiumJS application in React with Resium, the declarative React component wrapper. |
| `cesium-impl-aec-georef` | Placing a BIM, CAD, IFC, or city model into the globe at its real geographic location, or when the model renders at the wrong place. |
| `cesium-impl-build-deploy` | A bundled CesiumJS app shows a blank globe, logs 404s for Workers or Assets, fails with a bundler error, or when converting a Sandcastle example to production. |

## errors (4)

| Skill | Use when |
|-------|----------|
| `cesium-errors-rendering` | A globe renders blank, black, white, or frozen, the canvas shows nothing, the WebGL context is lost, or a render-loop error panel covers the canvas. |
| `cesium-errors-tileset` | A Cesium3DTileset does not load: CORS errors, 401 or 403, readyPromise undefined, fromUrl resolves but nothing appears, or a RuntimeError. |
| `cesium-errors-memory` | Memory keeps growing, the tab slows or crashes, or the console reports "Too many active WebGL contexts" or "This object was destroyed". |
| `cesium-errors-coordinates` | Geometry renders in the wrong place, falls through terrain, sits at the center of the Earth, never appears, or position values come back as NaN. |

## agents (2)

| Skill | Use when |
|-------|----------|
| `cesium-agents-scene-architect` | Starting a new CesiumJS application or feature and deciding how to structure the scene: viewer class, imagery, terrain, data representation, render mode. |
| `cesium-agents-skill-validator` | CesiumJS code has just been generated or reviewed and must be checked before it is called done, to catch deprecated APIs and common defects. |

## Dependency Flow

```
core  ->  syntax  ->  impl  ->  errors  ->  agents
```

`core` skills are foundational. `syntax` skills depend on `core`. `impl` skills
compose `syntax`. `errors` skills diagnose failures across all of them. `agents`
skills orchestrate and validate the rest.

## Discovery

- npm-agentskills manifest : [package.json](package.json) `agents.skills[]`
- OpenAI Codex : [agents/openai.yaml](agents/openai.yaml)
- GitHub topic : `agentskills`

Each skill directory contains `SKILL.md` plus `references/methods.md`,
`references/examples.md`, and `references/anti-patterns.md`.

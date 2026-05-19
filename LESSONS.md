# Lessons Learned

Observations and findings captured during skill package development.

---

## L-001 : WebGPU is NOT a CesiumJS backend

- **Date** : 2026-05-20
- **Phase** : 2 (deep research)
- **Finding** : The project brief assumed "WebGL2 + WebGPU backend (in transition)". Research (Cesium Community forum, CesiumGS/cesium issue #4989) confirmed CesiumJS renders EXCLUSIVELY on WebGL2 through the current 1.142 release line. WebGPU is a long-term roadmap item with no shipped backend toggle.
- **Action** : Masterplan scope line corrected. All skills MUST treat WebGL2 as the only backend and MUST NOT reference a WebGPU rendering path.

---

## L-002 : Async factory constructors are mandatory on 1.124+

- **Date** : 2026-05-20
- **Phase** : 2 (deep research)
- **Finding** : The `readyPromise` pattern and synchronous provider constructors were deprecated in 1.104 and fully removed in 1.107. CesiumJS 1.124+ requires async static factories: `Cesium3DTileset.fromUrl` / `.fromIonAssetId`, `Model.fromGltfAsync`, `CesiumTerrainProvider.fromUrl`, `IonImageryProvider.fromAssetId`, `ImageryLayer.fromProviderAsync`.
- **Action** : Skills MUST never emit `readyPromise`, `.ready`, or `new Cesium3DTileset({url})`. This is the single biggest source of broken tutorial code and a cross-cutting migration topic.

---

## L-003 : CesiumJS has no routing capability

- **Date** : 2026-05-20
- **Phase** : 2 (deep research)
- **Finding** : Cesium ion and CesiumJS provide geocoding (`GeocoderService`, `IonGeocoderService`) which is address-to-coordinate only. There is NO routing/directions API. The raw masterplan skill `cesium-impl-geocoding-routing` was mis-scoped.
- **Action** : Phase 3 renames the skill to `cesium-impl-geocoding`. Any routing claim in a skill would be a hallucination.

---

## L-004 : Parallel cluster-research for broad single-tech packages

- **Date** : 2026-05-20
- **Phase** : 2 (deep research)
- **Finding** : BOOTSTRAP-RUNBOOK §4 prescribes 1 research agent for single-tech packages. CesiumJS API surface is broad enough (rendering + data + ecosystem) that 3 parallel cluster-agents, each writing an isolated fragment file, produced verified research faster with no file conflicts. Main session consolidated.
- **Action** : Workflow-level insight for the Workflow Template: allow parallel cluster-research when a single-tech API surface is comparable in breadth to a multi-tech package.

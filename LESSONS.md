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

---

## L-005 : Account session-limit under parallel tmux-orchestration

- **Date** : 2026-05-20
- **Phase** : 5 (skill creation)
- **Finding** : Two tmux worker sessions (cesium-w1, cesium-w3) hit the Claude account session-limit during batch 9 (reset 6am Amsterdam). The machine was running multiple skill-package orchestrations concurrently (PostgreSQL, IFC, CesiumJS), so 6+ worker-claude REPLs plus orchestrators shared one account quota. The limit is account-wide, so respawning workers does not recover it.
- **Action** : The orchestrator built the final batch (batch 10, 3 skills) directly instead of waiting ~3 hours for the limit to reset. Workflow-level lesson for the Workflow Template: when running tmux-orchestration alongside other orchestrations on the same account, expect the session-limit; the orchestrator completing a stalled batch directly is a valid no-stall recovery, and worker session names must be package-prefixed (cesium-w*) to avoid hijacking another orchestration's generic worker-N sessions.

---

## L-006 : Two-step Enter unreliable for large pasted bundles

- **Date** : 2026-05-20
- **Phase** : 5 (skill creation)
- **Finding** : Injecting a worker context bundle via `tmux load-buffer` + `paste-buffer` leaves the paste collapsed in the claude REPL input ("paste again to expand"); a single follow-up Enter often does not submit it. Two spaced Enter key-sends (0.5s apart) reliably submit. Extra Enters on an already-submitted prompt are harmless no-ops.
- **Action** : The injection helper sends paste, then two `send-keys Enter` calls with a 0.5s gap. Recorded for the tmux-orchestration skill.

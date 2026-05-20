# Changelog

All notable changes to the CesiumJS Skill Package.

Format follows [Keep a Changelog](https://keepachangelog.com/).

## [1.0.0] - 2026-05-20

### Added
- 30 deterministic Claude skills for CesiumJS 1.124+ across 5 categories:
  core (5), syntax (12), impl (7), errors (4), agents (2).
- Deep research base: `docs/research/vooronderzoek-cesium.md` plus three
  WebFetch-verified cluster fragments.
- Executable masterplan with 7 refinement decisions and a 10-batch plan.
- Discovery manifests: `package.json` `agents.skills[]` and `agents/openai.yaml`.
- `INDEX.md` skill catalog and Phase 6 validation reports.
- Compliance audit: 100%.

### Notes
- CesiumJS 1.124+ is WebGL2 only; no WebGPU rendering path exists (L-001).
- All skills mandate async factories; no removed `readyPromise`,
  `new Cesium3DTileset`, `ModelExperimental`, or `defaultValue` (L-002).

# Functional Sample Test : CesiumJS Skill Package

> Phase 6 functional validation. Date : 2026-05-20.
> Method : one skill per category reviewed against a realistic user prompt for
> trigger match (description and Keywords), guidance soundness (against the
> verified research base), and absence of hallucinated or deprecated APIs.

## Scope

Five skills, one per category. Each is checked on three criteria:

1. **Trigger** : would the skill description and `Keywords:` line match a real
   user prompt that the skill should handle.
2. **Guidance** : is the SKILL.md advice correct and consistent with
   `docs/research/vooronderzoek-cesium.md`.
3. **No hallucination** : every API named is verified; no removed API is
   recommended.

## core : cesium-core-architecture

- **Prompt tested** : "My CesiumJS globe is blank and `scene.viewer` is
  undefined, how is a Cesium app structured?"
- **Trigger** : PASS. Keywords include "blank globe", "Viewer and CesiumWidget",
  "Scene", and the description opens on the Viewer/Scene/Globe relationship.
- **Guidance** : PASS. Correctly states `Globe` is a member of `Scene`, there is
  no `scene.viewer` back-reference, and access is `viewer.scene.globe`. Matches
  vooronderzoek section 1.
- **No hallucination** : PASS. WebGL2-only stated; no WebGPU path; no
  `readyPromise`.

## syntax : cesium-syntax-3d-tiles

- **Prompt tested** : "How do I load a 3D Tiles tileset in CesiumJS 1.13x?"
- **Trigger** : PASS. Keywords include "tileset not loading", "Cesium3DTileset",
  "fromUrl"; description opens on loading a tileset.
- **Guidance** : PASS. Mandates `Cesium3DTileset.fromUrl` / `.fromIonAssetId`
  async factories; documents b3dm/i3dm/pnts versus glTF tile content and the
  1.0 versus 1.1 distinction. Matches vooronderzoek section 2.
- **No hallucination** : PASS. Explicitly forbids `new Cesium3DTileset({url})`
  and `readyPromise`, both removed in 1.107.

## impl : cesium-impl-cesium-ion

- **Prompt tested** : "I get a 401 loading my Cesium ion asset, what is wrong?"
- **Trigger** : PASS. Keywords include "401 unauthorized", "ion token", "asset
  not loading".
- **Guidance** : PASS. Requires `Ion.defaultAccessToken` before any ion asset;
  documents `IonResource.fromAssetId` and `Cesium3DTileset.fromIonAssetId`.
  Matches vooronderzoek section 4. The skill correctly flags that exact ion
  REST endpoint paths must be re-verified rather than quoted blindly.
- **No hallucination** : PASS.

## errors : cesium-errors-rendering

- **Prompt tested** : "My CesiumJS app shows a black screen and nothing
  renders."
- **Trigger** : PASS. Keywords include "blank globe", "black screen", "nothing
  renders", "context lost".
- **Guidance** : PASS. Lists the dominant root causes in priority order :
  unset `CESIUM_BASE_URL`, missing ion token, omitted bundler asset copy, WebGL
  context loss. Each entry has symptom, root cause, prevention, recovery.
  Matches vooronderzoek section 7.
- **No hallucination** : PASS.

## agents : cesium-agents-skill-validator

- **Prompt tested** : "Check this CesiumJS code before I commit it."
- **Trigger** : PASS. Keywords include "validate CesiumJS code", "check before
  commit", "deprecated CesiumJS API".
- **Guidance** : PASS. Deterministic 8-check list; Checks 1 to 4, 6, 7 are
  BLOCKER, Checks 5 and 8 are WARN; verdict rule is explicit. The removed-API
  table matches `cesium-core-versioning` and the verified `CHANGES.md` matrix.
- **No hallucination** : PASS. Check 7 mandates WebFetch verification of every
  API name, the safeguard against hallucinated APIs.

## Result

| Category | Skill | Trigger | Guidance | No hallucination |
|----------|-------|---------|----------|------------------|
| core | cesium-core-architecture | PASS | PASS | PASS |
| syntax | cesium-syntax-3d-tiles | PASS | PASS | PASS |
| impl | cesium-impl-cesium-ion | PASS | PASS | PASS |
| errors | cesium-errors-rendering | PASS | PASS | PASS |
| agents | cesium-agents-skill-validator | PASS | PASS | PASS |

All five sampled skills pass on all three criteria. Combined with the automated
suite (frontmatter, line count, structure, language, em-dash all PASS) and the
compliance audit (100%), the package meets the Phase 6 quality bar.

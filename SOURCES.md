# Sources : CesiumJS Skill Package

## Approved Sources

All skill content MUST be verified against these approved sources. No unverified blog
posts or AI-generated content.

### Primary Sources

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| CesiumJS API Reference | https://cesium.com/learn/cesiumjs/ref-doc/ | Official API Documentation | Not yet |
| CesiumJS Learn / Tutorials | https://cesium.com/learn/cesiumjs-learn/ | Official Tutorials | Not yet |
| Sandcastle Examples | https://sandcastle.cesium.com/ | Official Live Examples | Not yet |
| CesiumGS/cesium (source) | https://github.com/CesiumGS/cesium | GitHub Repository | Not yet |
| CesiumGS/cesium CHANGES.md | https://github.com/CesiumGS/cesium/blob/main/CHANGES.md | Version History | Not yet |
| 3D Tiles Specification | https://github.com/CesiumGS/3d-tiles | GitHub Spec Repository | Not yet |
| glTF Specification | https://github.com/CesiumGS/glTF | GitHub Spec Repository | Not yet |

### Secondary Sources (use only when primary is insufficient)

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Cesium ion docs | https://cesium.com/learn/ion/ | Cloud Platform Docs | Not yet |
| Resium docs | https://resium.reearth.dev/ | React Wrapper Docs | Not yet |
| CesiumGS/cesium GitHub Issues | https://github.com/CesiumGS/cesium/issues | Real-world Anti-patterns | Not yet |

## Verification Rules

1. **Primary sources ONLY**: Official docs > source code > official tutorials
2. **NEVER use**: Random blog posts, unverified StackOverflow answers, AI-generated content without verification
3. **Version-check**: Ensure source matches target version (1.124+)
4. **Date-check**: Note last verification date per source
5. **Cross-reference**: If official docs are sparse, verify against source code
6. **WebFetch**: ALWAYS use WebFetch to verify against latest official documentation (D-006)

## Source Addition Protocol

When discovering a new source during research:
1. Verify it's official or maintained by core team (CesiumGS / reearth for Resium)
2. Add to appropriate table above
3. Set "Last Verified" to current date
4. Record in LESSONS.md if the source revealed significant insights

## Last-Verified Log

| Date | Phase | What was verified |
|------|-------|-------------------|
| 2026-05-19 | Phase 1 | Source URLs registered, not yet fetched |

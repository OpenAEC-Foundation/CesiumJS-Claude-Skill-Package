# Sources : CesiumJS Skill Package

## Approved Sources

All skill content MUST be verified against these approved sources. No unverified blog
posts or AI-generated content.

### Primary Sources

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| CesiumJS API Reference | https://cesium.com/learn/cesiumjs/ref-doc/ | Official API Documentation | 2026-05-20 |
| CesiumJS Learn / Tutorials | https://cesium.com/learn/cesiumjs-learn/ | Official Tutorials | 2026-05-20 |
| CesiumJS Quickstart | https://cesium.com/learn/cesiumjs-learn/cesiumjs-quickstart/ | Official Tutorial | 2026-05-20 |
| Sandcastle Examples | https://sandcastle.cesium.com/ | Official Live Examples | 2026-05-20 |
| CesiumGS/cesium (source) | https://github.com/CesiumGS/cesium | GitHub Repository | 2026-05-20 |
| CesiumGS/cesium CHANGES.md | https://raw.githubusercontent.com/CesiumGS/cesium/main/CHANGES.md | Version History | 2026-05-20 |
| 3D Tiles Specification | https://github.com/CesiumGS/3d-tiles | GitHub Spec Repository | 2026-05-20 |
| glTF Specification | https://github.com/CesiumGS/glTF | GitHub Spec Repository | Not yet |

### Secondary Sources (use only when primary is insufficient)

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Cesium ion docs | https://cesium.com/learn/ion/ | Cloud Platform Docs | 2026-05-20 |
| Resium docs | https://resium.reearth.dev/ | React Wrapper Docs | UNREACHABLE 2026-05-20 |
| Resium source | https://github.com/reearth/resium | React Wrapper Repo | 2026-05-20 |
| CesiumGS/cesium GitHub Issues | https://github.com/CesiumGS/cesium/issues | Real-world Anti-patterns | 2026-05-20 |
| Cesium Community Forum | https://community.cesium.com/ | Official Forum (CesiumGS-maintained) | 2026-05-20 |
| cesium-webpack-example | https://github.com/CesiumGS/cesium-webpack-example | Official Build Reference | Not yet |
| CZML Guide | https://github.com/AnalyticalGraphicsInc/czml-writer/wiki/CZML-Guide | CZML Format Spec | Not yet |

## Verification Rules

1. **Primary sources ONLY**: Official docs > source code > official tutorials
2. **NEVER use**: Random blog posts, unverified StackOverflow answers, AI-generated content without verification
3. **Version-check**: Ensure source matches target version (1.124+)
4. **Date-check**: Note last verification date per source
5. **Cross-reference**: If official docs are sparse, verify against source code
6. **WebFetch**: ALWAYS use WebFetch to verify against latest official documentation (D-006)
7. **CHANGES.md**: Use the `raw.githubusercontent.com` form; the rendered GitHub blob page does not expose changelog text to WebFetch.

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
| 2026-05-20 | Phase 2 | ref-doc, Learn, Quickstart, Sandcastle, CHANGES.md, 3d-tiles spec, ion docs, Resium repo, GitHub issues verified. Resium docs site unreachable (ECONNREFUSED). glTF spec + webpack example + CZML guide pending Phase 4. |

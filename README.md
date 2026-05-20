# CesiumJS : Claude Skill Package

<p align="center">
  <img src="docs/social-preview.png" alt="30 Deterministic Skills for CesiumJS" width="100%">
</p>

![Claude Code Ready](https://img.shields.io/badge/Claude_Code-Ready-blue?style=flat-square)
![CesiumJS](https://img.shields.io/badge/CesiumJS-1.124+-0A66C2?style=flat-square)
![Skills](https://img.shields.io/badge/Skills-30-green?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![Agent Skills](https://img.shields.io/badge/agent--skills-compatible-purple?style=flat-square)

**30 deterministic Claude AI skills for CesiumJS : the WebGL geospatial 3D platform. Covers Viewer and Scene architecture, 3D Tiles, glTF models, imagery and terrain providers, KML/CZML/GeoJSON data sources, Resium, Cesium ion, digital-twin and AEC geo-BIM workflows.**

Built on the [Agent Skills](https://agentskills.org) open standard. Discoverable via npm-agentskills manifest and OpenAI Codex skill discovery.

## Why This Exists

Without skills, Claude reaches for outdated CesiumJS patterns from pre-2023 training data. The synchronous tileset constructor and `readyPromise` were removed in CesiumJS 1.107, so that code throws on any 1.124+ target:

```js
// Wrong : the url constructor option and readyPromise were removed in 1.107
const tileset = new Cesium.Cesium3DTileset({
  url: Cesium.IonResource.fromAssetId(69380),
});
tileset.readyPromise.then(() => viewer.zoomTo(tileset));
```

With this skill package, Claude produces the modern async-factory pattern:

```js
// Correct : async factory; await replaces the readyPromise chain entirely
const tileset = await Cesium.Cesium3DTileset.fromIonAssetId(69380);
viewer.scene.primitives.add(tileset);
await viewer.zoomTo(tileset);
```

## What's Inside

| Category | Count | Purpose |
|----------|:-----:|---------|
| **core/** | 5 | Architecture, cross-cutting concerns |
| **syntax/** | 12 | API syntax, code patterns, signatures |
| **impl/** | 7 | Step-by-step development workflows |
| **errors/** | 4 | Error handling, debugging, anti-patterns |
| **agents/** | 2 | Validation, orchestration |
| **Total** | **30** | |

See [INDEX.md](INDEX.md) for the complete skill catalog with descriptions and the dependency flow.

## Installation

### Claude Code (recommended)

```bash
# Clone the full package
git clone https://github.com/OpenAEC-Foundation/CesiumJS-Claude-Skill-Package.git
cp -r CesiumJS-Claude-Skill-Package/skills/source/ ~/.claude/skills/cesium/
```

### As git submodule

```bash
git submodule add https://github.com/OpenAEC-Foundation/CesiumJS-Claude-Skill-Package.git .claude/skills/cesium
```

### Via npm-agentskills standard

```bash
npx skills add @openaec/cesium-claude-skill-package
```

### Claude.ai (web)

Upload individual SKILL.md files as project knowledge.

## Skill Structure

Every skill follows 3-level progressive disclosure:

```
cesium-{category}-{topic}/
├── SKILL.md              # Main guidance (< 500 lines)
└── references/
    ├── methods.md        # Complete API signatures
    ├── examples.md       # Working code examples
    └── anti-patterns.md  # What NOT to do (with explanations)
```

YAML frontmatter uses folded scalar `>`, a "Use when..." opener, and a `Keywords:` line with technical, symptom-based, and plain-language terms for maximum discoverability.

## Quality Guarantees

- **Deterministic language** : ALWAYS / NEVER, no "you might consider"
- **WebGL2-only** : CesiumJS 1.124+ renders on WebGL2; no WebGPU path is assumed
- **Async-factory correct** : no removed `readyPromise`, `new Cesium3DTileset`, `ModelExperimental`, or `defaultValue`
- **WebFetch-verified** : all code snippets validated against official docs
- **CI/CD validated** : frontmatter, line count, structure, language, em-dash checks on every push
- **Compliance audit** : 100% on the Phase 6 audit

## Companion Skills : Cross-Technology Integration

For projects combining CesiumJS with other AEC technologies, see [Cross-Tech-AEC-Claude-Skill-Package](https://github.com/OpenAEC-Foundation/Cross-Tech-AEC-Claude-Skill-Package). CesiumJS commonly pairs with QGIS (geo data), ThatOpen and IfcOpenShell (BIM to glTF), and Speckle (data federation); the `cesium-impl-aec-georef` skill covers the georeferencing boundary.

## Related Skill Packages (OpenAEC Foundation)

| Package | Skills | Repo |
|---------|--------|------|
| Blender-Bonsai-ifcOpenshell-Sverchok | 73 | [Link](https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package) |
| Frappe | 61 | [Link](https://github.com/OpenAEC-Foundation/Frappe_Claude_Skill_Package) |
| Speckle | 25 | [Link](https://github.com/OpenAEC-Foundation/Speckle-Claude-Skill-Package) |

See the full list at [OpenAEC-Foundation](https://github.com/OpenAEC-Foundation).

## License

MIT : OpenAEC Foundation

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Built with the [Skill Package Workflow Template](https://github.com/OpenAEC-Foundation/Skill-Package-Workflow-Template) methodology.

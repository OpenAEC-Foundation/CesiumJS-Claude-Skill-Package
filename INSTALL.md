# Install : CesiumJS Skill Package

## Claude Code (local)

```bash
git clone https://github.com/OpenAEC-Foundation/CesiumJS-Claude-Skill-Package.git
cp -r CesiumJS-Claude-Skill-Package/skills/source/ ~/.claude/skills/cesium/
```

Restart Claude Code if running. Skills auto-discover from `~/.claude/skills/`.

## As git submodule (recommended for project-specific use)

```bash
cd your-project
git submodule add https://github.com/OpenAEC-Foundation/CesiumJS-Claude-Skill-Package.git .claude/skills/cesium
git commit -m "feat: add cesium skills as submodule"
```

## Via npm-agentskills

```bash
npx skills add @openaec/cesiumjs-claude-skill-package
```

## Claude.ai (web)

1. Open a Claude.ai project
2. Upload individual `SKILL.md` files from `skills/source/**/` as project knowledge
3. Skills activate based on description triggers

## Verification

After install, ask Claude :

> {{VERIFICATION_PROMPT}}

If skill activates : install successful. If not : check `~/.claude/skills/cesium/` exists and contains category-folders.

## Requirements

- CesiumJS 1.124+
- Claude Code (latest)
- {{ADDITIONAL_REQUIREMENTS}}

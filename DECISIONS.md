# Architectural Decisions

Numbered decisions (D-XXX) with rationale. Immutable once recorded — new decisions can supersede but never delete old ones.

---

## D-001: English-Only Skills

- **Date**: 2026-05-19
- **Decision**: All skill content MUST be in English only.
- **Rationale**: Skills are instructions FOR Claude, not for end users. Claude reads English and responds in the user's language. Bilingual skills double maintenance with zero functional benefit. Proven in ERPNext (28 skills) and Blender-Bonsai (73 skills).
- **Consequence**: No translations needed. All descriptions, code comments, and documentation in English.

---

## D-002: MIT License

- **Date**: 2026-05-19
- **Decision**: Project uses MIT License.
- **Rationale**: Most permissive license, maximizes adoption. Consistent with OpenAEC Foundation standards.
- **Consequence**: No commercial restrictions. Community-friendly.

---

## D-003: SKILL.md Under 500 Lines

- **Date**: 2026-05-19
- **Decision**: SKILL.md files MUST be under 500 lines.
- **Rationale**: Keeps main skill focused on decision trees and quick reference. Heavy content belongs in references/ directory. Proven optimal in ERPNext (180-427 lines per skill).
- **Consequence**: Complex topics split between SKILL.md (quick reference + patterns) and references/ (complete API, examples, anti-patterns).

---

## D-004: 7-Phase Research-First Methodology

- **Date**: 2026-05-19
- **Decision**: Follow the 7-phase research-first development methodology.
- **Rationale**: Proven in ERPNext (28 skills), Blender-Bonsai (73 skills), and Tauri 2 (27 skills). Research prevents hallucination. Deterministic skills require deep understanding.
- **Consequence**: No skill creation without prior deep research. Phases are sequential with defined exit criteria.

---

## D-005: ROADMAP.md as Single Source of Truth

- **Date**: 2026-05-19
- **Decision**: ROADMAP.md is the ONLY place for project status.
- **Rationale**: Multiple status locations cause drift and "which is current?" confusion. Single source of truth enables reliable session recovery.
- **Consequence**: Never duplicate status in CLAUDE.md or other files. All status references point to ROADMAP.md.

---

## D-006: WebFetch for Research Verification

- **Date**: 2026-05-19
- **Decision**: Use WebFetch to verify all code examples against latest official documentation.
- **Rationale**: Technology APIs evolve. Training data may be stale. WebFetch ensures latest official docs are consulted, not outdated cached knowledge.
- **Consequence**: All code examples must be verified against current official documentation before inclusion in skills.

---

## D-007: GitHub Publication Under OpenAEC Foundation

- **Date**: 2026-05-19
- **Decision**: Publish all skill packages under the OpenAEC Foundation GitHub organization.
- **Rationale**: Centralized, consistent branding. Community ownership. Discoverability.
- **Consequence**: All repos follow OpenAEC naming conventions and include social preview banners with OpenAEC branding.

---

## D-008: 30 Skills Across 5 Standard Categories

- **Date**: 2026-05-20
- **Decision**: The CesiumJS package contains exactly 30 skills organized in the 5 standard categories (core 5, syntax 12, impl 7, errors 4, agents 2). No extra `tiles` or `geospatial` category is created.
- **Rationale**: 30 skills fits comfortably in the standard core → syntax → impl → errors → agents dependency chain. Extra categories are reserved for packages exceeding 60 skills (Frappe pattern, BOOTSTRAP-RUNBOOK §5.2). 3D Tiles depth is handled by splitting into a syntax skill (loading, formats) and an impl skill (styling, clipping).
- **Consequence**: Skill naming stays `cesium-{core|syntax|impl|errors|agents}-{topic}`. The full inventory and refinement decisions (D-01..D-07) live in `docs/masterplan/cesium-masterplan.md`.

# slint

Agent skill for Slint UI work (`.slint` + Rust).

Mandatory for every Slint `.slint` + Rust UI change — Matches tables, cards,
day bands, HUD overlay. Enforces Rust-owns-values, shared widths, 1280x720
floor, and screenshot proof. No direct `.slint` edits without it.

## What it covers

- Slint language essentials (components, bindings, callbacks, structs)
- Rust integration (build.rs, generated code, models, callbacks)
- Standard widgets (`std-widgets.slint`)
- Layout rules (shared widths, 1280x720 floor)
- Tooling (LSP, viewer, SlintPad, Figma inspector)
- Troubleshooting + version notes (1.x)
- Repo patterns for `outpeek-app` / `outpeek-overlay` (read
  `references/repo-patterns.md` first when touching that UI)

## Contents

- `SKILL.md` — the skill instructions
- `references/` — deep-dive docs (language, layout, rust-integration,
  widgets, tooling, troubleshooting, versions, showcase, repo-patterns)

## Install

Copy into your skills directory:

```bash
# personal install (DeepSeek Harness)
cp -r slint ~/.dsh/skills/

# or project-local install
cp -r slint <project>/.dsh/skills/
```

## Requirements

- Slint 1.19
- Rust (edition 2021; UI crates use Slint edition 1.19)

## License

MIT — see [LICENSE](LICENSE).

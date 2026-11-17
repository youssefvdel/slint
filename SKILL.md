---
name: slint
description: Mandatory for every Slint .slint + Rust UI change in this repo — Matches tables, cards, day bands, HUD overlay. Enforces Rust-owns-values, shared widths, 1280x720 floor, and screenshot proof. No direct .slint edits without it.
---

# Slint Outpeek — General Slint Mastery

Use for any `.slint` work: full language, Rust binding, std-widgets,
layout, tooling — plus this repo's `outpeek-app` / `outpeek-overlay` patterns.

Read `references/repo-patterns.md` first when touching this repo's UI;
read `references/layout.md` before changing any width;
read `references/language.md` before using unfamiliar syntax.

## Non-negotiables (repo)

1. **Rust owns every value; `.slint` renders, never formats a fact.**
2. Empty string = "not reported" = em dash in `Cell`; never `0` or invented text.
3. **Shared widths are single-sourced.** Column widths live in one `col-width` / `chip-width` property, never copy-pasted. If a fixed `width` is unavoidable, write the row sum in a comment.
4. **No invented APIs.** Use only `references/` + linked Slint docs. Anything else is marked `UNSURE` — stop and verify.
5. **Min floor is 1280x720.** `AppWindow { min-width: 1280px; min-height: 720px; }`; `main.rs` clamps restored size to it. Screenshot at the floor, not just comfortable size.
6. **Screenshot proof ships with the change.** `cargo build --features ui-debug`, run with `--mcp-port`, `take_screenshot`, read the image. Unseen = unverified.

## General Slint rules (all projects)

- `.slint` file = top-level items only (`component`, `import`/`export`, `struct`/`enum`, `global`). Comments `//` and `/* */`.
- Instantiate: `TypeName { }`. Named: `id := Type { }`. `id` unique per component; `self`/`parent`/`root` reserved. Built-ins need no import.
- App root is normally `export component X inherits Window` (or `Dialog`).
- Inheriting instance IS the base: body bindings apply to it, children become its children, inherits all base props.
- Non-private top-level props form the component interface (settable from outside + native API). Binding `prop: expr;` re-evaluates reactively.
- `pure` contexts (bindings): only pure functions/callbacks; assigning a prop or calling an impure callback there is an error.
- Custom layout containers place use-site children at `@children`.
- Fonts: use `font-*` or `Window.default-font-*`; custom fonts via `import "./Noto.ttf";` then `font-family: "Noto Sans";`.
- Math lives in global scope + `Math.*` + postfix: `abs(x)` / `Math.abs(x)` / `(-10).abs()`.

## Workflow

1. Rust first: format strings, `SharedString` / `ModelRc` / `Image` in Rust; expose via struct/global props.
2. `.slint` second: bind props, `if`/`for` render, `animate` only on binding/state change.
3. Check `build.rs`: edition `(1,19)` + include path `../outpeek-overlay/ui` intact; shared tokens imported, never copied.
4. General code: pick component/props/callbacks/functions/globals per `references/language.md`; wire Rust per `references/rust-integration.md`; pick widgets per `references/widgets.md`; check tooling per `references/tooling.md`.
5. Build + screenshot at `1280x720` and at wide; confirm no clip/collide/elide, bars left-anchored, portraits never collapse rows (missing `image` draws glyph at same size).
6. If a detail is off by one step (`4px` vs `8px`), it is still off — fix it.

## Width arithmetic (floor must fit)

- Rail `Shell.rail-width: 272px` + page gutters + Matches columns ≈ `940px` → floor `1280px`. If the row sum exceeds the card, it clips — recompute, don't shrink fonts.
- Match row baseline: `66px` tall, `8px` radius, `1px` border, `10px` gap, `4px` result spine, `44px` portrait, `28px` rank medal, kebab at right edge.
- Stat cells share the row: `horizontal-stretch: 1` + `min-width`; fixed cells (e.g. `K/D 54px`, `K/D/A 150px`, `DDΔ 64px`, `HS % 56px`, `ACS 56px`) must still sum inside the card. Header, rows, day bands use the same widths in the same order.
- Bars anchor left: `x: 0` + clamped share `parent.width * clamp(share,0,1)`. Centred fill reads as a wrong value.
- Numbers never touch: value-to-value needs a visible gutter; give the tail its own layout if two numbers merge.

## References

- `references/language.md` — full `.slint`: file/component/props/callbacks/functions/modules/structs/enums/globals/types/expressions/statements/if-for/states/animate/elements/gradients/images.
- `references/layout.md` — stretch algorithm, constraints, absolute geometry, Box variants, `@children`, floor arithmetic.
- `references/rust-integration.md` — `build.rs`, `include_modules!`, props/callbacks/models/images/event loop.
- `references/widgets.md` — std-widgets covered by inputs (Box helpers, `ScrollView`, `StandardButton`, menus) + `UNSURE` gaps.
- `references/tooling.md` — compiler config, editions, library paths, fonts/images, SlintPad notes.
- `references/showcase.md` — copy-paste patterns built only from `references/` syntax.
- `references/repo-patterns.md` — AppWindow shell, Theme/Tracker, Matches/cards/HUD patterns.
- `references/troubleshooting.md` — pitfalls + `UNSURE` gaps.
- `references/versions.md` — pinned editions/versions + doc links.

Gap rule: anything not in `references/` or findings is `UNSURE`. Do not guess Slint API — look it up, then add the source.

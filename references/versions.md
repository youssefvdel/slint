# Versions + Doc Links (pinned)

## Pinned in this repo

- **Slint 1.19** — both crates opt in via `CompilerConfiguration::new().with_edition((1, 19))`. Do not bump one crate alone; `if`/`for` sizing changes with the edition.
- **Rust 2021** — backend + tokio + `windows-rs`; target `x86_64-pc-windows-msvc`, Windows 10/11 only.
- **UI backend:** `winit` + `femtoVG-wgpu` (DX12 on Windows).
- **Shared UI:** `outpeek-app/build.rs` include path `../outpeek-overlay/ui`; keep crates siblings or fix `Cargo.toml` path dep + `build.rs` together.

## Doc links (only sources findings came from)

### Language book

- https://docs.slint.dev/latest/docs/slint/guide/language/coding/file/
- https://docs.slint.dev/latest/docs/slint/reference/language/file-structure/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/properties/
- https://docs.slint.dev/latest/docs/slint/reference/language/properties/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/functions-and-callbacks/
- https://docs.slint.dev/latest/docs/slint/reference/language/callbacks/
- https://docs.slint.dev/latest/docs/slint/reference/language/functions/
- https://docs.slint.dev/latest/docs/slint/reference/language/imports/
- https://docs.slint.dev/latest/docs/slint/reference/language/exports/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/structs-and-enums/
- https://docs.slint.dev/latest/docs/slint/reference/language/structs-and-enums/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/globals/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/expressions-and-statements/
- https://docs.slint.dev/latest/docs/slint/reference/language/operators/
- https://docs.slint.dev/latest/docs/slint/reference/language/repetition-and-conditional-elements/
- https://docs.slint.dev/latest/docs/slint/reference/language/states-and-transitions/
- https://docs.slint.dev/latest/docs/slint/reference/language/animations/
- https://docs.slint.dev/latest/docs/slint/reference/global-functions/math/
- https://docs.slint.dev/latest/docs/slint/reference/property-types/colors-and-brushes/
- https://docs.slint.dev/latest/docs/slint/reference/property-types/images/
- https://docs.slint.dev/latest/docs/slint/reference/language/lexical-structure/
- https://docs.slint.dev/latest/docs/slint/reference/language/expressions/
- https://docs.slint.dev/latest/docs/slint/guide/development/fonts/

### Layouts / elements / widgets / windows

- https://docs.slint.dev/latest/docs/slint/reference/layouts/verticallayout/ + horizontallayout/ + positioning-and-layouts
- https://docs.slint.dev/latest/docs/slint/reference/elements/text/
- https://docs.slint.dev/latest/docs/slint/reference/elements/image/
- https://docs.slint.dev/latest/docs/slint/reference/elements/rectangle/
- https://docs.slint.dev/latest/docs/slint/reference/gestures/toucharea/
- https://docs.slint.dev/latest/docs/slint/reference/keyboard-input/focusscope/
- https://docs.slint.dev/latest/docs/slint/reference/gestures/flickable/
- https://docs.slint.dev/latest/docs/slint/reference/std-widgets/views/scrollview/
- https://docs.slint.dev/latest/docs/slint/reference/window/window/
- https://docs.slint.dev/latest/docs/slint/reference/window/dialog/
- https://docs.slint.dev/latest/docs/slint/reference/window/popupwindow/
- https://docs.slint.dev/latest/docs/slint/guide/language/coding/file/ (library paths)

### Rust

- https://docs.slint.dev/latest/docs/rust/slint/ (Generated components, build.rs compile, include_modules!)
- https://docs.slint.dev/latest/docs/rust/slint/docs/type_mappings/
- https://docs.slint.dev/latest/docs/rust/slint_build/struct.CompilerConfiguration + src/slint_build/lib.rs
- https://docs.slint.dev/latest/docs/rust/slint/struct.ModelRc + trait.Model + struct.Image
- https://docs.slint.dev/latest/docs/rust/slint/struct.Brush (`Brush::color`)
- https://docs.slint.dev/latest/docs/rust/slint/node Brush interface
- C++ generated-code + rust SampleComponent (public-function `invoke_` pattern); globals Logic sample

### Repo (local, pinned to tree at skill creation)

- `outpeek-app/build.rs` + `outpeek-overlay/build.rs` (edition + include paths)
- `outpeek-app/src/main.rs` (`include_modules!`, `AppWindow`, `global::<Tracker>()`, `on_navigate`/`on_select_tab`/`on_refresh_rank`, `ModelRc`/`VecModel`/`SharedString`)
- `outpeek-app/ui/app.slint` + `state.slint` + `outpeek-overlay/ui/overlay.slint`, `tokens.slint`, `facts.slint`

## Currency rule

- If local Slint version ≠ docs `latest`, re-verify `animate` fields, `for`/`if` sizing, and `Model`/`Image` APIs before changing code. Mark drift `UNSURE` until checked.

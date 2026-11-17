# Troubleshooting (verified pitfalls + UNSURE gaps)

## Pitfalls (all verified — do not repeat)

1. **Strong capture leaks.** Capturing strong `AppWindow` in `on_` closure leaks; use `as_weak().upgrade()`.
2. **Off-thread UI touch fails.** Touching UI off UI thread fails; wrap in `invoke_from_event_loop` after loop starts (repo: `main.rs` + `hud.rs`).
3. **Edition drop reverts layout.** `with_edition((1,19))` changes `if`/`for` sizing; omitting it reverts layout (repo: both `build.rs` comments).
4. **Image decoders missing.** `Image::load_from_data` without `image-decoders` fails; SVG buffer without `<?xml`/`<svg` needs explicit `Some("svg")`.
5. **Bad `animate` fields error.** `animate` body allows only `delay/duration/easing/iteration-count/direction/enabled`; other fields error.
6. **`for` scope trap.** `for` model expr cannot see `name`/`index`; globals are per-component instances, not process singletons.
7. **Fixed-width row clip.** A row of fixed-width cells clips the last cell — recompute floor arithmetic, add `horizontal-stretch: 1` + `min-width`, never shrink fonts to fit.
8. **Centred bar lies.** Bar without `x: 0` reads as a different value — anchor left + clamp share.
9. **Layout overwrite on launch.** `window().set_size()` before first event-loop turn gets overwritten — set after first turn (`invoke_from_event_loop` / first-event resize).
10. **Copied tokens drift.** Shared tokens via include path, never copied — a copied `tokens.slint` silently forks Theme.

11. **Backdrop-blur silently missing off Skia.** `backdrop-blur` is Skia-renderer ONLY (verified 2026-11-17) — software/FemtoVG/Vello/Qt draw the rectangle without blur, no error. Only `winit-skia*` / `linuxkms-skia` backends serve it. OutPeek ships FemtoVG-wgpu/DX12 → expect NO blur; design translucent surfaces that read fine unblurred. Exact spelling `backdrop-blur: length = 0px`; zero/negative disables.
12. **Drop-shadow-spread ignored on Qt.** All 5 shadow props verified 2026-11-17 (`drop-shadow-offset-x/y`, `drop-shadow-blur`, `drop-shadow-spread`, `drop-shadow-color`, all `0px`/transparent defaults): `spread` works on Skia/FemtoVG/Vello/software, Qt ignores it. Children never contribute; transparent fill + no border = no shadow.
13. **Popup auto-closes by default.** `PopupWindow.close-policy` EXACT 3 (verified 2026-11-17, default `close-on-click`): `close-on-click` (any click or Escape), `close-on-click-outside`, `no-auto-close` (call `close()` yourself). Rust: `PopupClosePolicy::{CloseOnClick, CloseOnClickOutside, NoAutoClose}`.

## Symptoms → checks

- Window opens at min width (~996x819 outer) instead of preferred → `max-width` finite + content-derived max won over `preferred-width`; check layout preferred computation, keep 4K clamp.
- Matches/Agents/Maps stop short of window edge → missing `width/height: 100%` on `Flickable` + `content-width/height` (see app.slint comments).
- Numbers collide at 1280 → row sum exceeds card; recheck `col-width` sum + gutters + rail 272px.
- Empty image collapses row → missing `if art.width == 0: Glyph` fallback at same size.
- Em dash missing / `0` shown for unknown → Rust sent `""`? `Cell` must map empty → em dash, never format in `.slint`.

## UNSURE — stop and verify (do not guess)

- `image-fit` values beyond `contain`; `image-rendering` options — look up element docs.
- Callback arg/return limits beyond `string`/`int`; `invoke_` naming for camelCase — check generated-code docs.
- `export * from` glob behaviour in this Slint version — verify imports doc.
- `ComponentHandle::hide` on frameless `no-frame` windows — verify.
- `slint::Timer` repeat modes beyond `single_shot` — look up `slint::Timer` docs.
- `PhysicalSize` vs logical units in `window().set_size` — verify `window()` docs.
- Enum payload shapes in `.slint` ↔ Rust mapping — verify type_mappings.
- Anything not in these references or the linked docs is `UNSURE` by definition.

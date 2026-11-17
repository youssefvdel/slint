# Tooling — Compiler, Editions, Assets

> Anything not listed here is `UNSURE` — look it up, don't guess.

## build.rs (Rust projects)

```rust
slint_build::compile("ui/hello.slint")
slint_build::compile_with_config(path, config)
```

- Must run in cargo build script; `.unwrap()` / `.expect()` to fail the build.
- Full config (OutPeek pattern — copy verbatim for this repo):

```rust
let config = slint_build::CompilerConfiguration::new()
    .with_edition((1, 19))
    .with_include_paths(vec![std::path::PathBuf::from("../outpeek-overlay/ui")]);
slint_build::compile_with_config("ui/app.slint", config)
    .expect("outpeek-app: ui/app.slint must compile");
```

## Editions

- `with_edition((1, 19))` changes `if`/`for` sizing: children count toward parent preferred-size, state-kept props keep defaults. Omitting it reverts to pre-1.19 layout (no-state `x`/`y`/`w`/`h` were `0px`, `if`/`for` children excluded).
- Both crates must opt in together; never bump one crate alone.

## Include paths vs library paths

- `with_include_paths`: plain relative imports (`import { Theme } from "tokens.slint";`) resolve against file dir, then include paths. Repo pattern: shared tokens via include path, never copied.
- `with_library_paths`: `@`-imports (`import { X } from "@lib/file.slint";`) resolve via `slint-project.json` `library-paths` or `with_library_paths`. `UNSURE`: exact `slint-project.json` schema — verify docs before use.

## Fonts

- `import "./Noto.ttf";` (ttf/ttc/otf), then `font-family` / `default-font-family: "Noto Sans";`.
- `Text`/`TextInput` use `font-*`, everything else falls back to `Window.default-font-*` (`default-font-family`, `default-font-size`, `default-font-weight` — exact prop names beyond family/size `UNSURE`, verify window docs).
- `FontWeight` namespace: EXACT 9 hyphenated lowercase members (verified 2026-11-17) — `thin` 100, `extra-light` 200, `light` 300, `normal` 400, `medium` 500, `semi-bold` 600, `bold` 700, `extra-bold` 800, `black` 900. Access `FontWeight.extra-bold`. `rem` = `Window.default-font-size` × factor.
- Embedded targets: must embed the font + `EMBED_RESOURCES` / `SLINT_FONT_SIZES` / SDF options. `UNSURE`: exact flag spellings per target — verify guide/development/fonts.

## Images

- `@image-url("path")` / nine-slice `@image-url("path", nine-slice(t r b l))` (`1` = all, `2` = v h, `4` = trbl). Compile-time string literal; relative-to-file, then include-paths; `http` only in SlintPad; `data:` URIs ok.
- Formats: SVG + image-crate list (AVIF BMP DDS Farbfeld GIF HDR ICO JPEG EXR PNG PNM QOI TGA TIFF WebP). Rust default decodes PNG/JPEG/SVG (+`image-default-formats` cargo feature for more); web builds use browser decoders except `.svgz`.
- Rust loading needs `image-decoders` (or wasm+std): `Image::load_from_path` / `Image::load_from_data(&bytes, Some("png"))`. SVG buffer without `<?xml`/`<svg` needs explicit `Some("svg")`. Without decoders, load fails.
- `Image` element: animated GIF/WebP play; `.width`/`.height` int px (`0` if empty); `image-fit: fill|contain|cover|preserve`, `image-rendering: smooth|pixelated`, `colorize`, `source-clip-*` for atlases.

## SlintPad notes

- `http` imports / image URLs work in SlintPad; local relative paths resolve against the pad file. `UNSURE`: exact SlintPad import limits — verify docs before relying on it.
- SlintPad always runs latest Slint; if local Slint ≠ pad version, re-verify `animate` fields, `for`/`if` sizing, and `Model`/`Image` APIs. Mark drift `UNSURE` until checked.

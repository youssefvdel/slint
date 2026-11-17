# Rust Integration

> Rule: Rust owns every value; `.slint` renders. Format in Rust, bind in `.slint`.
> Anything not listed here is `UNSURE`.
> Covers general Slint+Rust plus the OutPeek pattern (copy the OutPeek `build.rs` verbatim for this repo).

## build.rs — minimal (any project)

```rust
slint_build::compile("ui/hello.slint")
// or
slint_build::compile_with_config(path, config)
```

- Must run in cargo build script, `.unwrap()` / `.expect()` to fail build (docs: rust/slint build.rs, slint_build lib.rs).

## build.rs — OutPeek pattern (copy this for this repo)

```rust
// outpeek-app/build.rs
let config = slint_build::CompilerConfiguration::new()
    .with_edition((1, 19))
    .with_include_paths(vec![std::path::PathBuf::from("../outpeek-overlay/ui")]);
slint_build::compile_with_config("ui/app.slint", config)
    .expect("outpeek-app: ui/app.slint must compile");
```

```rust
// outpeek-overlay/build.rs — edition only
let config = slint_build::CompilerConfiguration::new().with_edition((1, 19));
slint_build::compile_with_config("ui/overlay.slint", config)
    .expect("outpeek-overlay: ui/overlay.slint must compile");
```

- Docs: `with_include_paths`; repo: both `build.rs`.
- `with_edition((1,19))` changes `if`/`for` sizing (excluded from parent preferred-size unless ≥1.19); omitting it reverts layout.

## Library paths (`@example` imports, any project)

```rust
CompilerConfiguration::new().with_library_paths(HashMap{"example" -> path})
```

- For `import { X } from "@lib/file.slint"` via `slint-project.json` `library-paths` (docs: coding/file, with_library_paths).
- Repo pattern: shared tokens via include path, never copied.

## Generated access (any exported Window/Dialog)

```rust
slint::include_modules!();
let app = AppWindow::new().expect("...");
app.show().unwrap();
app.run().unwrap();
// handles
app.as_weak(); app.window(); app.global::<Tracker>();
```

- `ComponentHandle::{show,hide,run,clone_strong,as_weak,window,global<T>}` (docs: include_modules!, Generated components; repo: `main.rs`).
- Only exported `Window`/`Dialog` become Rust structs; `window().set_size(...)` only after first event-loop turn or layout overwrites — see Event-loop section.

## Generated members (dashes become `_`)

```rust
app.set_window_width(x); app.get_window_width();
app.on_navigate(|s| { ... });
app.invoke_toggle_hud();
let t: Tracker = app.global::<Tracker>();
t.set_window_width(x); t.on_select_match(|id| { ... });
```

- Per prop `get_<p>`/`set_<p>`, per callback `invoke_<cb>`/`on_<cb>(fn)`, per fn `invoke_<f>`, global via `app.global::<T>()` (docs: Generated components/SampleComponent; repo: `app.global::<Tracker>().set_window_width`, `app.on_navigate`).

## Native type mapping (general)

- `.slint string` ↔ `slint::SharedString`; `image` ↔ `slint::Image`; `color` ↔ `Color`; `brush` ↔ `Brush`; `array<[T]>` ↔ `ModelRc`; `struct` → Rust struct of same name; `enum` → Rust enum (payload shapes `UNSURE` — verify type_mappings before use).
- `struct` literal `{ name: "Foo", score: 100 }` maps to the Rust struct; JS object / Python dict / C++ class on those backends.
- `color` auto-converts to `brush`; `Brush::color()` returns solid color or first stop.

## Globals from native code (general)

```rust
app.global::<Logic>().on_magic_operation(|v| v * 2);
app.global::<Logic>().set_v(42);
```

- `export global Logic { in-out property<int> v; pure callback op(int)->int; }` (+ re-export from main file) exposes to native: Rust `app.global::<Logic>().on_magic_operation(...)`, C++ `app->global<Logic>()`, JS `component.Logic.x`.
- Globals are one instance per component instance, NOT process singletons; no `init`, no states/`animate` on globals.

## SharedString (.slint `string`)

```rust
app.set_tab(boot_tab.into());
app.set_title(SharedString::from(id));
```

- `.slint string` <-> `slint::SharedString` (docs: type_mappings, ModelRc example; repo: `set_tab`/`set_view`/`set_playlist`).

## Models (`.slint` arrays)

```rust
use slint::{ModelRc, VecModel};
let rows: Vec<MatchRowUi> = vec![...];
s.set_not_ported(ModelRc::new(VecModel::from(rows)));
```

- `Model` requires `row_count/row_data/model_tracker` + `push/set/remove/insert/iter/as_any`; UI takes `ModelRc` via `set_the_model(rc)` (docs: Model trait, ModelRc::new/row_count; repo pattern: `Rc+ModelRc`).

## Mutating a model (keep `Rc<VecModel<T>>`)

```rust
let m = std::rc::Rc::new(slint::VecModel::from(rows));
app.set_list(ModelRc::from(m.clone()));
m.push(row); // UI updates, VecModel notifies
// or read back:
app.get_list().as_any().downcast_ref::<VecModel<T>>().unwrap().push(row);
```

- Docs: ModelRc constructing example, Model downcast example; repo pattern: `Rc+ModelRc`.

## Callbacks in Rust (capture `as_weak`, not strong)

```rust
let weak = app.as_weak();
app.on_toggle_hud(move || { let _ = weak.upgrade(); });
tracker.on_select_match(move |id: SharedString| { ... });
app.invoke_toggle_hud(); // fire into .slint from Rust
```

- `on_<cb>(move||...)`, `invoke_<cb>(args)` into `.slint` (docs: Generated components; repo: `on_navigate`/`on_select_tab`/`on_refresh_rank`).
- Pitfall: capturing strong `AppWindow` in `on_` closure leaks; use `as_weak().upgrade()`.

## Public functions from Rust

```rust
app.invoke_do_something(42);
```

- `.slint` `public function do-something(x: int) -> bool` → Rust `invoke_do_something(...)`; dashes become `_` (docs: cpp/generated-code, rust SampleComponent).
- `UNSURE`: exact `invoke_` naming for camelCase fns — verify generated docs.

## Event-loop thread

```rust
slint::invoke_from_event_loop(move || { app.window().set_size(slint::PhysicalSize::new(w, h)); }).unwrap();
slint::Timer::single_shot(std::time::Duration::from_millis(50), move || { ... });
slint::quit_event_loop().unwrap();
```

- Touching UI off UI thread fails — wrap in `invoke_from_event_loop` after loop starts (repo: `main.rs`+`hud.rs`).
- Set size after first loop turn or layout overwrites (docs: Generated component run/event-loop; repo: first-event resize).

## Images from Rust

```rust
let img = slint::Image::load_from_path(path)?;
let img = slint::Image::load_from_data(&bytes, Some("png"))?;
// None guesses; SVG buffer without <?xml/<svg needs explicit Some("svg")
```

- Needs `image-decoders` or wasm+std (docs: rust Image load_from_path/load_from_data; repo: `assets.rs` URL art + RgbaImage test).
- Pitfall: without `image-decoders`, load fails.

## `UNSURE` gaps (do not guess)

- `ComponentHandle::hide` semantics on frameless `no-frame` windows — verify.
- `Timer` repeat modes beyond `single_shot` — look up `slint::Timer` docs.
- `PhysicalSize` vs logical `set_size` units — verify against `window()` docs.
- Enum payload shapes in `.slint` ↔ Rust mapping — verify type_mappings.
- Callback arg/return limits beyond `string`/`int` — verify generated-code docs.
- Anything beyond std `slint` crate features (e.g. `image-default-formats`) — verify `Cargo.toml` feature docs.

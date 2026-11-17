# Slint Language — Complete Reference

> Rule: `.slint` renders, never formats a fact. Rust owns every value.
> Anything not listed here is `UNSURE` — look it up, don't guess.

## File structure

`.slint` FILE = sequence of top-level items only (`component`, `import`/`export`, `struct`/`enum`, `global`). Comments `//` and `/* */`.

```slint
import { Button } from "std-widgets.slint";
export component Name inherits Base { ... }
```

- Instantiate: `TypeName { }`. Named: `id := Type { }`. `id` unique per component; `self` / `parent` / `root` reserved. Built-ins need no import.
- Sources: guide/file, ref/file-structure.

## Component / inherits

```slint
export component Name inherits Window { ... }
component Helper inherits Rectangle { ... }
```

- Base = built-in, earlier component, or imported. Inheriting instance IS a Base: body bindings apply to it, children become its children, inherits all Base props.
- App root normally `export component X inherits Window` (or `Dialog`).

## Properties

```slint
in property <int> count;
out property <string> label;
in-out property <brush> surface: #140e1b;
private property <bool> internal;
in-out property <int> doubled: count * 2;   // binding, re-evaluates reactively
```

- `in` = set by user, `out` = read by user, `in-out` = both, `private` = internal (default).
- Non-private top-level props form the component interface (settable from outside + native API).
- Binding `prop: expr;` re-evaluates reactively.
- `changed p => { ... }` fires once per event-loop pass when value differs (max ~10 passes).
- Expose inherited/child prop via new name + `<=>` two-way bind.
- Sources: guide/properties, ref/properties.

## Callbacks

```slint
callback toggled;
callback navigate(string);
callback pick(int, string) -> int;
pure callback transform(int) -> int;
```

- Declare: `callback name;` / `callback f(int, string);` / `callback f(int)->int;` (return needs parens). `pure callback` for pure contexts.
- Invoke: `root.hello()`. No handler = no-op (return type → type default).
- Handler: `cb(args) => { ... }` (≤1 per callback, same component or child).
- Alias: `callback clicked <=> area.clicked;`
- Built-in `init` runs outermost→inward; globals cannot declare/handle `init`.
- Sources: guide/functions-and-callbacks, ref/callbacks.

## Functions

```slint
[public|protected] [pure] function name(a: int) -> int { return a * 2; }
```

- Body required, not top-level/nested. `private` (default) = own component only; `public` = any holder element + native code if exported; `protected` = direct inheritor via `self`/`root` only.
- `pure` = side-effect-free, callable from bindings (assigning prop or calling impure callback = error).
- Unqualified call resolves `self` → enclosing → `root`. Callbacks differ: settable from native, may lack body, may `<=>` alias.
- Source: ref/functions.

## Modules — import / export

```slint
import { Button } from "./button.slint";
import { VBox } from "std-widgets.slint";
import "./Noto.ttf";                              // font file
import { X } from "@lib/file.slint";             // needs library-paths in slint-project.json
export component Name;
export { A, B, }
export { N as M }
export { I } from "path"
export * from "path"                             // max once
export global Logic { ... }
export struct Player { ... }
```

- Everything private by default. Sources: guide/file, ref/imports, ref/exports.

## Structs

```slint
export struct Player { name: string, score: int, }
// field defaults must be compile-time constants:
export struct WithDef { name: string, score: int: 100, }
```

- Literal: `{ name: "Foo", score: 100 }`; omitted fields take defaults/zero. Access `p.name`. Anonymous: `property<{name: string, score: int}>`.
- Native: Rust struct, C++ class, JS object, Python dict.
- Source: guide/structs-and-enums, ref/structs-and-enums.

## Enums

```slint
export enum CardSuit { clubs, diamonds, hearts, spade }
```

- ≥1 distinct values, default = first. Use short `spade` when type known, or `CardSuit.clubs`. Names map `-`=`_`; no value named same as enum.
- Source: guide/structs-and-enums.

## Globals / singletons

```slint
export global Palette { in-out property<color> primary: blue; }
export global Logic {
    in-out property<int> v;
    pure callback op(int) -> int;
}
```

- Accessed as `Palette.primary`. One instance per component instance (NOT process singleton).
- Expose to native: `export global Logic {...}` (+ re-export from main file): Rust `app.global::<Logic>().on_magic_operation(...)`, C++ `app->global<Logic>()`, JS `component.Logic.x`.
- No `init`, no states/`animate` on globals.
- Source: guide/globals.

## Types

| .slint | Notes (default) |
|---|---|
| `bool` | `true`/`false` = `false` |
| `int` | i32 = `0` |
| `float` | f32; `30%` → `0.30` = `0` |
| `string` | `".."` escapes `\"` `\\` `\n` `\u{}` `\{expr}`; styled `@markdown(..)` = `""` |
| `color` | names / `#RGB` / `#RRGGBBAA` / `Colors.red` / `rgb()` / `hsv()` / `oklch()` = transparent |
| `brush` | `color` \| gradient = transparent |
| `image` | `@image-url` = empty |
| `length` | `1px` / `pt` / `in` / `mm` / `cm` = `0px` |
| `physical-length` | `0phx` |
| `percent` | `50%` = `0%` |
| `duration` | `200ms` / `1s` = `0ms` |
| `angle` | `90deg` / `rad` / `turn` = `0deg` |
| `relative-font-size` | `rem` = `0rem` |
| `array [T]` | literals `[1,2,3]` = empty model |

Type helpers:

- No mixed-unit `+`/`-` (`10px+2ms` errors); `width/1px` → `float`; `42*1px` int→length.
- `float`: `.to-fixed` / `.to-precision` / `.to-string-unlocalized`.
- `string`: `.is-empty` / `.character-count` / `.to-lowercase` / `.to-uppercase()` / `.starts-with()` / `.ends-with()` / `.to-float()` / `.is-float()` / `.replace-all()`; template `"5+5=\{5+5}"`.
- `color`: `.red` / `.green` / `.blue` / `.alpha` `.brighter()` / `.darker()` / `.mix()` / `.transparentize()` / `.with-alpha()` / `.to-hsv()` / `.to-oklch()`.
- `array`: `.length`, `[i]`, `[i]=v`, `.push()` / `.remove()` / `.insert()`; OOB read → default, bad write → no-op.
- `color` auto-converts to `brush`.
- Builtin structs: `Point` `Size` `Edges` `FontMetrics` `KeyEvent` `KeyboardModifiers` `PointerEvent` `PointerScrollEvent` `StandardListViewItem` `TableColumn` `DropEvent` `InputMethodHints`.

## Expressions

Reactive. Precedence loosest→tightest: `?:` > `||` `&&` (non-assoc) > `==` `!=` `<` `>` `<=` `>=` (non-assoc) > `+` `-` > `*` `/` > unary `+` `-` `!` > `.` `[]`.

- No `%` modulo (percent unit) → `Math.mod`; no bitwise / `++` / `--` / assignment.
- `+` concatenates if either side is `string` (int/float auto-stringify).
- `==`/`!=` any common type; ordering only numbers/units/strings. `&&` `||` `!` bool, short-circuit. Ternary right-assoc, branches same type. Member `foo.x`, index `arr[i]`.

## Statements (callbacks / functions)

`let delta = 42;` immutable local; `if`/`else`; assignment `+=`; `return`; any expression + `;`. Block value = last statement unless `return`.

## if / for

```slint
if cond : Element { }
for name[index] in model : Element { }
```

- `if` adds/removes (unlike `visible` which keeps space); excluded from parent preferred-size unless edition ≥1.19.
- `for`: `name`/`index`/`id` all optional; `name` read-only; model in surrounding scope; model = `int` (repeat N) | array/model | literal `[..]`. Single element only (may nest `for`/`if`). In layouts repeaters act as children.

## States

```slint
states [
    name when cond : { prop: val; in/out/in-out { animate ... } }
]
```

- Any element except globals; multiple blocks merge; first active wins, none active → own bindings. Body: unqualified = owner, qualified = descendant (must resolve to property).
- Two-way-bound prop cannot be changed in state. Edition <1.19 no-state `x`/`y`/`w`/`h` were `0px`; now fill/fit defaults.

## Transitions

- `in` (entering) / `out` (leaving) / `in-out` (both, `in_out` accepted) blocks containing ONLY `animate` clauses; qualified names allowed (`animate root.background`).
- `animate *` catch-all only in transitions; skips bool/string + props with own animate; at most one per direction. Named props must be changed by that state.
- Legacy standalone `transitions[]` removed.

## Animate

```slint
animate background { duration: 250ms; }
animate x, y { duration: 250ms; easing: ease-in-out; }
```

- Every change animates. Own-element props only (qualified = error outside transitions). One block per property. Multi-prop `animate x, y {...}`.
- Animatable: `int` `float` `length` `physical-length` `color` `brush` `angle` only.
- Fields: `delay` (`0ms`) `duration` (`0ms`) `easing` (`linear`) `iteration-count` (`1`, negative = forever) `direction` (`normal|reverse|alternate|alternate-reverse`) `enabled`; angle-only `angle-interpolation: linear|shorter|longer|increasing|decreasing`.
- Easing spring: `spring` / `spring(bounce -1..1)`, 0 = critically damped. Reduced-motion OS setting forces `enabled: false`.

## Layouts (summary)

Absolute `x`/`y` (parent-relative) `width`/`height`; `px` logical (DPR-scaled, recommended), `phx` physical, `%` of parent. Defaults: `x`/`y` center; `Image`/`Text`/widgets fit content; `Rectangle`/`TouchArea`/`FocusScope`/`Flickable`/layouts fill parent. `preferred-width/height` from unset-`x`/`y` children. Core: `VerticalLayout`/`HorizontalLayout` (column/row), `GridLayout` (`Row{}` or `row`/`col`/`rowspan`/`colspan`, runtime-changeable, `for`/`if` ok). std-widgets `VerticalBox`/`HorizontalBox`/`GridBox` add padding+spacing. Props: `spacing`, `padding(+sides)`, `min/max-width/height`, `horizontal-stretch`/`vertical-stretch` (0 = never; parent-fillers default 1), `alignment` `stretch|start|end|center|space-between|space-around` + cross-axis alignment. Custom containers place use-site children at `@children`. Full rules: `layout.md`.

## Elements — Text

```slint
Text {
    text: "hi"; color: white; font-size: 13px; font-family: "Noto Sans";
    font-weight: 700; font-italic: false;
    horizontal-alignment: start|end|left|center|right;
    vertical-alignment: top|center|bottom;
    wrap: no-wrap|word-wrap|char-wrap; overflow: clip|elide;
    max-lines: 2; letter-spacing: 1px; line-height-factor: 1.2; // factors need UNSURE check
    // Text stroke (verified 2026-11-17): EXACT 3 props — `stroke: brush` (transparent), `stroke-width: length = 0px` (0 = 1-physical-px hairline), `stroke-style: TextStrokeStyle = outside`. `TextStrokeStyle` = EXACTLY 2 values: `outside` (inside edge at glyph edge, default) | `center` (centerline at glyph edge). Example: `Text { text: "Stroke"; stroke-width: 2px; stroke: darkblue; stroke-style: center; font-size: 80px; color: lightblue; }`
}
```

- `font-weight` numeric (default `0` = unset → falls back to enclosing `Window.default-font-size` chain; see Fonts). `FontWeight` namespace (verified 2026-11-17, built-in namespace of unitless `float` per name-resolution rule 6, access `FontWeight.<name>`): EXACT 9 members, hyphenated lowercase, CSS weights — `thin` 100, `extra-light` 200, `light` 300, `normal` 400, `medium` 500, `semi-bold` 600, `bold` 700, `extra-bold` 800, `black` 900 (= heaviest; a weight name, not a color). NO `extrabold`/`semibold`/`demi-bold`/`heavy`. Consuming prop `Text.font-weight: int, default 0`, range 100–900, 400 = normal. Example: `font-weight: FontWeight.extra-bold;`. Sources: `reference/global-namespaces/font-weight/`, `reference/elements/text/`, `reference/property-types/builtin-enums/`. `font-metrics` is `out`. Source: ref/elements/text.

## Elements — Image / Rectangle

```slint
Image {
    source: @image-url("art.png");
    image-fit: fill|contain|cover|preserve; image-rendering: smooth|pixelated;
    colorize: blue; // tints with brush
    source-clip-x/y/width/height: 0px; // sprite atlases
}
Rectangle {
    background: blue; border-width: 1px; border-color: gray;
    border-radius: 8px; // + per-corner variants
    clip: true;
    // Drop shadow (verified 2026-11-17, injected on every item, docs under Rectangle → "Drop Shadows"):
    // EXACT 5 props — `drop-shadow-offset-x: length = 0px`, `drop-shadow-offset-y: length = 0px`,
    // `drop-shadow-blur: length = 0px`, `drop-shadow-spread: length = 0px` (Skia/FemtoVG/Vello/software only — Qt backend ignores spread),
    // `drop-shadow-color: color` (transparent). Fill/border alpha shape the shadow; children do not contribute.
    // Backdrop blur (verified 2026-11-17): EXACT `backdrop-blur: length = 0px`; zero/negative disables; Skia renderer ONLY (software/FemtoVG/Vello/Qt draw without blur, no error). Translucent `background` + blur follow border-radius; expensive (re-filters every repaint).
    backdrop-blur: 4px; // Skia only — degrades gracefully elsewhere
}
```

- `Image`: animated GIF/WebP supported; `.width`/`.height` int px (`0` if empty). No `BorderRectangle`/`Clip` elements — use `Rectangle` border props / `clip: true`.
- Sources: ref/elements/image, ref/elements/rectangle.

## Elements — TouchArea / FocusScope

```slint
ta := TouchArea {
    enabled: true;
    clicked => { ... } double-clicked => { ... }
    moved => { ... }
    pointer-event(e) => { accept }        // -> EventResult
    scroll-event(e) => { accept }         // -> EventResult
}
// reads: ta.pressed(out) ta.has-hover(out) ta.mouse-x/y(out) ta.pressed-x/y(out)
// mouse-cursor: ...
fs := FocusScope {
    enabled: true; focus-on-click: true;
    focus-on-tab-navigation: true; forward-focus: other;
    key-pressed(e) => { accept } key-released(e) => { accept }
    capture-key-pressed(e) => { accept } capture-key-released(e) => { accept }
    focus-gained => { ... } focus-lost => { ... } focus-changed-event => { ... }
    // functions: fs.focus(); fs.clear-focus();
}
KeyBinding { keys: @keys(...); activated() => { ... } }  // child of FocusScope — @keys values verified, see notes below
```

- `EventResult`: EXACTLY 2 values — `accept` (accepted, won't be processed further) / `reject` (rejected, may be handled by parent). No `ignore` variant exists (TouchArea `scroll-event` prose says accept-or-ignore but enum is still `accept`/`reject`). Sources: ref/gestures/toucharea, ref/keyboard-input/focusscope. (verified 2026-11-17)
- `@keys()` / `KeyBinding` / `KeyEvent` (verified 2026-11-17): `KeyBinding.keys` is `keys` type, constructed via `@keys(...)`: `@keys(Control + X)`, `@keys(Control + Q)`, `@keys(Control + S)`, `@keys(Control + Z)`. Grammar: a key from `Key` namespace + zero or more of modifiers `Meta`, `Control`, `Shift`, `Alt` joined by `+`. Optional-modifier forms `Shift?`, `Alt?` confirmed, e.g. `@keys(Control + Shift? + Alt? + "€")`, equivalence `@keys(Control + Plus)` == `@keys(Control + Shift? + "+")`. String-literal fallback `@keys(Control + "ä")` matched against lowercased `KeyEvent.text`. Logical-key semantics: layout-dependent, numpad not distinguished from row digits, AltGr not available as modifier.
- `KeyEvent` struct fields: `text: string` (unicode representation), `modifiers: KeyboardModifiers`, `repeat: bool` (true only for press-repeat, always false for release). `KeyboardModifiers` fields: `alt: bool`, `control: bool` (Command on macOS), `shift: bool`, `meta: bool` (Control on macOS, Windows key on Windows).
- `Key` namespace exact member list (as `Key.<Name>`): Backspace, Tab, Return, Escape, Backtab, Delete, Shift, Control, Alt, AltGr, CapsLock, ShiftR, ControlR, Meta, MetaR, Space, UpArrow, DownArrow, LeftArrow, RightArrow, F1-F24, Insert, Home, End, PageUp, PageDown, ScrollLock, Pause, SysReq, Stop, Menu, Back, A-Z (represent lowercase a-z), Digit0-Digit9, Circumflex, Exclamation, DoubleQuote, Hash, Dollar, Percent, Ampersand, Underscore, OpenParen, CloseParen, Asterisk, Plus, Pipe, HyphenMinus, OpenCurlyBracket, CloseCurlyBracket, Tilde, Colon, Semicolon, LessThan, Equals, GreaterThan, QuestionMark, At, Comma, Period, Slash, BackQuote, OpenBracket, BackSlash, CloseBracket.
- `keys` member function: `to-string() -> string` (platform-native description, e.g. ⌘Z on macOS, Ctrl+Z elsewhere).

## Elements — Flickable / ScrollView

```slint
Flickable {
    content-width: 800px; content-height: 1200px;
    content-x: 0px; content-y: 0px; // in-out
    interactive: true; mouse-drag-pan-enabled: true;
    flicked => { ... }
    // scroll-to(pos: Point, behavior: ScrollBehavior) — verified 2026-11-17, see notes below
}
ScrollView { // std-widgets: Flickable + scrollbars; instantiates ALL children
    content-width: 800px; content-height: 1200px;
    vertical-scrollbar-policy: as-needed|always-off|always-on;
    horizontal-scrollbar-policy: as-needed|always-off|always-on;
    mouse-drag-pan-enabled: false; // default false
    scrolled => { ... }
}
```

- Sources: ref/gestures/flickable, ref/std-widgets/views/scrollview.
- `scroll-to(pos: Point, behavior: ScrollBehavior)` — owner: `Flickable` (verified 2026-11-17). Semantics: scrolls so content point `pos` (content coordinates, positive where content-x/y are negative) lands at Flickable top-left; out-of-range clamped. `ScrollBehavior` = EXACTLY 2 values: `instant` (jump), `smooth` (animate). Example: `scroll-to({ x: 0px, y: 100px }, ScrollBehavior.instant)` sets `content-y` to `-100px`. Related, NOT the same: `ensure-visible(behavior)` on any element scrolls enclosing `Flickable`(s) by smallest amount to bring it into view; no-op without a `Flickable` ancestor.

## Elements — Window / Dialog / PopupWindow

```slint
export component App inherits Window {
    title: "Hi"; background: white; icon: @image-url("icon.png");
    no-frame: true; resize-border-width: 6px;
    always-on-top: false; full-screen: false;
    maximized: false; minimized: false; // in-out
    default-font-family: "Noto Sans"; default-font-size: 14px; // + default-font-weight
    // MenuBar > Menu > MenuItem hierarchy — verified 2026-11-17, see widgets.md
}
// functions: close(); hide();
// Dialog: same props as Window, one main child + StandardButton row; per-kind `<kind>_clicked` aliases CONFIRMED (hyphen, `_` normalizes same) — see widgets.md StandardButton section
PopupWindow {
    close-policy: close-on-click|close-on-click-outside|no-auto-close; // EXACT 3, default close-on-click (verified 2026-11-17)
    // functions show(); close();
}
```

- Sources: ref/window/window, dialog, popupwindow.

## Fonts

`Text`/`TextInput` use `font-*`, else `Window.default-font-*`. `font-size: 0px` = unset → falls back to enclosing `Window.default-font-size` (explicit on `StyledText.default-font-size`; general rule in Fonts guide covers `font-family`/`font-size`/`font-weight`). Final pixel when `Window.default-font-size` is itself `0`: STILL-UNSURE (no page states it; needs renderer/style source or runtime probe). System fonts auto-picked. Custom: `import "./Noto.ttf";` (ttf/ttc/otf) then `font-family`/`default-font-family: "Noto Sans";`. Embedded targets must embed + `EMBED_RESOURCES`/`SLINT_FONT_SIZES`/SDF options. `FontWeight` namespace: EXACT 9 hyphenated members `thin 100`, `extra-light 200`, `light 300`, `normal 400`, `medium 500`, `semi-bold 600`, `bold 700`, `extra-bold 800`, `black 900` (access `FontWeight.extra-bold`). `rem` = `Window.default-font-size` × factor. Sources: guide/development/fonts, `reference/elements/styledtext/`, `reference/window/window/`, `reference/global-namespaces/font-weight/` (all verified 2026-11-17).

## Math API (`Math.*` + global + postfix)

`abs` `ceil` `clamp` `cos` `sin` `tan` `acos` `asin` `atan` `atan2` `exp` `floor` `log` `ln` `max` `min` `mod` (euclidean) `pow` `round` `sign` `sqrt`. T = any numeric. Trig args need angle units (`cos(90deg)`).

```slint
Math.abs(-10)  abs(-10)  (-10).abs()
```

- Source: ref/global-functions/math.

## Gradients (@ sigil)

```slint
@linear-gradient(90deg, red 0%, blue 100%)                       // along rotated line
@radial-gradient(circle, red 0%, blue 100%)                       // default center, farthest-corner
@radial-gradient(ellipse 60px 30px at 10px 20px, red 0%, blue 100%)
@conic-gradient(from 45deg at 50% 50%, red 0deg, blue 180deg)    // sweep 0–360deg, from rotates (up=0deg)
```

- No negative angles directly (use `270deg` or variable/subtraction). Source: ref/property-types/colors-and-brushes, ref/language/lexical-structure.

## @image-url

````
@image-url("path") / @image-url("path", nine-slice(t r b l))  // 1=all, 2=v h, 4=trbl
````

- Compile-time string literal (escapes decoded); relative-to-file then include-paths; http only in SlintPad; `data:` URIs ok. Formats: SVG + image-crate (AVIF BMP DDS Farbfeld GIF HDR ICO JPEG EXR PNG PNM QOI TGA TIFF WebP); Rust default PNG/JPEG/SVG (+`image-default-formats` feature); web = browser decoders except `.svgz`. Fields `.width`/`.height` int px (`0` if empty).
- Source: ref/property-types/images, ref/language/expressions.

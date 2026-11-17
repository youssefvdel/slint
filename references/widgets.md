# Std-Widgets (verified scope only)

> Only widgets below are verified by the inputs. Anything else is `UNSURE` — look it up, don't guess.
> Import path: `import { X } from "std-widgets.slint";`

## Layout helpers (verified)

```slint
import { VerticalBox, HorizontalBox, GridBox } from "std-widgets.slint";
VerticalBox { spacing: 8px; Text { text: "padded column"; } }
HorizontalBox { Text { text: "a"; } Text { text: "b"; } }
GridBox { Row { Text { text: "a"; } Text { text: "b"; } } }
```

- `VerticalBox`/`HorizontalBox`/`GridBox` = `VerticalLayout`/`HorizontalLayout`/`GridLayout` + default padding + spacing. Prefer for content cards; bare layouts for pixel-exact rows.

## Views (verified)

```slint
import { ScrollView } from "std-widgets.slint";
ScrollView {
    content-width: 800px; content-height: 1200px;
    vertical-scrollbar-policy: as-needed;   // as-needed|always-off|always-on
    horizontal-scrollbar-policy: as-needed; // as-needed|always-off|always-on
    scrolled => { ... }
}
```

- `ScrollView` exposes NO color/width/thickness/thumb styling knobs. The ONLY scrollbar knobs are `vertical-scrollbar-policy` / `horizontal-scrollbar-policy` (`ScrollBarPolicy`: `as-needed` (default) | `always-off` | `always-on`). Visual bar styling lives in the widget style, not per-instance. Full 11-entry prop list: `enabled: bool = true`, `mouse-drag-pan-enabled: bool (in) = false`, `has-focus: bool (in-out) = false`, `content-width/height/x/y: length (in-out) = 0px` (was `viewport-width/height/x/y`, kept as deprecated aliases), `visible-width/height: length (out)` (excludes scrollbar), `vertical-scrollbar-policy` / `horizontal-scrollbar-policy`. Callback: `scrolled()`. To style a bar: drop to `Flickable` + draw own bar from `content-x/y ÷ content-width/height`, or restyle at style level.
- `ScrollBehavior` = EXACTLY 2 values, lowercase: `instant`, `smooth`. Used only in `Flickable.scroll-to(pos: Point, behavior)` and `ensure-visible(behavior)`. No `animated`/`auto`/`none`.
- `VerticalBox`/`HorizontalBox`/`GridBox` declare ZERO own properties — each page body is one sentence delegating to the layout + `StyleMetrics`. Settable knobs = inherited layout knobs + theming globals. VBox/HBox inherit `VerticalLayout`/`HorizontalLayout`: `spacing = 0px`, `padding = 0px`, `padding-left/right/top/bottom = padding`, `alignment: LayoutAlignment = stretch` (`stretch|center|start|end|space-between|space-around|space-evenly`), `cross-axis-alignment: CrossAxisAlignment` (default = first value, behaves `stretch`; `auto` legal ONLY on `cross-axis-self-alignment`), per-child `cross-axis-self-alignment = auto`, `layout-order = 0`. `GridBox` inherits `GridLayout`: `spacing = 0px`, `spacing-horizontal = spacing`, `spacing-vertical = spacing`, `padding = 0px` + sides `= padding`, per-cell `row/col = auto` (setting `row` resets `col` to 0 unless both set), `rowspan/colspan = 1`. Theming globals: `StyleMetrics` (ONLY 2 props, both `out length`): `layout-spacing`, `layout-padding`; `Palette` (12 props, `brush (out)` except last): `background`, `foreground`, `alternate-background`, `alternate-foreground`, `control-background`, `control-foreground`, `accent-background`, `accent-foreground`, `selection-background`, `selection-foreground`, `border`, `color-scheme: ColorScheme (in-out)` (`unknown|dark|light`, only `in-out` switch; all styles except Qt support forcing dark/light). No `background`/`border-*`/`corner-radius`/`font-*`/`color` on any Box — wrap in owned `Rectangle`/`Border` and read `Palette.*`/`StyleMetrics.*`.
- Sources: `std-widgets/views/scrollview/`, `gestures/flickable/`, `std-widgets/layouts/verticalbox|horizontalbox|gridbox/`, `layouts/verticallayout|gridlayout|overview/`, `std-widgets/globals/stylemetrics|palette/`, `property-types/builtin-enums/`, `reference/common/` (all verified 2026-11-17).

## StandardButton (verified 2026-11-17, docs.slint.dev/latest)

```slint
import { StandardButton } from "std-widgets.slint";
StandardButton { kind: ok; dialog-button-role: none; }
```

- `kind: StandardButtonKind = ok` — EXACT 11 members, no `extra`: `ok`, `cancel`, `apply`, `close`, `reset`, `help`, `yes`, `no`, `abort`, `retry`, `ignore`.
- `dialog-button-role` (common property on any element in a `Dialog`): `DialogButtonRole = none` — EXACT 7 members, no `extra` / no `ok` / `cancel` / `yes` / `no`: `none`, `accept`, `reject`, `apply`, `reset`, `help`, `action`.
- `action` = role of any other button that performs another action — the `extra`-like catch-all, not named `extra`.
- Inside `Dialog` auto-generates `<kind>_clicked` aliases: CONFIRMED 2026-11-17 (compiler `lower_dialog_layout` + live Dialog docs). Each `StandardButton` that is a direct `Dialog` child with constant `kind` and no explicit `clicked` handler gains `ok-clicked`, `cancel-clicked`, `apply-clicked`, `close-clicked`, `reset-clicked`, `help-clicked`, `yes-clicked`, `no-clicked`, `abort-clicked`, `retry-clicked`, `ignore-clicked` (hyphen; `_` normalizes to same). Alias is `<=>`-style to the button's `clicked` (`InOut`, public API). Conditions: direct child, constant `kind`, no explicit handler, `Dialog` is the component root. Duplicate `kind` = compile error; `kind` unset = error; exactly one non-button main child required. `kind` → `DialogButtonRole`: `ok/yes/retry/ignore` → `accept`, `cancel/no/close/abort` → `reject`, `apply` → `apply`, `reset` → `reset`, `help` → `help`.
- `StandardButton` members: `in kind`, `callback clicked <=> base.clicked`, `in enabled`, `in primary`, `out has-focus`, `out pressed`.
- Sources: docs.slint.dev/latest `std-widgets/basic-widgets/standardbutton/`, `property-types/builtin-enums/`, `reference/common/`, `reference/window/dialog/`; compiler `builtin_elements.rs`, `passes/lower_layout.rs` (`lower_dialog_layout`), `common/enums.rs`.

## MenuBar > Menu > MenuItem (verified 2026-11-17, docs.slint.dev/latest)

```slint
export component App inherits Window {
    MenuBar {
        Menu {
            title: "File";
            MenuItem { title: "Quit"; activated => { ... } }
            MenuSeparator {}
            Menu { title: "More"; MenuItem { title: "Sub"; } }
        }
    }
}
```

- `Window > MenuBar > Menu > { MenuItem | Menu (sub-menu) | MenuSeparator }`.
- `MenuBar`: exactly 0..1 per `Window`, MUST be direct child of `Window`, MUST NOT be in `for`/`if`. Property: `visible: bool = true`. No callbacks. Shortcuts still work when invisible. Native on some platforms (macOS top-of-screen); Window width/height = client area excluding menubar.
- `Menu`: allowed parent = `MenuBar` | `ContextMenuArea` (max 1 `Menu` child, rest = visual children) | `Menu` (sub-menu). Children allowed = `MenuItem` | `Menu` | `MenuSeparator`. Properties: `title: string = ""`, `enabled: bool = true`, `icon: image = empty`. No callbacks/functions.
- `MenuItem`: MUST be child of `Menu`. Properties: `title: string = ""`, `enabled: bool = true`, `checkable: bool = false`, `checked: bool (in-out) = false`, `icon: image = empty`, `shortcut: keys = @keys()` [MenuBar-only], `role: MenuItemRole = none` [MenuBar-only]. Callback: `activated()`.
- `MenuItemRole`: `none | about | preferences | quit | services | hide | hide-others | show-all`. On macOS role items move to app menu; other platforms: about/preferences/quit stay in place, macOS-only roles hidden. `services/hide/hide-others/show-all`: `activated` not invoked, shortcut ignored. `preferences` default Cmd+`,`, `quit` default Cmd+Q on macOS.
- `MenuSeparator`: no children, no props, no callbacks. Leading/trailing invisible; consecutive merged.
- `ContextMenuArea` helpers (for completeness): `enabled: bool = true`, `show(Point)`, `close()`.
- Sources: `reference/window/window/`, `reference/window/contextmenuarea/`.

## Callbacks — complete inventory (verified 2026-11-17, docs.slint.dev/latest)

| Element | Complete callback list | Notes |
|---|---|---|
| `Menu` | **NONE** (props only: `title`, `enabled`, `icon`) | No `clicked`/`triggered`/`activated` |
| `MenuItem` | **`activated()` ONLY** | Slint equivalent of Qt `triggered`; NO `clicked`. macOS-only roles (`services`, `hide`, `hide-others`, `show-all`): `activated` not invoked, `shortcut` ignored |
| `MenuSeparator` | **NONE** (explicitly: no props or callbacks) | — |
| `MenuBar` | **NONE** (only `visible` prop) | — |
| `ContextMenuArea` | **NONE** (only `show(Point)`/`close()` functions + `enabled` prop) | — |
| `StandardButton` | **`clicked()` ONLY** | — |
| `Button` (plain) | **`clicked()` ONLY** | Joins `Dialog` button row via common `dialog-button-role` (`none|accept|reject|apply|reset|help|action`), not via callback |
| `Dialog` (itself) | **ZERO declared** + auto `<kind>_clicked` aliases (see StandardButton section) | NO `triggered`/`accepted`/`rejected` on `Dialog` |
| Ruled OUT | `triggered` exists on NONE of these pages; `accepted`/`canceled` exist ONLY on `DatePickerPopup` (`accepted(date)`/`canceled()`) and `TimePickerPopup` (`accepted(time)`/`canceled()`), NOT on `Dialog`/`Menu`/`MenuItem`/`StandardButton` | Confident negative from fetched TOC/sections |
| Generic | Every element implicitly declares `init` (instantiation, inside-to-out; cannot be set from app code via `on_init`); `accessible-action-*` (`-default`, `-set-value`, `-increment`, `-decrement`, `-expand`, `-set-selection-offsets`) exist only when `accessible-role` is set | Not widget-specific |

- Sources: `reference/window/contextmenuarea/`, `reference/window/dialog/`, `std-widgets/basic-widgets/standardbutton|button/`, `reference/window/window/`, `reference/common/` (all fetched 2026-11-17).

## Full std-widgets catalog (verified 2026-11-17, kebab-case as in docs)

1. **Button** — props: `checkable: bool = false`, `checked: bool (in-out) = false` (needs checkable), `enabled: bool = true`, `has-focus: bool (out)`, `icon: image = empty`, `icon-size: length = 0px`, `pressed: bool (out)`, `text: string = ""`, `primary: bool = false`, `colorize-icon: bool = false`. Callback: `clicked()`.
2. **CheckBox** — props: `checked: bool (in-out) = false`, `enabled: bool = true`, `font-size: length = 0px`, `font-weight: int = 0`, `has-focus: bool (out)`, `text: string = ""`. Callback: `toggled()`.
3. **ComboBox** — props: `current-index: int (in-out) = 0`, `current-value: string (in-out) = ""`, `enabled: bool = true`, `has-focus: bool (out)`, `model: [string] = []`. Callback: `selected(string)`. Writing `current-value` looks up model, moves `current-index` or clears to `-1`/`""` if not found.
4. **LineEdit** — props: `enabled: bool = true`, `font-size: length`, `font-family: string`, `font-italic: bool = false`, `has-focus: bool (out)`, `has-selection: bool (out)`, `horizontal-alignment: TextHorizontalAlignment = left`, `input-method-hints: InputMethodHints`, `input-type: InputType = text` (text|password|number|decimal|search), `placeholder-text: string = ""`, `read-only: bool = false`, `text: string (in-out) = ""`. Functions: `focus()`, `clear-focus()`, `set-selection-offsets(anchor: int, focus: int)`, `select-all()`, `clear-selection()`, `copy()`, `cut()`, `paste()`, `undo()`, `redo()`. Callbacks: `accepted(string)` [Enter], `edited(string)`, `key-pressed(KeyEvent) -> EventResult`, `key-released(KeyEvent) -> EventResult`.
5. **TextEdit** — props: `font-size: length = 0px`, `font-family: string = ""`, `font-italic: bool = false`, `text: string (in-out) = ""`, `has-focus: bool (out)`, `has-selection: bool (out)`, `enabled: bool = true`, `input-method-hints`, `read-only: bool = false`, `wrap: TextWrap = word-wrap`, `horizontal-alignment`, `placeholder-text: string (in) = ""`, `content-width/length (in-out) = 0px`, `content-height (in-out)`, `content-x (in-out)`, `content-y (in-out)`, `visible-width (out)`, `visible-height (out)`. Same 10 functions as LineEdit. Callbacks: `edited(string)`, `key-pressed -> EventResult`, `key-released -> EventResult`. NO `accepted()` (only LineEdit has it).
6. **Slider** — props: `enabled: bool = true`, `has-focus: bool (out)`, `value: float (in-out) = 0`, `step: float = 1`, `minimum: float = 0`, `maximum: float = 100`, `orientation: Orientation = horizontal`. Callbacks: `changed(float)`, `released(float)`.
7. **SpinBox** — props: `enabled: bool = true`, `has-focus: bool (out)`, `value: int (in-out) = 0`, `minimum: int = 0`, `maximum: int = 100`, `read-only: bool = false`, `step-size: int = 1` (NOT `step`), `horizontal-alignment = left`. Callback: `edited(int)`.
8. **ProgressIndicator** — props: `indeterminate: bool = false`, `progress: float = 0` (0..1, capped). No callbacks/functions.
9. **Switch** — props: `checked: bool (in-out) = false`, `enabled: bool = true`, `has-focus: bool (out)`, `text: string = ""`. Callback: `toggled()`.
10. **TabWidget** — props: `current-index: int = 0`, `orientation: Orientation = horizontal` (must be constant). Children: ONLY `Tab` elements, each with `title: string = ""`. No callbacks/functions documented.
11. **ListView** (plain) — NO `model` prop; use `for` + auto list layout, virtualized. Inherits ScrollView props: `enabled: bool = true`, `mouse-drag-pan-enabled: bool (in) = false`, `has-focus: bool (in-out) = false`, `content-width/height/x/y: length (in-out) = 0px`, `visible-width/height: length (out)`, `vertical-scrollbar-policy: ScrollBarPolicy = as-needed`, `horizontal-scrollbar-policy = as-needed`. Callback: `scrolled()`. Function: `ensure-row-visible(row: int, behavior: ScrollBehavior)`.
12. **StandardTableView** — there is NO low-level `TableView` element (CONFIRMED ABSENT 2026-11-17: zero `export component TableView` hits across all 5 styles × 2 versions + no `TableView` builtin in `builtin_elements.rs`; `fluent/tableview.slint` `TableViewColumn/Row/Cell` are private non-exported helpers). Do NOT use `StandardTableView` for custom-drawn cells (rank medals, spines, agent strips) — it renders each cell as plain `Text { text: cell.text }`. Hand-roll `ListView` + `for` instead. Full member list: `rows: [[StandardListViewItem]]` (in / in-out, style-dependent; qt: in), `columns: [TableColumn] (in-out)`, `current-row: int (in-out) = -1` (`-1` = none), `current-sort-column: int (out) = -1` (`-1` = unsorted), `enabled: bool (in)` (aliased to scroll-view), `visible-width/height: length (out)`, `content-width/height/x/y: length (in-out) @shadowable`, `viewport-width/height/x/y` (in-out, **deprecated**, use `content-*`), `vertical-scrollbar-policy / horizontal-scrollbar-policy: ScrollBarPolicy (in)`, `has-focus: bool (in-out)`, `mouse-drag-pan-enabled: bool (in) @shadowable`. Callbacks: `sort-ascending(int)`, `sort-descending(int)` (user clicked header — model sorts), `row-pointer-event(int, PointerEvent, Point)`, `current-row-changed(int)` (user selection change), `scrolled() (@shadowable)`. Function: `set-current-row(int)` — sets `current-row`, emits `current-row-changed`, scrolls into view; **ignores out-of-range** (`index < 0 || index >= rows.length` → return). `TableColumn` EXACT fields: `title: string, min-width: length, horizontal-stretch: float, sort-order: SortOrder, width: length` (`width` = live/actual width, mutated by built-in column-resize drag). `StandardListViewItem` EXACT fields: single `text: string`. Sources: fluent `tableview.slint` + interface + docs page; `builtin_structs.rs`; live structs page; compiler `builtin_elements.rs` (absence).
13. **GroupBox** — container (any children, typically VerticalLayout). Props: `content-padding: length` (style-dependent), `enabled: bool = true`, `title: string = ""`. No callbacks/functions.
- Sources: `std-widgets/basic-widgets/button/`, `checkbox/`, `combobox/`, `slider/`, `spinbox/`, `progressindicator/`, `switch/`, `std-widgets/views/lineedit/`, `textedit/`, `tabwidget/`, `listview/`, `scrollview/`, `standardlistview/`, `standardtableview/`, `std-widgets/layouts/groupbox/`.

## STILL-UNSURE — stop and verify (do not guess)

- `font-size: 0px` final pixel when `Window.default-font-size` is itself `0` — no page states a concrete pixel fallback (style/system size presumably, not written). Needs renderer/style source or runtime probe (open since 2026-11-17).
- No other menu/widget callbacks beyond those listed were found; absence is per fetched docs TOC/sections, not exhaustively code-verified (checked 2026-11-17).

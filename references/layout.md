# Layout + Rendering

> Read this before changing any width. Shared widths single-sourced; fixed
> `width` without a row-sum comment is a bug.

## Containers

- `HorizontalLayout` = row, `VerticalLayout` = column, `GridLayout` = grid.
- All nest, fill parent by default, support `for` (model expansion) and `if` (add/remove).
- `visible: false` still takes layout space — use `if` to remove.
- std-widgets Box variants: `VerticalBox` / `HorizontalBox` / `GridBox` = same layouts + default padding + spacing. Prefer them for content cards; use bare layouts for pixel-exact rows.
- `Flickable` is a viewport, not a layout: takes content size only from its layout child.

```slint
VerticalBox {
    spacing: 8px; padding: 12px;
    Text { text: "title"; }
    HorizontalLayout {
        spacing: 4px;
        Text { text: "a"; horizontal-stretch: 1; }
        Text { text: "b"; horizontal-stretch: 2; }
    }
}
GridLayout {
    spacing: 4px;
    Row { Text { text: "a"; } Text { text: "b"; } }
    Text { text: "c"; row: 1; col: 0; colspan: 2; }
}
```

## Verified defaults (2026-11-17, docs.slint.dev/latest)

- `GroupBox.content-padding: length` — NO numeric default; docs say "Depends on the style". Do NOT confuse with raw layouts: `HorizontalLayout`/`VerticalLayout` `padding = 0px`, `spacing = 0px`, side paddings each default to `padding`.
- `VerticalBox`/`HorizontalBox`/`GridBox` declare ZERO own properties — each is the raw layout with `padding`/`spacing` defaults pulled from `StyleMetrics` (`layout-spacing`, `layout-padding`: `length (out)`, no numeric default listed).
- `font-size: 0px` = unset → falls back to enclosing `Window.default-font-size` (explicit on `StyledText.default-font-size`; `Window.default-font-size: length = 0`, "forms the basis for relative font sizes"; Fonts guide general rule). STILL-UNSURE: final pixel when `Window.default-font-size` is itself `0` — no page states it; needs renderer/style source or runtime probe.
- `TabWidget.orientation: Orientation = horizontal` MUST be constant (`orientation: foo;` errors, `Orientation.vertical` valid). `Slider.orientation` has NO such note — do not generalize. `Orientation` = `horizontal | vertical`.
- `alignment: LayoutAlignment = stretch` — EXACT 7 values: `stretch|center|start|end|space-between|space-around|space-evenly`. `cross-axis-self-alignment: auto` — `auto` legal ONLY on the per-child knob (error on the container).
- Theming globals: `StyleMetrics` ONLY `layout-spacing` + `layout-padding` (`length (out)`); `Palette` 12 props (`brush (out)` except last): `background`, `foreground`, `alternate-background`, `alternate-foreground`, `control-background`, `control-foreground`, `accent-background`, `accent-foreground`, `selection-background`, `selection-foreground`, `border`, `color-scheme: ColorScheme (in-out)` (`unknown|dark|light`, only `in-out` switch; all styles except Qt support forcing dark/light). No `background`/`border-*`/`corner-radius`/`font-*`/`color` on any Box — wrap content in an owned `Rectangle`/`Border`.

## Common props

- `spacing` (between children), `padding` + `padding-left/right/top/bottom` (inside border).
- Grid children gain `row`/`col` (default 0) + `rowspan`/`colspan`, via `Row{}` sub-element or explicit `row`/`col`. Runtime-changeable; `for`/`if` allowed inside.
- Custom containers place use-site children at `@children`:

```slint
component Card inherits Rectangle {
    VerticalLayout {
        padding: 12px;
        @children
    }
}
export component App inherits Window {
    Card { Text { text: "use-site child lands at @children"; } }
}
```

## Alignment

- Layout `alignment`: `stretch` (default) / `start` / `end` / `center` / `space-between` / `space-around`.
- Elements only grow beyond preferred when `stretch`.
- Cross-axis: container `cross-axis-alignment` + child `cross-axis-self-alignment: auto/stretch/start/end/center` (`auto` = use container).
- `layout-order` (int, default 0) controls visual order ascending; focus stays declaration order.

## Sizing constraints (all visible items)

- `min-width/min-height`, `max-width/max-height`, `preferred-width/preferred-height`, `horizontal-stretch/vertical-stretch` (float, default 0.0; widgets 0 or 1).
- Explicit `width`/`height` = fixed size in layout, no stretch.

## Preferred size

- From content (`Text`/`Image`), inner layout, or biggest child whose `x`/`y` unset.
- `if`/`for` children excluded unless edition 1.19+ (this repo opts in — both `build.rs`).
- `Flickable`/`ScrollView` takes content only from layout child.
- `preferred-width: 100%` = parent-sized component.
- Unset `x`/`y` constraints auto-propagate to parent.

## Stretch algorithm

1. Start at preferred, clamped to `[min, max]`.
2. Extra space shared proportional to `stretch`, capped at `max`.
3. Deficit shrinks proportional to `stretch`, floored at `min`.
4. `min-height: 0px` allows shrink below preferred; `min > max` => min wins.
5. Factor 0 keeps preferred unless all 0 or others at max/min.
6. Content-sized default 0, parent-filling default 1; own factor = min(own, children) — override explicitly.

## Absolute geometry

- `x`/`y` offset from parent top-left, `width`/`height` extend right/down.
- `z` (float 0.0) = stacking + input order.
- `absolute-position: {x, y}` is read-only `Point` vs `Window`/`PopupWindow`.
- Defaults: `x = (parent.width - self.width)/2`, `y = (parent.height - self.height)/2`.
- `Rectangle`/`TouchArea`/`FocusScope`/`Flickable`/layouts default 100% parent; others content/preferred.
- Units `px` logical (DPI-scaled); `0phx` physical. Convert with `/1px`.

## Floor rule — 1280x720 (load-bearing)

```slint
export component AppWindow inherits Window {
    min-width: 1280px;
    min-height: 720px;
    max-width: 3840px;  // clamp, not launch size
    max-height: 2160px;
}
```

- `main.rs` clamps restored size to the same floor.
- 1280 = narrowest width widest content fits: Matches table needs ~940px of column arithmetic beside the 272px rail + page gutters. Below this numeric cells collide — window stops instead of eliding.
- Reference page is 1475px wide / 1068px card (72px rows, 18px values); shell floor 1280 leaves ~936px card — baseline is the reference's **ratio** at our width, not absolute numbers. If floor ever changes, re-derive the table.

## Width arithmetic (write the sum)

- Rail `Shell.rail-width: 272px` + gutters + Matches columns ≈ 940px → floor 1280px.
- Match row: 66px tall, 8px radius, 1px border, 10px gap, 4px result spine, 44px portrait, 28px rank medal, kebab right edge.
- Stat cells: `K/D 54px`, `K/D/A 150px`, `DDΔ 64px`, `HS % 56px`, `ACS 56px` (see `matches.slint` `StatCell` `col-width`); header/rows/day bands share the same widths in the same order.
- Cells share the row via `horizontal-stretch: 1` + `min-width`; a row of fixed-width cells clips the last agent cell — the floor arithmetic must still fit.
- Bars anchor left: `x: 0` + `parent.width * clamp(share, 0, 1)`. Centred fill = wrong value.
- Numbers never touch: visible gutter between adjacent values; tail gets its own layout if two numbers merge.
- Portraits/art never collapse a row: missing `image` draws glyph at same size.
- Empty = em dash via `Cell`; never 0 or invented text.

## Screenshot verification (ships with the change)

1. `cargo build --features ui-debug`, run with `--mcp-port`, `take_screenshot`, read the image.
2. Screenshot at **1280x720 floor** AND wide: nothing may clip, collide, or elide.
3. Header, rows, day bands agree (same columns, same widths); bars left-anchored; portraits full-size.
4. Off by one step (4px vs 8px) is still off.

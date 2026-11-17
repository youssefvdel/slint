# Repo Patterns — OutPeek UI (verified from tree)

> Shared UI is imported, never copied. `outpeek-app/build.rs` include path
> `../outpeek-overlay/ui` makes `tokens.slint` resolve from the overlay crate.
> Keep both crates siblings or fix `Cargo.toml` path dep + `build.rs` together.

## AppWindow shell (`outpeek-app/ui/app.slint`)

```slint
import { Theme } from "tokens.slint";
import { Tracker } from "state.slint";
import { Sidebar } from "components/sidebar.slint";
import { Header } from "components/header.slint";
import { TitleBar } from "components/titlebar.slint";
import { TrackerPage } from "components/cards.slint";
import { SettingsPage } from "components/settings.slint";
import { MatchesPage } from "components/matches.slint";

export { Tracker, Settings, SettingsItem } from "state.slint";
export { Theme } from "tokens.slint";

export component AppWindow inherits Window {
    no-frame: true;
    resize-border-width: 6px;
    preferred-width: Tracker.window-width;   // opens 1216x804
    preferred-height: Tracker.window-height;
    min-width: 1280px; min-height: 720px;    // floor — see layout.md
    max-width: 3840px; max-height: 2160px;   // clamp, not launch size
    background: Theme.surface;
    callback toggle-hud(); callback refresh-rank();
    callback navigate(string); callback select-tab(string);
    callback minimize-window(); callback toggle-maximize(); callback close-window();
    HorizontalLayout {
        Sidebar { /* full height, hairline to top edge */ }
        VerticalLayout {
            TitleBar { /* own chrome; Rust owns Win32 commands */ }
            Header { /* tabs; subtitle = account only */ }
            Rectangle {
                vertical-stretch: 1;
                flick := Flickable {
                    width: 100%; height: 100%;   // load-bearing: without them
                    content-width: self.width;   // column shrank / viewport grew
                    content-height: page.preferred-height;
                    page := VerticalLayout {
                        width: flick.content-width;
                        if Tracker.view == "tracker" && Tracker.tab == "overview": TrackerPage {}
                        if Tracker.view == "tracker" && Tracker.tab == "matches": MatchesPage {}
                        // agents / maps / live / settings switch the same way
                    }
                }
                if flick.content-height > flick.height: Rectangle { /* 3px thumb */ }
            }
        }
    }
}
```

- Frameless by construction (`no-frame: true` + drawn `titlebar.slint`); `win_style::apply` touches OVERLAY hwnd only, never app window (`roles::wants_clickthrough` tested).
- `Flickable` `width/height: 100%` + `content-width/height` are load-bearing; scrollbar is a drawn 3px thumb, only when content overflows.

## Theme + state (`tokens.slint`, `state.slint`)

- `export global Theme { in-out property<brush> surface/mint/coral/...; }` — brush tokens, single source.
- `export global Tracker { ... }` — every label/metric/view/tab owned by Rust; `.slint` binds.
- `export global Shell { in-out property<length> rail-width: 272px; }` — titlebar + rail share it; columns never disagree about content start.
- Structs pre-formatted by Rust: `MatchRowUi` (id/map/age/score/score-won/score-lost/placement/rank-art/rank-name/won/agent/trs/trs-grade/kd-low/clutch/clutch-bad/art/kda/kd/hs/dd/acs), `MatchAgentUi` (name/record/kd/win-share/art), `MatchSummaryUi` (record/kd/adr/win-share), `MatchDayUi` (title/count-label/wins-label/losses-label/kda-sub/kd/kda/dda/hs/acs), `MatchPlayerUi`, `MatchRoundUi` (label/blue/tip).
- `win-share: float` 0–1 widths the bar only, nothing else reads it.

## Matches table (`components/matches.slint`)

- `StatCell { in property<length> col-width: 52px; width: root.col-width; }` — widths single-sourced via `col-width`.
- Row: 4px spine + 44px portrait + 146px identity + `StatCell`s (54/150/64/56/56) + 20px kebab; `Rectangle { horizontal-stretch: 1; }` spacers absorb slack; `min-width: 150px` on flexible cells.
- Win bar: `width: parent.width * clamp(root.item.win-share, 0, 1)` anchored `x: 0`.
- Day band 50px / 8px radius / date 16px/700 secondary ink + 30x24 count chip; verdict `wins mint // faint, losses coral`; sub-1.0 K/D `coral-dim`.
- Scoreboard: `Cell { col-width: ...; }` 72/48/92/48/56/56/64 + header `ColumnLabel` same widths same order; banner first column, two 28px top agents (whole `%`).
- Maps table: one body card, 34px header, 56px rows, 62px season cell + three agent cells each with win bar.
- Missing art → glyph at same size (`if art.width == 0: Glyph`), never collapse.

## Cards (`components/cards.slint`)

- One `body` card: radius 16, 1px border, 14/16px padding; `CardTitle` + `MetricCell { horizontal-stretch: 1; }` grid (WINS mint / LOSSES coral / KILLS / DEATHS / ...).
- `ScoreMetric`, accuracy 76px, crests 34px / 56px with glyph fallback, `max-width: 620px` + 210/270px columns + stretch spacers.

## HUD overlay (`outpeek-overlay/ui/overlay.slint`)

- Second `Window` from overlay crate at runtime; `if selecting:` / `if peeking:` gate cluster; `animate opacity { duration: Motion.mid; }`.
- `OverlayWindow` sizing/states mirror app edition 1.19 rules.

## Rust side (`outpeek-app/src/main.rs` pattern)

```rust
slint::include_modules!();
let app = AppWindow::new().expect(...);
app.global::<Tracker>().set_window_width(...);
app.on_navigate(move |v| { ... }); app.on_select_tab(...); app.on_refresh_rank(...);
app.global::<Tracker>().set_not_ported(ModelRc::new(VecModel::from(rows)));
```

- `#[path = "../../legacy/src-tauri/src/..."]` compiles five legacy modules by path (`fs_migrate`, `live_facts`, `overlay_renderer`, `paths`, `riot_http`) — renaming `legacy/` needs all five lines updated.

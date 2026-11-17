# Showcase — Copy-Paste Patterns

> Every pattern below uses only syntax from `language.md` / `layout.md` / `widgets.md`.
> Replace the placeholder values; keep the structure. Anything beyond this is `UNSURE`.

## 1. Counter (props + callback + function)

```slint
export component Counter inherits Rectangle {
    in-out property<int> count: 0;
    callback stepped(int);
    pure function doubled() -> int { return count * 2; }
    changed count => { stepped(count); }

    VerticalLayout {
        spacing: 8px; padding: 12px;
        Text { text: "count: \{count} (x2=\{doubled()})"; }
        TouchArea {
            clicked => { count += 1; }
            Text { text: "tap to step"; }
        }
    }
}
```

## 2. Reactive list (for + if + states + animate)

```slint
export struct RowUi { name: string, score: int, hot: bool, }

export component ScoreList inherits Rectangle {
    in-out property<[RowUi]> rows;
    private property<int> selected: -1;

    VerticalLayout {
        spacing: 4px;
        for r[i] in rows : Rectangle {
            background: r.hot ? red : transparent;
            states [ active when selected == i : { background: blue; } ]
            animate background { duration: 250ms; }
            TouchArea { clicked => { selected = i; } }
            HorizontalLayout {
                Text { text: r.name; horizontal-stretch: 1; }
                if r.hot : Text { text: "\{r.score} HOT"; }
            }
        }
    }
}
```

## 3. Global theme + two-way bind

```slint
export global Palette { in-out property<color> primary: blue; }

export component Themed inherits Rectangle {
    // two-way bind exposes the child/global prop under a new name
    property<brush> surface <=> self.background;
    background: Palette.primary;
    border-radius: 8px;
}
```

## 4. Gradient card + image with fallback

```slint
export component ArtCard inherits Rectangle {
    in-out property<image> art;
    background: @linear-gradient(135deg, #1b1030 0%, #3b1d5e 100%);
    border-radius: 16px; border-width: 1px; border-color: #ffffff22;
    clip: true;

    HorizontalLayout {
        padding: 12px; spacing: 10px;
        if art.width == 0 : Rectangle {
            width: 44px; height: 44px; border-radius: 8px;
            background: @radial-gradient(circle, gray 0%, transparent 100%);
        }
        if art.width > 0 : Image { source: art; width: 44px; height: 44px; }
        VerticalLayout {
            Text { text: "title"; font-size: 16px; font-weight: 700; }
            Text { text: "sub \{Math.round(0.956 * 100)}%"; color: gray; }
        }
    }
}
```

## 5. Button (TouchArea + states + animation)

```slint
export component PillButton inherits Rectangle {
    in-out property<string> label;
    callback pressed();
    border-radius: 999px;
    background: #2a2140;
    states [ down when area.pressed : { background: #3b2d5e; } ]
    animate background { duration: 150ms; easing: ease-in-out; }

    area := TouchArea { clicked => { root.pressed(); } }
    Text {
        text: label; horizontal-alignment: center; vertical-alignment: center;
        x: 0; y: 0; width: parent.width; height: parent.height;
    }
}
```

## 6. Scrollable page (ScrollView + GridBox)

```slint
import { ScrollView, VerticalBox } from "std-widgets.slint";

export component Page inherits Window {
    width: 800px; height: 600px;
    ScrollView {
        width: 100%; height: 100%;
        content-width: self.width; content-height: body.preferred-height;
        body := VerticalBox {
            width: parent.width;
            Text { text: "header"; font-size: 18px; font-weight: 700; }
            for i in 20 : Text { text: "row \{i}"; }
        }
    }
}
```

## 7. Popup + dialog skeleton

```slint
export component App inherits Window {
    // PopupWindow: tooltip/menu surface; close-policy: close-on-click|close-on-click-outside|no-auto-close
    // Dialog root: same props as Window, one main child; StandardButton kind + dialog-button-role
    //   kind values + <kind>_clicked aliases verified 2026-11-17, see widgets.md
}
```

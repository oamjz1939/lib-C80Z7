---
name: textual-tui
description: Write or modify Textual TUI apps in Python. Use when the user asks to build a terminal UI, convert a GUI to TUI, or work with the Textual framework. Covers layout, CSS variable gotchas, button patterns, clipboard, keyboard bindings, and terminal height constraints.
---

# Textual TUI

## Dependencies

```
pip install textual pyperclip
```

- `textual` — TUI framework
- `pyperclip` — clipboard (Textual has no built-in clipboard API)

## CSS variable matrix

Textual CSS variables have strict usage constraints. Getting this wrong is the #1 source of errors.

| Variable | `color:` | `background:` | `border:` | Notes |
|----------|----------|---------------|-----------|-------|
| `$text` | OK | ERR | ERR | Resolves to `auto` for bg/border — invalid |
| `$text-muted` | OK | ERR | ERR | Same — only for `color` |
| `$primary` | OK | OK | OK | Default blue; override for monochrome |
| `$secondary` | OK | OK | OK | Default darker blue |
| `$boost` | OK | OK | OK | Auto light/dark contrast — best for monochrome |
| `$surface` | OK | OK | OK | App background |
| `$panel` | OK | OK | OK | Content container background |

### Monochrome theme recipe

Use `$boost` for ALL interactive elements (buttons, borders). In dark terminals `$boost` is lighter, in light terminals it's darker — automatic contrast.

```css
/* Solid button */
.primary-btn { background: $boost; color: $text; }
/* Outline button */
.outline-btn { background: transparent; border: solid $boost; color: $text; }
/* Hover: add opacity */
.primary-btn:hover { background: $boost 60%; }
.outline-btn:hover { background: $boost 20%; }
/* Container border */
#content-box { border: round $boost; }
```

## Button click handling

Use CSS classes to distinguish buttons. Handle clicks in `on_button_pressed`:

```python
def compose(self) -> ComposeResult:
    yield Button("Copy", classes="copy-btn")
    yield Button("Exit", classes="exit-btn")

def on_button_pressed(self, event: Button.Pressed) -> None:
    if event.button.has_class("copy-btn"):
        # handle copy
    elif event.button.has_class("exit-btn"):
        # handle exit
```

### Finding data associated with a button

When buttons live in rows with data labels, navigate the widget tree:

```python
def _get_row_data(self, button: Button) -> str:
    row = button.parent                    # the container widget
    rows = list(self.query(".row-class"))  # all rows
    idx = rows.index(row)
    return self._data_list[idx]
```

## Data refresh patterns

### Pattern 1: Query + update (preferred for static row count)

Keep data in a list, update labels in place:

```python
def refresh(self) -> None:
    self._data = generate_new_data()
    labels = self.query(".data-label")
    for i, label in enumerate(labels):
        label.update(self._data[i])
```

### Pattern 2: Rebuild all widgets (matches tkinter destroy+rebuild)

```python
def refresh(self) -> None:
    container = self.query_one("#content")
    container.remove_children()
    for item in new_data:
        container.mount(RowWidget(item))
```

Prefer Pattern 1 — it's faster and avoids flicker.

## Clipboard

Textual has no built-in clipboard. Use `pyperclip`:

```python
import pyperclip
pyperclip.copy("text to copy")  # synchronous — no flush needed
```

## Keyboard bindings

```python
from textual.binding import Binding

class MyApp(App):
    BINDINGS = [
        Binding("1", "copy(0)", "Copy item 1"),
        Binding("r", "refresh", "Refresh"),
        Binding("q", "quit", "Quit"),
    ]

    def action_copy(self, index: int) -> None:
        text = self._data[index]
        pyperclip.copy(text)
        self.notify(f"Copied: {text}", timeout=2)

    def action_refresh(self) -> None:
        self.refresh()
```

`Binding` supports parameterized actions via `action_name(args)`.

## Delayed exit

Mimics `tkinter.after(500, destroy)`:

```python
pyperclip.copy(text)
self.set_timer(0.5, self.exit)  # seconds, not milliseconds
```

## Terminal height budget

Default Windows terminal: **24 rows**. Calculate layout height:

| Element | Rows |
|---------|------|
| Border top + bottom | 2 |
| Widget margin top + bottom | 2 × margin value |
| Each widget | its `height` value |
| Content padding | `padding` values |

**Rules of thumb:**
- 5 rows × `height: 3` + refresh button × `height: 3` + margins ≈ 24 rows — fits without scrolling
- `height: 1` rows hide button text — only use if buttons are text-less
- Remove title/header/footer text to reclaim 3–6 rows
- Use `dock: bottom` on the Footer widget to pin it below scrollable content

## Layout composition pattern

```python
def compose(self) -> ComposeResult:
    with Vertical(id="outer"):
        with Horizontal(classes="row"):
            yield Static("label", classes="label")
            yield Button("Action", classes="action-btn")
    yield Button("Bottom", id="bottom-btn")

def on_mount(self) -> None:
    # First data population after widgets exist
    self.refresh()
```

Widgets created in `compose()` are available for `query()` and `mount()` in `on_mount()` and later.

## Notification / toast

```python
self.notify("Copied: 2026-05-11", timeout=2)  # auto-dismiss after 2s
```

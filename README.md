# term

Terminal toolkit for Neutron: newline-free output, cursor and screen control,
ANSI styling, text layout, key and line input, and a few ready-made widgets.

## Install

```console
$ ntsc pkg install term
```

```toml
[dependencies]
term = "^1.1.0"
```

## Use

Everything is reached through the `term` namespace:

```ntsc
use term

fun main() {
    say(term.bold("hello"))
    say(term.rule("build", 40))
    say(term.progress(3, 10, 20))
}
```

A full-screen program opens a session first. `open` enters raw mode and the
alternate screen; `restore` puts the terminal back, so keep the two paired in a
`finally`:

```ntsc
use term

fun main() {
    if (!term.is_tty()) {
        say("needs an interactive terminal")
        return
    }
    var handle = term.open()
    try {
        term.clear()
        term.move_to(0, 0)
        var name = term.prompt(handle, "name: ")
        term.clear()
        term.move_to(0, 0)
        term.write_line("hello " + name)
    } finally {
        term.restore(handle)
    }
}
```

## Output

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.write(text)` | | Writes without a newline and flushes. |
| `term.write_line(text)` | | Writes a line. |
| `term.write_at(column, row, text)` | | Moves the cursor, then writes. |
| `term.flush()` | | Flushes the output stream. |

Widgets return strings, so a frame can be assembled and written once.

## Screen

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.columns()` | `int` | Width in cells (`COLUMNS`, or 80). |
| `term.rows()` | `int` | Height in cells (`LINES`, or 24). |
| `term.clear()` | | Clears the screen and homes the cursor. |
| `term.clear_line()` | | Clears the current line. |
| `term.clear_to_end()` | | Clears from the cursor to the end of the screen. |
| `term.move_to(column, row)` | | Absolute cursor position, zero-based. |
| `term.move_up(n)` / `move_down(n)` | | Relative cursor movement. |
| `term.move_left(n)` / `move_right(n)` | | Relative cursor movement. |
| `term.hide_cursor()` / `show_cursor()` | | Cursor visibility. |
| `term.is_tty()` | `bool` | Whether stdout is interactive. |
| `term.open()` | `int` | Enters raw mode; returns the session handle. |
| `term.restore(handle)` | `bool` | Restores the terminal. |

Size comes from the `COLUMNS` and `LINES` environment variables, which most
shells export. Programs started without them fall back to 80×24.

## Style

Every styling function takes the text and returns it wrapped in escape
sequences, so it composes with concatenation and with the widgets.

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.bold(text)` … `term.strike(text)` | `string` | Bold, dim, italic, underline, blink, reverse, strike. |
| `term.fg(text, color)` | `string` | Foreground palette colour 0–255. |
| `term.bg(text, color)` | `string` | Background palette colour 0–255. |
| `term.styled(text, color, background)` | `string` | Both colours at once. |
| `term.reset()` | `string` | The sequence that clears every attribute. |
| `term.strip(text)` | `string` | The text without escape sequences. |

Colours have named constructors rather than constants, because a package can
only export functions:

`term.black()`, `term.red()`, `term.green()`, `term.yellow()`, `term.blue()`,
`term.magenta()`, `term.cyan()`, `term.white()`, and the `term.bright_*`
variants, returning palette indices 0–15.

```ntsc
say(term.fg("failed", term.red()) + " after " + term.bold("3 retries"))
```

## Text

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.width(text)` | `int` | Visible characters, ignoring escape sequences. |
| `term.pad(text, width)` | `string` | Pads on the right to `width`. |
| `term.pad_left(text, width)` | `string` | Pads on the left. |
| `term.pad_right(text, width)` | `string` | Pads on the right. |
| `term.center(text, width)` | `string` | Centres in `width` cells. |
| `term.truncate(text, width)` | `string` | First `width` visible characters; styling is dropped. |
| `term.wrap(text, width)` | `string` | Breaks into lines of at most `width`, joined by newlines. |
| `term.split(total, count, gap, index)` | `int` | Width of column `index` when `count` columns share `total`. |
| `term.center_offset(total, content)` | `int` | Left offset that centres `content`. |

`term.width` counts what the terminal shows, so padding and centring line up
even when the text already carries colour.

## Input

Input functions need a session from `term.open`, because raw mode is what makes
individual key presses readable.

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.read_key(handle, timeout_ms)` | `string` | Waits for a key; `""` on timeout. |
| `term.key_character()` | `string` | The character of the last key, if it was one. |
| `term.key_ctrl()` | `bool` | Whether the last key used Control. |
| `term.read_line(handle)` | `string` | Line editor with echo and backspace. |
| `term.prompt(handle, question)` | `string` | Writes a prompt and reads a line. |
| `term.confirm(handle, question)` | `bool` | Yes/no question; only `y`/`Y` is yes. |
| `term.select(handle, question, options)` | `int` | Arrow-key menu; `-1` when cancelled. |

Key names returned by `read_key` are `character`, `space`, `enter`, `escape`,
`tab`, `backspace`, `delete`, `home`, `end`, `page_up`, `page_down`, `up`,
`down`, `left`, `right`, `ctrl_c`, and `other`.

```ntsc
var choice = term.select(handle, "deploy where?", ["staging", "production"])
if (choice == 1) {
    term.write_line(term.fg("careful", term.yellow()))
}
```

## Widgets

| Function | Returns | Behavior |
| --- | --- | --- |
| `term.hline(width)` | `string` | A box-drawing horizontal line. |
| `term.hline_char(width, character)` | `string` | A line of any character. |
| `term.rule(title, width)` | `string` | A rule with a centred title. |
| `term.box(title, lines, width)` | `string` | A framed box; `lines` is an array of strings. |
| `term.progress(done, total, width)` | `string` | A progress bar including brackets. |
| `term.spinner(frame, label)` | `string` | One spinner frame with an optional label. |
| `term.table(headers, rows, widths)` | `string` | A table; each row is a tab-separated string. |

```ntsc
var headers = ["name", "version"]
var rows = ["term\t1.1.0", "toml\t1.0.1"]
var array[int] widths = arrays.fill(0, 0)
arrays.push(widths, term.split(40, 2, 2, 0))
arrays.push(widths, term.split(40, 2, 2, 1))
say(term.table(headers, rows, widths))
```

## Layout

The package keeps each concern in its own file:

```
src/lib.nt       the public API, delegating to the modules below
src/ansi.nt      escape sequence builders
src/style.nt     attributes, colours, strip
src/screen.nt    output, size, cursor, clear, sessions
src/text.nt      width, padding, truncation, wrapping, column widths
src/input.nt     keys, line editor, prompts, menu
src/widgets.nt   rules, boxes, progress, spinner, table
```

## License

Neutron Permissive License (NPL) 1.1. See [`LICENSE`](LICENSE).

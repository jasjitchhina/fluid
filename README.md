<div align="center">

<img src="assets/neru-appicon.png" alt="Neru" width="80" />

# Neru

Hints, grids and vim keys for your whole desktop. Free, open source, one binary, one TOML file.

|     macOS 14+     |   Linux (X11, Wayland)   |     Windows 10+     |
| :---------------: | :----------------------: | :-----------------: |
|     Stable ✅     |      Beta, daily 🔵      |    Beta, daily 🔵   |

<sub>[What the labels mean](docs/reference/platform-support.md#what-the-labels-mean)</sub>

</div>

---

https://github.com/user-attachments/assets/322e2666-d961-46a9-bd7c-39e42fda8819

Neru does what Vimium does in the browser, for every app, window, menu bar and dock item on your screen. Press a hotkey, type the label on the element you want, and the cursor jumps there and clicks.

## Why Neru

- **Works where accessibility trees don't.** Grid modes split pixels, not widgets, so they work in canvases, games and remote desktops. Hints use the accessibility tree, on-device OCR, or a pure-Go scan of the screen image.
- **Fast.** The event tap sits on every keystroke, and Neru treats any added latency as a bug.
- **No per-app hacks.** Good defaults, plus per-app overrides you write yourself, applied when focus changes.
- **Scriptable.** Hotkeys, the `neru` CLI and the IPC socket run the same commands, so a binding works unchanged from skhd, Hammerspoon, Raycast or a shell script. Hooks run your own steps when a mode opens or closes, the focused app changes, or Neru pauses. `neru watch` streams the same events live, filtered with `--event`. `neru query` reports displays, the cursor and hint targets, and `--json` on the status and query commands gives a program something to parse. The IPC protocol is versioned, with JSON schemas and a Go client package.
- **Private.** OCR and element detection run on-device. No telemetry, no accounts, and typed text never reaches the log.
- **Fails loudly.** A mistyped flag in a binding fails config validation instead of doing nothing when you press the key.

## Install

Build from source with `just build` (macOS needs the Xcode Command Line Tools).
Installing the app bundle, completions and uninstalling are in
[Installation](docs/guide/installation.md).

## Quick start

Run `neru launch`, grant the permissions `neru doctor` asks for, then press
`Primary+Shift+Space` for hints. The full first run, including Linux, is in
[Getting started](docs/guide/getting-started.md).

## Modes

<table>
<tr>
<td align="center" width="50%">
<img src="https://github.com/user-attachments/assets/4327c027-d3c8-4915-a740-a0c9c46b386d" alt="Hints Mode" /><br/>
<sub><b>Hints</b></sub>
</td>
<td align="center" width="50%">
<img src="https://github.com/user-attachments/assets/3b8a4ccb-f102-4971-902e-c4097d14afe5" alt="Vim Mode" /><br/>
<sub><b>Vim</b></sub>
</td>
</tr>
<tr>
<td align="center" width="50%">
<img src="https://github.com/user-attachments/assets/f72b43fb-9745-4c17-a80d-15ca4077a5a2" alt="Recursive Grid Mode" /><br/>
<sub><b>Recursive Grid</b></sub>
</td>
<td align="center" width="50%">
<img src="https://github.com/user-attachments/assets/6b18396a-9bb8-4d1b-a8f8-270c19c0f727" alt="Bisect Mode" /><br/>
<sub><b>Bisect</b></sub>
</td>
</tr>
<tr>
<td align="center" colspan="2">
<img src="https://github.com/user-attachments/assets/b0759650-a3fa-43f3-9123-237124afe870" alt="Grid Mode" width="50%" /><br/>
<sub><b>Grid</b></sub>
</td>
</tr>
</table>

| Mode                  | Default hotkey        | How it works                                                    | Best for                                    |
| :-------------------- | :-------------------- | :-------------------------------------------------------------- | :------------------------------------------ |
| **Recursive Grid** ⭐ | `Primary+Shift+C`     | Split 3x3 and pick cell after cell until the cursor lands       | Anything, including canvases and games      |
| **Bisect**            | `Primary+Shift+B`     | `hjkl` keep a half, `yubn` a quadrant, until the cursor lands   | Anything, without reading labels            |
| **Hints**             | `Primary+Shift+Space` | Labels every target via accessibility tree, OCR or a pixel scan | Native apps, Electron, browsers, Figma      |
| **Grid**              | `Primary+Shift+G`     | Type a row and column label to jump to a cell                   | Coarse jumps across big monitors            |
| **Scroll**            | `Primary+Shift+S`     | `j`/`k`, `u`/`d`, `gg`/`G`, `h`/`l` in any scroll view          | Reading without lifting your hands          |
| **Vim**               | unbound               | Reads the screen's text and moves a cursor over it with vim keys | Selecting, copying and editing any app's text |
| **Monitor Select**    | disabled, unbound     | Labels each display, type one to jump                           | Multi-monitor setups                        |
| **Your own**          | unbound               | A name, an indicator and a key table you declare in config      | Window management layers, app-specific keys |

`Primary` is `Cmd` on macOS and `Ctrl` on Linux and Windows. Linux ships without these defaults so they never collide with terminal shortcuts. Every binding is remappable.

Every mode except monitor select gives you left, right and middle click, double and triple click, drag with any button, and sticky modifiers. `[held_repeat]` adds held-key glide, with acceleration if you want it.

## Configuration

Everything is one TOML file. Apply edits with `neru config reload`, or change one value with `neru config set`.

```toml
[hotkeys]
"Primary+Shift+Space" = "hints --action left_click"   # click on select, Vimium style
"Primary+Shift+W" = "mode window"

[macros]
click_and_exit = ["action left_click --bail-on-error", "idle"]

[[hints.app_configs]]
bundle_id = "com.brave.Browser"   # bundle ID on macOS, WM_CLASS or app_id on Linux, full executable path on Windows
hotkeys = { "Return" = "macro click_and_exit" }

[modes.window]
indicator = "Window"

[modes.window.hotkeys]
"h" = "exec yabai -m window --focus west"
"l" = "exec yabai -m window --focus east"
```

Every hotkey command is also a shell command:

```bash
neru hints --role button --text save   # only buttons whose text matches
neru run "action save_cursor_pos" "hints --action left_click" \
         "action wait_for_mode_exit" "action restore_cursor_pos"
neru config set hints.strategy contour
neru doctor                            # diagnose permissions and backends
```

Themes, indicators, smooth cursor and scroll, virtual pointer, app exclusions and screen-share hiding are options too.

## Build on Neru

Status bars, launchers and voice tools can follow Neru and drive it:

```bash
neru watch --event mode_enter,mode_exit   # one JSON line per mode change
```

```go
neru := client.New()   // github.com/y3owk1n/neru/client
status, err := neru.Status(ctx)
_, err = neru.Send(ctx, "hints", "--action", "left_click")
```

Which interface fits which tool is in [Build on Neru](docs/guide/integrations.md).

## Documentation

Browse [docs/](docs/README.md).

## License

MIT. See [LICENSE](LICENSE).

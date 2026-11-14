# Installation

How to build, install and remove Neru from source. Supported systems are in
[Platform support](../reference/platform-support.md#platform-status). After
installing, continue with [Getting started](getting-started.md).

## From source

| Platform | Needs                                                                                               |
| :------- | :-------------------------------------------------------------------------------------------------- |
| All      | Go 1.27+ and [just](https://github.com/casey/just)                                                  |
| macOS    | Xcode Command Line Tools (`xcode-select --install`) for the Objective-C bridge                      |
| Linux    | A C compiler and the `-dev`/`-devel` packages the CGO build links against, listed in [Development setup](../contributing/development.md#development-setup) |
| Windows  | Git for Windows, whose `sh` runs the `just` recipes. The build has CGO off, so no C compiler        |

```bash
just build        # bin/neru
just dist         # build/dist: bin/neru, man pages and, on macOS, Neru.app
```

On macOS, copy `build/dist/Neru.app` to `/Applications` so the CLI and the
daemon share the app's Accessibility and Screen Recording permissions, and
link its program onto your `PATH`:

```bash
cp -R build/dist/Neru.app /Applications/
ln -sf /Applications/Neru.app/Contents/MacOS/neru /usr/local/bin/neru
```

Other build options are in the
[Development guide](../contributing/development.md).

## Shell completions

Write completions with `neru completion`:

```bash
neru completion bash > ~/.local/share/bash-completion/completions/neru
neru completion zsh  > ~/.zsh/completions/_neru     # needs ~/.zsh/completions on your fpath
neru completion fish > ~/.config/fish/completions/neru.fish
```

```powershell
neru completion powershell | Out-String | Invoke-Expression   # add to your $PROFILE to keep it
```

## Login service

To start Neru at login:

```bash
neru services install
```

What it registers on each platform, and when it refuses, is under
[`neru services`](../reference/cli.md#neru-services).

## Uninstallation

macOS:

```bash
neru services uninstall
rm -rf /Applications/Neru.app
rm /usr/local/bin/neru
rm -f ~/.config/fish/completions/neru.fish ~/.zsh/completions/_neru \
      ~/.local/share/bash-completion/completions/neru
rm -f /usr/local/share/man/man1/neru*.1
rm -rf ~/.config/neru ~/Library/Application\ Support/neru ~/Library/Logs/neru
```

Linux:

```bash
neru services uninstall
rm -f ~/.local/bin/neru
rm -f ~/.local/share/bash-completion/completions/neru ~/.zsh/completions/_neru \
      ~/.config/fish/completions/neru.fish
rm -f ~/.local/share/man/man1/neru*.1
rm -rf ~/.config/neru ~/.local/share/neru ~/.local/state/neru

# only if nothing else needs it
sudo gpasswd -d "$USER" input
```

To remove the systemd unit by hand, find where it lives in
[Systemd user service](linux.md#systemd-user-service).

Windows:

```powershell
neru services uninstall
Stop-Process -Name neru -Force -ErrorAction SilentlyContinue
Remove-Item "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Neru.lnk"
Remove-Item "$env:LOCALAPPDATA\Programs\neru" -Recurse
Remove-Item "$env:APPDATA\neru" -Recurse
Remove-Item "$env:LOCALAPPDATA\neru" -Recurse
```

Remove the PATH entry under *Edit environment variables for your account* in
Settings, or use `just uninstall`. Avoid `setx`, which truncates the value at
1024 characters and flattens other tools' `%VAR%` entries.

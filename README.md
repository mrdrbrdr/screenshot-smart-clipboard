# wl-clip-multi

**Multi-MIME Type Wayland Clipboard Utility**

If you use Wayland and have ever tried to paste a screenshot into a terminal application (VS Code terminal, Claude Code, etc.) and nothing happens (or even worse, binary stuff) instead of what you expected, this tool is for you.

Ie you can't paste it as an image in Discord AND as a file path in your terminal apps. You have to choose ONE format, which is annoying. 
For example, if i wanna debug complicated web development, i'd usually take a screenshot and show it to my LLM CLI. But when i take the damn picture, i cant just paste it in the terminal. I literally have to 1) take screenshot, manually save it, go to my file explorer, find the damn picture, copy it manually (bc ctrl+c doesnt cut it), and THEN paste it into the LLM CLI in the terminal. 
<img width="340" height="423" alt="image" src="https://github.com/user-attachments/assets/02c075e6-39a5-48a5-9c45-1be97225b426" />

## How It Works

1. Connects directly to Wayland compositor
2. Creates a clipboard data source
3. Offers 6 MIME types simultaneously
4. Serves the appropriate type when apps request clipboard content
5. Mimics GNOME Nautilus clipboard format for maximum compatibility


## Fix
<img width="220" height="165" alt="image" src="https://github.com/user-attachments/assets/9acd7caa-a142-48d6-9aa4-5e63b2760484" />

`wl-clip-multi` uses the Wayland protocol directly (via `pywayland`) to offer 6 MIME types simultaneously:

1. `x-special/gnome-copied-files` - File path (for terminals)
2. `text/plain;charset=utf-8` - Plain text path
3. `text/uri-list` - Standard URI format
4. `application/vnd.portal.filetransfer` - XDG portal
5. `application/vnd.portal.files` - XDG portal files
6. `image/png` (or appropriate) - Raw content (for visual apps)

Apps then choose their preferred format when you paste.

## Installation

### 1. Install Dependencies

```bash
pip install pywayland

sudo pacman -S wl-clip-persist  # Arch/Manjaro
# or
yay -S wl-clip-persist  # AUR
```

### 2. Install wl-clip-multi

```bash
git clone https://github.com/mrdrbrdr/wl-clip-multi.git
cd wl-clip-multi
sudo cp bin/wl-clip-multi /usr/local/bin/
sudo chmod +x /usr/local/bin/wl-clip-multi
```

### 3. Setup wl-clip-persist (Required)

Without this, your clipboard only works for a split second:

Many Wayland systems run clipboard managers (like `elephant-clipboard`, `cliphist`) that use `wl-paste --watch` to monitor the clipboard. When they detect our multi-MIME clipboard, they read it and re-copy it - **but only preserve 1-2 MIME types**, destroying 6-type clipboard.

`wl-clip-persist` fixes this by reading ALL MIME types and preserving them.

**Add to autostart:**

<details>
<summary><b>Hyprland</b></summary>

Add to `~/.config/hypr/autostart.conf`:
```bash
exec-once = wl-clip-persist --clipboard regular
```
</details>

<details>
<summary><b>Sway</b></summary>

Add to `~/.config/sway/config`:
```bash
exec wl-clip-persist --clipboard regular
```
</details>

<details>
<summary><b>GNOME (Wayland)</b></summary>

Create `~/.config/autostart/wl-clip-persist.desktop`:
```ini
[Desktop Entry]
Type=Application
Name=wl-clip-persist
Exec=wl-clip-persist --clipboard regular
X-GNOME-Autostart-enabled=true
```
</details>

<details>
<summary><b>KDE Plasma (Wayland)</b></summary>

Create `~/.config/autostart/wl-clip-persist.sh`:
```bash
#!/bin/bash
wl-clip-persist --clipboard regular
```
Make it executable: `chmod +x ~/.config/autostart/wl-clip-persist.sh`
</details>

**Then start it now (or reboot):**
```bash
wl-clip-persist --clipboard regular &
```

That's it. 


## Technical Details
See [TECHNICAL.md](TECHNICAL.md) for:
- MIME type specifications and byte-level formats
- Wayland protocol implementation details
- Comparison with other clipboard tools
- Debugging and development info

## Why This Works Universally

This is a **Wayland protocol solution** that works on Hyprland, Sway, GNOME (Wayland session), KDE Plasma (Wayland session), or any ny Wayland compositor

The problem (wl-clipboard limitation) affects everyone on Wayland (i think), so the solution works everywhere.

## License
GPL-3.0

## Links

- [wl-clipboard issue #71](https://github.com/bugaevc/wl-clipboard/issues/71) - Multi-MIME discussion
- [wl-clip-persist](https://github.com/Linus789/wl-clip-persist) - Essential companion tool
- [pywayland](https://github.com/flacjacket/pywayland) - Python Wayland bindings

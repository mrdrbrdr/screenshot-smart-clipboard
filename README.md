# wl-clip-multi

**Multi-MIME Type Wayland Clipboard Utility**

Standard Wayland clipboard tools (`wl-copy`, `wl-paste`) can only offer **one MIME type at a time**. This means you can't copy a screenshot and paste it as an **image** in Discord while also pasting it as a **file path** in your terminal.

`wl-clip-multi` solves this by offering **6 MIME types simultaneously**. Applications automatically choose their preferred format.

## The Problem

```bash
# With wl-copy
grim screenshot.png
wl-copy < screenshot.png
# Paste in terminal → ��PNG... (binary garbage)
# Paste in Discord → Works!

# With wl-clip-multi
grim screenshot.png
wl-clip-multi screenshot.png &
# Paste in terminal → /home/user/screenshot.png (file path!)
# Paste in Discord → Works!
```

## Installation

```bash
git clone https://github.com/mrdrbrdr/wl-clip-multi.git
cd wl-clip-multi
pip install pywayland
sudo cp bin/wl-clip-multi /usr/local/bin/
```

**Dependencies:** Python 3.6+, `pywayland`, Wayland compositor

**Recommended:** `wl-clip-persist` (keeps clipboard alive after wl-clip-multi exits)

## Usage

```bash
wl-clip-multi ~/Pictures/screenshot.png
```

That's it. The daemon runs, serving clipboard requests. Press Ctrl+C to stop, or run in background with `&`.

See `examples/screenshot-multi-clipboard` for integration with screenshot tools.

## How It Works

Uses the Wayland protocol directly (via `pywayland`) to offer 6 MIME types:
1. `x-special/gnome-copied-files` - File path format (terminals)
2. `text/plain;charset=utf-8` - Plain text path
3. `text/uri-list` - URI format
4. `application/vnd.portal.filetransfer` - XDG portal
5. `application/vnd.portal.files` - XDG portal files
6. `image/png` (or appropriate) - Raw content (visual apps)

Apps request their preferred type. Mimics GNOME Nautilus clipboard format for compatibility.

## Troubleshooting

**Clipboard clears immediately?**
Install `wl-clip-persist` to preserve the multi-MIME clipboard:
```bash
sudo pacman -S wl-clip-persist
wl-clip-persist --clipboard regular &
```

**PyWayland not found?**
```bash
pip install pywayland
```

## Why This Exists

The `wl-clipboard` maintainer won't add multi-MIME support due to CLI complexity ([issue #71](https://github.com/bugaevc/wl-clipboard/issues/71)). His recommendation: "Use the Wayland protocol directly."

That's what we did.

## Technical Details

See [TECHNICAL.md](TECHNICAL.md) for in-depth implementation details, MIME type specifications, and protocol information.

## License

GPL-3.0

## Author

Ozan Demirezen

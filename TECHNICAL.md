# Technical Details

## MIME Type Specifications

The tool offers MIME types in the **exact order** GNOME Nautilus uses for maximum compatibility:

### 1. x-special/gnome-copied-files
**Format:** `copy\nfile:///absolute/path` (NO trailing newline)

This is critical for terminal applications. The byte-level format matters - even a trailing newline breaks compatibility with apps like Claude Code.

Example bytes:
```
63 6f 70 79 0a 66 69 6c 65 3a 2f 2f 2f 68 6f 6d
c  o  p  y  \n f  i  l  e  :  /  /  /  h  o  m
```

### 2. text/plain;charset=utf-8
**Format:** `/absolute/path/to/file` (plain text, no prefix)

Used by some terminal applications that don't support the GNOME format.

### 3. text/uri-list
**Format:** `file:///absolute/path\n` (WITH trailing newline)

Standard URI list format per [RFC 2483](https://www.rfc-editor.org/rfc/rfc2483). Note the trailing newline - this format requires it, unlike x-special/gnome-copied-files.

### 4-5. Portal Types
- `application/vnd.portal.filetransfer`
- `application/vnd.portal.files`

XDG Desktop Portal formats. We offer these for compatibility but most apps don't request them. They're part of the Flatpak/Portal security model for file access.

### 6. Content MIME Type
`image/png`, `application/pdf`, etc. - Actual file content

Detected based on file extension. This is what visual applications (Discord, GIMP, browsers) request.

## Implementation Details

### Wayland Protocol Flow

```
1. Connect to Wayland compositor
   ↓
2. Get registry and bind to:
   - wl_seat (input device management)
   - wl_data_device_manager (clipboard management)
   ↓
3. Create wl_data_source
   ↓
4. Offer 6 MIME types via data_source.offer()
   ↓
5. Set as clipboard selection via data_device.set_selection()
   ↓
6. Flush and roundtrip to ensure compositor processes it
   ↓
7. Event loop: Handle send requests when apps paste
```

### Why PyWayland?

The standard `wl-copy` tool uses `libwayland-client` but is limited to offering one MIME type. PyWayland gives us direct access to the Wayland protocol, allowing multiple MIME type offerings.

Alternative implementations could use:
- Direct `libwayland-client` in C
- `wayland-rs` in Rust
- Any language with Wayland bindings

### Clipboard Persistence

Without `wl-clip-persist`, the clipboard data is only available while `wl-clip-multi` is running. When it exits, apps can no longer paste.

`wl-clip-persist` solves this by:
1. Watching for clipboard changes
2. Reading **all offered MIME types** (unlike simple watchers)
3. Re-offering all MIME types from its own data source
4. Running indefinitely

This is why `wl-clip-multi` + `wl-clip-persist` work together:
- `wl-clip-multi` creates the multi-MIME clipboard
- `wl-clip-persist` preserves it after `wl-clip-multi` exits

## Comparison with Alternatives

### wl-copy
- **Implementation:** C, uses libwayland-client
- **MIME types:** 1 (single type from stdin or file)
- **Limitation:** Architectural - designed for piping data through stdin

### wl-clipboard-rs
- **Implementation:** Rust reimplementation of wl-copy
- **MIME types:** 1 (same limitation as wl-copy)
- **Status:** Active, but maintainer also doesn't plan multi-MIME support

### Clipboard Managers (cliphist, clipman, etc.)
- **Purpose:** Store clipboard history
- **MIME types:** Read multiple types, but don't create multi-MIME clipboards
- **Relationship:** Complementary - they work with wl-clip-multi

### wl-clip-persist
- **Purpose:** Keep clipboard alive after source app closes
- **MIME types:** Preserves all offered types
- **Relationship:** Essential companion to wl-clip-multi

### GNOME Nautilus
- **Implementation:** GTK/GLib, uses Wayland protocol directly
- **MIME types:** 6 (our tool mimics its exact format)
- **Why we copied it:** Terminal apps like Claude Code expect this exact format

## Known Issues and Limitations

### File Type Detection
Currently uses simple extension mapping. Could be improved with:
- `python-magic` for content-based detection
- User-configurable MIME type mappings
- Fallback to `application/octet-stream` for unknown types

### Single File Only
Currently supports one file at a time. Multiple file support would require:
- Different clipboard format (multiple file:// URIs)
- Handling of mixed content types
- More complex clipboard data structure

### No Primary Selection Support
Only implements clipboard selection (Ctrl+C/Ctrl+V), not primary selection (middle-click paste). This is by design - most Wayland apps use clipboard selection.

## Debugging

Enable debug output:
```bash
wl-clip-multi --debug screenshot.png
```

Or set environment variable:
```bash
export WL_CLIP_MULTI_DEBUG=1
wl-clip-multi screenshot.png
```

Debug output shows:
- File path being served
- MIME types offered
- Which MIME type each application requests

Use `wl-paste --list-types` to verify what MIME types are available in clipboard.

## Why the wl-clipboard Maintainer Won't Fix This

From [issue #71](https://github.com/bugaevc/wl-clipboard/issues/71):

> "It's unclear how this would look in the command-line interface... this is getting too complicated for the command-line tool."

The problem: With single MIME type, data comes from stdin. With multiple MIME types, where does each type's data come from? Multiple stdin pipes? Multiple files? The CLI design doesn't have a clean answer.

His recommendation: Use a library or the Wayland protocol directly. That's what we did.

## Future Improvements

Potential enhancements:
- [ ] Better MIME type detection (python-magic)
- [ ] Multiple file support
- [ ] Configuration file for custom MIME mappings
- [ ] Integration with more screenshot tools
- [ ] Support for copying directories
- [ ] Video/audio file support with metadata
- [ ] DBus interface for easier integration
- [ ] Systemd service file for running as daemon

## Contributing

When contributing, maintain:
- Byte-perfect GNOME format compatibility
- Same MIME type order as Nautilus
- Clean separation of concerns (clipboard serving vs. file handling)
- Minimal dependencies

## References

- [Wayland Protocol Documentation](https://wayland.freedesktop.org/docs/html/)
- [PyWayland Documentation](https://pywayland.readthedocs.io/)
- [wl-clipboard Issue #71](https://github.com/bugaevc/wl-clipboard/issues/71)
- [RFC 2483 - URI List MIME Type](https://www.rfc-editor.org/rfc/rfc2483)
- [XDG Desktop Portal](https://flatpak.github.io/xdg-desktop-portal/)

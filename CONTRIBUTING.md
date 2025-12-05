# Contributing to wl-paste-anywhere

Thanks for your interest man! This project welcomes contributions.

## Reporting Issues

If the tool doesn't work for you:

1. Include your desktop environment (Hyprland, Sway, GNOME, etc.)
2. Output of `echo $XDG_SESSION_TYPE` (should be "wayland")
3. Is `wl-clip-persist` running? Check with `ps aux | grep wl-clip-persist`
4. What are you trying to paste into?

## Pull Requests

- Keep changes focused and minimal
- Test on at least 2 desktop environments if possible
- Update README.md if adding features

## Testing

Run the examples:
```bash
cd examples
./test-basic.sh
```

## Questions?

Open an issue or discussion - happy to help!

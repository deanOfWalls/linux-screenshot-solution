After recent updates, my old screenshot setup started showing unbearable visual artifacts, so I switched to a cleaner, headless setup. I wanted to maintain the ability to:
- Capture only the active display (where the cursor is) for full-screen screenshots.
- Perform manual select area captures.
- Pipe all screenshots directly to the clipboard.
- Avoid any annoying GUI popups or windows.

### Tools:
- `scrot`: for screenshots
- `xdotool`: to get the cursor position
- `xrandr`: to detect display geometry
- `xclip`: to pipe to clipboard

### Script:
```bash
#!/bin/bash

# Get cursor position
CURSOR_POS=$(xdotool getmouselocation --shell)
X_POS=$(echo "$CURSOR_POS" | grep 'X=' | cut -d'=' -f2)
Y_POS=$(echo "$CURSOR_POS" | grep 'Y=' | cut -d'=' -f2)

# Get display geometry
DISPLAYS=$(xrandr | grep ' connected' | grep -oP '\d+x\d+\+\d+\+\d+')

for DISP in $DISPLAYS; do
    DISP_WIDTH=$(echo $DISP | cut -d'x' -f1)
    DISP_HEIGHT=$(echo $DISP | cut -d'x' -f2 | cut -d'+' -f1)
    DISP_X=$(echo $DISP | cut -d'+' -f2)
    DISP_Y=$(echo $DISP | cut -d'+' -f3)

    # Check if cursor is within this display
    if [ "$X_POS" -ge "$DISP_X" ] && [ "$X_POS" -lt $(($DISP_X + $DISP_WIDTH)) ] &&
       [ "$Y_POS" -ge "$DISP_Y" ] && [ "$Y_POS" -lt $(($DISP_Y + $DISP_HEIGHT)) ]; then
        scrot -a "${DISP_X},${DISP_Y},${DISP_WIDTH},${DISP_HEIGHT}" ~/Pictures/Screenshots/screenshot_%Y%m%d%H%M%S.png -e 'xclip -selection clipboard -t image/png -i $f'
        exit 0
    fi
done
```

### Custom GNOME Shortcuts:
- **Full-Screen Active Display**:
  - Command: `/bin/bash /home/yourusername/active-display-screenshot.sh`
  - Shortcut: `Print Screen`
- **Area Selection to Clipboard**:
  - Command: `scrot -s -e 'xclip -selection clipboard -t image/png -i $f'`
  - Shortcut: `Ctrl + Print Screen`

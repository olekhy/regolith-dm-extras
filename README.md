# Regolith / i3 Configuration Guide

This document describes the setup for i3 on Regolith, including power/suspend shortcuts, workspace assignments, and environment configuration.

---

## 1. Power/Suspend Shortcut

**File:** `~/.config/regolith3/i3/config.d/config`

**Purpose:**  
Binds the Sleep/Power key to a **lock and suspend script**.

**Content:**

```bash
bindsym XF86Sleep exec --no-startup-id ~/.local/bin/lock-suspend.sh
````

**Explanation:**

* `XF86Sleep` → responds to the Sleep key
* `~/.local/bin/lock-suspend.sh` → script locks the screen and suspends the laptop
* `exec --no-startup-id` → runs the command without an i3 startup ID

---

### `lock-suspend.sh` Script

**Path:** `~/.local/bin/lock-suspend.sh`

```bash
#!/bin/bash

# --- Bildschirm Helligkeit sanft dimmen ---
current_brightness=$(brightnessctl g)
target_brightness=50  # helligkeit minimal
step=5

while [ "$current_brightness" -gt "$target_brightness" ]; do
    brightnessctl s $current_brightness
    current_brightness=$((current_brightness - step))
    sleep 0.02
done

# --- Lautstärke sanft runter ---
current_vol=$(amixer get Master | grep -oP '\[\d+%\]' | head -1 | tr -d '[]%')
while [ "$current_vol" -gt 0 ]; do
    amixer set Master ${current_vol}% 2>/dev/null
    current_vol=$((current_vol - 2))
    sleep 0.02
done

# --- sperren ---
i3lock -c 000000

sleep 0.2

# --- suspend ---
systemctl suspend

# --- nach aufwachen wiederherstellen ---
brightnessctl s 100%  # oder setze auf alten Wert
amixer set Master 100%

```

**Explanation:**

* Locks the screen with a black background
* Pauses briefly to make sure the lock is active
* Suspends the system afterwards

> Optional: You can also mute audio or dim the screen in this script.

---

## 2. Workspace Setup & Window Assignments

**File:** `~/.config/regolith3/i3/config.d/my-workspace-setup`

**Purpose:**

* Assign workspaces to specific monitors
* Automatically assign applications to workspaces
* Control window focus behavior

**Content:**

```bash
workspace 1 output DP-1.9
workspace 9 output DP-1.8
workspace 2 output HDMI-0
workspace 3 output HDMI-0
workspace 4 output HDMI-0
workspace 5 output HDMI-0
workspace 6 output HDMI-0
workspace 7 output HDMI-0
workspace 8 output HDMI-0

assign [class="firefox"] workspace 1
assign [class="Navigator"] workspace 1

assign [class="terminator"] workspace 3
assign [class="X-terminal-emulator"] workspace 3

assign [class="jetbrains-phpstorm"] workspace 2
assign [class="phpstorm"] workspace 2

assign [class="google-chrome"] workspace 4
assign [class="Google-chrome"] workspace 4

assign [class="gedit"] workspace 8
assign [class="Gedit"] workspace 8

focus_on_window_activation focus
```

---

### Workspaces & Monitors

* Assign a workspace to a monitor:

```bash
workspace N output MONITOR_NAME
```

* Find monitor names:

```bash
xrandr | grep " connected"
xrandr --listmonitors
```

Example output:

```
DP-1.9 connected 3840x2160+0+0
DP-1.8 connected 1920x1080+3840+0
HDMI-0 connected 2560x1440+5760+0
```

* Use the left-hand name (`DP-1.9`, `HDMI-0`) in workspace assignments.

---

### Window Assignments & Class Names

* Assign programs to workspaces:

```bash
assign [class="NAME"] workspace N
```

* To find the **class name**:

**Using `xprop`:**

```bash
xprop | grep WM_CLASS
```

* Click on the window → output example:

```
WM_CLASS(STRING) = "Navigator", "Firefox"
```

* Use the **second value** as the class in i3:

```bash
assign [class="Firefox"] workspace 1
```

**Alternative with `xdotool`:**

```bash
xdotool getactivewindow getwindowclassname
```

* Returns the class name of the currently active window.

---

### Focus Behavior

* `focus_on_window_activation focus` → new windows automatically get focus
* Alternatives: `smart | urgent | none`

---

## 3. PATH Setup

Ensure `~/.local/bin` is in your PATH so i3 and your shell can find scripts like `lock-suspend.sh`.

### Bash

Add to `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Reload:

```bash
source ~/.bashrc
```

Check:

```bash
echo $PATH
```

---

### Fish


```fish
fish_add_path ~/.local/bin
```

echo $PATH
which lock-suspend.sh
```

---


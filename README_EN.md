# macOS Focus Monitor

**Monitor window focus hijacking on macOS. Protect your input privacy.**

Some macOS apps silently steal window focus in the background, interrupting your typing and ruining your flow. Worse, certain apps can auto-capture screenshots and upload them without your knowledge — potentially exposing passwords, browsing history, and chat content.

This script monitors all window focus changes in real time, helping you identify apps that hijack focus.

---

## Install & Run

```bash
# 1. Clone the repo
git clone https://github.com/Rain-at-Spring-Dawn/macos-focus-monitor.git
cd macos-focus-monitor

# 2. Install dependencies
pip3 install pyobjc-framework-Quartz pyobjc-framework-Cocoa

# 3. Grant Accessibility permission (required!)
#    System Settings → Privacy & Security → Accessibility → check your terminal app

# 4. Run
python3 focus_monitor.py -v
```

Press `Ctrl+C` to stop and view the summary report.

---

## Usage

```bash
python3 focus_monitor.py              # Show only suspicious events
python3 focus_monitor.py -v           # Show all focus changes
python3 focus_monitor.py -t 0.5       # Custom threshold (default: 0.3s)
python3 focus_monitor.py -o log.txt   # Write log to file
```

| Flag | Description |
|------|-------------|
| `-v` | Show all focus switches, including normal ones |
| `-t 0.5` | Increase threshold to reduce false positives |
| `-o log.txt` | Save output to file for later review |
| `-i 0.2` | Polling interval (default: 0.1s) |

---

## How to Read the Output

### Normal switch
```
[14:35:12] ✓  Safari  (PID 1234)
```
You clicked it. Nothing to see here.

### Suspicious switch
```
[14:35:15] ⚠  SomeApp  (PID 5678, com.example.app)  Δt=2.10s  [LSUIElement]
```
No keyboard or mouse activity detected — the app activated itself.

### Flag meanings

| Flag | Meaning | Severity |
|------|---------|:---:|
| `LSUIElement` | No Dock icon, runs hidden in background | 🔴 High |
| `FloatPanel` | Floating window layer, overlays other windows | 🔴 High |
| `Idle 10s` | Activated after 10+ seconds of user inactivity | 🟡 Medium |
| `Delay 0.5s` | Activated shortly after user input ended | 🟡 Medium |

### Reading the report

```
⚠  Found 2 suspicious apps:

  📱 SomeApp  [com.example.app]
     Suspicious/Total: 48/52  (92%)        ← 48 out of 52 activations were not user-triggered
     Method: LSUIElement(no Dock icon), FloatPanel(floating window)
     Path: /Applications/SomeApp.app
     🔍 Locate: open -R "/Applications/SomeApp.app"
     🚫 Disable: mv "/Applications/SomeApp.app" "/Applications/SomeApp_disabled.app"
     💡 Renaming prevents the app from being auto-triggered
```

- **> 70%**: Frequent focus hijacking — worth investigating
- **30%–70%**: Some non-user-triggered activations — keep monitoring
- **< 30%**: Likely normal behavior or environment noise

---

## Common Focus Hijacking Techniques

macOS APIs that apps may use to grab focus:

| Technique | Description |
|-----------|-------------|
| `activateIgnoringOtherApps:` | Activates own windows, ignoring other apps |
| `makeKeyWindow` | Forces a window to become the key (focused) window |
| Floating window level | Keeps a window visually on top of others |
| `LSUIElement = true` | Background mode with no Dock icon |

---

## How It Works

```
while running:
    check current foreground app
    if it changed:
        read last user input time
        if time gap > threshold → flag as suspicious
        read Info.plist → check LSUIElement
        query window level → check FloatPanel
    sleep 0.1 seconds
```

---

## Requirements

- macOS 10.14+
- Python 3.8+

## License

MIT

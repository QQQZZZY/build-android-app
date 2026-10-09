---
name: "android-emulator-qa"
description: "Use when validating Android feature flows in an emulator with adb-driven launch, input, UI-tree inspection, screenshots, and logcat capture."
---

# Android Emulator QA

Validate Android app flows in an emulator using adb for launch, input, UI-tree inspection, screenshots, and logs.

## Build Android App integration

Use the device and build variant selected by `../android-build-debug/SKILL.md`.
When serve-avd MCP is available, prefer its UI tree, screenshot and interaction
tools, always passing the selected device. The ADB commands and helpers below
are the fallback. Use the project's Gradle Wrapper instead of requiring global
Gradle. The UI picking helper requires Python 3.10 or newer; use an available
compatible interpreter. Preserve existing Logcat output and store artifacts in
a run-specific folder outside the installed skill directory.

For the ADB helper examples, set the artifact directory and choose an available
Python 3.10+ interpreter before running commands:

```bash
ARTIFACT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/codex-android-qa.XXXXXX")"
PYTHON_BIN="<path to Python 3.10+>"
"$PYTHON_BIN" -c 'import sys; assert sys.version_info >= (3, 10)'
```

## When to use
- QA a feature flow in an Android emulator.
- Reproduce UI bugs by driving navigation with adb input events.
- Capture screenshots and logcat output while testing.

## Quick start
1) List emulators and pick a serial:
   - `adb devices`
2) Build and install the target variant:
   - `./gradlew :<module>:install<BuildVariant> --console=plain --quiet`
   - If unsure about task names: `./gradlew tasks --all | rg install`
3) Launch the app:
   - Resolve activity: `adb -s <serial> shell cmd package resolve-activity --brief <package>`
   - Start app: `adb -s <serial> shell am start -n <package>/<activity>`
4) Capture a screenshot for visual verification:
   - `adb -s <serial> exec-out screencap -p > "$ARTIFACT_DIR/emu.png"`

On multi-display/foldable emulators, serve-avd 0.1.3 may report `screencap
produced no image`: Android prepends a display-selection warning to the image
bytes. In that case inspect `adb -s <serial> shell dumpsys SurfaceFlinger
--display-id`, choose the active display of the selected emulator, and capture
with `adb -s <serial> exec-out screencap -p -d <display-id>` instead. Inspect the
saved image before reporting success. Do not retry the same failing MCP capture
or treat a file containing warning text as a valid screenshot.

## adb control commands
- Tap (use UI tree-derived coordinates):
  - `adb -s <serial> shell input tap <x> <y>`
- Swipe:
  - `adb -s <serial> shell input swipe <x1> <y1> <x2> <y2>`
  - Avoid edges (start ~150-200 px from left/right) to reduce accidental back gestures.
- Text:
  - `adb -s <serial> shell input text "hello"`
- Back:
  - `adb -s <serial> shell input keyevent 4`
- UI tree dump:
  - `adb -s <serial> exec-out uiautomator dump /dev/tty`

## Coordinate picking (UI tree only)
Always compute tap coordinates from the UI tree, not screenshots.

1) Dump the UI tree to a step-specific file:
   - `adb -s <serial> exec-out uiautomator dump /dev/tty > "$ARTIFACT_DIR/ui-settings.xml"`
2) Find the target node and derive center coordinates (`x y`) from bounds:
   - Bounds format: `bounds="[x1,y1][x2,y2]"`
   - Helper script:
   - `"$PYTHON_BIN" <path-to-skill>/scripts/ui_pick.py "$ARTIFACT_DIR/ui-settings.xml" "Settings"`
3) If the node is missing and there are `scrollable` elements:
   - swipe, re-dump, and re-search at least once before concluding the target is missing.
4) Tap the center:
   - `adb -s <serial> shell input tap <x> <y>`

## UI tree skeleton (helper)
Use this helper to create a compact, readable overview before inspecting full XML.

1) Dump full UI tree:
   - `adb -s <serial> exec-out uiautomator dump /dev/tty > "$ARTIFACT_DIR/ui-full.xml"`
2) Generate summary:
   - `"$PYTHON_BIN" <path-to-skill>/scripts/ui_tree_summarize.py "$ARTIFACT_DIR/ui-full.xml" "$ARTIFACT_DIR/ui-summary.txt"`
3) Review the generated `ui-summary.txt` to choose likely targets, then compute exact bounds from full XML.

## Logs (logcat)
1) Stream app process logs without clearing existing logs:
   - Resolve pid: `adb -s <serial> shell pidof -s <package>`
   - Stream: `adb -s <serial> logcat --pid <pid>`
2) Crash buffer only:
   - `adb -s <serial> logcat -b crash`
3) Save logs:
   - `adb -s <serial> logcat -d > /tmp/logcat.txt`

## Package shortcuts
- List installed packages:
  - `adb -s <serial> shell pm list packages`
- Filter to your namespace:
  - `adb -s <serial> shell pm list packages | rg <company_or_app_id>`
- Confirm the activity resolves before launching:
  - `adb -s <serial> shell cmd package resolve-activity --brief <package>`

# Keyboard Zoom Feature Validation

Feature: Keyboard-driven zoom controls (`Ctrl++`, `Ctrl+-`, `Ctrl+0`) for Windows Desktop application.

## Overview & Background

Previously, zoom in/out was exclusively triggered via mouse wheel scrolling while holding `Ctrl` (`Ctrl + mouse scroll wheel`), calling `applyZoomDelta(app, delta)`. When using Tinta on a desktop or laptop without a mouse (e.g. touchpad only) or for quick keyboard ergonomics, keyboard zoom shortcuts were unavailable.

Standard desktop conventions (Chrome, Firefox, VS Code, SumatraPDF, Edge, Windows native apps) define:
- `Ctrl + +` / `Ctrl + =` / `Ctrl + Numpad +`: Zoom in (increments zoom factor by +10%, up to 300%).
- `Ctrl + -` / `Ctrl + Numpad -`: Zoom out (decrements zoom factor by -10%, down to 50%).
- `Ctrl + 0` / `Ctrl + Numpad 0`: Reset zoom (resets zoom factor directly to 100% / 1.0f).

## Implementation Details

1. **Modifiers Filter Guard (`src/keymap.cpp`)**:
   - `shortcutModifiersAllowed` filters chords when `Ctrl` and `Shift` are simultaneously pressed. On US standard keyboards, `+` is typed with `Ctrl + Shift + =`. Added `VK_OEM_PLUS`, `VK_ADD`, `VK_OEM_MINUS`, `VK_SUBTRACT`, `'0'`, and `VK_NUMPAD0` to the permitted switch cases so `Ctrl + Shift + =` as well as unshifted `Ctrl + =` are allowed to pass through to `handleKeyDown`.

2. **Core Zoom Logic (`include/input.h`, `src/input.cpp`)**:
   - Exposed `applyZoomDelta(App& app, float delta)` and added `applyZoomReset(App& app)`.
   - Guarded zoom delta changes against threshold deadlocks (`fabsf(newZoom - oldZoom) < 0.001f`) to prevent redundant format recreations when already clamped at bounds (0.5f to 3.0f).
   - In `handleKeyDown`, added handlers within the global Ctrl check (`!app.confirmExitPending`):
     - `wParam == VK_OEM_PLUS || wParam == VK_ADD`: `applyZoomDelta(app, 1.0f)` (zoom in).
     - `wParam == VK_OEM_MINUS || wParam == VK_SUBTRACT`: `applyZoomDelta(app, -1.0f)` (zoom out).
     - `wParam == '0' || wParam == VK_NUMPAD0`: `applyZoomReset(app)` (reset zoom to 1.0f).
   - Handled `Ctrl++`, `Ctrl+-`, `Ctrl+0` in `app.showLightbox` to zoom and reset the image overlay consistently with wheel zooming.
   - Returning `true` consumes the keydown event so no stray `WM_CHAR` is produced or passed down to child controls.

3. **Internationalization & UI (`src/i18n.cpp`, `src/overlays.cpp`, `src/startpage.cpp`, `README.md`)**:
   - Added `help.nav.zoom_reset` key to all 7 supported UI languages (EN, ZH, JA, KO, DE, FR, IT).
   - Updated Help overlay navigation entries:
     - `Ctrl+Scroll / Ctrl++/-`: Zoom in / out
     - `Ctrl+0`: Reset zoom
   - Updated start page navigation hints and README shortcut reference table.

## Automated Verification

1. **Automated Unit & Regression Tests (`tests/input_tests.cpp`)**:
   - `Ctrl + =` (`VK_OEM_PLUS`) increments `zoomFactor` from 1.0 to 1.1.
   - `Ctrl + -` (`VK_OEM_MINUS`) decrements `zoomFactor` from 1.1 back to 1.0.
   - `Ctrl + Shift + =` (`VK_OEM_PLUS` with `VK_SHIFT` pressed) zooms in to 1.1.
   - `Ctrl + Numpad +` (`VK_ADD`) and `Ctrl + Numpad -` (`VK_SUBTRACT`) zoom in and out.
   - `Ctrl + 0` ('0') and `Ctrl + Numpad 0` (`VK_NUMPAD0`) reset zoom to 1.0.
   - Clamping limits verified: lower bound at 0.5f (50%), upper bound at 3.0f (300%).
   - Edit mode verification: `Ctrl++` and `Ctrl+0` operate in edit mode without modifying or dirtying the buffer.
   - Lightbox verification: `Ctrl++`, `Ctrl+-`, `Ctrl+0` properly scale and reset `lightboxZoom`.

2. **Keymap Tests (`tests/keymap_tests.cpp`)**:
   - Verified `shortcutModifiersAllowed` returns `true` for all zoom keys (`VK_OEM_PLUS`, `VK_ADD`, `VK_OEM_MINUS`, `VK_SUBTRACT`, `'0'`, `VK_NUMPAD0`) when `Ctrl` + `Shift` are held.

3. **Mixed Content Fixture (`tests/fixtures/zoom-shortcuts.md`)**:
   - Contains mixed headings, markdown table, math equations ($E = mc^2$, integral block), fenced code blocks, blockquotes, numbered and bulleted lists, links, and text formatting.
   - Confirms math equations, code blocks, and table borders scale proportionally with zoom factor without clipping or layout breaking.

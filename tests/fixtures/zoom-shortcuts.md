# Keyboard Zoom Feature Fixture

This document demonstrates and verifies keyboard-driven zoom controls in Tinta:
- **Zoom In**: `Ctrl + +` / `Ctrl + =` / `Ctrl + Numpad +`
- **Zoom Out**: `Ctrl + -` / `Ctrl + Numpad -`
- **Reset Zoom**: `Ctrl + 0` / `Ctrl + Numpad 0` (resets to 100%)

---

## Shortcut Reference Table

| Shortcut | Action | Step / Target | Notes |
| :--- | :--- | :--- | :--- |
| `Ctrl + +` / `Ctrl + =` | Zoom in | +10% (up to 300%) | Also works with `Ctrl + Numpad +` and `Ctrl + Shift + =` |
| `Ctrl + -` | Zoom out | -10% (down to 50%) | Also works with `Ctrl + Numpad -` |
| `Ctrl + 0` | Reset zoom | 100% (1.0f) | Restores base scale instantly; also works with `Ctrl + Numpad 0` |
| `Ctrl + Mouse Wheel` | Smooth zoom | ±10% per tick | Existing mouse behavior remains fully compatible |

---

## Mixed Content Verification

### Mathematical Equations

When zoomed in or out, equation rendering must preserve bounding boxes, baseline alignment, and font sizes proportionally:

$$
f(x) = \int_{-\infty}^{\infty} \hat{f}(\xi)\,e^{2\pi i \xi x}\,d\xi
$$

Inline equations: $E = mc^2$, $\sum_{k=1}^n k = \frac{n(n+1)}{2}$, and $\lim_{x \to 0} \frac{\sin x}{x} = 1$.

### Fenced Code Blocks

Code blocks and syntax highlighting scale with the document zoom factor:

```cpp
void applyZoomDelta(App& app, float delta) {
    float oldZoom = app.zoomFactor;
    float newZoom = std::max(0.5f, std::min(3.0f, app.zoomFactor + delta * 0.1f));
    if (fabsf(newZoom - oldZoom) < 0.001f) return;
    app.zoomFactor = newZoom;
    float zoomRatio = app.zoomFactor / oldZoom;
    app.scrollY *= zoomRatio;
    app.targetScrollY *= zoomRatio;
}
```

### Blockquotes and Lists

> **Usability Note**: Keyboard zoom shortcuts follow common standards across web browsers,
> code editors, and document viewers (VS Code, Obsidian, Chrome, Firefox).
>
> 1. Scaling scales font sizes and layout margins smoothly.
> 2. Scroll anchor is maintained so reading position does not jump.
> 3. Per-document zoom settings are persisted across sessions.

### Links and Formatting

- [Internal link to math section](#mathematical-equations)
- [README documentation](../../README.md)
- Formatting checks: **bold text**, *italic text*, ***bold italic***, `inline code`, and ~~strikethrough~~.

### Edit Mode Compatibility

Press `:` or `E` to enter edit mode:
1. Verify `Ctrl + +` / `Ctrl + -` zooms both editor and preview pane.
2. Verify `Ctrl + 0` restores 100% zoom.
3. Verify that typing text with `+`, `-`, or `0` without Ctrl enters characters as normal.

# Centering Cam V2 — Design Spec

## Overview
Enhance the Pokemon card centering camera tool with adaptive detection, new controls, and richer feedback. Single HTML file, deployed on GitHub Pages.

## Architecture
- **Web Worker** for detection: all frame processing moves off the main thread via an inlined Blob Worker (keeps single-file deployment)
- Main thread: camera, overlay rendering, UI, haptics
- Communication: `postMessage` with `ImageData` transfer

## 1. Detection — Otsu's Adaptive Threshold
- Replace hardcoded threshold (55) with Otsu's method
- Compute 256-bin grayscale histogram per frame
- Find threshold minimizing intra-class variance
- Apply to existing projection-based bounding box detection
- Handles light surfaces, varying lighting automatically

## 2. New Controls

### Flip Camera
- Toggle `facingMode` between `environment` and `user`
- Stop current stream, re-acquire with flipped constraint
- Button: camera flip icon in controls panel

### Torch/Flashlight
- Use `MediaStreamTrack.applyConstraints({ advanced: [{ torch: true }] })`
- Only shown when `torch` capability is available (back camera)
- Toggle button with on/off state

### Snapshot
- Composite: draw video frame on a temp canvas, draw overlay on top, add stats watermark bar at bottom
- Trigger download as PNG via `<a download>`
- Camera flash effect: white overlay that fades out over 200ms
- Button in controls panel

### Tap to Lock/Unlock
- Tap anywhere on the video area toggles lock state
- Same logic as current LOCK button
- Existing buttons remain for discoverability

## 3. Feedback

### Stability Indicator
- Ring buffer of last 12 card position readings (x, y, w, h)
- Compute variance of center-point across buffer
- Thresholds: MOVING (>15px variance) → ADJUSTING (>8) → SETTLING (>3) → STABLE (<=3)
- Display in status bar and grade panel
- Only show grade when SETTLING or STABLE

### Border Pixel Labels
- Render L/R/T/B margin pixel counts directly on the overlay
- Positioned at midpoint of each edge, outside the card box
- Small monospace text with semi-transparent background

### Haptic Feedback
- `navigator.vibrate(50)` on first card detection (transition from no-card to card)
- `navigator.vibrate([30, 20, 30])` on lock
- Feature-detect `navigator.vibrate` before calling

### Grade Block Border Color
- `.grade-block` border-color matches the current grade color
- Green (#00ff88) for 10, yellow-green (#aaff00) for 9, yellow (#ffcc00) for 8, orange (#ff8800) for 7, red (#ff2244) for 6-

## Constraints
- Must remain a single `index.html` file (Web Worker inlined as Blob)
- No build tools, no dependencies
- GitHub Pages static hosting

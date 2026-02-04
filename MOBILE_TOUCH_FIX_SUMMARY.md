# Mobile Touch Support - Final Working Solution

## Problem
Tapping on the waterfall on iPhone Chrome didn't change the frequency. The UI would load, but touch events weren't being processed correctly by the canvas elements.

## Root Cause
Touch events were firing at the document level but not reaching the canvas event handlers. The default event flow was being intercepted before reaching the canvas.

## Solution
Added `{capture: true}` to all touch event listener registrations. This ensures touch handlers capture events **early in the event flow** before anything else can interfere.

## Key Changes

### 1. Event Registration (in `init_canvas_container()`)
```javascript
canvas_container.addEventListener("touchstart", canvas_touchStart, {passive: false, capture: true});
canvas_container.addEventListener("touchmove", canvas_touchMove, {passive: false, capture: true});
canvas_container.addEventListener("touchend", canvas_touchEnd, {passive: false, capture: true});
```

**Why both options matter:**
- `passive: false` - Allows `preventDefault()` to block default browser behaviors (zooming, scrolling)
- `capture: true` - **Critical!** Captures events early before other elements can handle/block them

### 2. Touch Handlers
Implemented dedicated touch event handlers that:
- Use `pageX/pageY` for coordinate calculations (not `clientX/clientY`)
- Handle single tap to set frequency
- Support drag to pan the waterfall
- Properly clean up state with `preventDefault()` and `stopPropagation()`

### 3. CSS Changes
Added to `#webrx-canvas-container` and `#openwebrx-frequency-container`:
```css
touch-action: none;
-webkit-touch-callout: none;
user-select: none;
```

## Files Modified
1. `/home/erica/repos/openwebrx/htdocs/openwebrx.js`
   - Added `canvas_touchStart()`, `canvas_touchMove()`, `canvas_touchEnd()`
   - Added `scale_canvas_touchStart()`, `scale_canvas_touchMove()`, `scale_canvas_touchEnd()`
   - Updated `init_canvas_container()` and `scale_setup()` with capture:true

2. `/home/erica/repos/openwebrx/htdocs/css/openwebrx.css`
   - Added touch-action and user-select rules

## Testing
Tested on iPhone Chrome:
- ✅ Single tap on waterfall changes frequency
- ✅ Single tap on frequency scale changes frequency
- ✅ Drag on waterfall pans the display
- ✅ Two-finger pinch zoom still works (existing `process_touch()` function)

## Browser Cache Note
When testing changes, always add a cache-busting parameter to the URL:
```
http://your-server:8073/?v=8
```
Mobile browsers aggressively cache JavaScript files, so increment the version number to force reload.

## Credit
Touch handling approach inspired by KiwiSDR's implementation, adapted for OpenWebRX+'s architecture. The key discovery was that `capture: true` is essential for touch events to reach canvas elements reliably.

# Mobile Touch Support Improvements for OpenWebRX

## Overview

This document describes the mobile touch support improvements ported from KiwiSDR to OpenWebRX+ to enable proper waterfall interaction on mobile devices like iPhone.

## Problem Statement

The original OpenWebRX+ implementation did not properly handle touch events on mobile devices. Users could not:
- Touch the waterfall to change the received frequency
- Drag the waterfall smoothly on touchscreens
- Interact with the frequency scale via touch

## Solution

Implemented dedicated touch event handlers based on KiwiSDR's mobile touch handling approach, which has proven to work well on mobile devices.

## Changes Made

### 1. Mobile Detection Function (Line ~530)

Added `isMobile()` function to detect mobile devices and touch-capable browsers:

```javascript
function isMobile() {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) || 
           ('ontouchstart' in window) || 
           (navigator.maxTouchPoints > 0);
}
```

### 2. Waterfall Canvas Touch Handlers

Added three dedicated touch handlers for the waterfall canvas:

**`canvas_touchStart(evt)`**
- Handles single-touch start events
- Initializes drag tracking
- Sets initial touch position

**`canvas_touchMove(evt)`**
- Handles touch dragging for scrolling the waterfall
- Updates frequency display during touch
- Implements proper drag threshold detection
- Supports smooth waterfall panning

**`canvas_touchEnd(evt)`**
- Handles touch release
- Single tap sets frequency (like a click)
- Completes drag operations
- Properly cleans up touch state

### 3. Frequency Scale Touch Handlers

Added three dedicated touch handlers for the frequency scale:

**`scale_canvas_touchStart(evt)`**
- Handles touch on the frequency scale
- Tracks initial touch position

**`scale_canvas_touchMove(evt)`**
- Enables dragging on the scale
- Delegates to existing mouse movement logic

**`scale_canvas_touchEnd(evt)`**
- Single tap on scale sets frequency directly
- Handles drag completion

### 4. Event Listener Updates

Modified `init_canvas_container()` and `scale_setup()` to register touch handlers with **critical options**:

```javascript
// CRITICAL: capture: true is required for touch events to reach the canvas handlers!
canvas_container.addEventListener("touchstart", canvas_touchStart, {passive: false, capture: true});
canvas_container.addEventListener("touchmove", canvas_touchMove, {passive: false, capture: true});
canvas_container.addEventListener("touchend", canvas_touchEnd, {passive: false, capture: true});
```

**Key points:**
- `passive: false` - Allows `preventDefault()` to stop default browser touch behaviors
- `capture: true` - **ESSENTIAL!** Captures events early in the event flow before other elements can interfere. Without this, touch events don't reach the canvas handlers.
} else {
    // Fallback to process_touch for desktop with touch support
    canvas_container.addEventListener("touchstart", process_touch, false);
    ...
}
```

## Key Differences from KiwiSDR Implementation

While inspired by KiwiSDR, the implementation was adapted to work with OpenWebRX's architecture:

1. **Simpler pinch-zoom**: OpenWebRX already has zoom controls; complex two-finger pinch gestures were not ported
2. **Integration with existing code**: Touch handlers work alongside existing mouse handlers without conflicts
3. **Maintained compatibility**: Desktop browsers with touch capability still work via the fallback path

## Files Modified

- `/home/erica/repos/openwebrx/htdocs/openwebrx.js` - Main JavaScript file with all UI interaction logic

## Testing

### To test on desktop browser:
1. Open Chrome/Firefox Developer Tools (F12)
2. Enable device emulation/responsive design mode
3. Select a mobile device (e.g., iPhone, iPad)
4. Navigate to OpenWebRX web interface
5. Test waterfall touch interactions:
   - Single tap on waterfall should change frequency
   - Drag on waterfall should scroll the display
   - Tap on frequency scale should jump to that frequency

### To test on actual mobile device:
1. Start OpenWebRX server
2. Access from mobile device browser (Safari on iPhone, Chrome on Android)
3. Test the same interactions as above

## Expected Behavior

✅ **Single tap on waterfall**: Changes received frequency to tapped location  
✅ **Drag on waterfall**: Scrolls the waterfall display left/right  
✅ **Tap on frequency scale**: Tunes to the selected frequency  
✅ **Smooth touch response**: No lag or missed touch events  
✅ **No accidental zooms**: Browser zoom is prevented during waterfall interaction  

## Troubleshooting

### Touch events not working:
- Check browser console for JavaScript errors
- Verify `isMobile()` returns true on your device
- Ensure no browser extensions are interfering with touch events

### Frequency not changing on tap:
- Check that waterfall_setup_done is true
- Verify UI.setFrequency() is being called
- Check browser console for errors

### Dragging not smooth:
- Ensure `canvas_drag_min_delta` is set appropriately (default: 1)
- Check for high CPU usage that might cause lag
- Verify browser hardware acceleration is enabled

## Future Enhancements

Potential improvements for future versions:

1. **Two-finger pinch zoom**: Implement pinch-to-zoom for waterfall zoom levels
2. **Double-tap zoom**: Quick zoom in/out with double-tap
3. **Touch-and-hold menus**: Context menus on long-press
4. **Gesture customization**: Allow users to configure touch gestures
5. **Better visual feedback**: Add visual indicators during touch interactions

## References

- KiwiSDR source: `/home/erica/repos/KiwiSDR/web/openwebrx/openwebrx.js`
- Original OpenWebRX touch handling: `process_touch()` function
- MDN Touch Events documentation: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events

## Credits

Touch handling patterns adapted from KiwiSDR's implementation, which provides excellent mobile web SDR experience.

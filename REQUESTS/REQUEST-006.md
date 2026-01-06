# REQUEST-006: UI Framework Phase 1 Polish

## Goal
Implement Phase 1 UI polish from SETUPSTAGES.md: rounded corners, smooth animations, cobalt theming, and micro-interactions for a professional, modern client interface.

## Scope
- vesper-client only
- Core UI component library with consistent styling
- Animation system for smooth transitions
- Theming system (colors, fonts, spacing)
- Micro-interactions for better UX

## Constraints
- Must work within Minecraft's rendering system (OpenGL/LWJGL)
- Must be performant (target <1ms per frame for UI rendering)
- Must be consistent across all UI elements
- Must support Minecraft's GUI scale setting

## vesper-client AI Instructions
```
CREATE: src/main/java/ui/components/ (component library)
CREATE: src/main/java/ui/theme/Theme.java (theming system)
CREATE: src/main/java/ui/animation/Animator.java (animation framework)

---

Component Library Requirements:

1. RoundedPanel Component
   Features:
   - Rounded corners (6px radius, configurable)
   - Background color with alpha support
   - Optional border (1px, configurable color)
   - Optional shadow (soft drop shadow, 2px offset)
   - Padding support (inner spacing)
   
   Methods:
   - draw(x, y, width, height, color)
   - drawWithBorder(x, y, width, height, bgColor, borderColor)
   - drawWithShadow(x, y, width, height, color, shadowAlpha)

2. AnimatedButton Component
   Features:
   - Base state: Background color, text, icon (optional)
   - Hover state: Cobalt glow (0.3s ease transition)
   - Press state: Scale 1.0 → 0.98 (0.1s)
   - Release state: Scale back (0.15s easeOutCubic)
   - Disabled state: Grayed out, no interactions
   
   States:
   - IDLE: Default appearance
   - HOVER: Cobalt glow around border, slight brightness increase
   - PRESSED: Scaled down, darker background
   - DISABLED: 50% opacity, no hover effects
   
   Animations:
   - Hover transition: 300ms ease (background color, glow)
   - Press transition: 100ms easeInQuad (scale)
   - Release transition: 150ms easeOutCubic (scale)
   
   Methods:
   - draw()
   - onClick(callback)
   - setEnabled(boolean)
   - setIcon(ResourceLocation)

3. ModuleCard Component (Expandable Panel)
   Features:
   - Header: Title, subtitle, toggle button, expand/collapse arrow
   - Body: Content area (settings, controls)
   - Expand/collapse animation (120-200ms easeOutCubic)
   - Smooth content fade (opacity 0→1)
   
   Animation:
   - Height transition: Animate from collapsed height to full height
   - Arrow rotation: 0° (expanded) to 180° (collapsed)
   - Content fade: 0 opacity when collapsed, 1 when expanded
   - Timing: 150ms for height, 200ms for content fade (staggered)
   
   Methods:
   - setExpanded(boolean, animated)
   - addContent(Component)
   - setTitle(String)
   - setToggleCallback(Consumer<Boolean>)

4. CustomScrollView Component
   Features:
   - Thin scrollbar (4px width)
   - Auto-hide after 1 second of inactivity
   - Cobalt accent on hover
   - Smooth scroll animation (easing)
   - Mouse wheel support
   
   Scrollbar:
   - Track: 2px wide, semi-transparent gray
   - Thumb: 4px wide, cobalt blue, rounded ends
   - Hover: Increase opacity, brighten color
   - Fade out: Opacity 1.0 → 0.0 over 300ms after 1s idle
   
   Scroll behavior:
   - Wheel scroll: 20 pixels per notch with easing
   - Drag scroll: Direct manipulation
   - Smooth scroll: easeOutQuad over 200ms
   
   Methods:
   - scrollTo(y, animated)
   - addChild(Component)
   - setScrollSpeed(double)

5. Toggle Component (Switch)
   Features:
   - OFF state: Gray background, handle on left
   - ON state: Cobalt background, handle on right
   - Slide animation: Handle moves left/right (200ms easeOutCubic)
   - Background color transition: 250ms ease
   
   Animation:
   - Handle position: Interpolate X position smoothly
   - Background color: Gray → Cobalt (or vice versa)
   - Timing: 200ms for handle, 250ms for background (staggered)
   
   Methods:
   - setState(boolean, animated)
   - onToggle(Consumer<Boolean>)
   - setEnabled(boolean)

6. Slider Component
   Features:
   - Track: Rounded bar (2px height)
   - Filled portion: Cobalt accent (left of handle)
   - Handle: Circle (10px diameter) with shadow
   - Smooth drag: No stuttering, precise positioning
   - Optional snap-to-grid (for discrete values)
   
   Interactions:
   - Drag: Smooth follow with cursor
   - Click on track: Jump to position with animation (150ms)
   - Hover handle: Scale 1.0 → 1.1
   - Drag handle: Scale 1.1 → 1.15 (active state)
   
   Methods:
   - setValue(double, animated)
   - setRange(min, max)
   - setSnapInterval(double)
   - onChange(Consumer<Double>)
```

## Theme System Requirements
```
1. Color Palette (Theme.java)
// Primary colors
COBALT_PRIMARY = #0066FF
COBALT_HOVER = #3385FF
COBALT_PRESSED = #0052CC

// Background colors
BG_DARK = #1A1A1A
BG_MEDIUM = #2D2D2D
BG_LIGHT = #404040

// Text colors
TEXT_PRIMARY = #FFFFFF
TEXT_SECONDARY = #CCCCCC
TEXT_DISABLED = #808080

// State colors
SUCCESS = #00FF00
WARNING = #FFAA00
ERROR = #FF0000
INFO = #00AAFF

// Transparency levels
ALPHA_FULL = 255
ALPHA_HIGH = 204 (80%)
ALPHA_MEDIUM = 153 (60%)
ALPHA_LOW = 102 (40%)

2. Typography System
// Font sizes
TITLE = 16px (bold)
SUBTITLE = 14px (medium)
BODY = 12px (regular)
CAPTION = 10px (light)

// Font weights
BOLD = Minecraft bold rendering
MEDIUM = Normal + slight spacing
REGULAR = Default Minecraft font
LIGHT = Reduced opacity (90%)

3. Spacing System
// Standard spacing units (pixels)
SPACE_XS = 4
SPACE_SM = 8
SPACE_MD = 12
SPACE_LG = 16
SPACE_XL = 24

// Border radius
RADIUS_SM = 4
RADIUS_MD = 6
RADIUS_LG = 8

4. Shadow Styles
// Drop shadows (offset-x, offset-y, blur, color)
SHADOW_SM = (0, 1, 2, rgba(0,0,0,0.2))
SHADOW_MD = (0, 2, 4, rgba(0,0,0,0.3))
SHADOW_LG = (0, 4, 8, rgba(0,0,0,0.4))
```

## Animation Framework Requirements
```
1. Animator Class
Features:
- Keyframe-based animation system
- Multiple easing functions
- Chainable animations
- Pause/resume/stop controls

Easing Functions:
- linear
- easeInQuad, easeOutQuad, easeInOutQuad
- easeInCubic, easeOutCubic, easeInOutCubic
- easeInElastic, easeOutElastic

Methods:
- animate(from, to, duration, easing, onUpdate)
- chain(animation)
- delay(ms)
- repeat(count)
- stop()

2. Animation Examples
// Button hover animation
Animator.animate(currentColor, hoverColor, 300, easeInOut, color -> {
    this.backgroundColor = color;
});

// Scale animation
Animator.animate(1.0, 0.98, 100, easeInQuad, scale -> {
    this.scale = scale;
});

// Height expansion
Animator.animate(collapsedHeight, expandedHeight, 150, easeOutCubic, height -> {
    this.currentHeight = height;
});

3. Performance Optimization
- Max 60 FPS animation updates (16.67ms per frame)
- Batch animation updates (update all at once per frame)
- Skip animations if <1px change
- Cancel conflicting animations (e.g., two animations on same property)
```

## Micro-Interactions Requirements
```
1. Button Press
- Visual: Scale down 2% + darken background
- Audio: Soft click sound (optional)
- Duration: 100ms press, 150ms release

2. Toggle Switch
- Visual: Handle slides + background color fades
- Audio: Toggle click sound
- Duration: 200ms handle, 250ms background

3. Slider Drag
- Visual: Handle scales up + track fills
- Audio: None
- Duration: Instant response to drag

4. Card Expand
- Visual: Height expands + arrow rotates + content fades in
- Audio: Soft whoosh (optional)
- Duration: 150ms height, 200ms fade (staggered by 50ms)

5. Scrollbar Hover
- Visual: Fade in + brighten color
- Duration: 200ms fade in, 300ms fade out after idle
```

## Integration Requirements
```
1. Global UI Manager
- Manages all UI components
- Handles rendering order (z-index)
- Dispatches input events
- Applies theme globally

2. Rendering Pipeline
- Pre-render: Update animations
- Render: Draw all components back-to-front
- Post-render: Cleanup, cache invalidation

3. Input Handling
- Mouse: Click, hover, drag, scroll
- Keyboard: Focus navigation (Tab key)
- Gamepad: Optional (future enhancement)

4. Testing Checklist
- Test components render at different GUI scales
- Test animations are smooth (60 FPS)
- Test hover states activate/deactivate correctly
- Test buttons respond to clicks
- Test scrolling is smooth
- Test toggles animate properly
- Measure UI render time (<1ms per frame)
```

## Definition of Done
- All UI components implemented (RoundedPanel, AnimatedButton, ModuleCard, ScrollView, Toggle, Slider)
- Theme system defined with color palette, typography, spacing
- Animation framework implemented with easing functions
- Micro-interactions working for all interactive elements
- All animations smooth at 60 FPS
- UI render time <1ms per frame
- Components work at different Minecraft GUI scales
- Visual testing completed (screenshots of all states)
- Performance profiling completed

## Affected Repos
- vesper-client

## Parallelizable Tasks
- Individual components can be developed in parallel after Theme system is defined
- Animation framework can be developed independently

## Sequential Constraints
1. Theme system must be defined first
2. Animation framework should be completed early
3. Base components (RoundedPanel, AnimatedButton) before complex components (ModuleCard, ScrollView)
4. All other client features (REQUEST-004, REQUEST-005) should wait for this to complete

## Notes
- This is the foundation for all client UI
- Should be completed before other client features for consistency
- Reference: Modern UI libraries (Material Design, Fluent UI)
- Performance is critical (no lag on low-end systems)
- Future: Custom themes, user-created color schemes

---
Approval:
   Status: APPROVED
  AutoCommit: true
  CommitMessage: "REQUEST-006: UI framework Phase 1 polish"
  Scope:
    - vesper-client

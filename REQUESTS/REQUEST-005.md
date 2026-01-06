# REQUEST-005: QoL Features Suite (Slot Locking + Custom Health Bar)

## Goal
Implement quality-of-life features: hotbar slot locking and customizable health bar overlay (inspired by Skyblocker mod).

## Scope
- vesper-client only
- Slot locking system (prevent accidental item movement)
- Custom health bar HUD overlay (numeric, vanilla-style, or both)
- Draggable repositioning for HUD elements
- Configurable colors and thresholds

## Constraints
- Must integrate with existing HUD rendering system
- Must not conflict with Minecraft vanilla UI
- Settings must persist across sessions
- Must be toggleable independently

## vesper-client AI Instructions
```
CREATE: src/main/java/modules/SlotLockingModule.java
CREATE: src/main/java/modules/CustomHealthBarModule.java

---

SlotLockingModule Requirements:

1. Module Configuration
   Fields:
   - lockedSlots: Set<Integer> (slot indices 0-8 for hotbar)
   - enabled: Boolean
   - lockIconColor: Color (default: yellow)
   - preventDropping: Boolean (also prevent Q-dropping, default: true)

2. Slot Locking Logic
   a. Slot state tracking:
      - Store locked slots in Set<Integer>
      - Persist to config file on change
      - Restore on client startup
   
   b. Lock/unlock interaction:
      - Detect right-click on hotbar slot (in inventory GUI)
      - Toggle lock state for clicked slot
      - Play sound feedback (click sound)
      - Update lock icon visibility
   
   c. Item movement prevention:
      - Hook inventory click event
      - If source or destination slot is locked:
        * Cancel event (prevent item movement)
        * Show chat message: "Slot is locked"
        * Play error sound (optional)
      - If preventDropping enabled and locked slot:
        * Cancel drop item event (Q key)

3. Visual Rendering
   - Draw lock icon over locked slots:
     * Icon: Small padlock symbol (8x8 pixels)
     * Position: Top-right corner of slot
     * Color: Configurable (default: yellow with transparency)
     * Render in both inventory and HUD views
   - Hover effect:
     * Brighten icon when hovering over locked slot
     * Show tooltip: "Right-click to unlock"

4. UI Component (ModuleCard)
   Fields:
   - "Enable Slot Locking" Toggle
   - Visual hotbar display showing lock state of each slot
   - "Lock Icon Color" Color picker
   - "Prevent Dropping" Checkbox
   - "Lock All" button (lock all hotbar slots at once)
   - "Unlock All" button

5. Persistence
   - Save locked slots to config.json:
     ```json
     {
       "slotLocking": {
         "enabled": true,
         "lockedSlots": [0, 1, 8],
         "lockIconColor": "#FFFF00AA",
         "preventDropping": true
       }
     }
     ```

6. Edge Cases
   - Locked slot contains no item: Still locked (prevents accidental placement)
   - Player dies: Locked slots remain locked on respawn
   - Switching profiles: Load profile-specific locked slots

---

CustomHealthBarModule Requirements:

1. Module Configuration
   Fields:
   - enabled: Boolean
   - position: Point (x, y coordinates, default: center-top)
   - scale: Double (0.5x - 2.0x, default 1.0)
   - style: Enum (NUMERIC, VANILLA, BOTH)
   - showAbsorption: Boolean (blue hearts, default: true)
   - colorThresholds: List<Threshold> (health %, color)
     * Default: <30% red, 30-60% yellow, >60% green
   - backgroundColor: Color (semi-transparent black, default)
   - showMaxHealth: Boolean (display "20/20" vs "20", default: true)

2. Health Display Styles
   a. NUMERIC style:
      - Text: "❤ 20/20" or "20 HP"
      - Font: Minecraft default, configurable size
      - Color: Based on health percentage and thresholds
      - Shadow: Optional text shadow for readability
   
   b. VANILLA style:
      - Heart icons (same as Minecraft vanilla)
      - Half-heart support (damaged hearts)
      - Absorption hearts (blue, above normal hearts)
      - Shake effect on damage (brief animation)
   
   c. BOTH style:
      - Numeric display above heart icons
      - Combined layout with proper spacing

3. Rendering Logic (called every frame)
   a. Get player health data:
      - Current health: player.getHealth()
      - Max health: player.getMaxHealth()
      - Absorption: player.getAbsorptionAmount()
   
   b. Calculate display values:
      - Health percentage: (current / max) * 100
      - Determine color based on thresholds
      - Format text: "❤ 18.5/20" or "18.5 HP"
   
   c. Render at configured position:
      - Apply scale transformation
      - Draw background (rounded rectangle, semi-transparent)
      - Draw health display (numeric and/or icons)
      - Draw absorption hearts if enabled

4. Repositioning (Drag Mode)
   - Hold Shift + Left-click and drag to reposition
   - Show dashed outline while dragging
   - Snap to grid (optional, 10px grid)
   - Save position on release

5. Threshold Configuration
   - Threshold structure: { percentage: Int, color: Color }
   - Default thresholds:
     * 0-30%: #FF0000 (red)
     * 30-60%: #FFFF00 (yellow)
     * 60-100%: #00FF00 (green)
   - UI: Color picker + percentage slider for each threshold

6. UI Component (ModuleCard)
   Fields:
   - "Enable Custom Health Bar" Toggle
   - "Style" Dropdown (Numeric, Vanilla, Both)
   - "Position" Button: "Click to Reposition" (enters drag mode)
   - "Scale" Slider (0.5x - 2.0x, step 0.1)
   - "Show Absorption" Checkbox
   - "Show Max Health" Checkbox
   - "Thresholds" Expandable section:
     * List of threshold editors (% + color picker)
     * "Add Threshold" button
     * "Reset to Defaults" button
   - Live preview (shows current health in configured style)

7. Animations
   - Damage animation: Flash red and shake for 200ms
   - Heal animation: Pulse green for 150ms
   - Smooth health change: Interpolate over 300ms (not instant)
   - Absorption gain: Blue glow effect

8. Performance
   - Cache rendered text (don't re-render every frame if health unchanged)
   - Use VBO for heart icons (batch rendering)
   - Only update when health changes or every 20 ticks (1 second)

9. Integration with Minecraft UI
   - Option to hide vanilla health bar when custom bar enabled
   - Respect Minecraft GUI scale setting
   - Don't render when chat open or in F1 mode

10. Accessibility
    - High contrast mode (bold text, thicker borders)
    - Colorblind mode (use patterns instead of colors)
    - Font size override (for vision-impaired users)

11. Persistence
    - Save all settings to config.json
    - Include position, scale, style, thresholds, colors

12. Testing Checklist
    - Test damage (health decreases, color changes)
    - Test healing (health increases, animation plays)
    - Test absorption (blue hearts appear above normal)
    - Test max health changes (drinking potions, losing hearts)
    - Test repositioning (drag and snap)
    - Test style switching (numeric vs vanilla vs both)
    - Test threshold colors (verify correct color at each %)
```

## Definition of Done
- SlotLockingModule implemented with all features
- CustomHealthBarModule implemented with all styles
- Both modules have UI components (ModuleCards)
- Settings persist across client restarts
- Health bar repositioning works smoothly
- Threshold colors apply correctly
- Slot locking prevents item movement as expected
- Animations are smooth and performant
- No conflicts with vanilla Minecraft UI
- Manual testing completed on Hypixel SkyBlock

## Affected Repos
- vesper-client

## Parallelizable Tasks
- SlotLockingModule and CustomHealthBarModule can be developed independently
- Can be developed in parallel with Mining Macro (REQUEST-004)
- Can be developed in parallel with auth system (REQUEST-003)

## Sequential Constraints
1. UI framework (REQUEST-006) should be implemented first for consistent styling
2. Both modules can be developed simultaneously after UI framework is ready

## Notes
- Skyblocker reference: https://github.com/SkyblockerMod/Skyblocker
- Health bar should be lightweight (no performance impact)
- Slot locking is a highly requested feature for SkyBlock players
- Future enhancement: Lock chest slots, lock specific items by type

---
Approval:
   Status: APPROVED
  AutoCommit: true
  CommitMessage: "REQUEST-005: QoL features (slot locking + health bar)"
  Scope:
    - vesper-client

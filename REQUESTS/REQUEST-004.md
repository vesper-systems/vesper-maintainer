# REQUEST-004: Mining Macro Module (MVP)

## Goal
Implement a basic automated mining module with configurable block IDs, humanization, and safety features for Minecraft Hypixel SkyBlock.

## Scope
- vesper-client only
- Basic block targeting and auto-mining
- Configurable block ID whitelist
- Humanization (random delays, aim variation)
- Safety features (PvP detection, player proximity)

## Constraints
- Must integrate with existing module system
- Must use humanization framework from BehaviorMonitor
- Must respect multiplayer safety rules (disable when players nearby)
- No perfect timing patterns (must appear human-like)
- Must be toggleable via UI

## vesper-client AI Instructions
```
CREATE: src/main/java/modules/MiningMacroModule.java

Requirements:

1. Module Configuration (Settings)
   Fields:
   - blockIds: List<Integer> (comma-separated input from UI)
   - enabled: Boolean (toggle on/off)
   - range: Double (scan radius, default 4.5 blocks, range 3.0-6.0)
   - humanizationDelayMin: Int (minimum delay between actions, default 50ms)
   - humanizationDelayMax: Int (maximum delay, default 200ms)
   - aimOffsetDegrees: Double (random aim offset, default ±2 degrees)
   - pauseEveryNBlocks: Int (micro-pause frequency, default 5-10)
   - pauseDurationMs: Int (micro-pause length, default 100-300ms)

2. Core Mining Logic (onTick method, called every game tick)
   a. Safety checks (exit early if any fail):
      - If !enabled: return
      - If player in combat: return
      - If player in PvP zone: return
      - If other players within 20 blocks: return
      - If inventory full: return (optional: auto-drop junk)
   
   b. Block scanning:
      - Scan all blocks in sphere of radius <range>
      - Filter for blocks matching blockIds list
      - Sort by distance (nearest first)
      - Select nearest valid block
   
   c. Block targeting:
      - Rotate camera to target block
      - Add random offset: ±<aimOffsetDegrees> on yaw and pitch
      - Smooth rotation over 2-4 ticks (easeOutCubic)
   
   d. Block breaking:
      - Start left-click (attack) when aimed at block
      - Hold until block breaks
      - Add humanized delay: random(<humanizationDelayMin>, <humanizationDelayMax>)
      - Increment block counter
   
   e. Micro-pauses:
      - Every <pauseEveryNBlocks> blocks, pause for <pauseDurationMs>
      - Randomize pause duration ±20%
      - During pause: stop all actions, look around slightly

3. Humanization Integration
   - Use BehaviorMonitor.getDelay() for action delays
   - Use BehaviorMonitor.getAimOffset() for rotation randomness
   - Track action timestamps to avoid perfect patterns
   - Add occasional "missed" blocks (rotate past target, then correct)
   - Vary rotation speed (sometimes fast, sometimes slow)

4. Safety Features
   - PvP detection: Check if player in PvP-enabled zone (scoreboard or world name)
   - Player proximity: Scan for players within 20 blocks (use world.getPlayers())
   - Combat detection: Check if player has damage ticks or combat timer
   - Auto-disable on detection, show notification: "Mining paused (safety)"

5. UI Component (ModuleCard in settings panel)
   Fields:
   - "Block IDs" TextArea (comma-separated, e.g., "1,4,14,15,16,56")
     * Tooltip: "Minecraft block IDs to mine (Stone=1, Coal=16, etc.)"
   - "Range" Slider (3.0 - 6.0 blocks, step 0.5)
   - "Humanization Delay" RangeSlider (min/max, 50-300ms)
   - "Aim Variance" Slider (0.5 - 5.0 degrees, default 2.0)
   - "Micro-pause Frequency" Slider (3-15 blocks, default 7)
   - "Enable" Toggle button (green = on, red = off)
   
   Actions:
   - All settings auto-save on change
   - "Test Target" button: Highlights nearest matching block for 5 seconds
   - "Reset to Defaults" button

6. Error Handling
   - Invalid block IDs: Show warning, filter out invalid entries
   - No blocks in range: Idle (don't spam actions)
   - Camera rotation fails: Log warning, retry next tick
   - Exception in onTick: Catch, log, disable module

7. Performance Optimization
   - Cache block scan results for 3 ticks (reduce CPU)
   - Skip processing if player not moving and no blocks nearby
   - Use spatial hashing for player proximity checks

8. Telemetry (Opt-in)
   - Track: blocks mined per session, average time per block
   - Track: safety auto-disable events
   - Never log: exact block locations or player names

9. Testing Checklist
   - Test with single block type (e.g., only stone)
   - Test with multiple block types
   - Test safety features (approach another player → should pause)
   - Test humanization (record timing, verify variance)
   - Test inventory full scenario
   - Test rapid enable/disable (should not crash)

10. Dependencies
    - Minecraft client API (BlockPos, PlayerEntity, World)
    - BehaviorMonitor (humanization framework)
    - ModuleManager (registration and lifecycle)
```

## Definition of Done
- MiningMacroModule class created with all specified features
- UI component (ModuleCard) implemented with all settings
- Safety features tested and working (PvP, players, combat)
- Humanization verified (timing variance, aim randomness)
- Block scanning and targeting working correctly
- Micro-pauses implemented and randomized
- Error handling covers all edge cases
- Performance optimized (no lag spikes)
- Telemetry respects opt-in flag
- Manual testing completed on Hypixel SkyBlock

## Affected Repos
- vesper-client

## Parallelizable Tasks
- Can be developed independently of auth system (REQUEST-003)
- Can be developed in parallel with QoL features (REQUEST-005)
- Depends on UI framework (REQUEST-006) for styling

## Sequential Constraints
1. UI framework (REQUEST-006) should be implemented first for consistent styling
2. Mining macro can be developed in parallel with auth integration

## Notes
- Block IDs reference: https://minecraft.fandom.com/wiki/Java_Edition_data_values
- Common SkyBlock ores: Coal=16, Iron=15, Gold=14, Diamond=56, Emerald=129
- Humanization is critical to avoid detection
- Future enhancement: Route optimization, ore priority, auto-sell

---
Approval:
   Status: APPROVED
  AutoCommit: true
  CommitMessage: "REQUEST-004: Basic mining macro module"
  Scope:
    - vesper-client

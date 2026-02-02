# Hold or Toggle - Modular Architecture

## Overview

Modular architecture with pluggable stance systems and isolated features. Easy to extend and maintain.

## Structure

```
holdortoggle/
├── core.script              # Event dispatcher, loads modules
├── config.script            # MCM menu, option handling
├── state.script             # Shared state variables
├── utils.script             # holdfix(), is_modifier_key()
├── stance_utils.script      # Stance-related utilities
├── movement_base.script     # Basic crouch/walk/sprint
├── features/
│   ├── lean.script          # Lean hold/toggle/opposite cancel
│   ├── inventory.script     # Inventory/PDA on release
│   ├── autowalk.script      # Auto-walk feature
│   └── qaw.script           # Quick Action Wheel integration
└── stance_systems/
    ├── vanilla_prone.script      # Vanilla behavior (crouch+walk=prone)
    ├── fluid_stance.script       # Smooth crouch↔walk↔prone transitions
    └── progressive_stance.script # Tap=crouch, Hold=prone
```

## Core Files

**core.script** - Coordinates all modules, dispatches events (key press/hold/release)

**config.script** - MCM menu definition, stance system selection, ensures game toggles enabled

**state.script** - Pure state storage (no logic), prevents circular dependencies

**movement_base.script** - Handles crouch/walk/sprint when stance system inactive

## Features

**lean.script** - Lean hold/toggle with opposite lean cancel (in Hold+Toggle mode, held lean allows switching)

**inventory.script** - Opens inventory/PDA on key release (allows holding for other actions)

**autowalk.script** - Auto-walk toggle with press/double-tap/hold modes

**qaw.script** - Quick Action Wheel integration (modifier bypass on hold, close with reload key)

## Stance Systems

**vanilla_prone** - Vanilla behavior: crouch+walk=prone

**fluid_stance** - Smooth transitions, dedicated prone key, no crouch+walk=prone

**progressive_stance** - Tap crouch=immediate crouch, hold crouch=transition to prone

### Stance System Interface

All stance systems implement:
```lua
function initialize()
function shutdown()
function on_key_press(key) -- Returns true if handled
function on_key_hold(key)
function on_key_release(key)
function is_stance_active()
function is_handling_crouch()
function is_handling_walk()
function set_pronekey(key)
function set_pronekey_mode(mode) -- 0=Disabled, 1=Hold, 2=Toggle, 3=Hold+Toggle
```

## Event Flow

Key events → `core` → stance system (if active) → fallback to movement_base/features

Priority: stance system > inventory > lean > movement_base

## QoL Features

- **Alternative Sprint Cancel** - Sprint only cancels actions when moving forward (LSHIFT can be used for modifiers)
- **Opposite Lean Cancel** - In Toggle/Hold+Toggle mode, opposite lean key cancels current lean. In Hold+Toggle mode, held lean allows switching instead.
- **Inventory on Release** - Quick tap opens inventory, hold for other actions (e.g., Quick Action Wheel)
- **PDA on Release** - Quick tap opens PDA, hold for other actions (e.g., detector via Key Wrapper). Saves current weapon/detector state and restores it when closing.
- **Prevent Weapon with Detector** - When switching from two-handed weapon/item to detector, go to empty hands first.
- **QAW: Ignore Modifiers on Hold** - Long-press QAW key works even while holding modifiers (allows QAW while sprinting).
- **QAW: Close with Reload Key** - Reload key (R) closes QAW when open. Useful if you bind tap R to reload and hold R to ammo selector in Key Wrapper.

## Adding New Features

1. Create `features/your_feature.script` with standard interface
2. Add module reference in `core.script`
3. Call in appropriate event handlers (`on_key_press`, etc.)
4. Add MCM options in `config.script`
5. Call `your_feature.update_options()` in `config.on_option_change()`

## Performance

Module overhead: <0.01ms per keypress (unnoticeable). Maintainability gains outweigh microscopic performance cost.

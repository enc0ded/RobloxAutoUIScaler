# Roblox Universal UI Scaler

A drop-in solution for automatic UI scaling across all devices in Roblox. Design your UI once at a reference resolution, and it scales perfectly on mobile, tablet, and desktop.

## Features

- ✅ Automatic scaling across all screen sizes
- ✅ Consistent text sizing regardless of string length
- ✅ Zero configuration for new GUIs
- ✅ Works with nested ScreenGuis
- ✅ One-time setup with cleanup utility

## Installation

### 1. Install the Auto-Scaler

1. Copy the code from [`UniversalUIScaler.lua`](UniversalUIScaler.lua)
2. Create a **LocalScript** in `StarterPlayer > StarterPlayerScripts`
3. Name it `UniversalUIScaler`
4. Paste the code

### 2. Convert Existing UI (Optional)

If you have existing UI:

1. Copy the code from [`UICleanup.lua`](UICleanup.lua)
2. Create a **Script** (not LocalScript) in `ServerScriptService`
3. Press Play to run the conversion
4. Delete the script after it completes

## How It Works

The system scales all UI based on viewport width:

```
Scale = ViewportWidth / ReferenceWidth (default: 1920)
```

**Examples:**
- Mobile (414px): Scale = 0.22
- Desktop (1920px): Scale = 1.0
- 4K (3840px): Scale = 2.0

All UI elements scale proportionally while maintaining visual consistency.

## Design Rules

### ✅ Use Scale (not Offset)

```lua
-- ✅ CORRECT
element.Size = UDim2.new(0.3, 0, 0.15, 0)
element.Position = UDim2.new(0.5, 0, 0.5, 0)

-- ❌ WRONG
element.Size = UDim2.new(0, 400, 0, 200)
element.Position = UDim2.new(0, 960, 0, 540)
```

### ✅ Use Fixed TextSize (not TextScaled)

```lua
-- ✅ CORRECT
element.TextScaled = false
element.TextSize = 24  -- Body text
element.TextSize = 36  -- Headers
element.TextSize = 48  -- Large headers

-- ❌ WRONG
element.TextScaled = true  -- Makes text inconsistent
```

### Studio Editor Setup

When using the Properties panel:

| Property | Setting |
|----------|---------|
| Size → X.Scale | `0.0` - `1.0` |
| Size → X.Offset | `0` |
| Size → Y.Scale | `0.0` - `1.0` |
| Size → Y.Offset | `0` |
| Position → X.Scale | `0.0` - `1.0` |
| Position → X.Offset | `0` |
| Position → Y.Scale | `0.0` - `1.0` |
| Position → Y.Offset | `0` |
| TextScaled | ☐ Unchecked |
| TextSize | `24`, `36`, or `48` |

## Examples

### Creating a Button

```lua
local button = Instance.new("TextButton")
button.Parent = screenGui

-- Size and position using Scale
button.Size = UDim2.new(0.3, 0, 0.08, 0)
button.Position = UDim2.new(0.5, 0, 0.5, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)

-- Text properties
button.Text = "Click Me"
button.TextSize = 24
button.TextScaled = false

-- Visual properties
button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
button.BorderSizePixel = 0
```

### Creating a Centered Frame

```lua
local frame = Instance.new("Frame")
frame.Parent = screenGui
frame.Size = UDim2.new(0.4, 0, 0.6, 0)
frame.Position = UDim2.new(0.5, 0, 0.5, 0)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
```

### Creating a Title Label

```lua
local title = Instance.new("TextLabel")
title.Parent = screenGui
title.Size = UDim2.new(0.8, 0, 0.1, 0)
title.Position = UDim2.new(0.5, 0, 0.05, 0)
title.AnchorPoint = Vector2.new(0.5, 0)

title.Text = "Game Title"
title.TextSize = 48  -- Larger for titles
title.TextScaled = false
title.BackgroundTransparency = 1
```

## Testing

1. Set Studio viewport to 1920x1080 for design
2. Press Play (F5) to test
3. While playing, test different device emulations
4. Verify on smallest target device (iPhone)

## Configuration

Edit the auto-scaler script to adjust settings:

```lua
-- In UniversalUIScaler.lua
local REFERENCE_WIDTH = 1920   -- Design resolution width
local REFERENCE_HEIGHT = 1080  -- Design resolution height
local MIN_SCALE = 0.3          -- Minimum scale limit
local MAX_SCALE = 2.0          -- Maximum scale limit
local USE_WIDTH_SCALING = true -- Scale based on width vs height
```

## Troubleshooting

### UI Not Scaling

**Check:**
- Script is in `StarterPlayerScripts`
- Script is a `LocalScript` (not Script)
- Testing in Play mode (not edit mode)

**Expected console output:**
```
✓ Universal UI Auto-Scaler loaded!
  Reference Resolution: 1920x1080
  Current Scale: X.XX
  Tracking X ScreenGuis (including nested)
```

### Text Different Sizes

**Cause:** TextScaled enabled or inconsistent TextSize values

**Fix:** Ensure `TextScaled = false` and use consistent TextSize (24, 36, or 48)

### Elements Cut Off

**Cause:** Using Offset instead of Scale

**Fix:** Convert to Scale values: `UDim2.new(scale, 0, scale, 0)`

## Common Mistakes

| ❌ Mistake | ✅ Solution |
|-----------|-----------|
| Using Offset values | Use Scale only: `UDim2.new(0.3, 0, 0.1, 0)` |
| TextScaled = true | TextScaled = false + TextSize = 24 |
| Random TextSize values | Standardize: 24, 36, or 48 |
| Adding manual UIScale | Let auto-scaler handle it |
| Not testing on mobile | Always test on smallest device |

## Workflow Summary

1. **Setup once:** Install `UniversalUIScaler` in `StarterPlayerScripts`
2. **Design:** Create UI at 1920x1080 using Scale values
3. **Text:** Use fixed TextSize (24/36/48), never TextScaled
4. **Test:** Press Play and verify on multiple devices
5. **Deploy:** Works automatically for all players

## Requirements

- Roblox Studio
- LocalScript support (enabled by default)

## License

MIT License - See [LICENSE](LICENSE) file for details

## Contributing

Contributions welcome! Please open an issue or submit a pull request.

## Credits

Developed for the Roblox development community to simplify cross-device UI development.

---

**Quick Start:** Install → Design with Scale → Fixed TextSize → It just works™

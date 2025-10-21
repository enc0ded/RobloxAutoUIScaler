# Roblox Auto UI Scaler

An automatic UI scaling system for Roblox games. Converts existing UIs to scale correctly across all screen sizes and resolutions, then maintains that scaling for any new UI you create.

## Problem

Roblox GUIs often break on different devices. A UI designed on desktop looks wrong on mobile. Text sizes are inconsistent. Elements use pixel offsets that don't scale. You end up with either:
- Multiple versions of the same UI for different devices
- TextScaled causing inconsistent text sizes
- Manual scaling code scattered throughout your project

## Solution

This system uses two scripts:

**UICleanup** - Run once to convert your existing project. Finds every UI element in your game (StarterGui, ReplicatedStorage, ServerStorage) and converts it to use scale-based values. Handles Size, Position, Padding, Layout properties, and text sizing. Can also convert UI creation code in scripts you specify.

**UniversalUIScaler** - Install permanently. Automatically applies a UIScale multiplier to all ScreenGuis based on the player's screen size. Runs on every player's device with zero performance impact.

The result: You design UI once at a reference resolution (default 1920x1080), and it displays correctly on every device.

## Installation

### Step 1: Convert Your Existing UI

1. Copy `UICleanup.lua` into ServerScriptService as a Script
2. Configure the settings at the top of the file if needed
3. Press Play in Studio
4. Check the Output window for a conversion report
5. Stop Play and delete the UICleanup script

The cleanup script converts:
- All Size and Position values from Offset to Scale
- UIPadding values to scale-based
- UIListLayout, UIGridLayout, ScrollingFrame properties
- TextScaled from true to false
- TextSize values to standardized sizes
- Optionally: UI creation code in specified scripts

### Step 2: Install the Auto-Scaler

1. Copy `UniversalUIScaler.lua` into StarterPlayer > StarterPlayerScripts as a LocalScript
2. Configure reference resolution if needed (default: 1920x1080)

The auto-scaler applies a UIScale to every ScreenGui based on screen width. On mobile (414px), it scales down to 0.22x. On desktop (1920px), it displays at 1.0x. On 4K (3840px), it scales up to 2.0x.

## Usage

### Designing New UI in Studio

Set your viewport to your reference resolution (1920x1080 by default). When creating UI elements, follow these rules:

**In the Properties panel:**

For Size and Position, use only the Scale values (first number in each axis). Set Offset values to 0.

Example Size property:
```
X: [Scale: 0.3] [Offset: 0]
Y: [Scale: 0.15] [Offset: 0]
```

This makes the element 30% of its parent's width and 15% of its parent's height.

For text elements, uncheck TextScaled and set TextSize to a fixed value (recommended: 18, 24, 36, or 48).

For any element with UIPadding, use Scale values, not Offset.

### Why This Approach

**Scale vs Offset**: Offset uses pixels, which are fixed regardless of screen size. Scale uses percentages of parent size, which adapt automatically. A Size of `UDim2.new(0.5, 0, 0.5, 0)` means "50% of parent width and height" on any device.

**Fixed TextSize vs TextScaled**: TextScaled makes text fit its container, which means different length strings display at different sizes. Fixed TextSize keeps all text at consistent sizes relative to each other. The UIScale multiplier then scales everything proportionally.

**Reference Resolution**: You design at 1920x1080 (or your chosen resolution). The auto-scaler calculates a multiplier for other resolutions. This multiplier applies to everything - element sizes, positions, padding, and text.

## Configuration

### UniversalUIScaler Settings

Located at the top of the script:

- `REFERENCE_WIDTH` - The width you design at (default: 1920)
- `REFERENCE_HEIGHT` - The height you design at (default: 1080)
- `USE_WIDTH_SCALING` - Scale based on width vs height (default: true)
- `MIN_SCALE` - Minimum scale limit (default: 0.3)
- `MAX_SCALE` - Maximum scale limit (default: 2.0)

### UICleanup Settings

Located at the top of the script:

- `REFERENCE_WIDTH/HEIGHT` - Must match your scaler settings (default: 1920x1080)
- `STANDARD_FONT_SIZES` - Array of font sizes to standardize to (default: 14, 18, 24, 30, 36, 48, 60, 72)
- `STANDARDIZE_TO_CLOSEST` - Round font sizes to nearest standard size (default: true)
- `USE_KEYWORD_OVERRIDE` - Use element names to determine text size (default: true)
- `DRY_RUN` - Preview changes without applying them (default: false)
- `CONVERT_SCRIPTS` - Enable script source code conversion (default: true)
- `SCRIPTS_TO_CONVERT` - Array of script paths to convert (default: empty)

### Script Conversion

The cleanup script can convert UI creation code in your scripts. Add script paths to the `SCRIPTS_TO_CONVERT` array:

```lua
local SCRIPTS_TO_CONVERT = {
    "ReplicatedStorage.UIScripts.NotificationManager",
    "StarterPlayer.StarterPlayerScripts.ShopClient",
}
```

The script converter finds patterns like `UDim2.new(0, 400, 0, 200)` and converts them to scale-based values like `UDim2.new(0.208333, 0, 0.185185, 0)`. It also converts `TextScaled = true` to `false` and standardizes TextSize values.

## How Scaling Works

The system uses a simple calculation:

```
Scale = ViewportWidth / ReferenceWidth
```

If you design at 1920px width:
- iPhone (414px): Scale = 0.22
- iPad (768px): Scale = 0.4
- Desktop 1080p (1920px): Scale = 1.0
- 4K Monitor (3840px): Scale = 2.0

This multiplier applies to everything. A button with Size `UDim2.new(0.3, 0, 0.08, 0)` and TextSize 24:
- iPhone: 30% of 414px = 124px wide, text at 5.3px (24 * 0.22)
- Desktop: 30% of 1920px = 576px wide, text at 24px (24 * 1.0)
- 4K: 30% of 3840px = 1152px wide, text at 48px (24 * 2.0)

Everything maintains the same proportions relative to screen size.

## What Gets Converted

The cleanup script handles all properties that affect scaling:

**UI Elements:**
- Size (UDim2)
- Position (UDim2)
- UIPadding (UDim for all four sides)
- TextScaled (disabled)
- TextSize (standardized to preset values)

**Layout Objects:**
- UIListLayout.Padding (UDim)
- UIGridLayout.CellPadding (UDim2)
- UIGridLayout.CellSize (UDim2)
- ScrollingFrame.CanvasSize (UDim2)

**Constraints:**
- UITextSizeConstraint (adjusted to allow target size)
- UISizeConstraint (removed - doesn't scale properly)
- Conflicting UIScale objects (removed)

**Scripts (Optional):**
- UDim2.new patterns with offsets
- UDim.new patterns with offsets
- TextScaled assignments
- TextSize assignments

## Advanced Features

### Excluding GUIs from Auto-Scaling

Set the `DisableAutoScale` attribute to true on any ScreenGui:

```lua
screenGui:SetAttribute("DisableAutoScale", true)
```

### Excluding GUIs from Cleanup

Set the `SkipCleanup` attribute to true on any ScreenGui before running the cleanup script.

### Accessing Current Scale

Other scripts can read the current scale value:

```lua
local scaler = game.StarterPlayer.StarterPlayerScripts.UniversalUIScaler
local currentScale = scaler:GetAttribute("CurrentScale")
```

### Preview Mode

Set `DRY_RUN = true` in the cleanup script to see what would change without actually changing anything.

## Troubleshooting

**Auto-scaler not working:**
- Verify the script is in StarterPlayerScripts
- Verify it's a LocalScript, not a Script
- Check the Output window for "Universal UI Auto-Scaler loaded!"
- Make sure you're testing in Play mode, not edit mode

**Text appears at different sizes:**
- Verify TextScaled is false on all text elements
- Verify all text elements use the same TextSize value (e.g., all body text uses 24)
- Check if cleanup script ran successfully

**Elements cut off or wrong size:**
- Verify Size and Position use Scale, not Offset
- Check that X.Offset and Y.Offset are 0 in Properties
- Make sure reference resolution matches between cleanup and scaler scripts

**Cleanup reports zero-size elements:**
- Some elements weren't rendered yet (AbsoluteSize = 0)
- Run the cleanup script again after those GUIs are visible
- Or convert those elements manually

## Requirements

- Roblox Studio
- Game with existing UI or new UI to create
- Understanding of Roblox UI basics (Size, Position, UDim2)

## Limitations

- UISizeConstraint with pixel-based min/max are removed (they don't scale well)
- Elements with AbsoluteSize of 0 cannot be converted automatically
- Script conversion uses pattern matching and may not catch all cases
- Designed for UI elements - does not handle 3D space calculations

## License

MIT License - see LICENSE file for details.

## Contributing

Contributions welcome. Please test thoroughly across multiple device types before submitting pull requests.

## Support

Report issues on the GitHub repository issue tracker.

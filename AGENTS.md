# AGENTS.md - ZMK Sofle Keyboard Firmware

This guide is for agentic coding assistants working in this ZMK firmware repository.

## Project Overview

**Type:** ZMK (Zephyr Mechanical Keyboard) firmware configuration for Eyelash Sofle split keyboard  
**Hardware:** Nordic nRF52840, wireless split keyboard with OLED displays, RGB underglow, rotary encoder  
**Key Features:** Bluetooth LE, mouse emulation, ZMK Studio support, soft-off mode, Swedish character macros

## Build Commands

### Standard Build (GitHub Actions - Recommended)
```bash
# Push changes to trigger automated build
git push

# Watch build progress
gh run watch

# Download compiled firmware (.uf2 files) to firmware/ directory
# IMPORTANT: Always use --dir ./firmware to keep downloads organized
gh run download --dir ./firmware
```

### Local Build (Advanced)
ZMK uses Zephyr's west build system. For local builds:
```bash
# Setup (first time only)
west init -l config/
west update
west zephyr-export

# Build specific board/shield
west build -d build/left -b eyelash_sofle_left -- -DSHIELD=nice_view
west build -d build/right -b eyelash_sofle_right -- -DSHIELD=nice_view

# Build with ZMK Studio support (left side only)
west build -d build/studio -b eyelash_sofle_left -- -DSHIELD=nice_view \
  -DCONFIG_ZMK_STUDIO=y -DCONFIG_ZMK_STUDIO_LOCKING=n \
  -DEXTRA_CONF_FILE=studio-rpc-usb-uart.conf

# Flash firmware
west flash -d build/left
```

### Keymap Visualization
```bash
# Keymap visualizations auto-generate on push via GitHub Actions
# Manual generation requires keymap-drawer tool:
keymap parse -c keymap_drawer.config.yaml -z config/eyelash_sofle.keymap \
  > keymap-drawer/eyelash_sofle.yaml
keymap draw keymap-drawer/eyelash_sofle.yaml > keymap-drawer/eyelash_sofle.svg
```

## Testing

**No automated testing infrastructure.** This is typical for keyboard firmware projects.

**Manual Testing Process:**
1. Flash firmware to both keyboard halves via .uf2 files
2. Test all key positions, layers, combos, and macros
3. Verify RGB, OLED displays, encoder, Bluetooth pairing
4. Test power consumption and sleep modes
5. Validate ZMK Studio connectivity (left side USB)

## Code Style Guidelines

### File Organization

```
config/                    # Primary configuration location
├── eyelash_sofle.keymap  # Keymap definition (behaviors, layers, combos)
├── eyelash_sofle.conf    # Feature flags and hardware settings
├── eyelash_sofle.json    # Physical layout for keymap-drawer
└── west.yml              # Dependency management

boards/arm/eyelash_sofle/ # Board hardware definition
├── *.dts, *.dtsi         # Device tree files
├── *_defconfig           # Kconfig defaults
└── *.keymap              # Default fallback keymap
```

### Device Tree (.dts/.dtsi) Style

**Includes:** Place at top in this order:
```c
#include <input/processors.dtsi>
#include <zephyr/dt-bindings/input/input-event-codes.h>
#include <behaviors.dtsi>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/pointing.h>
```

**Formatting:**
- Use 4-space indentation (not tabs)
- Node names use underscores: `aa_lower`, `scroll_encoder`
- Property names use hyphens: `hold-time-ms`, `tap-ms`
- Align property values for readability
- Single line for simple properties, multi-line for bindings

**Comments:**
```c
// Use C++-style comments for single lines
/* Multi-line comments for blocks */
```

### Keymap Structure

**Behavior Definitions:**
```c
/ {
    behaviors {
        name: label {
            compatible = "zmk,behavior-type";
            #binding-cells = <0>;
            wait-ms = <10>;
            tap-ms = <10>;
            bindings = <...>;
        };
    };
};
```

**Layer Convention:**
- Layer 0: Default/base layer
- Layer 1: Lower (symbols, numbers)
- Layer 2: Raise (navigation, function keys)
- Layer 3: Adjust (system controls, Bluetooth)
- Use descriptive comments for each layer

**Key Bindings:**
- Format: `&behavior KEY_CODE` or `&custom_behavior`
- Transparent keys: `&trans`
- No-operation: `&none`
- Align bindings in columns matching physical layout

### Configuration (.conf) Style

**Format:**
```conf
# Use clear section comments
CONFIG_OPTION_NAME=y    # Enable with =y
CONFIG_OPTION_NAME=n    # Disable with =n
CONFIG_NUMERIC_OPTION=3600000  # Numeric values
```

**Organization:** Group related settings with blank lines and comments

**Common Patterns:**
- Power management: `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT`, `CONFIG_ZMK_PM_SOFT_OFF`
- RGB: `CONFIG_ZMK_RGB_UNDERGLOW*` options
- Bluetooth: `CONFIG_BT_CTLR_TX_PWR_*`
- Peripherals: `CONFIG_EC11`, `CONFIG_ZMK_POINTING`

### Naming Conventions

**Device Tree Nodes:** `lowercase_with_underscores`  
**Behavior Labels:** `descriptive_name` (e.g., `aa_lower`, `scroll_encoder`)  
**Macros:** Follow ZMK conventions (e.g., `ZMK_POINTING_DEFAULT_MOVE_VAL`)  
**Config Options:** `SCREAMING_SNAKE_CASE` with `CONFIG_` prefix

### Timing Values

**Standard Timing:**
- Macro timing: `wait-ms = <10>`, `tap-ms = <10>`
- Soft-off hold: `hold-time-ms = <2000>`
- Debounce: `8ms` for press/release
- Sleep timeout: `3600000ms` (1 hour)

### Error Handling

**No explicit error handling** - ZMK/Zephyr handle hardware errors internally.

**Best Practices:**
- Validate GPIO pin assignments against PCB schematic
- Check I2C/SPI addresses for displays/RGB
- Verify Bluetooth profile limits (5 max)
- Test power draw to prevent battery issues

## Special Features

### Swedish Character Macros
Requires macOS U.S. International - PC keyboard layout:
- `&aa_lower` / `&aa_upper`: å/Å (Option+A / Option+Shift+A)
- `&ae_lower` / `&ae_upper`: ä/Ä (Option+U dead key, then A/Shift+A)
- `&oe_lower` / `&oe_upper`: ö/Ö (Option+U dead key, then O/Shift+O)

### Soft-Off Combo
Press Q+S+Z simultaneously for 2 seconds to enter deep sleep. Wake via reset button.

### ZMK Studio
Dynamic keymap editing available on left side via USB connection. Build with:
```yaml
snippet: studio-rpc-usb-uart
cmake-args: -DCONFIG_ZMK_STUDIO=y -DCONFIG_ZMK_STUDIO_LOCKING=n
```

### Mouse Emulation
Enabled via `CONFIG_ZMK_POINTING=y`. Behaviors:
- `&mmv`: Mouse movement with acceleration
- `&msc`: Mouse scroll with custom scaling

## Git Workflow

1. Edit config files in `config/` directory
2. Commit with descriptive messages (e.g., "Adjust home row mod timing")
3. Push to trigger GitHub Actions build
4. Download firmware: `gh run download --dir ./firmware` (always use --dir flag)
5. Flash .uf2 files from firmware/ folder to keyboard (drag-drop to USB drive)

## Common Tasks

**Add new key binding:** Edit `config/eyelash_sofle.keymap`, update layer bindings  
**Change RGB settings:** Modify `CONFIG_ZMK_RGB_UNDERGLOW_*` in `.conf`  
**Adjust sleep timeout:** Update `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` (milliseconds)  
**Add combo:** Define in `combos` node with key positions and timeout  
**Update dependencies:** Edit `config/west.yml`, change ZMK revision hash

## Important Notes

- **Always build both halves** - left and right firmware differ (central vs peripheral)
- **Test after every change** - firmware bugs require reflashing both sides
- **Battery safety** - Monitor power draw; excessive RGB/backlight drains batteries
- **ZMK Studio compatibility** - Only works on left side with specific build flags
- **Physical layout** - 60 keys + 1 encoder (left side), 60 keys (right side)

## Maintenance Tasks

### ZMK Zephyr 4.1 Compatibility Monitoring

**Current Status (March 18, 2026):**
- ZMK pinned to commit `83eafcbf9b12226f1d0dbb30a77e6ddc332a4210` (Dec 6, 2025 - pre-Zephyr 4.1)
- Reason: External board repository (`a741725193/zmk-sofle`) hasn't been updated for Zephyr 4.1 board variant requirements
- Root cause: ZMK upgraded to Zephyr 4.1 on Dec 10, 2025, requiring board-specific ZMK variants that the external board repo doesn't provide

**Monthly Check Procedure:**

Perform this check on the 18th of each month:

1. **Check upstream board repository for Zephyr 4.1 support:**
   ```bash
   # Visit: https://github.com/a741725193/zmk-sofle/commits/main
   # Look for commits mentioning "Zephyr 4.1", "board variant", or "zmk_board"
   ```

2. **Test ZMK upgrade to latest:**
   ```bash
   # Edit config/west.yml, change ZMK revision to: main
   # (Keep board repo at revision: main)
   git add config/west.yml
   git commit -m "Test: Upgrade ZMK to latest for Zephyr 4.1 compatibility"
   git push
   gh run watch  # Monitor build
   ```

3. **If build succeeds:**
   - Update `config/west.yml` ZMK revision to `main` or latest stable
   - Update "Last checked" date in `west.yml` comments
   - Update this section's "Current Status" date
   - Test firmware on hardware (especially Swedish character macros, ZMK Studio)

4. **If build fails with "Missing ZMK Compat" or "KeyError: 'qualifiers'":**
   - Revert the test commit: `git revert HEAD && git push`
   - Keep existing pinned ZMK version
   - Update "Last checked" date in this section only
   - Try again next month

**Next check due:** April 18, 2026

**Maintenance owner:** To be assigned

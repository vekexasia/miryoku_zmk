# Veke Branch Customizations

This document describes the differences between the `veke` branch and `master` in the miryoku_zmk repository.

## Overview

The `veke` branch contains personal customizations for a **Temper keyboard** with **Nice Nano v2** controller, including custom symbol/media layers and macros.

## Files Modified/Added

### 1. `.github/workflows/build-veke.yml` (NEW)

Custom GitHub Actions workflow for building the Temper keyboard firmware.

**Workflow Configuration:**
```yaml
name: 'Temper'
on:
  push:
  pull_request:
  workflow_dispatch:
jobs:
  build:
    if: github.repository_owner == 'vekexasia'
    uses: ./.github/workflows/main.yml
    secrets: inherit
    with:
      board: '["nice_nano_v2"]'
      shield: '["temper_left","temper_right","settings_reset"]'
      alphas: '["QWERTY"]'
      nav: '["vi"]'
      kconfig: '["CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y\nCONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y\nCONFIG_BT_CTLR_TX_PWR_PLUS_8=y"]'
```

**What it builds:**
- **Board:** `nice_nano_v2` (Nice Nano v2 nRF52840 controller)
- **Shields:**
  - `temper_left` - Left half of the Temper split keyboard
  - `temper_right` - Right half
  - `settings_reset` - Utility firmware to reset BT bonds
- **Alphas:** `QWERTY` layout
- **Navigation:** `vi` mode (HJKL navigation)

**Kconfig options enabled:**
- `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y` - Proxy battery level from peripheral to central
- `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y` - Central fetches battery from peripherals
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` - Increased Bluetooth TX power (+8 dBm) for better range

### 2. `config/temper.keymap` (NEW)

Custom keymap file for the Temper keyboard that includes:

**Macros defined:**
- `arrowfn` - Types `=> {}` and positions cursor inside braces (for JS arrow functions)
- `email` - Types email address `vekexasia@gmail.com`
- Italian accent macros (using WinCompose/Linux compose key)

**Combos (press both keys together):**

| Combo | Keys | Result |
|-------|------|--------|
| A + S | `à` | agrave |
| E + R | `è` | egrave |
| I + O | `ì` | igrave |
| O + P | `ò` | ograve |
| U + I | `ù` | ugrave |
| W + E | `é` | eacute |

**Combo prerequisites:**
- **Windows:** Install [WinCompose](https://github.com/samhocevar/wincompose/releases)
- **Ubuntu:** Enable compose key in GNOME Tweaks → Keyboard → Compose Key → Right Alt

### 4. `config/temper.conf` (NEW)

Kconfig options for BT improvements:
```
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
```

### 5. `miryoku/custom_config.h` (MODIFIED)

Custom layer overrides:

#### Symbol Layer (`MIRYOKU_LAYER_SYM`)
Complete redesign of the symbol layer layout:

```
Row 1: `  ~  #  &  |  ^  {  }  [  ]
Row 2: !  _  :  =  $  @  (  )  _  ;
Row 3: %  ?  *  +  \  /  -  <  >  "
```

#### Media Layer (`MIRYOKU_LAYER_MEDIA`)
Custom media layer with:
- **Left side:** Boot mode, layer toggles, GUI/Alt/Ctrl/Shift modifiers, system reset
- **Right side:** Arrow function macro, email macro, RGB controls, media controls (prev/next, vol up/down), Bluetooth device selection (0-3)

## Key Differences Summary

| Feature | Master | Veke |
|---------|--------|------|
| Build workflow | Generic examples only | Custom Temper build |
| Symbol layer | Default Miryoku | Completely redesigned |
| Media layer | Default Miryoku | Custom with macros |
| Macros | None | Arrow function, email |
| BT power | Default | +8 dBm |
| Battery proxy | No | Yes (split keyboard) |

## Building

### Via GitHub Actions (Recommended)

The easiest way to build firmware is using GitHub Actions:

1. Push changes to the `veke` branch
2. Go to GitHub Actions tab
3. Find the "Temper" workflow run
4. Download the firmware artifacts (`.uf2` files)

**Workflow triggers:**
- Automatically on push to `veke`
- Automatically on pull requests
- Manually via "Run workflow" button

### Local Development Environment Setup

If you want to build locally:

```bash
# 1. Create and activate Python virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 2. Install west and build dependencies
pip install west cmake ninja

# 3. Download and set up Zephyr SDK
mkdir -p ~/zephyr-sdk && cd ~/zephyr-sdk
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64_minimal.tar.xz -O sdk.tar.xz
tar xf sdk.tar.xz
cd zephyr-sdk-0.16.8
./setup.sh -t arm-zephyr-eabi -h -c

# 4. Initialize ZMK workspace
mkdir -p ~/zmk-workspace && cd ~/zmk-workspace
west init -m https://github.com/zmkfirmware/zmk.git --mr main --mf app/west.yml
west update

# 5. Install Zephyr Python requirements
pip install -r ~/zmk-workspace/zephyr/scripts/requirements.txt

# 6. Export Zephyr cmake package
cd ~/zmk-workspace/zephyr && west zephyr-export
```

### Local Build Commands

**Note:** The Temper shield is an "outboard" (external) keyboard definition. For local builds with the Temper, you would need to manually clone the shield definition. For standard ZMK keyboards (like Corne), you can build directly:

```bash
# Activate environment and set variables
source /path/to/miryoku_zmk/.venv/bin/activate
export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
export ZEPHYR_SDK_INSTALL_DIR=$HOME/zephyr-sdk/zephyr-sdk-0.16.8

# Build Corne (included in ZMK) - example
cd ~/zmk-workspace
west build -s zmk.git/app -b nice_nano -- \
  -DSHIELD=corne_left \
  -DZMK_CONFIG=/path/to/miryoku_zmk/config

# Output: build/zephyr/zmk.uf2
```

For the Temper keyboard specifically, use the GitHub Actions workflow as it automatically handles the outboard shield definition.

## Merge Status

Master has been merged into veke. The following keyboards were added from master:
- dasbob
- ergonaut_one / ergonaut_one_s
- le_chiffre_stm32
- pipar_sool
- swoop
- willis

## Flashing Firmware

1. Double-tap the reset button on the Nice Nano (enters bootloader)
2. The keyboard appears as a USB mass storage device
3. Copy the `.uf2` file to the device
4. Keyboard resets automatically and runs new firmware

For split keyboards, flash both halves separately with their respective firmware files.

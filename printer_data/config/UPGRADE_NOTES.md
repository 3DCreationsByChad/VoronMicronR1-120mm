# Configuration Upgrade Notes - 2025

## Overview
This document details the configuration updates made to bring your Voron Micron R1 120mm up to date with the latest Klipper and Cartographer3D features.

## What's New

### 1. Input Shaping with Cartographer ADXL345

**What Changed:**
- Enabled the built-in ADXL345 accelerometer on your Cartographer v3 probe
- Added `[input_shaper]` configuration section
- Configured `[resonance_tester]` for automatic calibration

**Benefits:**
- Reduces ringing and ghosting artifacts in prints
- Enables higher acceleration without quality loss
- Uses the Cartographer's built-in accelerometer (no additional hardware needed)

**How to Use:**
1. Run `SHAPER_CALIBRATE` command to automatically measure resonances
2. The system will test X and Y axes and recommend optimal input shaper settings
3. Results will be saved automatically to your config
4. For full calibration: use the `CALIBRATE_PRINTER` macro

**Configuration Location:** `printer.cfg:320-336`

---

### 2. Survey Touch - Dual Mode Probing

**What Changed:**
- Enhanced Cartographer configuration with Survey Touch comments
- Added macros for switching between scan and touch modes
- Touch mode provides contact-based probing for more accurate Z-offset

**Benefits:**
- More accurate Z-offset calibration when switching nozzles or build plates
- Flexibility to use scan mode for speed or touch mode for accuracy
- Auto Z-offset capability

**How to Use:**

**Switching Modes:**
```gcode
SWITCH_TO_TOUCH_MODE  # Switch to contact probing
SWITCH_TO_SCAN_MODE   # Switch to eddy current scanning
```

**Z-Offset Calibration:**
```gcode
CALIBRATE_Z_TOUCH     # Use Survey Touch for Z calibration
```

**Note:** Survey Touch requires Cartographer firmware 5.1 or later. Run `CARTOGRAPHER_STATUS` to check your firmware version.

**Configuration Location:** `printer.cfg:253-271`, `macro.cfg:189-206`

---

### 3. Modern Klipper Features

**What Changed:**
- Added `minimum_cruise_ratio: 0.5` to `[printer]` section
- This replaces the deprecated `max_accel_to_decel` parameter

**Benefits:**
- Better control over acceleration/deceleration behavior
- Improved cornering performance
- More predictable motion planning

**Configuration Location:** `printer.cfg:27`

---

### 4. Enhanced Macros

**New Macros Added:**

| Macro | Description |
|-------|-------------|
| `SWITCH_TO_TOUCH_MODE` | Switch Cartographer to touch mode |
| `SWITCH_TO_SCAN_MODE` | Switch Cartographer to scan mode |
| `CALIBRATE_Z_TOUCH` | Calibrate Z offset using Survey Touch |
| `CALIBRATE_PRINTER` | Full calibration (input shaping) |

**Updated Macros:**
- `PRINT_START`: Now uses adaptive bed meshing and Survey Touch
- Improved comments and documentation throughout

**Configuration Location:** `macro.cfg`

---

## First-Time Setup After Upgrade

### Step 1: Verify Cartographer Firmware
```gcode
CARTOGRAPHER_STATUS
```
Ensure you have firmware 5.1+ for full Survey Touch support. If not, update via Moonraker's update manager.

### Step 2: Run Input Shaper Calibration
```gcode
CALIBRATE_PRINTER
```
This will:
1. Home all axes
2. Run resonance tests on X and Y axes
3. Calculate optimal input shaper values
4. Save results to your config

**Important:** This process takes 5-10 minutes and makes noise. The toolhead will vibrate at various frequencies.

### Step 3: Calibrate Z-Offset with Survey Touch
```gcode
SWITCH_TO_TOUCH_MODE
CALIBRATE_Z_TOUCH
```
This provides a more accurate Z-offset than the previous scan-only method.

### Step 4: Run a Test Print
After calibration, run a test print to verify everything works correctly. Recommended test prints:
- Calibration cube for dimensional accuracy
- Resonance test tower to verify input shaping is working

---

## Troubleshooting

### ADXL345 Connection Errors

**Error:** "Invalid adxl345 id"

**Solutions:**
1. Verify Cartographer firmware is up to date
2. Check that `cs_pin: scanner:PA3` is correct for your probe version
3. Restart Klipper after config changes

### Survey Touch Not Working

**Error:** "Unknown command: PROBE_SWITCH"

**Solutions:**
1. Update Cartographer Klipper module via Moonraker
2. Ensure firmware version is 5.1 or higher
3. Restart both Klipper and firmware

### Input Shaper Calibration Fails

**Error:** "Timer too close" during SHAPER_CALIBRATE

**Solutions:**
1. This can happen on lower-powered SBCs
2. Try reducing `max_accel` temporarily: `SET_VELOCITY_LIMIT ACCEL=3000`
3. Close other running services during calibration
4. Ensure CAN bus is running at 1,000,000 baudrate

---

## Performance Tuning Recommendations

### After Input Shaper Calibration

Once input shaping is calibrated, you can potentially:
1. **Increase max_accel** in `printer.cfg` [printer] section
   - Default: 5000 mm/s²
   - With input shaping: potentially up to 7000-10000 mm/s²
   - Test incrementally using `TEST_SPEED` macro

2. **Optimize pressure advance**
   - Re-tune pressure advance after enabling input shaping
   - Current value: 0.036 (see `printer.cfg:193`)

3. **Fine-tune square_corner_velocity**
   - Current: 5.0 mm/s
   - Can potentially increase to 8-10 mm/s with input shaping

### Bed Mesh vs Survey Touch

**Recommendation:**
- Use SCAN mode for routine bed meshing (faster)
- Use TOUCH mode when:
  - Changing nozzles
  - Changing build plates
  - After significant mechanical changes
  - When highest Z accuracy is needed

---

## Configuration Backup

Your original configuration has been preserved in git history. To view previous config:
```bash
git log --oneline
git show <commit-hash>:printer_data/config/printer.cfg
```

---

## Additional Resources

- **Klipper Input Shaping Docs:** https://www.klipper3d.org/Resonance_Compensation.html
- **Cartographer3D Docs:** https://docs.cartographer3d.com
- **Survey Touch Guide:** https://docs.cartographer3d.com/cartographer-probe/survey-touch/
- **Klipper Config Changes:** https://github.com/Klipper3d/klipper/blob/master/docs/Config_Changes.md

---

## Summary of Changes

### Files Modified:
1. **printer.cfg**
   - Enabled ADXL345 accelerometer (lines 325-331)
   - Added input_shaper section (lines 333-336)
   - Added minimum_cruise_ratio (line 27)
   - Enhanced Survey Touch documentation (lines 268-271)

2. **macro.cfg**
   - Added SWITCH_TO_TOUCH_MODE macro
   - Added SWITCH_TO_SCAN_MODE macro
   - Added CALIBRATE_Z_TOUCH macro
   - Updated CALIBRATE_PRINTER with description
   - Improved PRINT_START comments

3. **moonraker.conf**
   - No changes needed (already has Cartographer update manager)

### Configuration Compatibility:
- ✅ Backward compatible - all existing functionality preserved
- ✅ New features are additive - nothing removed
- ✅ Requires firmware 5.1+ for full Survey Touch support
- ✅ ADXL345 works with existing Cartographer v3 hardware

---

## Next Steps

1. **Update Cartographer firmware** (if needed) via Moonraker
2. **Run CALIBRATE_PRINTER** to set up input shaping
3. **Test Survey Touch** with CALIBRATE_Z_TOUCH
4. **Print a calibration cube** to verify dimensional accuracy
5. **Run TEST_SPEED** macro to test new acceleration limits

---

*Configuration updated: 2025-11-05*
*Klipper features as of: January 2025*
*Cartographer3D features: v3 with Survey Touch*

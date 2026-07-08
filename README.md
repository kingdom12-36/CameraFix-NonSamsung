# CameraFix-NonSamsung

Samsung Camera 15.0.03.35 patched for cross-device ROMs (tested: Note 10+ running S24 FE OneUI 7 port by Unica).

## Why everything was crashing

The S24 FE camera app hardcodes Samsung-specific feature mappings for S24 FE hardware. On a different device (Note 10+ with original vendor), every settings page, sub-menu, and widget crashes because:

| Root Cause | Fix |
|---|---|
| S24 FE analytics key mappings missing on Note 10+ hardware | Patch throws → return null/void |
| Samsung native JNI libs not in Note 10+ vendor | no-op the callers |
| Notification icon is Samsung-proprietary `.bmp` | Swap to valid system drawable |
| Settings key count mismatch crashes app at init | return-void instead of crash |
| `isDarkScreen()` reads Samsung's `lcd_curtain` setting — Note 10+ display driver sets this to `1` for its own dim feature, which triggers a confusing "turn off Dark Screen?" dialog | Patch to always return false |

## All patches — v3 (latest)

| # | What | Fix |
|---|---|---|
| 1a/b | CameraPreferenceFragment (×2) | return null instead of RuntimeException |
| 2 | NotificationService | notification icon → android.R.drawable.ic_menu_camera |
| 3 | WatermarkNode.mergeWatermarkImage | return false (skips broken JNI) |
| 4 | PreferenceSettingFragment.updatePreferenceAttr (×3) | return-void |
| 5 | SaveOptionsFragment.updatePreferenceAttr (×4) | return-void |
| 6 | WatermarkFragment.updatePreferenceAttr | return-void |
| 7 | WidgetCustomFragment.updatePreferenceAttr | return-void |
| 8 | WidgetWatermarkFragment.updatePreferenceAttr | return-void |
| 9 | AbstractCameraSettings.initializeDefaultValueGetterMap | return-void on key count mismatch |
| **10** | **SystemSettingsUtil.isDarkScreen()** | **always return false — kills Dark Screen popup** |
| **11** | **Mode bar spacing (sw411dp-xxxhdpi dimens)** | **22.8dp → 10dp, inner margin 13.7dp → 8dp** |

## Mode bar fix explained

The S24 FE app sets `shooting_mode_shortcut_list_view_spacing = 22.8dp` specifically for sw411dp-xxxhdpi devices (Note 10+ falls in this bucket). This makes mode items spread 22.8dp apart — so only ~3 modes are visible at once and the bar feels very wide. Reduced to 10dp so more modes fit on screen simultaneously.

## Files

| File | Version | Description |
|---|---|---|
| `camera_rebuilt_v3-aligned-signed.apk` | **v3** | Latest — all 11 patches |
| `samsung-camera-fix-v3.zip` | **v3** | Same, zipped |
| `camera_rebuilt_v2-aligned-signed.apk` | v2 | 9 patches |
| `camera_rebuilt-aligned-signed.apk` | v1 | 4 patches |

## Install

```bash
adb install camera_rebuilt_v3-aligned-signed.apk
```

## Build pipeline

```
apktool d camera.apk -o src/
# edit smali + res
apktool b src/ -o rebuilt.apk
uber-apk-signer -a rebuilt.apk --ks debug.jks ... -o signed/
# order matters: align THEN sign
```

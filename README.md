# CameraFix-NonSamsung

Samsung Camera 15.0.03.35 patched for cross-device ROMs (tested: Note 10+ running S24 FE OneUI 7 port by Unica).

## Why everything was crashing

The S24 FE camera app hardcodes Samsung Analytics event-ID mappings for **S24 FE-specific hardware features**. When running on a different device (Note 10+ with original vendor), every settings page, sub-menu, save option, and widget hits a `RuntimeException` because the S24 FE-specific key → event-ID mapping doesn't exist in the Note 10+ hardware config. This is why **every button** crashed — it's not random; it's the same root cause everywhere.

There are 4 categories of failures:

| Root Cause | Scope | Fix |
|---|---|---|
| S24 FE analytics key mappings missing on Note 10+ hardware | 193 files, every settings interaction | Patch throws → return null/void |
| Samsung native libs chain-depend on Samsung system libs absent from Note 10+ vendor | 65 JNI files | no-op the callers |
| Notification icon is Samsung-proprietary `.bmp` (Android 12+ rejects it) | NotificationService | Swap to valid system drawable |
| Settings key count mismatch crashes app at initialization | AbstractCameraSettings | return-void instead of crash |

## All patches — v2

| # | File | Method | Patch |
|---|---|---|---|
| 1a | CameraPreferenceFragment | getOriginalString | return null |
| 1b | CameraPreferenceFragment | getEventId-style | return null |
| 2 | NotificationService | notification builder | icon → android.R.drawable.ic_menu_camera |
| 3 | WatermarkNode | mergeWatermarkImage | return false (skips missing JNI) |
| 4 | PreferenceSettingFragment | updatePreferenceAttr ×3 | return-void |
| 5 | SaveOptionsFragment | updatePreferenceAttr ×4 | return-void |
| 6 | WatermarkFragment | updatePreferenceAttr | return-void |
| 7 | WidgetCustomFragment | updatePreferenceAttr | return-void |
| 8 | WidgetWatermarkFragment | updatePreferenceAttr | return-void |
| 9 | AbstractCameraSettings | initializeDefaultValueGetterMap | return-void (critical startup fix) |

## Files

| File | Description |
|---|---|
| `camera_rebuilt_v2-aligned-signed.apk` | **Latest** — v2, all 9 patches, v3 signed |
| `samsung-camera-fix-v2.zip` | Same APK zipped |
| `camera_rebuilt-aligned-signed.apk` | v1 — 4 patches only |
| `samsung-camera-fix.zip` | v1 zipped |

## Install

```bash
adb install camera_rebuilt_v2-aligned-signed.apk
```

## Build pipeline used

```
apktool d camera.apk -o src/        # decompile
# edit smali files
apktool b src/ -o rebuilt.apk       # rebuild
uber-apk-signer -a rebuilt.apk ...  # zipalign + sign v1/v2/v3 in one step
# CRITICAL: align THEN sign — signing covers the whole file byte-for-byte
```

# CameraFix-NonSamsung

Samsung Camera 15.0.03.35 patched to run on non-Samsung devices.

## Patches applied

| # | Crash | Fix |
|---|-------|-----|
| 1 & 3 | `RuntimeException: No defined key` (CameraSettingActivity) | `CameraPreferenceFragment` — return null instead of throwing for unknown setting keys |
| 2 | `BadForegroundServiceNotificationException` bad icon `0x7f080b31` | `NotificationService` — swapped broken `.bmp` resource for `android.R.drawable.ic_menu_camera` |
| 4 | `UnsatisfiedLinkError: nativeBlendImage` (WatermarkNode PostProcess) | `WatermarkNode.mergeWatermarkImage` — no-op (return false), skips Samsung-only native lib |

## Root cause

Samsung Camera depends on 4 categories of Samsung-only dependencies not present on stock Android:

1. **Samsung system services** (People API, SecMp, BixbyKey events, SemContextManager) — OS-level, 193 files affected, cannot be bundled in an APK
2. **Samsung native lib chain** — bundled `.so` files depend on `libsecimaging_pdk.camera.samsung.so` (Samsung system lib in `/system/lib64/`) which does not exist on stock ROMs; breaks all JNI calls in the chain
3. **Resource format incompatibility** — `saving_notification.bmp` is a Samsung-proprietary format; Android 12+ rejects non-PNG/vector notification icons
4. **Settings key mapping gaps** — Samsung-specific hardware keys (Bixby, DeX, S Pen shortcuts) have no equivalent in stock Android's `CameraSettings.Key` enum

**Providing the missing libs is not viable** — Samsung's system libs are hardware-tied, compiled against Samsung's camera HAL, and signed with Samsung's keys.

## Files

- `samsung-camera-fix.zip` — ZIP containing the signed APK
- `camera_rebuilt-aligned-signed.apk` — Signed APK (v3 signature, debug keystore)

## Install

```bash
adb install camera_rebuilt-aligned-signed.apk
```

## Build pipeline

```
apktool d camera.apk -o src/       # decompile
# edit smali files
apktool b src/ -o rebuilt.apk      # rebuild
uber-apk-signer -a rebuilt.apk ... # align + sign (order: align THEN sign)
```

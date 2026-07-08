# CameraFix-NonSamsung

Samsung Camera 15.0.03.35 patched for cross-device ROMs (tested: Note 10+ running S24 FE OneUI 7 port by Unica).

## Root cause summary

The S24 FE camera app is hardwired to S24 FE hardware signatures across 4 failure categories:

| Category | Symptoms | Fix |
|---|---|---|
| S24 FE analytics key mappings absent on Note 10+ | Every settings screen/widget/sub-page crashes | `throw RuntimeException` → `return null/void` |
| Samsung JNI native libs absent from Note 10+ vendor | Watermark crash, bokeh freeze | no-op callers, fix bridge NPEs |
| Notification icons are Samsung-proprietary drawables | App crashes immediately on photo taken | Swap to `android.R.drawable.ic_menu_camera` |
| Settings key count mismatch at init | Crash before settings open | `return-void` |
| `lcd_curtain` system setting misread as "Dark Screen ON" | Confusing popup dialog on every settings open | `isDarkScreen()` always returns false |
| Note 10+ sw411dp-xxxhdpi gets 22.8dp mode-tab spacing | Mode bar too wide, hard to scroll | Spacing reduced to 10dp |
| `CameraResolution` lookup fails on Note 10+ resolution set | 0.5× crash, portrait crash | `throw IllegalArgumentException` → `return null` |
| Post-processing notification uses bad icon | Normal photo capture crashes app | Icon patched in 2 notification classes |
| `cancelTakePicture()` bridge method throws null | Portrait mode crash on cancel/switch | `throw null` → `return-void` |

---

## All patches — v4 (latest, 29 total)

### Settings crashes (v1–v2)
| # | File | Fix |
|---|---|---|
| 1a/b | CameraPreferenceFragment (×2) | return null |
| 2 | NotificationService | notification icon → system drawable |
| 3 | WatermarkNode.mergeWatermarkImage | return false (skips broken JNI) |
| 4–8 | PreferenceSettingFragment, SaveOptionsFragment, WatermarkFragment, WidgetCustomFragment, WidgetWatermarkFragment | return-void on updatePreferenceAttr |
| 9 | AbstractCameraSettings.initializeDefaultValueGetterMap | return-void |

### UI / UX fixes (v3)
| # | File | Fix |
|---|---|---|
| 10 | SystemSettingsUtil.isDarkScreen() | always return false |
| 11 | values-sw411dp-xxxhdpi/dimens.xml | mode tab spacing 22.8→10dp, inner margin 13.7→8dp |

### Photo capture / lens crashes (v4)
| # | File | Fix |
|---|---|---|
| 12 | PostProcessNotification | setSmallIcon 0x7f080ce2 → 0x01080075 |
| 13 | PostProcessorLoggingService | setSmallIcon 0x7f080997 → 0x01080075 |
| 14–22 | CameraResolution (×9 methods) | IllegalArgumentException → return null (wide/ultrawide/live-focus resolution lookups) |
| 23 | CameraResolution | "not supported picture ratio" → return null |
| 24 | SingleBokehPhotoMaker.cancelTakePicture() | throw null → return-void |
| 25 | DualBokehPhotoMaker.cancelTakePicture() | throw null → return-void |
| 26 | PortraitZoomPhotoMaker.cancelTakePicture() | throw null → return-void |
| 27–29 | SingleBokehPhotoMaker, DualBokehPhotoMaker, BokehVideoMaker | all null-throw bridge methods → return-void |

---

## Files

| File | Version | Patches |
|---|---|---|
| `camera_rebuilt_v4-aligned-signed.apk` | **v4** | All 29 patches — use this |
| `samsung-camera-fix-v4.zip` | **v4** | Same, zipped |
| `camera_rebuilt_v3-aligned-signed.apk` | v3 | 11 patches |
| `camera_rebuilt_v2-aligned-signed.apk` | v2 | 9 patches |
| `camera_rebuilt-aligned-signed.apk` | v1 | 4 patches |

## Install

```bash
adb install camera_rebuilt_v4-aligned-signed.apk
```

## Build pipeline

```
apktool d camera.apk -o src/
# patch smali + res XML
apktool b src/ -o rebuilt.apk
uber-apk-signer -a rebuilt.apk --ks debug.jks --ksAlias androiddebugkey --ksKeyPass android --ksPass android --allowResign -o signed/
# CRITICAL: zipalign first, THEN sign — uber-apk-signer handles both in one step
```

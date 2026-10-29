# expo-dev-launcher crashes on Android 7 (API 24/25)

Minimal reproduction for a crash in `expo-dev-launcher` on Android 7.0/7.1.

This is `expo-template-blank` (SDK 58) with only `expo-dev-client` added (`npx expo install expo-dev-client`). There are no other changes. The app uses the default `minSdkVersion` (24) and doesn't enable core library desugaring.

## Steps to reproduce

1. Create an Android emulator running API 24 or 25 (for example "Pixel" with the Android 7.1.1 / API 25 system image) and start it.
2. Install dependencies and build the development build:

   ```sh
   npm install
   npx expo run:android
   ```

3. The app crashes as soon as it starts, before any JavaScript runs.

**Expected:** the dev launcher opens. The app's `minSdkVersion` is 24, so Android 7 is supported.

**Actual:** the app crashes in `Application.onCreate`:

```
FATAL EXCEPTION: main
Process: com.example.devlauncherandroid7repro, PID: 5716
java.lang.NoClassDefFoundError: Failed resolution of: Ljava/time/Duration;
  at expo.modules.devlauncher.nsd.NsdDiscoveryBaseKt.<clinit>(NsdDiscoveryBase.kt:30)
  at expo.modules.devlauncher.nsd.NsdDiscoveryBaseKt.access$getHEALTH_CHECK_TIMEOUT$p(NsdDiscoveryBase.kt:1)
  at expo.modules.devlauncher.nsd.NsdDiscoveryBase.<init>(NsdDiscoveryBase.kt:40)
  at expo.modules.devlauncher.nsd.NsdDiscoveryLegacy.<init>(NsdDiscoveryLegacy.kt:13)
  at expo.modules.devlauncher.nsd.NsdDiscoveryKt.NsdDiscovery(NsdDiscovery.kt:23)
  at expo.modules.devlauncher.services.PackagerService.<init>(PackagerService.kt:19)
  at expo.modules.devlauncher.services.DependencyInjection.init(DependencyInjection.kt:75)
  at expo.modules.devlauncher.DevLauncherController$Companion.initialize$expo_dev_launcher_debug(DevLauncherController.kt:527)
  at expo.modules.devlauncher.DevLauncherController$Companion.initialize(DevLauncherController.kt:554)
  at expo.modules.devlauncher.DevLauncherController$Companion.initialize$default(DevLauncherController.kt:547)
  at expo.modules.devlauncher.DevLauncherPackageDelegate$createApplicationLifecycleListeners$1.onCreate(DevLauncherPackageDelegate.kt:40)
  at expo.modules.ApplicationLifecycleDispatcher.onApplicationCreate(ApplicationLifecycleDispatcher.kt:20)
  at com.example.devlauncherandroid7repro.MainApplication.onCreate(MainApplication.kt:38)
Caused by: java.lang.ClassNotFoundException: Didn't find class "java.time.Duration"
  ... 23 more
```

## Cause

`packages/expo-dev-launcher/android/src/debug/java/expo/modules/devlauncher/nsd/NsdDiscoveryBase.kt` has a top-level property:

```kotlin
private val HEALTH_CHECK_TIMEOUT = 5.toDuration(DurationUnit.SECONDS).toJavaDuration()
```

`java.time.Duration` was added in API 26. Because the property is top-level, it's set up in the file's static initialiser, which runs as soon as `PackagerService` creates `NsdDiscoveryLegacy`.

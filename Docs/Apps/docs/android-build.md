# Android Build and Run

Status: NOT VERIFIED. A signed universal APK and sideload walkthrough have not been produced from this workspace.

Install a supported JDK and Android SDK, configure `ANDROID_HOME`, and start an emulator or connect a device. Start Metro with `npm start`; in another terminal run `npm run android` for a development build.

For M1 distribution, configure a release keystore outside source control and build a signed APK (not an AAB) with the Android Gradle release task. Verify installation and all six routes on a clean device before reporting PASS. Do not commit keystore files or signing credentials.
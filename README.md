# QNTMEX Wallet — Native App Shell (Capacitor)

This wraps `walletqntmex.com` (the single-file `index.html` wallet) into a
Capacitor project so it can be built as a real Android **.aab** and iOS **.ipa**.

## What's included
- `www/index.html` — the wallet app (copied from QNTMEX_JUNE_8_V3.zip)
- `android/` — native Android project (Gradle), app id `com.qntmex.wallet`, version 4.0.0 (versionCode 4)
- `ios/` — native Xcode project, app id `com.qntmex.wallet`
- `.github/workflows/android-aab.yml` — builds the `.aab` on every push to `main`
- `.github/workflows/ios-ipa.yml` — builds the `.ipa` on a macOS GitHub runner

Builds run in CI because Android SDK + Gradle and Xcode/macOS toolchains
aren't available in this sandbox. Push this repo to GitHub and the workflows
will produce downloadable artifacts.

## 1. Push to GitHub
```bash
git init
git add .
git commit -m "Capacitor wrapper for QNTMEX Wallet"
git remote add origin https://github.com/quantumex/<your-repo>.git
git push -u origin main
```

## 2. Android (.aab)
Runs out of the box and produces an **unsigned** release AAB.

To get a Play Store–ready **signed** AAB, add these repo secrets
(Settings → Secrets and variables → Actions):
- `ANDROID_KEYSTORE_BASE64` — `base64 -i your-release.keystore | pbcopy`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

Generate a keystore (one-time, do this locally and keep it safe):
```bash
keytool -genkey -v -keystore release.keystore -alias qntmex \
  -keyalg RSA -keysize 2048 -validity 10000
```

## 3. iOS (.ipa)
Requires an Apple Developer account. Add these repo secrets:
- `BUILD_CERTIFICATE_BASE64` — exported `.p12` distribution cert, base64-encoded
- `P12_PASSWORD`
- `BUILD_PROVISION_PROFILE_BASE64` — exported `.mobileprovision`, base64-encoded
- `KEYCHAIN_PASSWORD` — any string, used for a temporary CI keychain
- `APPLE_TEAM_ID` — found in Apple Developer portal (Membership)

Without these secrets the iOS workflow will fail at the signing step —
that's expected until you add Apple credentials.

## 4. Local builds (if you have the toolchains)
```bash
npm install
npx cap sync

# Android
cd android && ./gradlew bundleRelease
# -> android/app/build/outputs/bundle/release/app-release.aab

# iOS (macOS + Xcode only)
cd ios/App && pod install
open App.xcworkspace   # build & archive via Xcode, or use xcodebuild as in the workflow
```

## Notes
- QNTMEX branding (gold crown/shield logo, void-black background `#040302`) is
  applied to app icons, adaptive icon foreground/background, and splash
  screens for both Android and iOS.
- `capacitor.config.json` sets `appId: com.qntmex.wallet` to match the
  existing Play Console listing and the React Native app's bundle id.
- The wallet is fully client-side (ethers.js, TweetNaCl, etc.) — no native
  plugins are required beyond Capacitor's WebView shell.

# Production Build Guide for TaskAlarm

This guide explains how to build a production-ready APK or AAB (Android App Bundle) for TaskAlarm.

## Prerequisites

- Android SDK installed and configured
- Java JDK 17 or 21 installed
- `keytool` command available (comes with JDK)

## Step 1: Generate Production Keystore

**⚠️ IMPORTANT: Keep your keystore file and passwords secure! You'll need them for all future app updates.**

### Generate Keystore

Navigate to the `android/app` directory and run:

```bash
cd android/app

keytool -genkeypair -v -storetype PKCS12 -keystore taskAlarm-release.keystore -alias taskAlarm-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

You'll be prompted for:

- **Keystore password**: Choose a strong password (remember this!)
- **Key password**: Can be the same as keystore password
- **Your name**: Your name or organization name
- **Organizational Unit**: Department or team name
- **Organization**: Company name
- **City**: Your city
- **State**: Your state/province
- **Country code**: Two-letter country code (e.g., US, IN)

### Verify Keystore

```bash
keytool -list -v -keystore taskAlarm-release.keystore -alias taskAlarm-key-alias
```

## Step 2: Configure Signing Credentials

Create or edit `android/gradle.properties` and add:

```properties
# Production Release Signing Configuration
# ⚠️ DO NOT COMMIT THIS FILE IF IT CONTAINS PASSWORDS
MYAPP_RELEASE_STORE_FILE=taskAlarm-release.keystore
MYAPP_RELEASE_KEY_ALIAS=taskAlarm-key-alias
MYAPP_RELEASE_STORE_PASSWORD=your-keystore-password-here
MYAPP_RELEASE_KEY_PASSWORD=your-key-password-here
```

**Security Note**:

- The `gradle.properties` file is already in `.gitignore` for sensitive data
- For CI/CD, use environment variables or secure secrets management
- Never commit passwords to version control

## Step 3: Build Production APK

### Option A: Using npm scripts (Recommended)

```bash
# Clean previous builds
npm run build:android:clean

# Build release APK
npm run build:android:release
```

### Option B: Using Gradle directly

```bash
cd android

# Clean previous builds
./gradlew clean

# Build release APK
./gradlew assembleRelease

# On Windows:
gradlew.bat clean
gradlew.bat assembleRelease
```

### Output Location

The signed APK will be generated at:

```
android/app/build/outputs/apk/release/taskAlarm-release-v1.0-1-YYYYMMDD.apk
```

## Step 4: Build Android App Bundle (AAB) for Play Store

For Google Play Store, you need to build an AAB instead of APK:

```bash
# Using npm script
npm run build:android:bundle

# Or using Gradle
cd android && ./gradlew bundleRelease
```

### Output Location

The AAB will be generated at:

```
android/app/build/outputs/bundle/release/app-release.aab
```

## Step 5: Verify the Build

### Check APK Signature

```bash
jarsigner -verify -verbose -certs android/app/build/outputs/apk/release/taskAlarm-release-*.apk
```

### Get APK Information

```bash
# Requires Android SDK build-tools
aapt dump badging android/app/build/outputs/apk/release/taskAlarm-release-*.apk
```

### Install and Test

```bash
# Install on connected device
adb install android/app/build/outputs/apk/release/taskAlarm-release-*.apk

# Or use Gradle
npm run build:android:install
```

## Step 6: Update Version Before Release

Before each production release, update the version in `android/app/build.gradle`:

```gradle
defaultConfig {
    applicationId "com.taskalarm"
    versionCode 2        // Increment for each release (required)
    versionName "1.0.1"  // Update version string (optional)
}
```

**Version Code Rules:**

- Must be an integer
- Must be higher than previous releases
- Google Play requires incrementing for each upload

## Production Build Checklist

Before publishing to Play Store:

- [ ] Keystore generated and securely backed up
- [ ] Signing credentials configured in `gradle.properties`
- [ ] Version code and version name updated
- [ ] Release build tested on real device
- [ ] APK/AAB signature verified
- [ ] ProGuard rules tested (if enabled)
- [ ] App functionality verified in release build
- [ ] AAB uploaded to Play Console (for Play Store)

## Troubleshooting

### Build fails with "Keystore file not found"

- Ensure `taskAlarm-release.keystore` is in `android/app/` directory
- Check `MYAPP_RELEASE_STORE_FILE` path in `gradle.properties`

### Build fails with "Password incorrect"

- Verify passwords in `gradle.properties` match keystore
- Check for extra spaces or special characters

### APK not signed

- Verify `signingConfig signingConfigs.release` is set in `buildTypes.release`
- Check that all `MYAPP_RELEASE_*` properties are set in `gradle.properties`

### Build is too large

- Enable ProGuard: Set `enableProguardInReleaseBuilds = true` in `build.gradle`
- Resource shrinking is already enabled with `shrinkResources true`

## Security Best Practices

1. **Never commit keystore files** - Already in `.gitignore`
2. **Never commit passwords** - Use environment variables in CI/CD
3. **Backup keystore** - Store in secure location (password manager, encrypted storage)
4. **Use different keystores** - Separate keystores for different apps/environments
5. **Rotate if compromised** - If keystore is compromised, you'll need to create a new app listing

## CI/CD Integration

For automated builds, use environment variables instead of `gradle.properties`:

```bash
export MYAPP_RELEASE_STORE_FILE=taskAlarm-release.keystore
export MYAPP_RELEASE_STORE_PASSWORD=your-password
export MYAPP_RELEASE_KEY_ALIAS=taskAlarm-key-alias
export MYAPP_RELEASE_KEY_PASSWORD=your-password
```

Then reference in `build.gradle`:

```gradle
storeFile file(System.getenv("MYAPP_RELEASE_STORE_FILE"))
storePassword System.getenv("MYAPP_RELEASE_STORE_PASSWORD")
```

## Additional Resources

- [React Native Signing Guide](https://reactnative.dev/docs/signed-apk-android)
- [Android App Signing](https://developer.android.com/studio/publish/app-signing)
- [Google Play App Signing](https://support.google.com/googleplay/android-developer/answer/9842756)

## Step 3: Build the Production APK or AAB

Navigate to the project root directory:

```bash
cd ..
```

### To generate a **signed APK**:

```bash
cd android
./gradlew assembleRelease
```

- The APK will be created at `android/app/build/outputs/apk/release/app-release.apk`.

### To generate a **signed AAB (Android App Bundle)** (recommended for Play Store):

```bash
cd android
./gradlew bundleRelease
```

- The AAB file will be at `android/app/build/outputs/bundle/release/app-release.aab`.

---

## Step 4: Test Your Production Build

- **Local device:** Install the APK on a device/emulator and verify functionality.
- **AAB:** Use the "internal test track" on Google Play Console for bundle testing.

## Step 5: Prepare for Distribution

- **For Google Play:** Use the `.aab` file and upload it in the Google Play Console.
- **For direct download:** You can distribute the `.apk` file, but .aab is required for Play Store.

---

## Common Useful Gradle Commands

- **Clean build artifacts:**
  ```bash
  ./gradlew clean
  ```
- **Check builds without code changes:**
  ```bash
  ./gradlew assembleRelease
  ```
- **List all tasks:**
  ```bash
  ./gradlew tasks
  ```

---

**Your app is now production-ready!**

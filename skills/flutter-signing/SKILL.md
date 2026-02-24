---
name: flutter-signing
description: Use when the user needs to set up code signing for iOS or Android in a Flutter project. Covers iOS certificates and provisioning profiles, Android keystores and signing configurations, and CI/CD signing setup.
---

# Flutter Code Signing

## Overview

Code signing is required to distribute apps on both iOS and Android. iOS uses certificates and provisioning profiles managed through Apple Developer Program. Android uses Java keystores containing a private key. Both must be properly configured before building release versions.

## When to Use

- User needs to build a release version for the first time
- User sees signing-related build errors
- User is setting up CI/CD and needs to configure signing
- User is preparing to submit to App Store or Google Play
- User needs to create or manage signing certificates/keystores
- **NOT** for debug builds (Flutter handles debug signing automatically)

## Quick Reference

| Platform | Signing Mechanism | Key Files | Where Configured |
|---|---|---|---|
| iOS (auto) | Xcode Automatic Signing | Apple Developer account | Xcode project settings |
| iOS (manual) | Certificate + Provisioning Profile | `.cer`, `.p12`, `.mobileprovision` | Xcode project settings |
| Android | Java Keystore | `.jks` or `.keystore` file | `key.properties` + `build.gradle` |

## Part 1: iOS Code Signing

### Prerequisites

- Apple Developer Program membership ($99/year)
- Xcode installed on macOS
- Apple ID added to Xcode (Xcode → Settings → Accounts)

### Automatic Signing (Recommended for Development)

Automatic signing lets Xcode manage certificates and provisioning profiles.

**Setup in Xcode**:

1. Open `ios/Runner.xcworkspace` in Xcode
2. Select the **Runner** project in the navigator
3. Select the **Runner** target
4. Go to **Signing & Capabilities** tab
5. Check **Automatically manage signing**
6. Select your **Team** from the dropdown

Xcode will automatically:
- Create a signing certificate if needed
- Create and update provisioning profiles
- Handle capability entitlements

**In `project.pbxproj`**, automatic signing looks like:

```
CODE_SIGN_STYLE = Automatic;
DEVELOPMENT_TEAM = YOUR_TEAM_ID;
```

### Manual Signing (Required for CI/CD and Distribution)

For App Store distribution or CI/CD, you need manual signing with explicit certificates and profiles.

#### Step 1: Create a Signing Certificate

**Via Apple Developer Portal**:

1. Go to [developer.apple.com/account](https://developer.apple.com/account)
2. Navigate to **Certificates, Identifiers & Profiles** → **Certificates**
3. Click **+** to create a new certificate
4. Choose the certificate type:
   - **Apple Development** — for development/testing
   - **Apple Distribution** — for App Store and Ad Hoc distribution
5. Upload a Certificate Signing Request (CSR):
   - Open **Keychain Access** on macOS
   - Menu: **Keychain Access** → **Certificate Assistant** → **Request a Certificate From a Certificate Authority**
   - Enter your email, select **Saved to disk**, click **Continue**
   - Upload the generated `.certSigningRequest` file
6. Download the `.cer` file and double-click to install in Keychain

**Export as .p12 (for CI/CD)**:

1. Open **Keychain Access**
2. Find your certificate under **My Certificates**
3. Right-click → **Export**
4. Save as `.p12` format
5. Set a strong password (you will need this in CI/CD)

#### Step 2: Register an App ID

1. Go to **Certificates, Identifiers & Profiles** → **Identifiers**
2. Click **+** → **App IDs** → **App**
3. Enter a description and your **Bundle ID** (e.g., `com.company.appName`)
4. Select required **Capabilities** (Push Notifications, Sign in with Apple, etc.)
5. Click **Register**

#### Step 3: Create a Provisioning Profile

1. Go to **Certificates, Identifiers & Profiles** → **Profiles**
2. Click **+** to create a new profile
3. Choose the profile type:
   - **iOS App Development** — for development
   - **App Store Connect** — for App Store distribution
   - **Ad Hoc** — for limited distribution outside the store
4. Select the **App ID** you registered
5. Select the **Certificate** to include
6. For Development/Ad Hoc: Select the **Devices** to include
7. Name the profile and click **Generate**
8. Download the `.mobileprovision` file

#### Step 4: Configure Xcode for Manual Signing

In Xcode:

1. Uncheck **Automatically manage signing**
2. Under **Signing (Release)**, select the provisioning profile
3. The certificate should auto-populate

In `project.pbxproj`, manual signing looks like:

```
CODE_SIGN_STYLE = Manual;
DEVELOPMENT_TEAM = YOUR_TEAM_ID;
PROVISIONING_PROFILE_SPECIFIER = "Your Profile Name";
CODE_SIGN_IDENTITY = "Apple Distribution";
```

### iOS Signing for CLI Builds

When building from the command line:

```bash
# Build with automatic signing
flutter build ipa

# Build with specific export options
flutter build ipa --export-options-plist=ios/ExportOptions.plist
```

**ExportOptions.plist** for App Store distribution:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store-connect</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>provisioningProfiles</key>
    <dict>
        <key>com.company.appName</key>
        <string>Your Distribution Profile Name</string>
    </dict>
    <key>signingCertificate</key>
    <string>Apple Distribution</string>
</dict>
</plist>
```

**ExportOptions.plist** for Ad Hoc distribution:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>ad-hoc</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>provisioningProfiles</key>
    <dict>
        <key>com.company.appName</key>
        <string>Your Ad Hoc Profile Name</string>
    </dict>
    <key>signingCertificate</key>
    <string>Apple Distribution</string>
</dict>
</plist>
```

### iOS CI/CD Signing

For CI/CD environments (GitHub Actions, Codemagic, Bitrise, etc.) that don't have Xcode GUI:

#### Using Fastlane Match (Recommended)

Fastlane Match stores certificates and profiles in a Git repo or cloud storage:

```bash
# Install fastlane
gem install fastlane

# Initialize match (in ios/ directory)
cd ios
fastlane match init
```

**Matchfile** configuration:

```ruby
git_url("https://github.com/your-org/certificates.git")
storage_mode("git")
type("appstore")  # or "development", "adhoc"
app_identifier("com.company.appName")
team_id("YOUR_TEAM_ID")
```

```bash
# Generate/download certificates and profiles
fastlane match appstore
fastlane match development
```

#### Manual CI/CD Setup (GitHub Actions Example)

```yaml
- name: Install Apple certificate and provisioning profile
  env:
    BUILD_CERTIFICATE_BASE64: ${{ secrets.BUILD_CERTIFICATE_BASE64 }}
    P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
    BUILD_PROVISION_PROFILE_BASE64: ${{ secrets.BUILD_PROVISION_PROFILE_BASE64 }}
    KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
  run: |
    # Create variables
    CERTIFICATE_PATH=$RUNNER_TEMP/build_certificate.p12
    PP_PATH=$RUNNER_TEMP/build_pp.mobileprovision
    KEYCHAIN_PATH=$RUNNER_TEMP/app-signing.keychain-db

    # Decode from base64
    echo -n "$BUILD_CERTIFICATE_BASE64" | base64 --decode -o $CERTIFICATE_PATH
    echo -n "$BUILD_PROVISION_PROFILE_BASE64" | base64 --decode -o $PP_PATH

    # Create temporary keychain
    security create-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH
    security set-keychain-settings -lut 21600 $KEYCHAIN_PATH
    security unlock-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH

    # Import certificate
    security import $CERTIFICATE_PATH -P "$P12_PASSWORD" \
      -A -t cert -f pkcs12 -k $KEYCHAIN_PATH
    security set-key-partition-list -S apple-tool:,apple: \
      -k "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH
    security list-keychain -d user -s $KEYCHAIN_PATH

    # Install provisioning profile
    mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
    cp $PP_PATH ~/Library/MobileDevice/Provisioning\ Profiles
```

To encode your files for CI secrets:

```bash
# Encode certificate
base64 -i certificate.p12 | pbcopy

# Encode provisioning profile
base64 -i profile.mobileprovision | pbcopy
```

## Part 2: Android Code Signing

### Step 1: Create a Keystore

```bash
keytool -genkey -v \
  -keystore ~/upload-keystore.jks \
  -storetype JKS \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias upload
```

You will be prompted for:
- **Keystore password**: Strong password for the keystore file
- **Key password**: Password for the individual key (can match keystore password)
- **Distinguished name fields**: Name, organization, city, state, country code

**Important**: Store the keystore file and passwords securely. If you lose them, you cannot update your app on Google Play (unless using Play App Signing).

### Step 2: Create key.properties

Create `android/key.properties` (this file should NOT be committed to version control):

```properties
storePassword=your_keystore_password
keyPassword=your_key_password
keyAlias=upload
storeFile=/Users/username/upload-keystore.jks
```

For team projects, use a relative path or environment variable:

```properties
storePassword=your_keystore_password
keyPassword=your_key_password
keyAlias=upload
storeFile=../keystore/upload-keystore.jks
```

### Step 3: Configure build.gradle

Edit `android/app/build.gradle`:

```groovy
// Add above the android { } block
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ... existing config ...

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            // Enables code shrinking, obfuscation, and optimization
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Step 4: Add to .gitignore

Ensure sensitive files are not committed:

```gitignore
# Android signing
android/key.properties
*.jks
*.keystore
```

### Google Play App Signing

Google Play App Signing adds an extra layer of security. Google holds the app signing key and you upload with an "upload key":

- **App signing key**: Held by Google, used to sign the APK/AAB delivered to users
- **Upload key**: Your keystore, used to authenticate uploads to Google Play

If you lose your upload key, you can contact Google to reset it. You cannot lose the app signing key because Google manages it.

**Enrollment**: When uploading your first AAB to Google Play Console, you're automatically enrolled in Play App Signing.

### Android CI/CD Signing

#### Environment Variable Approach

```groovy
// android/app/build.gradle
signingConfigs {
    release {
        keyAlias System.getenv("KEY_ALIAS") ?: keystoreProperties['keyAlias']
        keyPassword System.getenv("KEY_PASSWORD") ?: keystoreProperties['keyPassword']
        storeFile file(System.getenv("KEYSTORE_PATH") ?: keystoreProperties['storeFile'] ?: 'dummy')
        storePassword System.getenv("STORE_PASSWORD") ?: keystoreProperties['storePassword']
    }
}
```

#### GitHub Actions Example

```yaml
- name: Decode keystore
  env:
    KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
  run: |
    echo "$KEYSTORE_BASE64" | base64 --decode > android/app/upload-keystore.jks

- name: Build AAB
  env:
    KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
    KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
    STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
  run: |
    # Create key.properties for the build
    cat > android/key.properties <<EOF
    storePassword=$STORE_PASSWORD
    keyPassword=$KEY_PASSWORD
    keyAlias=$KEY_ALIAS
    storeFile=upload-keystore.jks
    EOF

    flutter build appbundle
```

To encode keystore for CI secrets:

```bash
base64 -i upload-keystore.jks | pbcopy
```

#### Codemagic / Bitrise

Most CI/CD platforms have built-in support for Android code signing:
- Upload the keystore file to the platform's secure files
- Set environment variables for passwords and alias
- The platform injects them at build time

## Signing Configuration Summary

### iOS File Locations

| File | Purpose |
|---|---|
| `ios/Runner.xcodeproj/project.pbxproj` | Signing style, team ID, profile specifier |
| `ios/Runner/Info.plist` | Bundle identifier (used to match profiles) |
| `ios/ExportOptions.plist` | Export configuration for CLI builds |
| `~/Library/MobileDevice/Provisioning Profiles/` | Installed provisioning profiles |
| Keychain | Installed certificates |

### Android File Locations

| File | Purpose |
|---|---|
| `android/key.properties` | Keystore credentials (NOT in version control) |
| `android/app/build.gradle` | Signing config referencing key.properties |
| `upload-keystore.jks` | The keystore file (store securely) |

## Common Mistakes

1. **Committing key.properties or keystore to Git** — These contain sensitive credentials. Add `android/key.properties`, `*.jks`, and `*.keystore` to `.gitignore` immediately. If accidentally committed, rotate the credentials.

2. **Expired provisioning profile** — iOS provisioning profiles expire after 1 year. If builds fail with "no valid provisioning profile," regenerate the profile in the Apple Developer Portal and download it.

3. **Certificate/profile mismatch** — The provisioning profile must include the certificate you're signing with. If you create a new certificate, you must regenerate your provisioning profiles to include it.

4. **Wrong bundle ID in provisioning profile** — The bundle ID in the provisioning profile must exactly match `PRODUCT_BUNDLE_IDENTIFIER` in the Xcode project. Wildcards (e.g., `com.company.*`) work for development but not App Store distribution.

5. **Lost Android keystore** — If you lose the keystore file or forget the passwords and are NOT enrolled in Google Play App Signing, you cannot update your app. You would need to publish as a new app with a new package name. Always back up your keystore securely.

6. **Using debug signing for release** — If `signingConfig` is not set for the release build type, Gradle may silently fall back to debug signing. The resulting APK/AAB will be rejected by Google Play.

7. **Forgetting to configure build.gradle** — Creating `key.properties` is not enough. The `build.gradle` file must load and reference it in the `signingConfigs` block. Without this, the keystore is ignored.

8. **Xcode automatic signing in CI** — Automatic signing requires Xcode to be logged into an Apple Developer account, which is not available in headless CI environments. Use manual signing with explicit certificates and profiles for CI/CD.

9. **Multiple keystores for different flavors** — If your app has build flavors (dev, staging, production), each may need its own signing config. Configure separate `signingConfigs` blocks for each flavor in `build.gradle`.

10. **Not enabling Play App Signing** — Google Play App Signing protects against keystore loss and enables key upgrades. All new apps are automatically enrolled. For existing apps, enrollment is strongly recommended but requires careful migration.

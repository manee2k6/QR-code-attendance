# AttendQR - App Store & Play Store Deployment Guide

## Prerequisites

- Flutter SDK >= 3.2.0 installed
- Xcode >= 15.0 (for iOS)
- Android Studio + Android SDK (for Android)
- Apple Developer Account ($99/year) for App Store
- Google Play Developer Account ($25 one-time) for Play Store

---

## Part 1: Pre-Deployment Setup

### 1.1 Update App Version

Edit `pubspec.yaml`:
```yaml
version: 1.0.0+1
# Format: major.minor.patch+buildNumber
# Increment buildNumber for every store upload
```

### 1.2 Configure Environment

Create `lib/config/env.dart` for production settings:
```dart
class Env {
  static const String apiBaseUrl = 'https://your-api-domain.com';
  static const String qrSecretKey = 'your-production-secret';
}
```

### 1.3 App Icons

Generate app icons using `flutter_launcher_icons`:
```yaml
# pubspec.yaml
dev_dependencies:
  flutter_launcher_icons: ^0.14.1

flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/images/app_icon.png"
  min_sdk_android: 21
```

```bash
dart run flutter_launcher_icons
```

### 1.4 Splash Screen

Configure native splash using `flutter_native_splash`:
```yaml
dev_dependencies:
  flutter_native_splash: ^2.4.0

flutter_native_splash:
  color: "#0A0E1A"
  image: assets/images/splash_logo.png
  android_12:
    color: "#0A0E1A"
    image: assets/images/splash_logo.png
```

```bash
dart run flutter_native_splash:create
```

---

## Part 2: Google Play Store (Android)

### 2.1 Create Signing Key

```bash
keytool -genkey -v -keystore ~/attendqr-upload-key.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias attendqr-key
```

**IMPORTANT**: Back up this keystore file securely. You cannot update your app without it.

### 2.2 Configure Signing

Create `android/key.properties` (do NOT commit to git):
```properties
storePassword=your-store-password
keyPassword=your-key-password
keyAlias=attendqr-key
storeFile=/Users/yourname/attendqr-upload-key.jks
```

Add to `.gitignore`:
```
android/key.properties
*.jks
```

### 2.3 Update android/app/build.gradle

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ...
    defaultConfig {
        applicationId "com.attendqr.enterprise"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }

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
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

### 2.4 Android Permissions

Ensure `android/app/src/main/AndroidManifest.xml` has:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

### 2.5 Build App Bundle

```bash
flutter clean
flutter pub get
flutter build appbundle --release
```

Output: `build/app/outputs/bundle/release/app-release.aab`

### 2.6 Upload to Play Console

1. Go to [Google Play Console](https://play.google.com/console)
2. **Create app** → Fill in app details
3. **Store listing**:
   - App name: `AttendQR Enterprise`
   - Short description: `QR-based attendance management with geofencing`
   - Full description: (see below)
   - Screenshots: Phone (2-8), Tablet (optional), 7-inch (optional)
   - Feature graphic: 1024x500 PNG
   - App icon: 512x512 PNG
4. **Content rating**: Complete the questionnaire
5. **Pricing & distribution**: Set countries, free/paid
6. **App content**: Privacy policy URL required
7. **Production** → **Create release** → Upload `.aab` file
8. **Review** → Submit for review (typically 1-3 days)

### 2.7 Play Store Listing Copy

**Short Description (80 chars):**
```
QR-based attendance with geofencing, analytics & payroll integration
```

**Full Description:**
```
AttendQR Enterprise is a comprehensive attendance management system that combines QR code scanning with GPS geofencing to ensure accurate, tamper-proof attendance tracking.

KEY FEATURES:
• QR Code Attendance — Generate time-limited QR codes, scan to mark attendance
• GPS Geofencing — Verify employee presence within office radius
• Device Fingerprinting — Prevent duplicate check-ins from same device
• Real-time Dashboard — Live attendance stats, charts, and employee logs
• Attendance Analytics — Weekly trends, percentage tracking, date filtering
• Payroll Integration — Connect with Zoho, Slack HR, Paychex, and more
• White Label — Full brand customization for multi-tenant deployments
• Database Config — MongoDB and PostgreSQL support
• Security — HMAC-SHA256 QR signing, rate limiting, session management

IDEAL FOR:
• Schools & Universities
• Corporates & Enterprises
• Co-working Spaces
• Events & Conferences

Built with enterprise-grade security and designed for scale.
```

---

## Part 3: Apple App Store (iOS)

### 3.1 Prerequisites

- Mac with Xcode 15+
- Apple Developer Program membership ($99/year)
- Valid Apple ID enrolled in the program

### 3.2 Xcode Configuration

Open `ios/Runner.xcworkspace` in Xcode:

1. **Signing & Capabilities**:
   - Team: Select your Apple Developer team
   - Bundle Identifier: `com.attendqr.enterprise`
   - Signing: Automatic

2. **Info.plist permissions** (ios/Runner/Info.plist):
```xml
<key>NSCameraUsageDescription</key>
<string>AttendQR needs camera access to scan QR codes for attendance marking</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>AttendQR needs your location to verify you are within the office geofence</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>AttendQR uses background location for geofence monitoring</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>AttendQR needs photo library access to upload profile pictures and logos</string>
```

3. **Deployment target**: iOS 15.0 minimum

### 3.3 Update ios/Runner/Info.plist

Ensure these are set:
```xml
<key>CFBundleDisplayName</key>
<string>AttendQR</string>
<key>CFBundleName</key>
<string>AttendQR</string>
<key>CFBundleVersion</key>
<string>1</string>
<key>CFBundleShortVersionString</key>
<string>1.0.0</string>
```

### 3.4 Build for iOS

```bash
flutter clean
flutter pub get
flutter build ios --release
```

### 3.5 Archive & Upload

**Option A: Xcode (Recommended)**
1. Open `ios/Runner.xcworkspace` in Xcode
2. Select `Any iOS Device` as destination
3. Menu: **Product** → **Archive**
4. In Organizer: **Distribute App** → **App Store Connect**
5. Follow prompts to upload

**Option B: Command Line**
```bash
flutter build ipa --release
```
Output: `build/ios/ipa/attendqr.ipa`

Upload using:
```bash
xcrun altool --upload-app \
  --type ios \
  --file build/ios/ipa/attendqr.ipa \
  --apiKey YOUR_API_KEY \
  --apiIssuer YOUR_ISSUER_ID
```

### 3.6 App Store Connect Setup

1. Go to [App Store Connect](https://appstoreconnect.apple.com)
2. **My Apps** → **+** → **New App**
3. Fill in:
   - Platform: iOS
   - Name: AttendQR Enterprise
   - Primary Language: English (US)
   - Bundle ID: com.attendqr.enterprise
   - SKU: attendqr-enterprise-001
4. **App Information**:
   - Category: Business
   - Subcategory: Productivity
   - Content Rights: Does not contain third-party content
5. **Pricing and Availability**: Set price tier / free
6. **Version Information**:
   - Screenshots: 6.7" (1290x2796), 6.5" (1284x2778), 5.5" (1242x2208)
   - iPad screenshots (optional): 12.9" (2048x2732)
   - Description, Keywords, Support URL, Marketing URL
   - Build: Select the uploaded build
7. **App Review Information**:
   - Demo account credentials: admin / admin123
   - Notes: Explain QR scanning requires camera
8. **Submit for Review** (typically 1-2 days)

### 3.7 App Store Keywords
```
qr attendance, attendance tracker, geofence attendance, employee tracking,
qr code scanner, office check-in, attendance management, payroll integration,
enterprise attendance, workplace attendance
```

---

## Part 4: CI/CD with GitHub Actions

### 4.1 Android CI/CD

Create `.github/workflows/android.yml`:
```yaml
name: Android Build & Deploy

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          channel: 'stable'

      - run: flutter pub get

      - run: flutter test

      - run: flutter build appbundle --release
        env:
          SIGNING_KEY: ${{ secrets.SIGNING_KEY }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}

      - uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.PLAY_SERVICE_ACCOUNT }}
          packageName: com.attendqr.enterprise
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: internal
```

### 4.2 iOS CI/CD

Create `.github/workflows/ios.yml`:
```yaml
name: iOS Build & Deploy

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          channel: 'stable'

      - run: flutter pub get

      - run: flutter test

      - run: flutter build ipa --release --export-options-plist=ios/ExportOptions.plist

      - uses: apple-actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/attendqr.ipa
          issuer-id: ${{ secrets.APP_STORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APP_STORE_KEY_ID }}
          api-private-key: ${{ secrets.APP_STORE_PRIVATE_KEY }}
```

---

## Part 5: Post-Deployment

### 5.1 Monitoring

- **Android**: Google Play Console → Android Vitals (crashes, ANRs)
- **iOS**: App Store Connect → App Analytics, Xcode Organizer (crash logs)
- **Backend**: Monitor API health at `/health` endpoint

### 5.2 Updates

1. Increment version in `pubspec.yaml`:
   ```yaml
   version: 1.1.0+2  # version+buildNumber
   ```
2. Build new bundle/IPA
3. Upload to respective store
4. Submit for review

### 5.3 Testing Tracks

- **Android**: Internal testing → Closed testing → Open testing → Production
- **iOS**: TestFlight (internal → external) → App Store

### 5.4 Checklist Before Every Release

- [ ] Version bumped in `pubspec.yaml`
- [ ] All API endpoints point to production
- [ ] QR secret key is production key
- [ ] Demo credentials removed or documented
- [ ] Privacy policy URL is live
- [ ] Screenshots updated for new features
- [ ] All permissions have user-facing descriptions
- [ ] `flutter analyze` passes with no errors
- [ ] `flutter test` passes
- [ ] Tested on physical iOS and Android devices
- [ ] ProGuard rules don't strip needed code
- [ ] App icons and splash screen configured
- [ ] Deep links configured (if applicable)

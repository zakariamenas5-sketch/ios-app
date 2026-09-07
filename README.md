# Maktabti (مكتبتي) — iOS App

A lightweight SwiftUI wrapper app that opens a local Flask web application (personal movie/TV library) inside `SFSafariViewController`.

## App Details

- **App Name**: Maktabti (مكتبتي = "My Library")
- **Bundle Identifier**: `com.hackx.maktabti`
- **Minimum iOS Version**: 14.0
- **Repository**: https://github.com/zakariamenas5-sketch/ios-app

## Features

- **Local Server Access**: Opens a user-configurable local Flask web app (default: `http://192.168.1.100:5000`)
- **Settings Screen**: Arabic-first UI (RTL) to change server URL at runtime, stored in `AppStorage`
- **Dark Mode**: Default dark color scheme
- **Portrait + Landscape**: Supports both orientations
- **Unsigned Build**: IPA is built **unsigned** via GitHub Actions; signing is done manually on-device with KSign

## How It Works

1. **Build Phase** (CI): GitHub Actions generates an unsigned IPA using XcodeGen + `xcodebuild`
2. **Distribution**: The unsigned IPA is available as an artifact
3. **Signing Phase** (Manual): On the iPhone, KSign signs the IPA directly (no Apple certificates in the repo or CI)

## Building Locally

### Prerequisites

- macOS with Xcode 15+
- `xcodegen` (install via `brew install xcodegen`)

### Steps

1. **Generate Xcode project**:
   ```bash
   xcodegen generate
   ```

2. **Build unsigned archive**:
   ```bash
   xcodebuild archive \
     -project MaktabtiApp.xcodeproj \
     -scheme Maktabti \
     -configuration Release \
     -derivedDataPath build \
     -archivePath build/Maktabti.xcarchive \
     CODE_SIGNING_ALLOWED=NO \
     CODE_SIGNING_REQUIRED=NO \
     CODE_SIGN_IDENTITY=""
   ```

3. **Export unsigned IPA**:
   ```bash
   xcodebuild -exportArchive \
     -archivePath build/Maktabti.xcarchive \
     -exportOptionsPlist exportOptions.plist \
     -exportPath build/export
   ```

   The IPA will be at `build/export/Maktabti.ipa`

## Re-triggering the Workflow

After making changes:

```bash
git add .
git commit -m "Your commit message"
git push origin main
```

This will automatically trigger the **Build unsigned IPA** workflow. Monitor it at:
```
https://github.com/zakariamenas5-sketch/ios-app/actions
```

Or manually trigger:
```bash
gh workflow run build-ios.yml
```

## Project Structure

```
ios-app/
├── Sources/
│   ├── MaktabtiApp.swift       # App entry point
│   └── ContentView.swift       # Safari wrapper + Settings
├── project.yml                 # XcodeGen configuration
├── Info.plist                  # iOS app metadata
├── exportOptions.plist         # Xcode export configuration
├── .github/workflows/
│   └── build-ios.yml          # GitHub Actions workflow
├── .gitignore
└── README.md
```

## Configuration

### Server URL

The default server URL is `http://192.168.1.100:5000` (stored in code at `Sources/ContentView.swift`).

Users can change it at runtime via the Settings screen (gear icon, top-right).

**Important**: The iPhone and Flask server must be on the same Wi-Fi network.

### Display Name

- **App Display Name**: مكتبتي (Maktabti)
- **Configured in**: `project.yml` → `CFBundleDisplayName`

## Constraints

- ✅ No code signing configuration, certificates, or provisioning profiles in the repo or CI
- ✅ Signing happens **manually** on-device via KSign
- ✅ Swift and XcodeGen build process is unsigned-friendly
- ✅ Core behavior unchanged: SFSafariViewController + user-editable AppStorage URL

## Troubleshooting

### Build fails with "code signing required"
Ensure `CODE_SIGNING_ALLOWED=NO` and `CODE_SIGNING_REQUIRED=NO` are set in the build command and Xcode project settings.

### Artifact is empty
Check the GitHub Actions logs:
```bash
gh run view <run-id> --log
```

### IPA doesn't contain app bundle
After downloading the artifact, inspect it:
```bash
unzip Maktabti-unsigned-ipa.zip
ls -la Payload/Maktabti.app/
cat Payload/Maktabti.app/Info.plist
```

## Author

Built for a personal movie/TV library management system.

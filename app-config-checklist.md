# React Native App Configuration & Distribution Checklist

Configuration and distribution checklist for React Native and Expo applications.

**References:**
- [Expo app.json / app.config.js](https://docs.expo.dev/versions/latest/config/app/)
- [EAS Build Configuration](https://docs.expo.dev/build/eas-json/)
- [EAS Update](https://docs.expo.dev/eas-update/introduction/)
- [Expo Permissions](https://docs.expo.dev/guides/permissions/)

## Reference Summary

### 1. app.json / app.config.js Fields

**Required fields:**
- `name` -- display name shown to users
- `slug` -- URL-friendly identifier for the project (used in Expo URLs)
- `version` -- user-facing version string (maps to `CFBundleShortVersionString` on iOS, `versionName` on Android)

**Platform identifiers (required for store builds):**
- `ios.bundleIdentifier` -- unique reverse-DNS identifier for the App Store (e.g., `com.company.myapp`)
- `ios.buildNumber` -- maps to `CFBundleVersion`, must be incremented for each App Store submission
- `android.package` -- reverse-DNS package name for the Play Store
- `android.versionCode` -- positive integer, must be strictly incremented for each Play Store release

**Recommended fields:**
- `icon` -- 1024x1024 PNG, no transparency, used as the base app icon
- `scheme` -- URL scheme for deep linking (must begin with a lowercase letter)
- `userInterfaceStyle` -- `"light"`, `"dark"`, or `"automatic"` (follows system setting)
- `orientation` -- `"default"`, `"portrait"`, or `"landscape"`
- `owner` -- Expo account that owns the project
- `sdkVersion` -- Expo SDK version the project targets
- `runtimeVersion` -- compatibility key for OTA updates; ensures updates only reach compatible native builds
- `platforms` -- defaults to `["ios", "android"]`

**iOS-specific:**
- `ios.infoPlist` -- arbitrary Info.plist key-value pairs (permission descriptions, custom settings)
- `ios.associatedDomains` -- array for Universal Links (e.g., `["applinks:example.com"]`)
- `ios.appleTeamId` -- Apple Developer team identifier
- `ios.supportsTablet` -- defaults to `false`

**Android-specific:**
- `android.permissions` -- array of Android permissions to include
- `android.intentFilters` -- deep linking and App Links configuration
- `primaryColor` -- tint color in the Android multitasker

**Other notable fields:**
- `backgroundColor` -- root view background color
- `extra` -- custom data accessible at runtime via `Constants.expoConfig.extra`
- `plugins` -- config plugins that modify native projects at prebuild time
- `developmentClient` -- settings for Expo dev client builds
- `updates` -- configure `expo-updates` with automatic check behavior, `fallbackToCacheTimeout`, and code signing

### 2. EAS Build Profile Configuration (eas.json)

**Standard profiles:**
- **development** -- `developmentClient: true`, `distribution: "internal"`. Used for local dev with the Expo dev client.
- **preview** -- no dev tools, `distribution: "internal"`. Used for team/QA testing via direct install.
- **production** -- default profile for `eas build`. Used for store submission.

**Key configuration options:**
- Custom profile names are supported; `eas build --profile <name>` selects the profile (`production` is the default when `--profile` is omitted)
- `extends` -- inherit from another profile to share config (up to 5 levels of nesting)
- `distribution` -- `"internal"` for direct device install (ad hoc / internal testing), omit or `"store"` for App Store / Play Store
- `developmentClient: true` -- builds include the Expo dev client
- `simulator: true` -- produces an iOS Simulator build (use with development profile)
- Platform-specific overrides via `android` and `ios` objects nested within a profile
- `env` -- environment variables injected into the build
- Build tool versions: specify `node`, `npm`, `yarn`, `ruby` versions per profile
- `resourceClass` -- `"medium"` (default) or `"large"` for more build resources

**Best practices:**
- Use `extends` to share common config between profiles and reduce duplication
- Create separate simulator profiles for iOS Simulator builds
- Use `autoIncrement` in production profiles to avoid manual version bumps

### 3. EAS Update Setup and Limitations

**What it does:**
- Cloud service for over-the-air (OTA) updates via the `expo-updates` library
- Updates JavaScript, styling, images, and other non-native assets without going through the app store

**Setup steps:**
1. Install `expo-updates` package
2. Run `eas update:configure` to set up the project
3. Create a new native build that includes `expo-updates`
4. Publish updates: `eas update --channel production --message "description of change"`

**Channels and runtime versions:**
- Updates are published to a specific channel (e.g., `"production"`, `"preview"`)
- Each build profile in `eas.json` should specify a `channel`
- `runtimeVersion` policy ensures updates are only delivered to compatible native builds -- this is critical to prevent crashes from native/JS mismatches

**Update behavior:**
- By default, checks for updates on each app launch
- Customizable via the Updates API and the `useUpdates()` hook
- Deployments dashboard provides visibility into update reach
- Republish feature allows reverting a problematic update quickly
- Supports percentage-based rollouts and CI/CD automation

**What CAN be updated OTA:**
- JavaScript bug fixes
- UI and styling changes
- Translations and copy changes
- Image and asset swaps

**What CANNOT be updated OTA (requires new native build):**
- Native code changes (new native modules, native config changes)
- Permission modifications (Info.plist or AndroidManifest changes)
- Expo SDK version upgrades
- Any change that alters the native binary

**Compliance:** Updates must comply with Apple App Store and Google Play Store guidelines for OTA updates.

**Billing:** Monthly active users are counted as unique installations that download at least one update.

### 4. Permission Declaration Patterns

**Android permissions:**
- Use `android.permissions` in app.config to explicitly add permissions
- Most permissions are auto-included by config plugins when you install the corresponding Expo package (e.g., installing `expo-camera` auto-adds `CAMERA`)
- Only manually add permissions for extras not covered by plugins (e.g., `SCHEDULE_EXACT_ALARM`)
- Use `android.blockedPermissions` to remove unwanted permissions that were auto-added (operates at the package level only)

**iOS permissions:**
- Set permission usage description strings in `ios.infoPlist` (Apple requires a human-readable reason for each permission)
- Example: `"NSCameraUsageDescription": "Take photos for your profile"`
- Alternative: use config plugin properties (e.g., `expo-media-library`'s `photosPermission` prop) which set the Info.plist value for you
- Important: Info.plist changes (including permission descriptions) CANNOT be updated OTA -- they require building and submitting a new native binary

**Web permissions:**
- Requires HTTPS or localhost for permission APIs to work

**Testing permissions:**
- To re-test a rejected permission prompt, you must uninstall and reinstall the app (the OS prevents repeated permission prompts after a denial)

**Audit checklist for permissions:**
- Every declared permission should have a corresponding package in `package.json` that actually uses it
- Every package that requires a permission should have the permission declared in config
- iOS permission descriptions should be clear, specific, and explain why the app needs access (vague descriptions cause App Store rejections)
- Check for over-declared permissions that inflate the permission list unnecessarily
- Verify `android.blockedPermissions` is used to strip any auto-added permissions the app does not need

---

## 1. App Identity

#### What to Check:
```json
// app.json or app.config.js
{
  "expo": {
    "name": "My App",                    // Required: display name
    "slug": "my-app",                    // Required: URL-safe identifier
    "version": "1.0.0",                  // Required: user-facing version
    "orientation": "portrait",           // Should be explicitly set
    "icon": "./assets/icon.png",         // Required: 1024x1024
    "ios": {
      "bundleIdentifier": "com.company.myapp",  // Required: reverse domain
      "buildNumber": "1"                          // Required: increment per build
    },
    "android": {
      "package": "com.company.myapp",    // Required: reverse domain
      "versionCode": 1                   // Required: increment per build
    }
  }
}
```

#### Grep Patterns:
- `bundleIdentifier` — check format (reverse domain notation)
- `"package"` in android config — check format
- `"version"` — verify present and follows semver
- `buildNumber` and `versionCode` — verify present

#### Severity: HIGH for missing identifiers, MEDIUM for format issues

---

## 2. Build Configuration (EAS)

#### What to Check:
```json
// eas.json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

#### Grep Patterns:
- `eas.json` file presence
- `"development"`, `"preview"`, `"production"` profiles — all three should exist
- `"autoIncrement"` — recommended for production builds
- `"distribution"` — should be `"store"` for production

#### Severity: MEDIUM for missing profiles, LOW for missing autoIncrement

---

## 3. Permissions Audit

#### What to Check:
```json
// app.json - Only declare permissions that are actually used
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "Take profile photos",
        "NSLocationWhenInUseUsageDescription": "Find nearby events"
      }
    },
    "android": {
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION"
      ]
    }
  }
}
```

#### Cross-Reference Check:
For each permission declared, verify the corresponding package is in `package.json`:
- `NSCameraUsageDescription` → `expo-camera` or `expo-image-picker`
- `NSLocationWhenInUseUsageDescription` → `expo-location`
- `NSMicrophoneUsageDescription` → `expo-av` or `expo-camera`
- `NSPhotoLibraryUsageDescription` → `expo-image-picker` or `expo-media-library`
- `NSContactsUsageDescription` → `expo-contacts`
- `NSCalendarsUsageDescription` → `expo-calendar`
- `CAMERA` → `expo-camera`
- `ACCESS_FINE_LOCATION` → `expo-location`
- `RECORD_AUDIO` → `expo-av`
- `READ_CONTACTS` → `expo-contacts`

#### Grep Patterns:
- `infoPlist` — check declared iOS permissions
- `"permissions"` in android config — check declared Android permissions
- `expo-camera`, `expo-location`, `expo-notifications`, etc. in `package.json` — cross-reference with declared permissions

#### Severity: HIGH for undeclared permissions (app rejection risk), MEDIUM for unused declared permissions

---

## 4. Over-the-Air Updates

#### What to Check:
```json
// app.json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/project-id",
      "enabled": true,
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": {
      "policy": "appVersion"
    }
  }
}

// eas.json
{
  "build": {
    "production": {
      "channel": "production"
    }
  }
}
```

#### Grep Patterns:
- `expo-updates` in `package.json` — check if OTA is configured
- `"updates"` in `app.json` — check URL and enabled status
- `"runtimeVersion"` — verify policy is set (prevents incompatible updates)
- `"channel"` in eas.json build profiles — verify channels configured

#### Severity: MEDIUM for missing OTA config, HIGH for missing runtimeVersion policy

---

## 5. Splash Screen & App Icon

#### What to Check:
```json
// app.json
{
  "expo": {
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    }
  }
}
```

#### Grep Patterns:
- `"icon"` — verify icon path exists and is 1024x1024
- `"splash"` — verify splash configuration
- `"adaptiveIcon"` — verify Android adaptive icon configured
- `expo-splash-screen` in `package.json` — check for programmatic splash control

#### Severity: MEDIUM for missing splash config, LOW for missing adaptive icon

---

## 6. Deep Linking

#### What to Check:
```json
// app.json
{
  "expo": {
    "scheme": "myapp",
    "ios": {
      "associatedDomains": ["applinks:example.com"]
    },
    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [{ "scheme": "https", "host": "example.com" }],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

#### Grep Patterns:
- `"scheme"` in app.json — check URL scheme configured
- `associatedDomains` — check universal links (iOS)
- `intentFilters` — check App Links (Android)
- `expo-linking` or `expo-router` — check deep link handling code

#### Severity: MEDIUM for missing scheme, LOW for missing universal/app links

---

## 7. Environment Configuration

#### What to Check:
```
# .env files should be in .gitignore
.env
.env.local
.env.production

# EXPO_PUBLIC_ vars are bundled into JS — no secrets!
EXPO_PUBLIC_API_URL=https://api.example.com   # OK
EXPO_PUBLIC_API_KEY=sk_live_abc123            # CRITICAL - secret exposed!
```

#### Grep Patterns:
- `.env` in `.gitignore` — verify env files excluded
- `EXPO_PUBLIC_` in `.env*` files — check for secrets
- `process.env.EXPO_PUBLIC_` — check usage in code

#### Severity: CRITICAL for secrets in EXPO_PUBLIC_, HIGH for .env not in .gitignore

---

## 8. Dark Mode & Theming

#### What to Check:
```json
// app.json
{
  "expo": {
    "userInterfaceStyle": "automatic"
  }
}
```

Options: `"automatic"` (follows system), `"light"`, `"dark"`

#### Grep Patterns:
- `userInterfaceStyle` — check if configured
- `useColorScheme` — check if dark mode is handled in code
- `Appearance` — check for appearance change listeners

#### Severity: LOW

---

## Configuration Severity Levels

### CRITICAL (Fix Immediately)
- Secrets in `EXPO_PUBLIC_` environment variables
- `.env` files not in `.gitignore`

### HIGH (Fix This Week)
- Missing bundle identifier or package name
- Missing version/buildNumber/versionCode
- Permissions declared but not used (or used but not declared)
- Missing runtimeVersion policy for OTA updates

### MEDIUM (Fix This Sprint)
- Missing EAS build profiles
- Missing OTA update configuration
- Missing splash screen configuration
- Missing deep linking scheme
- Missing app icon

### LOW (Nice to Have)
- Missing adaptive icon (Android)
- Missing dark mode support
- Missing autoIncrement in build config

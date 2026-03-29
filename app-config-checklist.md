# React Native App Configuration & Distribution Checklist

Configuration and distribution checklist for React Native and Expo applications.

**References:**
- [Expo app.json / app.config.js](https://docs.expo.dev/versions/latest/config/app/)
- [EAS Build Configuration](https://docs.expo.dev/build/eas-json/)
- [EAS Update](https://docs.expo.dev/eas-update/introduction/)
- [Expo Permissions](https://docs.expo.dev/guides/permissions/)

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

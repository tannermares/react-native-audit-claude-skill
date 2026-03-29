---
name: react-native-audit
description: Comprehensive React Native/Expo application audit covering coding standards, accessibility, security, performance, testing, app configuration, and generates results as a Markdown report
user-invocable: true
argument-hint: output-filename
---

# React Native Application Audit

Perform a comprehensive audit of a React Native application covering:

1. **Coding Standards & Best Practices**
2. **Component Architecture**
3. **Accessibility (A11y)**
4. **Performance**
5. **Security**
6. **Testing & Coverage**
7. **Error Handling & Resilience**
8. **App Configuration & Distribution**

## Audit Process

### Phase 1: Project Discovery & Setup

1. **Identify Project Type**
   - Read `package.json` to understand dependencies and React Native version
   - Detect Expo: `expo` in dependencies → Expo project
   - Detect bare RN: `react-native` without `expo` → bare React Native
   - Check Expo SDK version from `expo` dependency

2. **Detect Navigation**
   - `expo-router` in deps → Expo Router (file-based routing in `app/`)
   - `@react-navigation/*` in deps → React Navigation
   - Check for navigation configuration files

3. **Detect Styling Approach**
   - `nativewind` in deps → NativeWind/Tailwind
   - `styled-components` in deps → styled-components
   - `tamagui` in deps → Tamagui
   - Default → StyleSheet.create

4. **Detect Architecture & Engine**
   - Check `app.json` for `"newArchEnabled": true` → New Architecture
   - Check `app.json` for `"jsEngine": "hermes"` → Hermes engine
   - Check for `eas.json` → EAS Build configured

5. **Detect Tooling**
   - TypeScript: `tsconfig.json` presence, count `.ts`/`.tsx` vs `.js`/`.jsx`
   - Linting: `eslint.config.*`, `.eslintrc.*`, `biome.json`
   - Testing: `jest.config.*`, `detox.config.*`, `maestro/` or `.maestro/` directory

6. **Establish Source Root & Metrics**
   - Auto-detect: `app/`, `src/`, `screens/`, `components/`
   - Read `.gitignore` to build exclusion list
   - Count screens, components, hooks, test files, native modules

### Phase 2: Coding Standards & Best Practices Audit

Evaluate code against React Native standards. Reference: `coding-standards-reference.md`

#### React Hooks (same as web)
- Search for hooks called conditionally: `if.*use[A-Z]`
- Search for index-as-key: `key={index}`, `key={i}`, `key={idx}`
- Search for `forceUpdate` usage

#### React Native Specific
- **Inline styles**: Search for `style={{` in render functions (should use `StyleSheet.create`)
- **Dimensions**: Search for `Dimensions.get(` — flag, suggest `useWindowDimensions`
- **SafeAreaView**: Check import source — `react-native-safe-area-context` (correct) vs `react-native` (deprecated)
- **KeyboardAvoidingView**: Check for platform-correct `behavior` prop
- **AppState**: Search for `AppState.addEventListener` — check for cleanup
- **StatusBar**: Check for StatusBar management

#### Navigation Patterns
- Screen/component separation — screens should compose components
- Deep linking configuration presence
- Typed navigation params (if TypeScript)

#### Styling (adapt to detected approach)
- NativeWind: check `className=` usage, verify config
- StyleSheet: flag inline `style={{}}`, verify `StyleSheet.create` usage
- Check for responsive patterns (`useWindowDimensions`)

### Phase 3: Component Architecture Audit

#### Component Size & Complexity
- Count lines per component (flag >300)
- Count `useState` calls per component (flag >5)
- Count `useEffect` calls per component (flag >3)
- Estimate props count (flag >8)

#### Separation of Concerns
- API calls directly in screens vs custom hooks
- Look for custom hooks: `use*.{ts,tsx,js,jsx}` files
- Business logic mixed with rendering

#### Reusability
- Duplicated patterns across screens
- Hardcoded strings that should be constants
- Opportunities to extract shared components/hooks

#### TypeScript Quality (if TS project)
- Search for `: any`, `as any` (flag each)
- Search for `@ts-ignore`, `@ts-expect-error`
- Search for excessive `!.` non-null assertions

#### Navigation Structure
- Nested navigator complexity
- Screen/component file organization
- Native module usage (wrapped in try/catch?)

### Phase 4: Accessibility (A11y) Audit

Reference: `a11y-checklist.md`

#### Accessible Labels
- Search for `<Pressable` without `accessibilityLabel`
- Search for `<TouchableOpacity` without `accessibilityLabel`
- Search for `<Image` without `accessibilityLabel` (meaningful images)
- Search for `<TextInput` without `accessibilityLabel`
- Check icon-only buttons for labels

#### Accessibility Roles
- Search for `<Pressable` without `accessibilityRole`
- Check for `accessibilityRole="button"` on interactive elements
- Check for `accessibilityRole="header"` on section headings

#### Accessibility State
- Search for `disabled={` on Pressable/Touchable — check for `accessibilityState`
- Check toggles/switches for `accessibilityState={{ checked: ... }}`

#### Touch Target Sizing
- Flag interactive elements with width/height below 44pt
- Check for `hitSlop` on small targets
- Check `minWidth`/`minHeight` on interactive elements

#### Font Scaling
- Search for `allowFontScaling={false}` — blocks user preference (HIGH)
- Search for `maxFontSizeMultiplier={1}` — effectively disables scaling

#### Focus Management
- Search for `<Modal` — check for `accessibilityViewIsModal`
- Search for `accessibilityLiveRegion` for dynamic content
- Check for `AccessibilityInfo.announceForAccessibility` usage

#### Gestures
- Search for `Swipeable`, `PanGestureHandler` — check for non-gesture alternatives
- Check for `accessibilityActions` and `onAccessibilityAction`

### Phase 5: Performance Audit

Reference: `coding-standards-reference.md` (anti-patterns section)

#### List Rendering
- Search for `<ScrollView` containing `.map(` — should be FlatList
- Search for `<FlatList` without `keyExtractor`
- Check for `getItemLayout` on known-height lists
- Check for `ListEmptyComponent` on FlatList
- Search for inline functions in `renderItem` prop

#### Animation Performance
- Search for `Animated.timing` or `Animated.spring` — check for `useNativeDriver: true`
- Search for `useNativeDriver: false` — flag
- Check for `react-native-reanimated` usage (preferred for complex animations)

#### Image Optimization
- Check for `expo-image` or `react-native-fast-image` vs raw `<Image`
- Search for `require(` inside render functions — should be top-level
- Check for image caching configuration

#### Bundle & Engine
- Verify Hermes engine enabled
- Count production dependencies (flag >50)
- Flag heavy deps: `moment` (suggest dayjs), full `lodash` (suggest lodash-es)
- Check for `console.log` without `__DEV__` guard (bridge traffic)

#### Memory Leaks
- Search for `navigation.addListener` — check for cleanup
- Search for `addEventListener` in useEffect — check for cleanup
- Search for `setInterval` in useEffect — check for cleanup

#### Rendering Optimization
- Search for `React.memo` usage
- Search for `useMemo`, `useCallback` usage
- Check for inline objects/functions in frequently-rendered list items

### Phase 6: Security Audit

Reference: `security-checklist.md`

#### Sensitive Data Storage
- Search for `AsyncStorage.setItem` with token/password/secret/auth data (CRITICAL)
- Check for `expo-secure-store` or `react-native-keychain` for sensitive data
- Search for hardcoded secrets: `API_KEY`, `SECRET`, `TOKEN`, `PASSWORD` with string values (CRITICAL)
- Check `EXPO_PUBLIC_` env vars for secrets

#### Network Security
- Search for `fetch(.*http://` — non-HTTPS calls
- Check for certificate pinning library
- Search for `credentials: 'include'` on cross-origin requests

#### Deep Linking
- Search for `Linking.openURL` — check URL validation
- Search for `Linking.addEventListener` — check URL validation in handler
- Check WebView usage: `javaScriptEnabled`, `originWhitelist`, `injectedJavaScript`

#### Code Injection
- Search for `eval(`, `new Function(` (CRITICAL)
- Search for `setTimeout(` with string argument

#### Debug Code
- Search for `console.log` without `__DEV__` guard
- Check `.env` files are in `.gitignore`

#### Third-Party Risks
- Run `npm audit` if package-lock.json exists
- Check for known vulnerable packages

### Phase 7: Testing & Coverage Audit

#### Test Framework Detection
- Check for Jest, React Native Testing Library, Detox, Maestro
- Count test files: `**/*.test.*`, `**/*.spec.*`, `**/__tests__/**`
- Check for Maestro flows: `maestro/`, `.maestro/` directories
- Calculate test-to-screen and test-to-component ratios

#### Test Quality (if tests exist)
- Search for `getByTestId` overuse (prefer `getByRole`, `getByLabelText`, `getByText`)
- Check for `fireEvent` vs `userEvent` usage
- Snapshot test overuse: `toMatchSnapshot`, `toMatchInlineSnapshot`
- Missing async handling: `waitFor`, `findBy`

#### E2E Testing
- Detox configuration presence
- Maestro flow files presence
- Critical user flow coverage

### Phase 8: Error Handling & Resilience Audit

#### Error Boundaries
- Search for `componentDidCatch`, `react-error-boundary`, or custom error boundaries
- Check if error boundaries wrap major navigation sections
- Check for granular boundaries (not just root)

#### Loading States
- Search for `ActivityIndicator`, `isLoading`, `loading`, `Skeleton`
- Verify async operations show loading state

#### Empty States
- Search for `ListEmptyComponent` on FlatList
- Search for `length === 0` empty state handling

#### Offline Handling (critical for mobile)
- Check for `@react-native-community/netinfo` or `NetInfo`
- Search for offline queue patterns
- Check for connectivity-aware data fetching

#### Permission Handling
- Check for graceful degradation when permissions denied
- Camera, location, notifications — check for denial handling

#### Crash Reporting
- Check for `sentry-expo`, `@sentry/react-native`, `@bugsnag`, `crashlytics`

#### App Lifecycle
- Search for `AppState` change handling (background/foreground)
- Check for cleanup on app background

#### Network Resilience
- Search for `.catch` or `try...catch` around API calls
- Check for timeout configuration
- Check for retry logic

### Phase 9: App Configuration & Distribution Audit

Reference: `app-config-checklist.md`

#### App Identity
- Check `app.json`/`app.config.js` completeness
- Verify bundle identifier format (`ios.bundleIdentifier`)
- Verify package name format (`android.package`)
- Check version and buildNumber/versionCode

#### Build Configuration
- Check for `eas.json` with dev/preview/production profiles
- Verify production distribution settings
- Check for autoIncrement

#### Permissions
- Cross-reference declared permissions with used packages
- Flag unused permission declarations
- Flag missing permission declarations for used packages

#### OTA Updates
- Check `expo-updates` configuration
- Verify runtimeVersion policy
- Check EAS Update channel configuration

#### Assets
- Verify app icon configuration
- Verify splash screen configuration
- Check Android adaptive icon

#### Deep Linking
- Check `scheme` in app.json
- Check universal links / App Links configuration

#### Environment
- Verify `.env` in `.gitignore`
- Check `EXPO_PUBLIC_` vars for secrets

### Phase 10: Generate Audit Report

1. **Create Markdown Report**: Write findings using `report-template.md`

2. **Scoring**:
   - 90-100: Excellent
   - 80-89: Good
   - 70-79: Fair
   - 60-69: Needs Attention
   - <60: Critical

3. **Output**: Save as `react-native-audit-report-[date].md` (or custom filename)

## Tool Usage Guidelines

1. **Respect .gitignore**: Exclude `node_modules/`, `android/`, `ios/` build dirs, `.expo/`
2. **Use Glob for file discovery**: Find TSX/JSX files, config files, test files
3. **Use Grep for pattern matching**: Search for anti-patterns, vulnerabilities
4. **Use Read for detailed analysis**: Read config files, critical screen files
5. **Use Bash sparingly**: Only for `npm audit` and similar system commands

## Command Usage

When invoked via `/react-native-audit [filename]`:
- If filename provided: Use as output filename
- If no filename: Use default `react-native-audit-report-YYYY-MM-DD.md`

## Execution Workflow

1. Announce audit start and estimated scope
2. Read `.gitignore` and build exclusion list
3. Discover project structure (Expo vs bare RN, navigation, styling)
4. Run each audit phase sequentially
5. Collect findings and metrics
6. Generate markdown report
7. Present summary and file location

## Important Notes

- Adapt all checks to detected project type (Expo vs bare RN)
- Provide specific file and line references
- Give constructive recommendations, not just criticism
- Prioritize findings by severity
- Include positive findings alongside issues
- Make the report actionable with clear next steps
- Do NOT generate PDF — output Markdown only
- Do NOT include any permissions or tool allowances

---

Begin the comprehensive React Native application audit now. Work through each phase systematically and generate a professional audit report.

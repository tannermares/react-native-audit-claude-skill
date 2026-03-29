# React Native Security Audit Checklist

Comprehensive security checklist for React Native and Expo applications.

**References:**
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [OWASP Mobile Application Security Checklist](https://mas.owasp.org/checklists/)
- [React Native Security](https://reactnative.dev/docs/security)

---

## 1. Sensitive Data Storage

#### What to Look For:
```jsx
// CRITICAL - Sensitive data in AsyncStorage (unencrypted, extractable)
await AsyncStorage.setItem('auth_token', token);
await AsyncStorage.setItem('user_password', password);
await AsyncStorage.setItem('api_secret', secret);

// SAFE - Use secure storage for sensitive data
// Expo:
import * as SecureStore from 'expo-secure-store';
await SecureStore.setItemAsync('auth_token', token);

// Bare RN:
import * as Keychain from 'react-native-keychain';
await Keychain.setGenericPassword(username, password);
```

#### Grep Patterns:
- `AsyncStorage\.setItem.*token` — token in AsyncStorage
- `AsyncStorage\.setItem.*password` — password in AsyncStorage
- `AsyncStorage\.setItem.*secret` — secret in AsyncStorage
- `AsyncStorage\.setItem.*auth` — auth data in AsyncStorage
- `AsyncStorage\.setItem.*key` — potential API key in AsyncStorage

#### Severity: CRITICAL for auth tokens/passwords, MEDIUM for non-sensitive preferences

---

## 2. JavaScript Bundle Secrets

#### What to Look For:
```jsx
// CRITICAL - Secrets in JS code (extractable from app binary)
const API_KEY = 'sk_live_abc123xyz';
const SECRET = 'my-super-secret-key';
const DATABASE_URL = 'postgres://user:pass@host/db';

// CRITICAL - Secrets in EXPO_PUBLIC_ env vars (bundled into JS)
const apiKey = process.env.EXPO_PUBLIC_API_SECRET;

// SAFE - Use server-side proxy for sensitive API calls
// SAFE - Use native config (not accessible from JS bundle)
// SAFE - EXPO_PUBLIC_ only for non-sensitive config (API base URL, feature flags)
```

#### Grep Patterns:
- `API_KEY\s*=\s*["']` — hardcoded API key
- `SECRET\s*=\s*["']` — hardcoded secret
- `PASSWORD\s*=\s*["']` — hardcoded password
- `TOKEN\s*=\s*["']` — hardcoded token
- `EXPO_PUBLIC_.*SECRET` — secret in public env var
- `EXPO_PUBLIC_.*KEY` — potential API key in public env var (review context)
- `EXPO_PUBLIC_.*PASSWORD` — password in public env var

#### Severity: CRITICAL

---

## 3. Network Security

#### What to Look For:
```jsx
// HIGH - Non-HTTPS API calls
fetch('http://api.example.com/data');
const BASE_URL = 'http://api.example.com';

// SAFE - HTTPS only
fetch('https://api.example.com/data');

// CHECK - Certificate pinning for high-security apps
// Using react-native-ssl-pinning or custom TrustManager
```

#### Grep Patterns:
- `fetch\(.*http:\/\/` — non-HTTPS fetch
- `http:\/\/` in API base URL constants — non-HTTPS configuration
- `ssl-pinning` or `certificate-pinning` — check for pinning library

#### Severity: HIGH for non-HTTPS, MEDIUM for missing cert pinning

---

## 4. Deep Linking & URL Handling

#### What to Look For:
```jsx
// HIGH - Opening URLs without validation
Linking.openURL(userProvidedUrl);

// HIGH - Custom URL scheme without validation
Linking.addEventListener('url', ({ url }) => {
  // No validation of URL content!
  navigateToScreen(url);
});

// SAFE - Validate URL before opening
const isValid = url.startsWith('https://') || url.startsWith('myapp://');
if (isValid) Linking.openURL(url);

// SAFE - Validate deep link parameters
Linking.addEventListener('url', ({ url }) => {
  const parsed = Linking.parse(url);
  if (isValidRoute(parsed.path)) {
    navigateToScreen(parsed.path);
  }
});
```

#### Grep Patterns:
- `Linking\.openURL\(` — check if URL is validated before opening
- `Linking\.addEventListener.*url` — deep link handler, check for validation
- `expo-linking` or `expo-router` — check deep link configuration

#### Severity: HIGH

---

## 5. WebView Security

#### What to Look For:
```jsx
// HIGH - WebView with JavaScript enabled and user-controlled URL
<WebView
  source={{ uri: userProvidedUrl }}
  javaScriptEnabled={true}
/>

// HIGH - Wildcard origin whitelist
<WebView originWhitelist={['*']} />

// HIGH - Injecting dynamic JavaScript
<WebView
  injectedJavaScript={`document.title = "${userInput}"`}
/>

// SAFE - Restricted WebView
<WebView
  source={{ uri: 'https://trusted-domain.com' }}
  originWhitelist={['https://trusted-domain.com']}
  javaScriptEnabled={false}
/>
```

#### Grep Patterns:
- `<WebView` — flag all WebView usage for review
- `originWhitelist=\{?\[?\s*["']\*` — wildcard origin
- `injectedJavaScript` — check for dynamic content injection
- `javaScriptEnabled=\{true\}` with dynamic `source` — potential risk

#### Severity: HIGH for user-controlled WebView, MEDIUM for static WebView

---

## 6. Code Injection

#### What to Look For:
```jsx
// CRITICAL - eval with any input
eval(userCode);

// CRITICAL - new Function with dynamic code
const fn = new Function(userInput);

// CRITICAL - setTimeout with string argument
setTimeout('alert("xss")', 1000);
```

#### Grep Patterns:
- `\beval\(` — direct eval usage
- `new\s+Function\(` — dynamic function creation
- `setTimeout\(\s*["']` — setTimeout with string

#### Severity: CRITICAL

---

## 7. Authentication & Biometrics

#### What to Look For:
```jsx
// CHECK - Biometric auth should have fallback
import * as LocalAuthentication from 'expo-local-authentication';
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: 'Authenticate',
  fallbackLabel: 'Use passcode',  // Should have fallback
});

// CHECK - Token refresh patterns
// CHECK - Session expiry handling
// CHECK - Background state handling (clear sensitive data)
```

#### Grep Patterns:
- `expo-local-authentication` or `react-native-biometrics` — check for fallback
- `refreshToken` or `token.*refresh` — verify refresh logic exists
- `AppState.*background` — check if sensitive data cleared on background

#### Severity: HIGH for missing auth fallback, MEDIUM for missing token refresh

---

## 8. Device Security

#### What to Look For:
```jsx
// CHECK - Jailbreak/root detection for high-security apps
import DeviceInfo from 'react-native-device-info';
const isRooted = await DeviceInfo.isRooted(); // Android
const isJailBroken = await DeviceInfo.isJailBroken(); // iOS

// CHECK - Screenshot prevention for sensitive screens
// Android: FLAG_SECURE
// iOS: UIScreen notifications
```

#### Grep Patterns:
- `isRooted` or `isJailBroken` or `jail-monkey` — jailbreak detection
- `FLAG_SECURE` — screenshot prevention

#### Severity: LOW for most apps, HIGH for financial/health apps

---

## 9. Debug & Development Code

#### What to Look For:
```jsx
// MEDIUM - Console output in production (bridge traffic on old arch, extractable)
console.log('User data:', userData);
console.log('API response:', response);

// CHECK - __DEV__ guard for development-only code
if (__DEV__) {
  console.log('Debug info');  // OK - only in development
}

// CHECK - Source maps not shipped in production
```

#### Grep Patterns:
- `console\.(log|debug|info|warn)` — console output (check for `__DEV__` guard)
- `__DEV__` — verify used to guard debug code
- `.env` files committed to repo (should be in .gitignore)

#### Severity: MEDIUM

---

## Security Severity Levels

### CRITICAL (Fix Immediately)
- Sensitive data (tokens, passwords) stored in AsyncStorage
- Hardcoded API keys or secrets in JS bundle
- Secrets in `EXPO_PUBLIC_` environment variables
- `eval()` or `new Function()` with dynamic input
- Authentication bypass

### HIGH (Fix This Week)
- Non-HTTPS API calls
- Deep linking without URL validation
- WebView with user-controlled URLs and JavaScript enabled
- Missing biometric auth fallback
- Wildcard origin whitelist on WebView

### MEDIUM (Fix This Sprint)
- `console.log` in production without `__DEV__` guard
- Missing certificate pinning (for high-security apps)
- Missing token refresh logic
- AsyncStorage for non-critical but private user data
- Source maps in production builds

### LOW (Fix When Convenient)
- Missing jailbreak/root detection
- Missing screenshot prevention
- Outdated dependencies (no known CVEs)

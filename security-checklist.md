# React Native Security Audit Checklist

Comprehensive security checklist for React Native and Expo applications.

**References:**
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [OWASP Mobile Application Security Checklist](https://mas.owasp.org/checklists/)
- [React Native Security](https://reactnative.dev/docs/security)

## Reference Summary

The following is a consolidated summary of the above references so auditors can work without visiting external URLs.

### OWASP Mobile Top 10 (2024)

| # | Category | Exploitability | Key Concerns |
|---|----------|---------------|--------------|
| M1 | **Improper Credential Usage** | EASY | Hardcoded credentials, improper credential handling, missing encryption for stored creds, lack of API key rotation |
| M2 | **Inadequate Supply Chain Security** | — | Unvetted third-party libraries, malicious SDKs, lack of dependency auditing |
| M3 | **Insecure Authentication/Authorization** | — | Weak auth mechanisms, missing server-side enforcement, improper session handling |
| M4 | **Insufficient Input/Output Validation** | — | Missing input sanitization, injection vectors, improper output encoding |
| M5 | **Insecure Communication** | EASY | Deprecated protocols, invalid certs, inconsistent SSL/TLS usage. Prevention: SSL/TLS everywhere, strong cipher suites, trusted CAs, certificate pinning, verify SSL chain, encrypt before transmission. iOS: fail-closed cert validation, NSURL pinning. Android: no unconditional cert acceptance, proper `checkServerTrusted` |
| M6 | **Inadequate Privacy Controls** | — | Excessive data collection, missing consent, PII leakage |
| M7 | **Insufficient Binary Protections** | — | Missing code obfuscation, reverse engineering exposure, lack of tamper detection |
| M8 | **Security Misconfiguration** | DIFFICULT | Insecure defaults, weak encryption, world-readable permissions, debugging enabled in prod, cleartext HTTP allowed. Prevention: secure defaults, no hardcoded creds, least privilege, disallow cleartext traffic, cert pinning, disable debugging in prod, disable Android backup for sensitive data |
| M9 | **Insecure Data Storage** | EASY | Weak/absent encryption, plaintext on filesystem, insufficient access controls. Prevention: strong encryption at rest and in transit, platform-specific storage (Keychain/Keystore), access controls, input validation, session management |
| M10 | **Insufficient Cryptography** | — | Weak algorithms, improper key management, deprecated crypto functions |

### OWASP MAS Checklist Categories (mapped to React Native)

**MASVS-STORAGE** -- Secure storage, backup exclusion, prevent exposure through logs/UI/keyboard caches.
- RN concern: AsyncStorage is unencrypted. Use `expo-secure-store` or `react-native-keychain` for sensitive data. Exclude sensitive data from Android backups. Watch for Redux persist writing tokens to AsyncStorage.

**MASVS-CRYPTO** -- Proper key generation, secure algorithms, no hardcoded credentials.
- RN concern: Never embed API keys or secrets in JS bundle (extractable in plain text). `react-native-dotenv` and `react-native-config` are NOT for secrets. Use server-side orchestration for secret-dependent API calls.

**MASVS-AUTH** -- Biometric auth, credential management, step-up auth, server-side enforcement.
- RN concern: Biometric auth (`expo-local-authentication`, `react-native-biometrics`) must have fallback. OAuth2 must use PKCE (SHA-256). Recommended library: `react-native-app-auth`. Server-side session validation is mandatory.

**MASVS-NETWORK** -- Encrypted transmission, TLS config, cert pinning, hostname verification, no cleartext.
- RN concern: All endpoints must use HTTPS. SSL pinning prevents MITM but requires cert rotation planning (certs expire every 1-2 years; app updates needed when server cert renews). Check `NSAppTransportSecurity` (iOS) and `NetworkSecurityConfig` (Android) for cleartext exceptions.

**MASVS-PLATFORM** -- Permission management, WebView security, deep link validation, IPC protection, no sensitive data in notifications/screenshots.
- RN concern: Deep linking is NOT secure -- no centralized URL scheme registration, malicious apps can hijack custom schemes. Never send sensitive info in deep links. Prefer Universal Links (iOS) over custom URL schemes. WebViews with user-controlled URLs and JS enabled are high risk.

**MASVS-CODE** -- Dependency vulnerabilities, input validation, injection prevention, safe deserialization, updated platform versions.
- RN concern: `eval()`, `new Function()`, `setTimeout` with string args are injection vectors. Audit npm dependencies regularly. Keep React Native and Expo SDK versions current.

**MASVS-RESILIENCE** -- Code obfuscation, root/jailbreak detection, debugger detection, app integrity verification.
- RN concern: JS bundle is readable by default. For high-security apps, add jailbreak/root detection (`react-native-device-info`, `jail-monkey`), debugger detection, and consider Hermes bytecode for light obfuscation. Use `FLAG_SECURE` (Android) for screenshot prevention on sensitive screens.

### React Native Official Security Guidance

**Secrets and API Keys:**
- NEVER hardcode API keys in code -- anything in the JS bundle is accessible in plain text
- `react-native-dotenv` and `react-native-config` set environment-specific variables but are NOT for secrets (still bundled)
- Build a server-side orchestration layer (serverless functions, API gateway) to proxy calls that require secrets

**Storage:**
- `AsyncStorage` is unencrypted key-value storage -- use ONLY for non-sensitive data (preferences, UI state)
- NEVER store tokens, secrets, or passwords in AsyncStorage
- iOS: use Keychain Services. Android: use `EncryptedSharedPreferences` or Keystore
- Recommended libraries: `expo-secure-store`, `react-native-keychain`
- Caution: do not persist sensitive form data through Redux to AsyncStorage; do not send tokens to monitoring services (Sentry, Crashlytics)

**Deep Linking:**
- Deep linking has no centralized URL scheme registration -- malicious apps can register the same scheme
- Never send sensitive information via deep links
- iOS: prefer Universal Links (associated domains, more secure) over custom URL schemes
- Always validate deep link parameters before acting on them

**Authentication:**
- OAuth2 flows must use PKCE (Proof Key for Code Exchange, SHA-256 based)
- Recommended library: `react-native-app-auth`

**Network Security:**
- All API communication must use SSL/TLS (HTTPS only)
- SSL Pinning: embed trusted certificates in the app to prevent MITM attacks
- Pinning caveat: certificates expire every 1-2 years; you must update the app when the server certificate renews -- plan a rotation strategy

**Quick Audit Checklist (from React Native docs):**
1. No hardcoded keys or secrets in JS bundle
2. Backend orchestration for secret-dependent API calls
3. Appropriate storage (secure store for sensitive data, AsyncStorage only for preferences)
4. No sensitive data transmitted via deep links
5. PKCE for all OAuth2 flows
6. SSL/TLS on every endpoint
7. Certificate pinning for sensitive apps with a cert rotation plan
8. Audit all data persisted to disk (AsyncStorage, Redux persist, file system)
9. Universal Links preferred over custom URL schemes on iOS

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

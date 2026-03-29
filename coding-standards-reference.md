# React Native Coding Standards Reference

Standards for auditing React Native applications.

**References:**
- [React Native](https://reactnative.dev/)
- [Expo Router](https://docs.expo.dev/router/introduction/)
- [React Navigation](https://reactnavigation.org/docs/getting-started)

---

## React Hooks Rules

Same rules as React web — hooks must be called at the top level, never conditionally.

```jsx
// GOOD - Hooks at top level
function ProfileScreen({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <ActivityIndicator />;
  return <UserProfile user={user} />;
}

// BAD - Hook after conditional return
function ProfileScreen({ userId }) {
  if (!userId) return null;  // Early return BEFORE hooks!
  const [user, setUser] = useState(null);  // Violates rules of hooks
}
```

#### Grep Patterns:
- `if.*\buse[A-Z]` — conditional hook call
- `for.*\buse[A-Z]` or `\.forEach.*\buse[A-Z]` — hook in loop
- `\?\s*use[A-Z]` — ternary hook call

---

## Naming Conventions

```
GOOD:
  ProfileScreen.tsx      PascalCase screen file
  UserAvatar.tsx         PascalCase component file
  useAuth.ts             camelCase with use prefix for hooks
  formatDate.ts          camelCase for utilities
  constants.ts           camelCase for config files
  MAX_RETRIES = 3        SCREAMING_SNAKE_CASE for constants

BAD:
  profile-screen.tsx     kebab-case
  userAvatar.tsx         camelCase component
  UseAuth.ts             PascalCase hook
```

---

## StyleSheet Patterns

```jsx
// GOOD - StyleSheet.create outside component (cached, optimized)
const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 24, fontWeight: 'bold' },
});

function MyComponent() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello</Text>
    </View>
  );
}

// BAD - Inline style objects (new object created every render)
function MyComponent() {
  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text style={{ fontSize: 24, fontWeight: 'bold' }}>Hello</Text>
    </View>
  );
}

// ACCEPTABLE - Dynamic styles with array syntax
<View style={[styles.container, { backgroundColor: theme.bg }]} />

// ACCEPTABLE - NativeWind/Tailwind className
<View className="flex-1 p-4">
  <Text className="text-2xl font-bold">Hello</Text>
</View>
```

#### Grep Patterns:
- `style=\{\{` in render — inline style objects (flag in non-NativeWind projects)
- `StyleSheet\.create` — verify usage exists
- `className=` — verify NativeWind is configured if used

---

## Platform-Specific Code

```jsx
// GOOD - Platform.OS for simple branching
import { Platform } from 'react-native';

const padding = Platform.OS === 'ios' ? 20 : 16;

// GOOD - Platform.select for object selection
const styles = StyleSheet.create({
  shadow: Platform.select({
    ios: { shadowColor: '#000', shadowOffset: { width: 0, height: 2 } },
    android: { elevation: 4 },
  }),
});

// GOOD - Platform-specific file extensions
// MyComponent.ios.tsx
// MyComponent.android.tsx

// BAD - Checking Platform.OS deep in business logic (should be at component level)
```

#### Grep Patterns:
- `Platform\.OS` — check for platform-specific handling
- `Platform\.select` — check for platform selection patterns
- `.ios.tsx` / `.android.tsx` — check for platform files

---

## Safe Area Handling

```jsx
// GOOD - SafeAreaView from react-native-safe-area-context
import { SafeAreaView } from 'react-native-safe-area-context';

// GOOD - useSafeAreaInsets for granular control
import { useSafeAreaInsets } from 'react-native-safe-area-context';
function MyScreen() {
  const insets = useSafeAreaInsets();
  return <View style={{ paddingTop: insets.top }}>{children}</View>;
}

// BAD - SafeAreaView from react-native (deprecated, iOS only)
import { SafeAreaView } from 'react-native';
```

#### Grep Patterns:
- `from 'react-native-safe-area-context'` — correct import
- `SafeAreaView.*from 'react-native'` — deprecated import (flag)
- `useSafeAreaInsets` — check for usage

---

## Keyboard Handling

```jsx
// GOOD - KeyboardAvoidingView with platform-correct behavior
import { KeyboardAvoidingView, Platform } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput />
</KeyboardAvoidingView>

// BAD - Wrong behavior prop per platform
<KeyboardAvoidingView behavior="padding">  {/* Android needs 'height' */}
```

#### Grep Patterns:
- `KeyboardAvoidingView` — check for platform-correct `behavior` prop
- `keyboard-aware-scroll-view` — alternative keyboard handling

---

## Dimensions & Responsive Design

```jsx
// BAD - Static Dimensions.get (doesn't update on rotation/resize)
const { width } = Dimensions.get('window');

// GOOD - useWindowDimensions hook (reactive)
import { useWindowDimensions } from 'react-native';
function MyComponent() {
  const { width, height } = useWindowDimensions();
  return <View style={{ width: width * 0.9 }} />;
}
```

#### Grep Patterns:
- `Dimensions\.get\(` — flag, suggest `useWindowDimensions`
- `useWindowDimensions` — check for reactive dimensions

---

## AppState & Lifecycle

```jsx
// GOOD - AppState listener with cleanup
useEffect(() => {
  const subscription = AppState.addEventListener('change', handleAppStateChange);
  return () => subscription.remove();
}, []);

// BAD - No cleanup
useEffect(() => {
  AppState.addEventListener('change', handleAppStateChange);
  // Missing cleanup!
}, []);
```

#### Grep Patterns:
- `AppState\.addEventListener` — check for cleanup in useEffect return
- `addEventListener` inside `useEffect` — check for cleanup generally

---

## Effect Cleanup

```jsx
// BAD - No cleanup for subscriptions
useEffect(() => {
  const interval = setInterval(fetchData, 5000);
  // Missing cleanup!
}, []);

// GOOD - Cleanup
useEffect(() => {
  const interval = setInterval(fetchData, 5000);
  return () => clearInterval(interval);
}, []);

// BAD - Navigation listener without cleanup
useEffect(() => {
  navigation.addListener('focus', handleFocus);
  // Missing cleanup — memory leak!
}, []);

// GOOD - Navigation listener cleanup
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', handleFocus);
  return unsubscribe;
}, [navigation]);
```

#### Grep Patterns:
- `setInterval` inside `useEffect` — check for cleanup
- `addEventListener` inside `useEffect` — check for cleanup
- `navigation\.addListener` — check for cleanup (common memory leak)
- `subscribe` inside `useEffect` — check for unsubscribe

---

## Key Props

```jsx
// BAD - Index as key
{items.map((item, index) => (
  <ListItem key={index} item={item} />
))}

// GOOD - Stable unique ID
{items.map(item => (
  <ListItem key={item.id} item={item} />
))}
```

#### Grep Patterns:
- `key=\{.*index` or `key=\{i\}` — index as key
- `\.map\(` — check that mapped elements have `key` prop

---

## Common Anti-Patterns

### 1. ScrollView for Long Lists
```jsx
// BAD - ScrollView with dynamic list
<ScrollView>
  {items.map(item => <Item key={item.id} {...item} />)}
</ScrollView>

// GOOD - FlatList for dynamic lists
<FlatList
  data={items}
  keyExtractor={item => item.id}
  renderItem={({ item }) => <Item {...item} />}
/>
```

### 2. Animated Without Native Driver
```jsx
// BAD - JS-driven animation (blocks JS thread)
Animated.timing(opacity, { toValue: 1, duration: 300 }).start();

// GOOD - Native driver animation
Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }).start();
```

### 3. Images in Render
```jsx
// BAD - require() inside render (re-evaluated each render)
function Avatar() {
  return <Image source={require('./avatar.png')} />;
}

// GOOD - require() at module level
const avatarImage = require('./avatar.png');
function Avatar() {
  return <Image source={avatarImage} />;
}
```

---

## Component Size Thresholds

| Metric | Good | Flag | Investigate |
|--------|------|------|-------------|
| Lines per component | <150 | 150-300 | >300 |
| useState calls | 1-3 | 4-5 | >5 |
| useEffect calls | 0-1 | 2-3 | >3 |
| Props count | 1-5 | 6-8 | >8 |

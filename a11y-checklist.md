# React Native Accessibility (A11y) Audit Checklist

Accessibility checklist for React Native applications targeting VoiceOver (iOS) and TalkBack (Android).

**References:**
- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [Apple HIG - Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Material Design - Accessible Design](https://m3.material.io/foundations/accessible-design/overview)

---

## 1. Accessible Labels

```jsx
// VULNERABLE - Interactive element without label
<Pressable onPress={handleDelete}>
  <TrashIcon />
</Pressable>

// VULNERABLE - Image without description
<Image source={profilePic} />

// SAFE - Labeled interactive element
<Pressable onPress={handleDelete} accessibilityLabel="Delete item">
  <TrashIcon />
</Pressable>

// SAFE - Labeled image
<Image source={profilePic} accessibilityLabel="Profile photo" />

// SAFE - TextInput with label
<TextInput
  placeholder="Email"
  accessibilityLabel="Email address"
/>
```

#### Grep Patterns:
- `<Pressable(?![^>]*accessibilityLabel)` — Pressable without label
- `<TouchableOpacity(?![^>]*accessibilityLabel)` — TouchableOpacity without label
- `<TouchableHighlight(?![^>]*accessibilityLabel)` — TouchableHighlight without label
- `<Image(?![^>]*(accessibilityLabel|accessible=\{false\}))` — meaningful Image without label
- `<TextInput(?![^>]*accessibilityLabel)` — TextInput without label (when no visible label)

#### Severity: HIGH for interactive elements, MEDIUM for images

---

## 2. Accessibility Roles

```jsx
// VULNERABLE - Custom button without role
<Pressable onPress={handleSubmit}>
  <Text>Submit</Text>
</Pressable>

// SAFE - Button with proper role
<Pressable
  onPress={handleSubmit}
  accessibilityRole="button"
>
  <Text>Submit</Text>
</Pressable>

// Common roles to check for:
// "button" - interactive pressables
// "link" - navigation links
// "header" - section headings
// "search" - search inputs
// "image" - meaningful images
// "tab" - tab bar items
// "switch" - toggle switches
// "checkbox" - checkboxes
// "adjustable" - sliders
```

#### Grep Patterns:
- `<Pressable(?![^>]*accessibilityRole)` — Pressable without role
- `<TouchableOpacity(?![^>]*accessibilityRole)` — TouchableOpacity without role
- `accessibilityRole=["']header["']` on section titles — verify presence

#### Severity: HIGH for interactive elements

---

## 3. Accessibility State

```jsx
// VULNERABLE - Disabled button without state announcement
<Pressable onPress={handleSubmit} disabled={true}>
  <Text>Submit</Text>
</Pressable>

// SAFE - State communicated to screen reader
<Pressable
  onPress={handleSubmit}
  disabled={isDisabled}
  accessibilityState={{ disabled: isDisabled }}
>
  <Text>Submit</Text>
</Pressable>

// SAFE - Toggle with checked state
<Pressable
  onPress={toggleFilter}
  accessibilityRole="checkbox"
  accessibilityState={{ checked: isChecked }}
>
  <Text>Filter active</Text>
</Pressable>

// SAFE - Expandable section
<Pressable
  onPress={toggleSection}
  accessibilityState={{ expanded: isExpanded }}
>
  <Text>Details</Text>
</Pressable>
```

#### Grep Patterns:
- `disabled=\{` on Pressable/Touchable — check for matching `accessibilityState`
- Toggle/switch components — check for `accessibilityState={{ checked: ... }}`
- Expandable sections — check for `accessibilityState={{ expanded: ... }}`

#### Severity: MEDIUM

---

## 4. Touch Target Sizing

```jsx
// VULNERABLE - Touch target too small (below 44x44pt iOS / 48x48dp Android)
<Pressable style={{ width: 24, height: 24 }} onPress={handleTap}>
  <Icon size={24} />
</Pressable>

// SAFE - Adequate touch target
<Pressable style={{ width: 48, height: 48 }} onPress={handleTap}>
  <Icon size={24} />
</Pressable>

// SAFE - Using hitSlop for small visual elements
<Pressable
  hitSlop={{ top: 12, bottom: 12, left: 12, right: 12 }}
  onPress={handleTap}
>
  <Icon size={24} />
</Pressable>

// SAFE - Minimum size with centered content
<Pressable style={{ minWidth: 44, minHeight: 44, alignItems: 'center', justifyContent: 'center' }}>
  <Icon size={20} />
</Pressable>
```

#### Grep Patterns:
- `(width|height):\s*(1[0-9]|2[0-9]|3[0-9]|4[0-3])\b` on Pressable/Touchable — below 44pt
- `hitSlop` — verify used for small targets
- `minWidth` and `minHeight` — check minimum sizing on interactive elements

#### Severity: HIGH

---

## 5. Font Scaling

```jsx
// HIGH - Disabling user font size preferences
<Text allowFontScaling={false}>Important text</Text>
<TextInput allowFontScaling={false} />

// HIGH - Overly restrictive max size
<Text maxFontSizeMultiplier={1.0}>Text</Text>

// SAFE - Allow font scaling (default behavior)
<Text>Text scales with user preference</Text>

// SAFE - Reasonable max to prevent layout breaking
<Text maxFontSizeMultiplier={1.5}>Text</Text>
```

#### Grep Patterns:
- `allowFontScaling=\{false\}` — disables user text size preference
- `maxFontSizeMultiplier=\{1(\.0)?\}` — effectively disables scaling

#### Severity: HIGH for disabled scaling, LOW for reasonable maxFontSizeMultiplier

---

## 6. Screen Reader Navigation

```jsx
// SAFE - Hide decorative elements from screen reader
<View accessibilityElementsHidden={true}>
  <DecorativeIcon />
</View>

// Android equivalent
<View importantForAccessibility="no-hide-descendants">
  <DecorativeIcon />
</View>

// SAFE - Group related elements for screen reader
<View accessible={true} accessibilityLabel="Song: Title by Artist, 3:45">
  <Text>Title</Text>
  <Text>Artist</Text>
  <Text>3:45</Text>
</View>

// SAFE - Mark headings for navigation
<Text accessibilityRole="header">Section Title</Text>
```

#### Grep Patterns:
- `accessibilityElementsHidden` — verify used on decorative elements
- `importantForAccessibility` — check Android-specific hiding
- `accessibilityRole=["']header["']` — check section headings are marked
- `accessible=\{true\}` — check grouping of related elements

#### Severity: MEDIUM

---

## 7. Dynamic Content

```jsx
// VULNERABLE - Content updates without announcement
{error && <Text style={styles.error}>{error}</Text>}

// SAFE - Android live region for dynamic content
<View accessibilityLiveRegion="polite">
  {error && <Text>{error}</Text>}
</View>

// SAFE - Programmatic announcement
import { AccessibilityInfo } from 'react-native';
AccessibilityInfo.announceForAccessibility('Item added to cart');
```

#### Grep Patterns:
- `accessibilityLiveRegion` — check presence for dynamic content areas
- `announceForAccessibility` — check usage for important state changes
- Error message rendering — check for live region wrapping

#### Severity: MEDIUM

---

## 8. Focus Management

```jsx
// CHECK - Modal focus trapping
<Modal>
  <View accessibilityViewIsModal={true}>
    {/* Screen reader focus is trapped inside */}
    {children}
  </View>
</Modal>

// CHECK - Focus after navigation
import { AccessibilityInfo } from 'react-native';
AccessibilityInfo.setAccessibilityFocus(viewRef);
```

#### Grep Patterns:
- `<Modal` — check for `accessibilityViewIsModal` on content container
- `accessibilityViewIsModal` — verify used in modals/overlays
- `setAccessibilityFocus` — check for focus management after navigation/modal

#### Severity: HIGH for modals, MEDIUM for navigation transitions

---

## 9. Gestures & Custom Interactions

```jsx
// VULNERABLE - Swipe-only action without alternative
<Swipeable onSwipeLeft={deleteItem}>
  <ListItem />
</Swipeable>
// No delete button available!

// SAFE - Gesture with accessible alternative
<Swipeable onSwipeLeft={deleteItem}>
  <ListItem />
  {/* Also accessible via long-press menu or button */}
</Swipeable>
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Delete item"
  onPress={deleteItem}
>
  <TrashIcon />
</Pressable>

// SAFE - Custom accessibility actions
<View
  accessibilityActions={[
    { name: 'delete', label: 'Delete item' },
    { name: 'archive', label: 'Archive item' },
  ]}
  onAccessibilityAction={(event) => {
    switch (event.nativeEvent.actionName) {
      case 'delete': deleteItem(); break;
      case 'archive': archiveItem(); break;
    }
  }}
>
```

#### Grep Patterns:
- `Swipeable` or `PanGestureHandler` — check for non-gesture alternative
- `accessibilityActions` — verify custom actions defined for gesture-based UI
- `onAccessibilityAction` — verify handler exists

#### Severity: HIGH for gesture-only interactions

---

## 10. Lists & FlatList

```jsx
// CHECK - FlatList accessibility
<FlatList
  data={items}
  renderItem={({ item }) => (
    <View accessible={true} accessibilityLabel={`${item.title} by ${item.artist}`}>
      <Text>{item.title}</Text>
      <Text>{item.artist}</Text>
    </View>
  )}
/>

// SAFE - Empty state announced
<FlatList
  data={items}
  ListEmptyComponent={
    <Text accessibilityRole="header">No items found</Text>
  }
/>
```

#### Grep Patterns:
- `<FlatList` — check renderItem for accessibility props
- `ListEmptyComponent` — check empty state accessibility

#### Severity: MEDIUM

---

## Accessibility Severity Levels

### CRITICAL (Fix Immediately)
- No screen reader access to essential functionality
- Focus trapped with no escape (missing `accessibilityViewIsModal`)

### HIGH (Fix This Week)
- Interactive elements without `accessibilityLabel`
- Missing `accessibilityRole` on buttons/links
- Touch targets below 44x44pt
- `allowFontScaling={false}` on text
- Gesture-only interactions without alternatives
- Modals without focus management

### MEDIUM (Fix This Sprint)
- Missing `accessibilityState` on stateful elements
- Dynamic content without `accessibilityLiveRegion`
- Decorative elements not hidden from screen reader
- Section headings not marked as headers
- List items not grouped with `accessible={true}`

### LOW (Nice to Have)
- Missing `accessibilityHint` on complex interactions
- Missing `accessibilityOrder` customization
- Reasonable `maxFontSizeMultiplier` limits

---

## Runtime Testing Recommendations

Code analysis cannot catch everything. Recommend running:

1. **VoiceOver (iOS)**: Settings > Accessibility > VoiceOver — navigate entire app
2. **TalkBack (Android)**: Settings > Accessibility > TalkBack — navigate entire app
3. **Accessibility Inspector** (Xcode): Audit individual screens
4. **React Native accessibility testing**: `@testing-library/react-native` with `getByRole`, `getByLabelText`

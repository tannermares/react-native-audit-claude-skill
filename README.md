# React Native Audit Skill

A Claude Code skill that performs a comprehensive audit of your React Native/Expo application and generates a Markdown report.

## Safety

This skill **only reads** your code and **writes one file** (the markdown report). It runs `npm audit` (read-only) but makes no changes to your project, dependencies, or configuration. No permissions or tool allowances are configured.

## Install

```bash
git clone https://github.com/tannermares/react-native-audit-claude-skill.git ~/.claude/skills/react-native-audit-claude-skill
```

## Usage

```bash
/react-native-audit                # generates react-native-audit-report-YYYY-MM-DD.md
/react-native-audit my-report      # generates my-report.md
```

## What It Checks

- **Coding Standards** — hooks rules, StyleSheet patterns, platform-specific code, safe area handling, navigation
- **Component Architecture** — complexity, screen/component separation, reusability, TypeScript quality
- **Accessibility** — VoiceOver/TalkBack support, labels, roles, touch targets, font scaling, gestures
- **Performance** — FlatList usage, native driver animations, image optimization, Hermes, memory leaks
- **Security** — secure storage, bundle secrets, deep linking, network security, device security
- **Testing** — unit coverage, E2E (Detox/Maestro), navigation testing
- **Error Handling** — error boundaries, offline support, permission handling, crash reporting
- **App Configuration** — app.json, EAS Build, permissions, OTA updates, deep linking

## Auto-Detected Stacks

The skill auto-detects and adapts checks for:

Expo, bare React Native, Expo Router, React Navigation, NativeWind, StyleSheet, TypeScript, Hermes, New Architecture, Jest, Detox, Maestro, EAS Build

## Report Format

Markdown-only output with:
- Executive summary and score (0-100)
- Findings by category with severity levels (Critical/High/Medium/Low)
- Specific file and line references
- Prioritized recommendations (immediate/short-term/long-term)
- Positive findings and best practice comparison

## Customization

Edit these files to adjust audit criteria:

- `coding-standards-reference.md` — React Native best practices and thresholds
- `security-checklist.md` — mobile security vulnerability patterns
- `a11y-checklist.md` — VoiceOver/TalkBack accessibility checks
- `app-config-checklist.md` — app configuration and distribution checks
- `report-template.md` — report structure

## References

Standards are sourced from:

- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [React Native Security](https://reactnative.dev/docs/security)
- [Apple HIG - Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Material Design - Accessible Design](https://m3.material.io/foundations/accessible-design/overview)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [OWASP Mobile Application Security Checklist](https://mas.owasp.org/checklists/)
- [Expo Configuration](https://docs.expo.dev/versions/latest/config/app/)
- [EAS Build](https://docs.expo.dev/build/eas-json/)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Detox](https://wix.github.io/Detox/)
- [Maestro](https://maestro.mobile.dev/)

## Issues

https://github.com/tannermares/react-native-audit-claude-skill/issues

## Inspiration

Inspired by the [React Audit Claude Skill](https://github.com/tannermares/react-audit-claude-skill) and the [Rails Audit Claude Skill](https://github.com/jcuervo/rails-audit-claude-skill).

## License

MIT

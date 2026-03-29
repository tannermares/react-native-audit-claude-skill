# React Native Application Audit Report

**Project**: [PROJECT_NAME]
**Audit Date**: [DATE]
**React Native Version**: [RN_VERSION]
**Expo SDK**: [EXPO_SDK_VERSION or N/A]
**Navigation**: [Expo Router / React Navigation / Other]
**Auditor**: Claude Code

---

# Executive Summary

## Overall Health Score: [SCORE]/100

| Priority | Count |
|----------|-------|
| 🔴 Critical | [N] |
| 🟠 High | [N] |
| 🟡 Medium | [N] |
| 🟢 Low | [N] |

**Status**: [EXCELLENT/GOOD/FAIR/NEEDS_ATTENTION/CRITICAL]

---

# Audit Scope

## Project Statistics

- **Total Files Analyzed**: [N]
- **Total Lines of Code**: [N]
- **Screens**: [N]
- **Components**: [N]
- **Custom Hooks**: [N]
- **Test Files**: [N]
- **Native Modules**: [N]
- **Dependencies**: [N] production / [N] dev

## Stack Detected

- **Project Type**: [Expo / Bare React Native]
- **Language**: [TypeScript / JavaScript]
- **Styling**: [StyleSheet / NativeWind / styled-components / Other]
- **Navigation**: [Expo Router / React Navigation / Other]
- **State Management**: [Context / Redux / Zustand / Jotai / Other / None]
- **Linting**: [ESLint / Biome / None]
- **Test Framework**: [Jest / Detox / Maestro / None]
- **Architecture**: [New Arch / Old Arch (Bridge)]
- **JS Engine**: [Hermes / JSC]

---

# Coding Standards Assessment

## Score: [X]/100

### Issues Found

| Category | Count | Severity |
|----------|-------|----------|
| [Category 1] | [N] | [Level] |
| [Category 2] | [N] | [Level] |
| [Category 3] | [N] | [Level] |

---

# Coding Standards - Top Issues

## 1. [Issue Title]
**File**: [filename.tsx:line]
**Severity**: [Critical/High/Medium/Low]
**Description**: [Brief description]

## 2. [Issue Title]
**File**: [filename.tsx:line]
**Severity**: [Critical/High/Medium/Low]
**Description**: [Brief description]

## 3. [Issue Title]
**File**: [filename.tsx:line]
**Severity**: [Critical/High/Medium/Low]
**Description**: [Brief description]

---

# Component Architecture

## Complexity Metrics

| Metric | Average | Max | Target |
|--------|---------|-----|--------|
| Lines per component | [N] | [N] | <150 |
| useState calls | [N] | [N] | <5 |
| useEffect calls | [N] | [N] | <3 |
| Props per component | [N] | [N] | <8 |

## Patterns Assessment

| Pattern | Status | Notes |
|---------|--------|-------|
| Screen/Component Separation | [Well Used/Missing] | [Notes] |
| Custom Hooks | [Well Used/Missing] | [Notes] |
| Navigation Structure | [Well Organized/Needs Work] | [Notes] |
| Type Safety | [Well Used/Missing/N/A] | [Notes] |

---

# Accessibility Assessment

## Issues Found

| Priority | Count |
|----------|-------|
| 🔴 Critical | [N] |
| 🟠 High | [N] |
| 🟡 Medium | [N] |
| 🟢 Low | [N] |

**Overall A11y Score**: [X]/100

**Platform Notes**: Issues affect VoiceOver (iOS), TalkBack (Android), or both.

---

# Accessibility - Top Issues

## 🟠 [Issue Type]
**File**: [filename.tsx:line]
**Platform**: [iOS / Android / Both]
**Impact**: [Description of impact on users]
**Recommendation**: [How to fix]

---

# Testing & Coverage

## Overall Coverage: [X]% (or N/A if no tests)

| Component Type | Files | Tested | Coverage |
|----------------|-------|--------|----------|
| Screens | [N] | [N] | [X]% |
| Components | [N] | [N] | [X]% |
| Hooks | [N] | [N] | [X]% |
| Utils | [N] | [N] | [X]% |

**Test-to-Screen Ratio**: [X:1]
**E2E Coverage**: [Detox / Maestro / None]

## Coverage Gaps

1. **[Screen/Component]**
   - Missing: [Description]

---

# Security Assessment

## Vulnerabilities Found

| Priority | Count |
|----------|-------|
| 🔴 Critical | [N] |
| 🟠 High | [N] |
| 🟡 Medium | [N] |
| 🟢 Low | [N] |

**Overall Security Score**: [X]/100

---

# Security - Top Issues

## 🔴 [Vulnerability Type]
**File**: [filename.tsx:line]
**Risk**: Critical
**Impact**: [Description — e.g., "Auth tokens extractable from device"]
**Recommendation**: [How to fix]

---

# Performance Analysis

## List Rendering

- **FlatList Usage**: [Proper / ScrollView+map detected]
- **keyExtractor**: [Present / Missing on [N] lists]
- **getItemLayout**: [Used / Missing]

## Animation Performance

- **Native Driver**: [All animations / [N] missing useNativeDriver]
- **Reanimated**: [Used / Not used]

## Image Optimization

- **Image Library**: [expo-image / react-native-fast-image / Raw Image]
- **Caching**: [Configured / Not configured]

## Bundle & Engine

- **Hermes**: [Enabled / Disabled]
- **New Architecture**: [Enabled / Disabled]
- **Heavy Dependencies**: [List any oversized deps]
- **Production Deps**: [N]

## Identified Bottlenecks

1. [Performance issue with file reference]
2. [Performance issue with file reference]

---

# Error Handling & Resilience

## Coverage

| Pattern | Present | Notes |
|---------|---------|-------|
| Error Boundaries | [Yes/No/Partial] | [Notes] |
| Loading States | [Yes/No/Partial] | [Notes] |
| Empty States | [Yes/No/Partial] | [Notes] |
| Offline Handling | [Yes/No/Partial] | [Notes] |
| Permission Denial | [Yes/No/Partial] | [Notes] |
| Crash Reporting | [Yes/No/Partial] | [Notes] |
| AppState Handling | [Yes/No/Partial] | [Notes] |

---

# App Configuration & Distribution

## Configuration Completeness

| Item | Status | Notes |
|------|--------|-------|
| Bundle Identifier | [✓/✗] | [Notes] |
| Version Management | [✓/✗] | [Notes] |
| EAS Build Profiles | [✓/✗/N/A] | [Notes] |
| Permissions Audit | [✓/✗] | [Notes] |
| OTA Updates | [✓/✗/N/A] | [Notes] |
| Splash Screen | [✓/✗] | [Notes] |
| App Icon | [✓/✗] | [Notes] |
| Deep Linking | [✓/✗] | [Notes] |
| Dark Mode | [✓/✗] | [Notes] |

---

# Dependency Analysis

## npm Audit Results

| Severity | Count |
|----------|-------|
| Critical | [N] |
| High | [N] |
| Moderate | [N] |
| Low | [N] |

## Outdated Dependencies

| Package | Current | Latest | Risk |
|---------|---------|--------|------|
| [package] | [version] | [version] | [High/Medium/Low] |

---

# Critical Issues Summary

## Must Fix Immediately

1. **[Issue]** - [filename.tsx:line]
   Impact: [Description]

2. **[Issue]** - [filename.tsx:line]
   Impact: [Description]

---

# Recommendations - Immediate Actions

## Fix This Week

1. **[Action Item]**
   - Priority: Critical
   - Effort: [High/Medium/Low]
   - Impact: [High/Medium/Low]

---

# Recommendations - Short Term

## Fix This Sprint (2-4 weeks)

1. **[Action Item]**
   - Priority: High
   - Effort: [High/Medium/Low]
   - Files Affected: [N]

---

# Recommendations - Long Term

## Fix This Quarter

1. **[Action Item]**
   - Priority: Medium
   - Effort: [High/Medium/Low]
   - Strategic Value: [High/Medium/Low]

---

# Technical Debt Estimation

- **Estimated Time to Fix All Issues**: [N] hours
- **Critical Issues**: [N] hours
- **High Priority**: [N] hours
- **Medium Priority**: [N] hours
- **Low Priority**: [N] hours

---

# Positive Findings

## What's Working Well

1. **[Positive aspect]**
   - [Details]

2. **[Positive aspect]**
   - [Details]

3. **[Positive aspect]**
   - [Details]

---

# Comparison to React Native Best Practices

| Practice | Status | Notes |
|----------|--------|-------|
| Function Components | [✓/✗/Partial] | [Notes] |
| Hooks Usage | [✓/✗/Partial] | [Notes] |
| TypeScript | [✓/✗/Partial] | [Notes] |
| FlatList for Lists | [✓/✗/Partial] | [Notes] |
| StyleSheet.create | [✓/✗/Partial] | [Notes] |
| Safe Area Handling | [✓/✗/Partial] | [Notes] |
| Accessibility | [✓/✗/Partial] | [Notes] |
| Error Boundaries | [✓/✗/Partial] | [Notes] |
| Offline Support | [✓/✗/Partial] | [Notes] |
| Hermes Engine | [✓/✗/Partial] | [Notes] |

---

# Next Steps

## Recommended Action Plan

1. **Week 1**: Fix critical security and accessibility issues
2. **Week 2-3**: Address performance bottlenecks and high-priority a11y
3. **Week 4-6**: Add test coverage to critical screens
4. **Month 2**: Refactor architecture issues
5. **Month 3**: Optimize app configuration and distribution
6. **Ongoing**: Maintain quality standards

---

# Conclusion

## Overall Assessment

[Summary paragraph about the overall state of the React Native application]

## Key Takeaways

1. [Main takeaway]
2. [Main takeaway]
3. [Main takeaway]

## Final Score: [X]/100

**Recommendation**: [Ready for store submission / Address critical issues first / Major refactoring needed]

---

# Appendix

## Audit Methodology

- Tools Used: Claude Code (Glob, Grep, Read, Bash)
- Standards Applied: React Native Best Practices, Apple HIG, Material Design, OWASP Mobile Top 10
- Files Analyzed: [N]

## Contact

- Generated by: Claude Code
- Date: [DATE]
- Skill Version: 1.0.0

---

**Next Audit Recommended**: [DATE + 3 months]

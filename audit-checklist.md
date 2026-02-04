# HIG Audit Checklist

Systematic evaluation criteria organized by HIG domain. Each item includes severity classification.

---

## 1. Color System Check

### Semantic Colors (Critical)
- [ ] Uses `.primary`, `.secondary`, `.tertiary` for text hierarchy
- [ ] Uses `Color(.systemBackground)` family for backgrounds
- [ ] Uses `Color(.systemGroupedBackground)` for grouped lists
- [ ] Avoids hardcoded `.white`, `.black` for UI elements
- [ ] Custom colors defined in Assets.xcassets with Light/Dark variants

**Anti-patterns to flag:**
```swift
// ❌ Hardcoded colors
Text("Label").foregroundStyle(.white)
.background(Color(red: 0.95, green: 0.95, blue: 0.95))

// ✅ Semantic colors
Text("Label").foregroundStyle(.primary)
.background(Color(.systemBackground))
```

### Color Contrast (Critical)
- [ ] Normal text meets 4.5:1 minimum contrast
- [ ] Large text (18pt+) meets 3:1 minimum
- [ ] Tested in both Light and Dark modes
- [ ] Tested with Increase Contrast enabled

### Color Independence (Critical)
- [ ] Information conveyed by more than color alone
- [ ] Status indicators use shape + color + text
- [ ] Links distinguishable without relying on color

**Anti-pattern:**
```swift
// ❌ Color-only status
Circle().fill(isActive ? .green : .red)

// ✅ Multiple indicators
HStack {
    Image(systemName: isActive ? "checkmark.circle.fill" : "xmark.circle.fill")
    Text(isActive ? "Active" : "Inactive")
}
```

---

## 2. Typography Check

### Dynamic Type (Critical)
- [ ] All text uses system text styles (`.body`, `.headline`, etc.)
- [ ] No fixed `.system(size: X)` without `relativeTo:`
- [ ] Layout adapts to larger text sizes
- [ ] Text doesn't truncate at accessibility sizes

**Anti-pattern:**
```swift
// ❌ Fixed size
.font(.system(size: 17))

// ✅ Scalable
.font(.body)
// or
.font(.custom("MyFont", size: 17, relativeTo: .body))
```

### Font Weight (Important)
- [ ] Avoids `.ultralight`, `.thin`, `.light` weights
- [ ] Uses `.regular` minimum for body text
- [ ] Headers use `.semibold` or `.bold`

### Text Hierarchy (Important)
- [ ] Clear visual hierarchy (title → body → caption)
- [ ] Consistent use of text styles throughout
- [ ] Secondary content uses `.secondary` foreground

---

## 3. Layout & Spacing Check

### Touch Targets (Critical)
- [ ] All interactive elements minimum 44x44 points
- [ ] Adequate spacing between targets (12-24pt)
- [ ] Small visual elements have expanded hit areas

**Anti-pattern:**
```swift
// ❌ Too small
Button("X") { }.frame(width: 24, height: 24)

// ✅ Adequate target
Button("X") { }.frame(minWidth: 44, minHeight: 44)
```

### Safe Areas (Important)
- [ ] Content respects safe areas
- [ ] Backgrounds extend to edges appropriately
- [ ] No content hidden by notch/Dynamic Island/home indicator

### Alignment (Important)
- [ ] Text baselines align across rows
- [ ] Controls align on common grid
- [ ] Consistent margins throughout

### Adaptability (Important)
- [ ] Works in portrait and landscape
- [ ] Adapts to different screen sizes
- [ ] Handles split view on iPad

---

## 4. Accessibility Check

### VoiceOver (Critical)
- [ ] All interactive elements have accessibility labels
- [ ] Images have descriptions or are marked decorative
- [ ] Custom controls provide accessibility traits
- [ ] Reading order is logical

**Anti-pattern:**
```swift
// ❌ No label
Button { } label: { Image(systemName: "gear") }

// ✅ Labeled
Button { } label: { Image(systemName: "gear") }
    .accessibilityLabel("Settings")
```

### Reduce Motion (Critical)
- [ ] Checks `accessibilityReduceMotion` environment value
- [ ] Provides simpler or no animation alternative
- [ ] No essential information conveyed only through motion

**Required pattern:**
```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

.animation(reduceMotion ? nil : .spring(), value: state)
```

### Bold Text (Important)
- [ ] Responds to Bold Text accessibility setting
- [ ] Custom fonts handle weight increase gracefully

### Increase Contrast (Important)
- [ ] Higher contrast variants available
- [ ] Tested with setting enabled

---

## 5. Platform Conventions Check

### Navigation (Critical)
- [ ] Uses standard navigation patterns (NavigationStack, TabView)
- [ ] Back navigation works via swipe (iOS)
- [ ] Doesn't override system gestures

### Standard Components (Important)
- [ ] Uses system controls where appropriate
- [ ] Custom controls follow platform conventions
- [ ] Familiar patterns for common actions (share, delete, edit)

### Platform-Specific (Context-Dependent)
- [ ] iOS: Tab bar for top-level navigation
- [ ] iPadOS: Sidebar navigation, pointer support
- [ ] macOS: Menu bar commands, keyboard shortcuts
- [ ] watchOS: Digital Crown integration
- [ ] visionOS: Comfortable depth, no head-anchored content

---

## 6. Motion & Animation Check

### Purposeful Motion (Important)
- [ ] Animations serve functional purpose
- [ ] No gratuitous or decorative-only animations
- [ ] Frequent interactions have minimal/no animation

### Interruptibility (Important)
- [ ] Animations can be interrupted
- [ ] Users not blocked waiting for animations
- [ ] State changes are immediate, animation is visual polish

### Duration (Context-Dependent)
- [ ] High-frequency UI: Under 300ms (180ms ideal)
- [ ] Transitions: 200-400ms
- [ ] Complex sequences: May be longer if purposeful

---

## 7. Branding Check

### Restraint (Important)
- [ ] Logo not displayed throughout app
- [ ] Branding doesn't compete with content
- [ ] System backgrounds used (not brand colors everywhere)

### Launch Screen (Critical for App Store)
- [ ] No logos on launch screen
- [ ] No text on launch screen
- [ ] Matches first screen appearance
- [ ] Adapts to light/dark mode

### Appropriate Placement (Important)
- [ ] Accent color used for tint
- [ ] Branding in onboarding (not launch)
- [ ] Brand voice in copy, not visual chrome

---

## 8. Icons & Symbols Check

### SF Symbols (Important)
- [ ] Uses SF Symbols where available
- [ ] Symbols scale with text (Dynamic Type)
- [ ] Appropriate symbol weights match adjacent text

### Custom Icons (Important)
- [ ] Consistent size and detail level
- [ ] Vector format (PDF/SVG)
- [ ] Provides accessibility labels

### Icon vs Text Decision (Context-Dependent)
- [ ] Clear, universal meaning → icon okay
- [ ] Ambiguous action → use text label
- [ ] Multiple similar actions → prefer text

---

## Severity Classification Summary

| Severity | Must Address | Examples |
|----------|--------------|----------|
| **Critical** | Before release | Accessibility violations, platform breaks |
| **Important** | High priority | Hardcoded colors, fixed fonts |
| **Context-Dependent** | Evaluate case-by-case | Branding, custom animations |
| **Nice to Have** | Polish phase | Optical alignment, material refinement |

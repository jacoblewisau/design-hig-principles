# Common HIG Mistakes

Anti-patterns and violations frequently found in iOS/macOS apps, with fixes.

---

## Color Mistakes

### Hardcoded Colors

**Problem**: Using literal colors that don't adapt to appearance modes.

```swift
// ❌ WRONG
Text("Title").foregroundStyle(.white)
.background(Color(red: 0.1, green: 0.1, blue: 0.1))
.background(Color(hex: "#F5F5F5"))
```

**Fix**:
```swift
// ✅ CORRECT
Text("Title").foregroundStyle(.primary)
.background(Color(.systemBackground))
.background(Color("CustomBackground")) // Asset catalog with variants
```

**Why it matters**: Users expect apps to respect their appearance preference. Hardcoded colors break in Dark Mode.

---

### Missing Dark Mode Variants

**Problem**: Custom colors without dark mode definitions.

```swift
// ❌ Asset catalog color with only "Any Appearance"
Color("BrandBlue") // Same blue in both modes
```

**Fix**:
1. Open Assets.xcassets
2. Select color
3. In Attributes Inspector, set "Appearances" to "Any, Dark"
4. Define distinct colors for each

---

### Color-Only Status Indicators

**Problem**: Colorblind users can't distinguish states.

```swift
// ❌ WRONG
Circle().fill(isOnline ? .green : .gray)
```

**Fix**:
```swift
// ✅ CORRECT - Multiple cues
HStack {
    Circle().fill(isOnline ? .green : .gray)
    Text(isOnline ? "Online" : "Offline")
}

// Or use different shapes
Image(systemName: isOnline ? "checkmark.circle.fill" : "circle")
    .foregroundStyle(isOnline ? .green : .secondary)
```

---

## Typography Mistakes

### Fixed Font Sizes

**Problem**: Text won't scale with Dynamic Type.

```swift
// ❌ WRONG
.font(.system(size: 17))
.font(.custom("Avenir", size: 14))
```

**Fix**:
```swift
// ✅ CORRECT
.font(.body)
.font(.custom("Avenir", size: 14, relativeTo: .body))
```

---

### Light Font Weights

**Problem**: Poor legibility, especially in bright conditions.

```swift
// ❌ WRONG
.font(.system(size: 16, weight: .ultralight))
.font(.system(size: 16, weight: .thin))
```

**Fix**:
```swift
// ✅ CORRECT
.font(.system(size: 16, weight: .regular))
.font(.body) // Let system choose appropriate weight
```

---

### Truncating Essential Content

**Problem**: Important information cut off at large text sizes.

```swift
// ❌ WRONG
Text(userName)
    .lineLimit(1) // Truncates long names at large sizes
```

**Fix**:
```swift
// ✅ CORRECT - Allow wrapping for essential content
Text(userName)
    .lineLimit(2)
    .minimumScaleFactor(0.8) // Slight shrink before wrap
```

---

## Layout Mistakes

### Small Touch Targets

**Problem**: Buttons too small to tap reliably.

```swift
// ❌ WRONG
Button { } label: { Image(systemName: "xmark") }
    .frame(width: 24, height: 24)
```

**Fix**:
```swift
// ✅ CORRECT - Visual can be small, hit area must be 44pt
Button { } label: { Image(systemName: "xmark").font(.caption) }
    .frame(minWidth: 44, minHeight: 44)
    .contentShape(Rectangle())
```

---

### Ignoring Safe Areas Incorrectly

**Problem**: Content hidden behind system UI.

```swift
// ❌ WRONG - Important content in unsafe area
VStack {
    ImportantHeader()
}
.ignoresSafeArea() // Header behind Dynamic Island
```

**Fix**:
```swift
// ✅ CORRECT - Only backgrounds extend, content respects safe areas
ZStack {
    Color.blue.ignoresSafeArea() // Background extends
    VStack {
        ImportantHeader() // Content in safe area
    }
}
```

---

### Fixed Layouts That Don't Adapt

**Problem**: Layouts break on different devices or orientations.

```swift
// ❌ WRONG
HStack {
    Column1().frame(width: 200)
    Column2().frame(width: 200)
}
// Breaks on iPhone SE, breaks in landscape
```

**Fix**:
```swift
// ✅ CORRECT - Adaptive layout
ViewThatFits {
    HStack { Column1(); Column2() } // Preferred
    VStack { Column1(); Column2() } // Fallback
}
```

---

## Accessibility Mistakes

### Missing VoiceOver Labels

**Problem**: VoiceOver announces unhelpful information.

```swift
// ❌ WRONG - VoiceOver says "Button"
Button { share() } label: { Image(systemName: "square.and.arrow.up") }

// ❌ WRONG - VoiceOver reads filename
Image("decorative-divider")
```

**Fix**:
```swift
// ✅ CORRECT
Button { share() } label: { Image(systemName: "square.and.arrow.up") }
    .accessibilityLabel("Share")

Image("decorative-divider")
    .accessibilityHidden(true)
```

---

### Ignoring Reduce Motion

**Problem**: Motion-sensitive users experience discomfort.

```swift
// ❌ WRONG - Always animates
.animation(.spring(), value: isExpanded)
```

**Fix**:
```swift
// ✅ CORRECT
@Environment(\.accessibilityReduceMotion) var reduceMotion

.animation(reduceMotion ? nil : .spring(), value: isExpanded)
```

---

### No Gesture Alternatives

**Problem**: Users who can't swipe have no alternative.

```swift
// ❌ WRONG - Swipe-only action
.swipeActions { Button("Delete") { delete() } }
```

**Fix**:
```swift
// ✅ CORRECT - Multiple interaction methods
.swipeActions { Button("Delete") { delete() } }
.contextMenu { Button("Delete", role: .destructive) { delete() } }
```

---

## Platform Convention Mistakes

### Non-Standard Navigation

**Problem**: Breaking expected navigation patterns.

```swift
// ❌ WRONG on iOS - Hamburger menu
Button { showMenu.toggle() } label: { Image(systemName: "line.3.horizontal") }

// ❌ WRONG - Custom back button that doesn't support swipe
.navigationBarBackButtonHidden(true)
.toolbar { Button("Back") { dismiss() } }
```

**Fix**:
```swift
// ✅ CORRECT iOS - Tab bar for top-level
TabView { ... }

// ✅ CORRECT - Use system back button
// Don't hide it unless absolutely necessary
```

---

### Overriding System Gestures

**Problem**: Interfering with platform-expected behaviors.

```swift
// ❌ WRONG - Conflicts with system edge swipe
.gesture(
    DragGesture()
        .onChanged { ... } // Captures edge swipes
)
```

**Fix**:
```swift
// ✅ CORRECT - Don't intercept system gestures
// Use .highPriorityGesture only when necessary
// Test that edge swipe back still works
```

---

## Branding Mistakes

### Logo Everywhere

**Problem**: Brand competes with content.

```swift
// ❌ WRONG
.toolbar {
    ToolbarItem(placement: .principal) {
        Image("logo") // Logo in every screen's nav bar
    }
}
```

**Fix**:
```swift
// ✅ CORRECT - Restraint
// Logo in onboarding or about screen only
// Use accent color for brand presence
.tint(Color("BrandColor"))
```

---

### Branded Launch Screen

**Problem**: App Store guidelines violation, poor UX.

```swift
// ❌ WRONG - Launch screen with logo/text
// LaunchScreen.storyboard contains app logo
```

**Fix**:
```swift
// ✅ CORRECT
// Launch screen matches first screen appearance
// No text, no logos
// Branding goes in onboarding (separate from launch)
```

---

### Custom Dark Mode Toggle

**Problem**: Conflicts with system appearance setting.

```swift
// ❌ WRONG - App has its own appearance toggle
@AppStorage("darkMode") var darkMode = false
.preferredColorScheme(darkMode ? .dark : .light)
```

**Fix**:
```swift
// ✅ CORRECT - Respect system preference
// Remove custom toggle
// Let Settings app control appearance
```

---

## Animation Mistakes

### Animations on Frequent Interactions

**Problem**: Repeated animations become tiresome.

```swift
// ❌ WRONG - Animates every keystroke
TextField("Search", text: $query)
    .onChange(of: query) {
        withAnimation { updateResults() }
    }
```

**Fix**:
```swift
// ✅ CORRECT - No animation for high-frequency actions
TextField("Search", text: $query)
    .onChange(of: query) { updateResults() } // Instant
```

---

### Blocking Animations

**Problem**: Users forced to wait for animation to complete.

```swift
// ❌ WRONG - Disables interaction during animation
.disabled(isAnimating)
```

**Fix**:
```swift
// ✅ CORRECT - Animation is visual polish, not blocker
// Users can interact immediately
// Animation catches up or cancels
```

---

## Quick Reference: What to Search For

| Pattern to Find | Likely Issue |
|-----------------|--------------|
| `.foregroundStyle(.white)` | Hardcoded color |
| `.foregroundStyle(.black)` | Hardcoded color |
| `Color(red:` | Hardcoded color |
| `Color(hex:` | Hardcoded color |
| `.font(.system(size:` | Fixed font size |
| `.weight(.ultralight)` | Poor legibility |
| `.weight(.thin)` | Poor legibility |
| `.frame(width:` < 44 on Button | Small touch target |
| `Image(systemName:` without `.accessibilityLabel` | Missing VoiceOver |
| `withAnimation` without `reduceMotion` | Motion not optional |
| `.swipeActions` without `.contextMenu` | No gesture alternative |
| `.navigationBarBackButtonHidden(true)` | Non-standard navigation |

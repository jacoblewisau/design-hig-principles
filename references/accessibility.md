# Accessibility Requirements

Accessibility compliance is **mandatory** for every HIG audit. These aren't suggestions—they're requirements.

---

## Vision Accessibility

### Dynamic Type (Critical)

**Requirement**: Support text scaling of at least **200%** (140% for watchOS).

**Implementation**:
```swift
// ✅ CORRECT - Uses text style, scales automatically
Text("Title")
    .font(.title)

// ✅ CORRECT - Custom font with scaling
Text("Title")
    .font(.custom("MyFont", size: 24, relativeTo: .title))

// ❌ WRONG - Fixed size, won't scale
Text("Title")
    .font(.system(size: 24))
```

**Audit check**: Search for `.font(.system(size:` without `relativeTo:`.

**Layout requirements at large sizes**:
- Reduce multi-column layouts to single column
- Stack horizontal elements vertically
- Never truncate essential content
- Test at maximum accessibility size (AX5)

### Color Contrast (Critical)

**WCAG Level AA standards**:

| Text Size | Minimum Contrast |
|-----------|------------------|
| Normal (<18pt) | 4.5:1 |
| Large (18pt+ regular, 14pt+ bold) | 3:1 |
| Small text | 7:1 recommended |

**Implementation**:
```swift
// ✅ Semantic colors (automatic compliance)
Text("Label").foregroundStyle(.primary)
Text("Secondary").foregroundStyle(.secondary)

// ⚠️ Custom colors - verify contrast
Text("Custom").foregroundStyle(Color("BrandBlue"))
// Must test with contrast calculator

// ❌ System gray may fail
Text("Label").foregroundStyle(.gray)
// Gray is 5.74:1 on white, but only 3.5:1 on light backgrounds
```

**Audit check**:
- Test with Xcode Accessibility Inspector
- Check both Light and Dark modes
- Enable "Increase Contrast" and verify

### Color Independence (Critical)

**Requirement**: "Convey information with more than color alone."

**Bad pattern**:
```swift
// ❌ Only color conveys status
Circle()
    .fill(status == .success ? .green : .red)
```

**Good pattern**:
```swift
// ✅ Shape + color + text
HStack {
    Image(systemName: status == .success ? "checkmark.circle.fill" : "exclamationmark.circle.fill")
    Text(status == .success ? "Success" : "Error")
}
.foregroundStyle(status == .success ? .green : .red)
```

**Audit check**: For every use of `.green`, `.red`, `.yellow` as status indicators, verify alternative cues exist.

### VoiceOver Labels (Critical)

**Requirement**: All interactive elements must have accessibility labels.

**Bad patterns**:
```swift
// ❌ Icon button without label
Button { action() } label: {
    Image(systemName: "gear")
}
// VoiceOver: "Button"

// ❌ Decorative image not hidden
Image("divider")
// VoiceOver reads filename
```

**Good patterns**:
```swift
// ✅ Labeled button
Button { action() } label: {
    Image(systemName: "gear")
}
.accessibilityLabel("Settings")

// ✅ Decorative image hidden
Image("divider")
    .accessibilityHidden(true)

// ✅ Complex control with hint
Slider(value: $volume)
    .accessibilityLabel("Volume")
    .accessibilityValue("\(Int(volume * 100))%")
    .accessibilityHint("Adjust playback volume")
```

**Audit check**: Search for `Button` and `Image(systemName:` without `.accessibilityLabel`.

---

## Mobility Accessibility

### Touch Targets (Critical)

**Requirement**: Minimum **44x44 points** for all interactive elements.

**Implementation**:
```swift
// ❌ Too small
Button("X") { dismiss() }
    .frame(width: 24, height: 24)

// ✅ Visual size can be small, hit area must be large
Button { dismiss() } label: {
    Image(systemName: "xmark")
        .font(.caption)
}
.frame(minWidth: 44, minHeight: 44)
.contentShape(Rectangle()) // Expands hit area
```

**Spacing requirement**: 12-24 points between adjacent touch targets.

**Audit check**: Search for `.frame(width:` or `.frame(height:` with values under 44 on Buttons or other interactive elements.

### Gesture Alternatives (Important)

**Requirement**: "Give people more than one way to interact."

**Implementation**:
```swift
// ✅ Swipe action has menu alternative
ForEach(items) { item in
    ItemRow(item: item)
        .swipeActions {
            Button("Delete", role: .destructive) { delete(item) }
        }
        .contextMenu {
            Button("Delete", role: .destructive) { delete(item) }
        }
}
```

**Audit check**: For every `.swipeActions` or custom gesture, verify a button/menu alternative exists.

---

## Cognitive Accessibility

### Reduce Motion (Critical)

**Requirement**: Respect `accessibilityReduceMotion` setting.

**Implementation**:
```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

var body: some View {
    content
        .animation(reduceMotion ? nil : .spring(), value: isExpanded)
}

// Alternative: Simpler animation instead of none
.animation(reduceMotion ? .linear(duration: 0.1) : .spring(duration: 0.4), value: isExpanded)
```

**What to reduce**:
- Bouncy/spring animations → Linear or none
- Parallax effects → Static
- Auto-playing videos → Paused with play button
- Continuous motion → Static
- Z-axis changes → 2D transitions

**Audit check**: Search for `withAnimation` or `.animation` without checking `reduceMotion`.

### Avoid Auto-Dismiss (Important)

**Requirement**: Avoid time-based auto-dismissing views.

**Bad pattern**:
```swift
// ❌ Auto-dismiss toast
.onAppear {
    DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
        dismiss()
    }
}
```

**Good pattern**:
```swift
// ✅ User-dismissable with optional auto-dismiss
Toast(message: message)
    .onTapGesture { dismiss() }
    .accessibilityAddTraits(.isButton)
    .accessibilityHint("Tap to dismiss")
```

---

## Hearing Accessibility

### Media Alternatives (Important)

For video/audio content:
- Captions for dialogue
- Subtitles for different languages
- Audio descriptions for visual information
- Transcripts for long-form content

### Audio Cues (Important)

Pair audio signals with:
- Haptic feedback
- Visual indicators

```swift
// ✅ Audio + haptic + visual
func showSuccess() {
    AudioServicesPlaySystemSound(1001) // Audio
    UINotificationFeedbackGenerator().notificationOccurred(.success) // Haptic
    showSuccessBanner = true // Visual
}
```

---

## Platform-Specific Accessibility

### visionOS

- Maintain horizontal layouts (vertical is uncomfortable)
- Reduce animation speed
- Avoid head-anchored content (breaks assistive tech)
- Center important content in field of view

### watchOS

- Digital Crown as alternative to touch
- Complications must be glanceable
- Larger text support within space constraints

### macOS

- Full Keyboard Access support
- VoiceOver works with custom controls
- Keyboard shortcuts for common actions

---

## Accessibility Audit Checklist Summary

| Category | Check | Severity |
|----------|-------|----------|
| Dynamic Type | All text uses text styles | Critical |
| Contrast | 4.5:1 minimum for normal text | Critical |
| Color Independence | Status not color-only | Critical |
| VoiceOver | All controls labeled | Critical |
| Touch Targets | Minimum 44x44pt | Critical |
| Reduce Motion | Environment check exists | Critical |
| Gesture Alternatives | Non-gesture options exist | Important |
| Auto-Dismiss | User can control timing | Important |
| Bold Text | Responds to setting | Important |
| Increase Contrast | Tested with setting | Important |

**Every audit must verify all Critical items are compliant.**

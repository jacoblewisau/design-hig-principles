# Haptics & Feedback Reference

Complete guide to haptic feedback patterns, UIFeedbackGenerator usage, and multi-modal feedback.

---

## Haptic Philosophy

**Apple's haptic principles:**
- Haptics should reinforce visual and audio feedback
- Use haptics purposefully, not gratuitously
- Match haptic intensity to action importance
- Haptics should feel natural and expected

---

## UIFeedbackGenerator Types

### Impact Feedback

Physical collision or impact sensations.

```swift
// Light - Subtle feedback
let light = UIImpactFeedbackGenerator(style: .light)
light.impactOccurred()

// Medium - Standard feedback
let medium = UIImpactFeedbackGenerator(style: .medium)
medium.impactOccurred()

// Heavy - Strong feedback
let heavy = UIImpactFeedbackGenerator(style: .heavy)
heavy.impactOccurred()

// Soft - iOS 13+ muted impact
let soft = UIImpactFeedbackGenerator(style: .soft)
soft.impactOccurred()

// Rigid - iOS 13+ sharp impact
let rigid = UIImpactFeedbackGenerator(style: .rigid)
rigid.impactOccurred()

// With intensity (0.0 - 1.0)
medium.impactOccurred(intensity: 0.7)
```

### Selection Feedback

For changing selection states.

```swift
let selection = UISelectionFeedbackGenerator()

// Prepare before frequent use
selection.prepare()

// Trigger on selection change
selection.selectionChanged()
```

### Notification Feedback

For outcomes and alerts.

```swift
let notification = UINotificationFeedbackGenerator()

// Success - Task completed
notification.notificationOccurred(.success)

// Warning - Attention needed
notification.notificationOccurred(.warning)

// Error - Task failed
notification.notificationOccurred(.error)
```

---

## When to Use Each Type

| Feedback Type | Use Case | Example |
|---------------|----------|---------|
| Impact (light) | Subtle confirmation | Checkbox toggle |
| Impact (medium) | Standard interaction | Button press |
| Impact (heavy) | Important action | Pull to refresh completion |
| Impact (soft) | Gentle feedback | Slider change |
| Impact (rigid) | Sharp feedback | Snap to position |
| Selection | Selection change | Picker scroll, segment change |
| Notification (success) | Completed action | Upload complete |
| Notification (warning) | Caution | Approaching limit |
| Notification (error) | Failed action | Invalid input |

---

## SwiftUI Integration

### sensoryFeedback Modifier (iOS 17+)

```swift
// Automatic haptic on value change
Toggle("Feature", isOn: $isEnabled)
    .sensoryFeedback(.selection, trigger: isEnabled)

Button("Submit") { submit() }
    .sensoryFeedback(.success, trigger: isSubmitted)

// With condition
.sensoryFeedback(feedbackType, trigger: value) { oldValue, newValue in
    newValue > oldValue // Only when increasing
}
```

### Available Feedback Types

```swift
.sensoryFeedback(.success, trigger: value)
.sensoryFeedback(.warning, trigger: value)
.sensoryFeedback(.error, trigger: value)
.sensoryFeedback(.selection, trigger: value)
.sensoryFeedback(.increase, trigger: value)
.sensoryFeedback(.decrease, trigger: value)
.sensoryFeedback(.start, trigger: value)
.sensoryFeedback(.stop, trigger: value)
.sensoryFeedback(.alignment, trigger: value)
.sensoryFeedback(.levelChange, trigger: value)
.sensoryFeedback(.impact, trigger: value)
.sensoryFeedback(.impact(weight: .heavy), trigger: value)
.sensoryFeedback(.impact(flexibility: .rigid), trigger: value)
```

### Manual Haptic in SwiftUI

```swift
struct HapticButton: View {
    private let impact = UIImpactFeedbackGenerator(style: .medium)

    var body: some View {
        Button("Press Me") {
            impact.impactOccurred()
            performAction()
        }
        .onAppear {
            impact.prepare()
        }
    }
}
```

---

## Performance Optimization

### Preparing Generators

```swift
// Prepare before expected use
class HapticManager {
    private let impact = UIImpactFeedbackGenerator(style: .medium)
    private let selection = UISelectionFeedbackGenerator()
    private let notification = UINotificationFeedbackGenerator()

    init() {
        // Prepare for immediate use
        impact.prepare()
        selection.prepare()
        notification.prepare()
    }

    func triggerImpact() {
        impact.impactOccurred()
        impact.prepare() // Prepare for next use
    }
}
```

### Avoiding Excessive Haptics

```swift
// ❌ Haptic on every scroll position
ScrollView {
    content
}
.onChange(of: scrollOffset) { _, _ in
    haptic.impactOccurred() // Too frequent!
}

// ✅ Haptic only at snap points
.onChange(of: currentPage) { _, _ in
    selection.selectionChanged() // Only on page change
}
```

---

## Common Haptic Patterns

### Button Press

```swift
Button("Action") {
    UIImpactFeedbackGenerator(style: .medium).impactOccurred()
    performAction()
}
```

### Toggle Switch

```swift
Toggle("Setting", isOn: $isEnabled)
    .sensoryFeedback(.selection, trigger: isEnabled)
```

### Pull to Refresh

```swift
// Haptic when refresh triggers (not during pull)
.refreshable {
    UIImpactFeedbackGenerator(style: .heavy).impactOccurred()
    await refresh()
}
```

### Swipe Action

```swift
.swipeActions {
    Button("Delete", role: .destructive) {
        UINotificationFeedbackGenerator().notificationOccurred(.warning)
        delete()
    }
}
```

### Slider Ticks

```swift
@State private var value: Double = 0
private let tickValues = stride(from: 0.0, through: 1.0, by: 0.25)

Slider(value: $value)
    .onChange(of: value) { old, new in
        // Haptic at tick marks
        if tickValues.contains(where: { abs(new - $0) < 0.02 && abs(old - $0) >= 0.02 }) {
            UISelectionFeedbackGenerator().selectionChanged()
        }
    }
```

### Success/Error Completion

```swift
func submitForm() async {
    do {
        try await submit()
        UINotificationFeedbackGenerator().notificationOccurred(.success)
        showSuccessUI()
    } catch {
        UINotificationFeedbackGenerator().notificationOccurred(.error)
        showErrorUI()
    }
}
```

### Long Press Confirmation

```swift
.onLongPressGesture(minimumDuration: 0.5) {
    UINotificationFeedbackGenerator().notificationOccurred(.warning)
    confirmDeletion()
}
```

### Drag Snapping

```swift
DragGesture()
    .onChanged { value in
        let snappedPosition = round(value.translation.width / gridSize) * gridSize
        if snappedPosition != lastSnappedPosition {
            UIImpactFeedbackGenerator(style: .rigid).impactOccurred()
            lastSnappedPosition = snappedPosition
        }
    }
```

---

## Coordinating with Audio

### Multi-Modal Feedback

```swift
func showSuccess() {
    // Visual
    withAnimation { showCheckmark = true }

    // Haptic
    UINotificationFeedbackGenerator().notificationOccurred(.success)

    // Audio (optional - use sparingly)
    AudioServicesPlaySystemSound(1001)
}
```

### When to Add Sound

| Action | Haptic | Sound |
|--------|--------|-------|
| Button tap | Yes | No |
| Toggle | Yes | No |
| Success completion | Yes | Optional |
| Error | Yes | Optional |
| Message sent | Yes | Yes |
| Payment complete | Yes | Yes |
| Camera shutter | Yes | Yes |

---

## Platform Availability

### Checking Capabilities

```swift
// Haptics available on most modern iPhones
// Not available on iPad (no Taptic Engine)
// Not available on Mac Catalyst

#if os(iOS)
if UIDevice.current.userInterfaceIdiom == .phone {
    UIImpactFeedbackGenerator(style: .medium).impactOccurred()
}
#endif
```

### Graceful Degradation

```swift
func provideFeedback() {
    #if os(iOS)
    if UIDevice.current.userInterfaceIdiom == .phone {
        UIImpactFeedbackGenerator(style: .medium).impactOccurred()
    }
    #endif
    // Visual feedback always available
    withAnimation { highlightElement = true }
}
```

---

## Accessibility Considerations

### Reduce Motion Setting

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

// Haptics are independent of Reduce Motion
// Still provide haptic feedback
func onSelection() {
    UISelectionFeedbackGenerator().selectionChanged()

    // But reduce visual motion
    if reduceMotion {
        showChange()
    } else {
        withAnimation { showChange() }
    }
}
```

### Over-Reliance on Haptics

```swift
// ❌ Haptic-only feedback
func showStatus(_ status: Status) {
    switch status {
    case .success: notification.notificationOccurred(.success)
    case .error: notification.notificationOccurred(.error)
    }
}

// ✅ Visual + haptic feedback
func showStatus(_ status: Status) {
    switch status {
    case .success:
        notification.notificationOccurred(.success)
        showSuccessBanner()
    case .error:
        notification.notificationOccurred(.error)
        showErrorBanner()
    }
}
```

---

## Core Haptics (Advanced)

For custom haptic patterns beyond UIFeedbackGenerator.

```swift
import CoreHaptics

class CustomHaptics {
    private var engine: CHHapticEngine?

    init() {
        guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }

        do {
            engine = try CHHapticEngine()
            try engine?.start()
        } catch {
            print("Haptic engine error: \(error)")
        }
    }

    func playPattern() {
        guard let engine else { return }

        // Define events
        let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.7)
        let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5)

        let event = CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [intensity, sharpness],
            relativeTime: 0
        )

        do {
            let pattern = try CHHapticPattern(events: [event], parameters: [])
            let player = try engine.makePlayer(with: pattern)
            try player.start(atTime: 0)
        } catch {
            print("Haptic pattern error: \(error)")
        }
    }
}
```

---

## Anti-Patterns

```swift
// ❌ Haptic on every scroll
.onReceive(scrollPublisher) { _ in
    haptic.impactOccurred() // Annoying, drains battery
}

// ✅ Haptic only at meaningful points
.onReceive(pageChangePublisher) { _ in
    haptic.selectionChanged()
}

// ❌ Heavy haptic for minor action
Button("Cancel") {
    UIImpactFeedbackGenerator(style: .heavy).impactOccurred()
}

// ✅ Match intensity to importance
Button("Cancel") {
    UIImpactFeedbackGenerator(style: .light).impactOccurred()
}

// ❌ Multiple haptics in sequence
func celebrate() {
    notification.notificationOccurred(.success)
    impact.impactOccurred()
    impact.impactOccurred()
}

// ✅ Single appropriate haptic
func celebrate() {
    notification.notificationOccurred(.success)
}

// ❌ No visual companion to haptic
func deleteItem() {
    haptic.impactOccurred()
    // No visual feedback!
}

// ✅ Haptic accompanies visual
func deleteItem() {
    haptic.impactOccurred()
    withAnimation { items.remove(item) }
}
```

---

## Quick Reference

| Action | Generator | Style |
|--------|-----------|-------|
| Button press | Impact | .medium |
| Toggle | Selection | - |
| Segment change | Selection | - |
| Picker scroll | Selection | - |
| Pull to refresh | Impact | .heavy |
| Snap to grid | Impact | .rigid |
| Task success | Notification | .success |
| Task error | Notification | .error |
| Warning | Notification | .warning |
| Delete confirm | Notification | .warning |
| Subtle feedback | Impact | .soft |
| Strong feedback | Impact | .rigid |

| iOS 17+ Sensory Feedback |
|--------------------------|
| .success |
| .warning |
| .error |
| .selection |
| .increase |
| .decrease |
| .start |
| .stop |
| .alignment |
| .levelChange |
| .impact |
| .impact(weight:) |
| .impact(flexibility:) |

# Gestures & Touch Reference

Complete guide to gesture design, touch targets, and interaction patterns.

---

## Gesture Philosophy

**Apple's gesture principles:**
- Gestures should be discoverable or have visible alternatives
- Standard gestures should work as expected
- Never override system gestures
- Provide multiple ways to accomplish actions

---

## Standard Gestures

### Tap

```swift
// Single tap - primary action
.onTapGesture {
    performAction()
}

// Double tap - secondary action (zoom, like)
.onTapGesture(count: 2) {
    toggleZoom()
}
```

### Long Press

```swift
// Context menu trigger
.onLongPressGesture {
    showOptions()
}

// With minimum duration
.onLongPressGesture(minimumDuration: 0.5) {
    showMenu()
}

// Preferred: Use contextMenu
.contextMenu {
    Button("Edit") { }
    Button("Delete", role: .destructive) { }
}
```

### Swipe

```swift
// Swipe actions on list rows
.swipeActions(edge: .trailing) {
    Button("Delete", role: .destructive) { delete() }
}
.swipeActions(edge: .leading) {
    Button("Archive") { archive() }
}

// Custom swipe gesture
.gesture(
    DragGesture(minimumDistance: 30)
        .onEnded { value in
            if value.translation.width > 100 {
                // Swipe right
            } else if value.translation.width < -100 {
                // Swipe left
            }
        }
)
```

### Pan/Drag

```swift
@State private var offset = CGSize.zero

content
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = value.translation
            }
            .onEnded { value in
                withAnimation {
                    offset = .zero
                }
            }
    )
```

### Pinch (Scale)

```swift
@State private var scale: CGFloat = 1.0

content
    .scaleEffect(scale)
    .gesture(
        MagnifyGesture()
            .onChanged { value in
                scale = value.magnification
            }
            .onEnded { _ in
                withAnimation {
                    scale = max(1, min(scale, 4))
                }
            }
    )
```

### Rotation

```swift
@State private var angle: Angle = .zero

content
    .rotationEffect(angle)
    .gesture(
        RotateGesture()
            .onChanged { value in
                angle = value.rotation
            }
    )
```

---

## Gesture State

### Tracking Gesture State

```swift
@GestureState private var isDragging = false

content
    .opacity(isDragging ? 0.7 : 1.0)
    .gesture(
        DragGesture()
            .updating($isDragging) { _, state, _ in
                state = true
            }
    )
```

### Gesture Value Access

```swift
@GestureState private var dragOffset = CGSize.zero

content
    .offset(dragOffset)
    .gesture(
        DragGesture()
            .updating($dragOffset) { value, state, _ in
                state = value.translation
            }
            .onEnded { value in
                // Use final value
            }
    )
```

---

## Combining Gestures

### Simultaneous Gestures

```swift
// Both gestures active at once
.gesture(
    rotationGesture
        .simultaneously(with: magnificationGesture)
)
```

### Sequential Gestures

```swift
// Second gesture after first completes
.gesture(
    longPressGesture
        .sequenced(before: dragGesture)
)
```

### Exclusive Gestures

```swift
// Only one gesture wins
.gesture(
    tapGesture
        .exclusively(before: longPressGesture)
)
```

### Priority Gestures

```swift
// High priority gesture takes precedence
.highPriorityGesture(
    DragGesture()
        .onChanged { ... }
)
```

---

## Touch Targets

### Minimum Size Requirements

| Platform | Minimum Touch Target |
|----------|---------------------|
| iOS | 44 x 44 points |
| iPadOS | 44 x 44 points |
| watchOS | 38 x 38 points |
| macOS | Smaller OK (pointer) |

### Implementing Touch Targets

```swift
// ❌ Too small
Button { } label: {
    Image(systemName: "xmark")
        .font(.caption)
}

// ✅ Visual can be small, hit area is 44pt
Button { } label: {
    Image(systemName: "xmark")
        .font(.caption)
}
.frame(minWidth: 44, minHeight: 44)
.contentShape(Rectangle())
```

### Expanded Hit Areas

```swift
// Make small visual element tappable
Image(systemName: "xmark")
    .font(.caption)
    .padding(14) // Expands to 44pt total
    .contentShape(Rectangle())
    .onTapGesture { dismiss() }
```

### Touch Target Spacing

| Context | Minimum Spacing |
|---------|-----------------|
| Buttons | 8-12pt |
| List items | Full width |
| Icons in toolbar | 12-16pt |
| Dense layouts | 8pt minimum |

---

## Hit Testing

### Content Shape

```swift
// Make entire area tappable
HStack {
    Image(systemName: "star")
    Text("Favorite")
    Spacer()
}
.contentShape(Rectangle())
.onTapGesture { toggleFavorite() }
```

### Custom Hit Testing

```swift
// Circle hit area
Circle()
    .fill(Color.blue)
    .frame(width: 60, height: 60)
    .contentShape(Circle())
    .onTapGesture { }

// Capsule hit area
Capsule()
    .contentShape(Capsule())
```

### Allowing Hit Testing Through

```swift
// Allow touches to pass through
Color.clear
    .allowsHitTesting(false)

// Conditional hit testing
.allowsHitTesting(isInteractive)
```

---

## System Gesture Interactions

### Edge Swipes

```swift
// ⚠️ Don't override system edge swipes
// Back navigation (left edge)
// Control Center (top edge)
// Home indicator (bottom edge)

// If you must use edge, use with care
.edgesIgnoringSafeArea(.all)
.gesture(
    DragGesture()
        .onChanged { value in
            // Check if from edge
            if value.startLocation.x < 20 {
                // Handle edge gesture
            }
        }
)
```

### Preserving System Gestures

```swift
// Let NavigationStack handle back swipe
NavigationStack {
    content
    // Don't add conflicting drag gesture
}
```

---

## Gesture Feedback

### Visual Feedback

```swift
@State private var isPressed = false

Button { } label: {
    Text("Button")
        .scaleEffect(isPressed ? 0.95 : 1.0)
        .animation(.easeOut(duration: 0.1), value: isPressed)
}
.simultaneousGesture(
    DragGesture(minimumDistance: 0)
        .onChanged { _ in isPressed = true }
        .onEnded { _ in isPressed = false }
)
```

### Haptic Feedback

```swift
// Selection change
let selection = UISelectionFeedbackGenerator()
selection.selectionChanged()

// Impact
let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()

// Notification
let notification = UINotificationFeedbackGenerator()
notification.notificationOccurred(.success)
```

### Gesture-Driven Animation

```swift
@GestureState private var dragOffset = CGSize.zero

Card()
    .offset(dragOffset)
    .rotationEffect(.degrees(Double(dragOffset.width) / 10))
    .animation(.interactiveSpring(), value: dragOffset)
    .gesture(
        DragGesture()
            .updating($dragOffset) { value, state, _ in
                state = value.translation
            }
    )
```

---

## Velocity and Momentum

### Using Predicted End

```swift
DragGesture()
    .onEnded { value in
        let predictedEnd = value.predictedEndTranslation

        // Use velocity for throw effect
        if predictedEnd.width > 200 {
            // Strong swipe right
            withAnimation(.spring()) {
                dismissCard()
            }
        }
    }
```

### Snapping

```swift
@State private var position = 0

DragGesture()
    .onEnded { value in
        let threshold: CGFloat = 100
        let velocity = value.predictedEndTranslation.width

        withAnimation(.spring()) {
            if velocity > threshold {
                position = min(position + 1, maxPosition)
            } else if velocity < -threshold {
                position = max(position - 1, 0)
            }
        }
    }
```

---

## Accessibility Alternatives

### Gesture Alternatives Required

```swift
// ❌ Swipe-only action
.swipeActions {
    Button("Delete") { delete() }
}

// ✅ Swipe + context menu
.swipeActions {
    Button("Delete") { delete() }
}
.contextMenu {
    Button("Delete", role: .destructive) { delete() }
}
```

### VoiceOver Gestures

```swift
// Custom actions for VoiceOver
.accessibilityAction(named: "Delete") {
    delete()
}

.accessibilityAction(.magicTap) {
    togglePlayback()
}
```

### Full Keyboard Access

```swift
// Focusable for keyboard users
.focusable()
.onKeyPress(.space) {
    activate()
    return .handled
}
```

---

## Common Gesture Patterns

### Pull to Refresh

```swift
List {
    content
}
.refreshable {
    await refreshData()
}
```

### Drag to Reorder

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
    .onMove { from, to in
        items.move(fromOffsets: from, toOffset: to)
    }
}
```

### Swipe to Delete

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
    .onDelete { indexSet in
        items.remove(atOffsets: indexSet)
    }
}
```

### Dismissible Card

```swift
@State private var offset = CGSize.zero
@State private var isDismissed = false

Card()
    .offset(offset)
    .rotationEffect(.degrees(Double(offset.width / 10)))
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = value.translation
            }
            .onEnded { value in
                if abs(value.translation.width) > 150 {
                    withAnimation {
                        offset.width = value.translation.width > 0 ? 500 : -500
                        isDismissed = true
                    }
                } else {
                    withAnimation(.spring()) {
                        offset = .zero
                    }
                }
            }
    )
    .opacity(isDismissed ? 0 : 1)
```

### Zoomable Image

```swift
@State private var scale: CGFloat = 1.0
@State private var lastScale: CGFloat = 1.0

Image("photo")
    .scaleEffect(scale)
    .gesture(
        MagnifyGesture()
            .onChanged { value in
                scale = lastScale * value.magnification
            }
            .onEnded { _ in
                lastScale = scale
                withAnimation {
                    scale = max(1, min(scale, 4))
                    lastScale = scale
                }
            }
    )
    .onTapGesture(count: 2) {
        withAnimation {
            scale = scale > 1 ? 1 : 2
            lastScale = scale
        }
    }
```

---

## Anti-Patterns

```swift
// ❌ Overriding system edge swipe
.gesture(
    DragGesture()
        .onChanged { value in
            if value.startLocation.x < 20 {
                // Conflicts with back navigation
            }
        }
)

// ✅ Use NavigationStack's built-in back gesture

// ❌ Gesture-only interaction
.gesture(LongPressGesture().onEnded { showMenu() })

// ✅ Visible alternative
.contextMenu { menuContent }

// ❌ Tiny touch targets
Button { } label: {
    Image(systemName: "xmark")
}
.frame(width: 20, height: 20)

// ✅ Adequate touch area
Button { } label: {
    Image(systemName: "xmark")
}
.frame(minWidth: 44, minHeight: 44)

// ❌ No gesture feedback
.onTapGesture { action() }

// ✅ Visual + haptic feedback
.onTapGesture {
    withAnimation { highlight = true }
    UIImpactFeedbackGenerator(style: .light).impactOccurred()
    action()
}
```

---

## Quick Reference

| Gesture | Usage | Min Target |
|---------|-------|------------|
| Tap | Primary action | 44x44pt |
| Double tap | Zoom, like | 44x44pt |
| Long press | Context menu | 44x44pt |
| Swipe | Row actions | Full width |
| Pan/Drag | Move, dismiss | 44x44pt |
| Pinch | Zoom | Content area |
| Rotate | Rotate content | Content area |

| System Gesture | Edge | Don't Override |
|----------------|------|----------------|
| Back navigation | Left | NavigationStack |
| Control Center | Top | Full screen |
| Home | Bottom | System |
| Notification | Top | System |

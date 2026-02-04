# Motion & Animation Reference

Apple's motion design creates interfaces that feel alive, responsive, and natural. This reference covers timing, physics, choreography, and accessibility.

---

## Motion Philosophy

**Core principle:** "Motion should be meaningful, not decorative."

Apple's motion design serves these purposes:
1. **Feedback** — Confirm user actions
2. **Orientation** — Show where elements came from/go to
3. **Focus** — Direct attention to important changes
4. **Continuity** — Connect related states
5. **Delight** — Make interactions feel polished (sparingly)

---

## Timing Curves

### System Curves

```swift
// Default - Natural deceleration
.animation(.easeOut, value: state)

// Symmetric - Gentle acceleration and deceleration
.animation(.easeInOut, value: state)

// Accelerating - Exiting animations
.animation(.easeIn, value: state)

// Linear - Constant speed (rare)
.animation(.linear, value: state)
```

### When to Use Each Curve

| Curve | When | Example |
|-------|------|---------|
| **easeOut** | Element appearing, user initiated | Button tap, view appear |
| **easeInOut** | State changes, continuous motion | Toggle, expand/collapse |
| **easeIn** | Element leaving, dismissing | Swipe away, fade out |
| **linear** | Progress indicators, loops | Loading spinner |

---

## Duration Guidelines

### High-Frequency UI (Buttons, Toggles)

```swift
// Fast - 100-200ms
.animation(.easeOut(duration: 0.15), value: isPressed)
.animation(.easeOut(duration: 0.18), value: isToggled)
```

### Standard Transitions

```swift
// Medium - 200-350ms
.animation(.easeInOut(duration: 0.25), value: isExpanded)
.animation(.easeInOut(duration: 0.3), value: selectedTab)
```

### Complex Sequences

```swift
// Longer - 350-500ms
.animation(.easeInOut(duration: 0.4), value: isDetailShown)
```

### Duration by Context

| Context | Duration | Notes |
|---------|----------|-------|
| Button press feedback | 100-150ms | Immediate feel |
| Toggle state change | 150-200ms | Quick but visible |
| Card expand/collapse | 250-300ms | Show transformation |
| Screen transition | 300-400ms | Orientation context |
| Complex choreography | 400-600ms | Multiple elements |

---

## Spring Animations

Springs create natural, physics-based motion.

### Standard Springs

```swift
// Default spring - Balanced
.animation(.spring(), value: state)

// Bouncy - Playful elements
.animation(.spring(response: 0.5, dampingFraction: 0.6), value: state)

// Snappy - Quick, minimal overshoot
.animation(.spring(response: 0.3, dampingFraction: 0.8), value: state)

// Smooth - No overshoot
.animation(.spring(response: 0.4, dampingFraction: 1.0), value: state)
```

### Spring Parameters

```swift
.spring(
    response: 0.5,           // Duration-like (how long)
    dampingFraction: 0.7,    // Bounciness (0 = infinite, 1 = no bounce)
    blendDuration: 0         // Transition blend
)
```

### Spring Selection Guide

| Damping | Feel | Use Case |
|---------|------|----------|
| 0.5-0.6 | Bouncy | Playful UI, games |
| 0.7-0.8 | Responsive | General UI |
| 0.9-1.0 | Smooth | Professional, subtle |

---

## Interactive Springs (iOS 17+)

```swift
// Interruptible spring for gestures
.animation(.interactiveSpring(response: 0.3), value: offset)

// Snappy spring for direct manipulation
.animation(.snappy, value: state)

// Smooth spring for ambient motion
.animation(.smooth, value: state)

// Bouncy spring for playful feedback
.animation(.bouncy, value: state)
```

---

## Transition Animations

### View Transitions

```swift
// Opacity
.transition(.opacity)

// Scale
.transition(.scale)

// Slide
.transition(.slide)

// Move from edge
.transition(.move(edge: .bottom))

// Combined
.transition(.scale.combined(with: .opacity))

// Asymmetric (different in/out)
.transition(.asymmetric(
    insertion: .scale.combined(with: .opacity),
    removal: .opacity
))
```

### Custom Transitions

```swift
extension AnyTransition {
    static var slideUp: AnyTransition {
        .asymmetric(
            insertion: .move(edge: .bottom).combined(with: .opacity),
            removal: .move(edge: .top).combined(with: .opacity)
        )
    }
}

// Usage
content
    .transition(.slideUp)
```

---

## Choreography

### Staggered Animations

```swift
ForEach(Array(items.enumerated()), id: \.element.id) { index, item in
    ItemView(item: item)
        .animation(
            .spring().delay(Double(index) * 0.05),
            value: isVisible
        )
}
```

### Sequenced Animations

```swift
// Phase 1: Container appears
withAnimation(.easeOut(duration: 0.2)) {
    showContainer = true
}

// Phase 2: Content fades in
withAnimation(.easeOut(duration: 0.2).delay(0.1)) {
    showContent = true
}

// Phase 3: Actions slide up
withAnimation(.spring().delay(0.2)) {
    showActions = true
}
```

### Keyframe Animations (iOS 17+)

```swift
content
    .keyframeAnimator(
        initialValue: AnimationValues(),
        trigger: trigger
    ) { content, value in
        content
            .scaleEffect(value.scale)
            .opacity(value.opacity)
    } keyframes: { _ in
        KeyframeTrack(\.scale) {
            SpringKeyframe(1.2, duration: 0.2)
            SpringKeyframe(1.0, duration: 0.3)
        }
        KeyframeTrack(\.opacity) {
            LinearKeyframe(1.0, duration: 0.1)
        }
    }
```

---

## Gesture-Driven Animation

### Following Gesture Position

```swift
@GestureState private var dragOffset = CGSize.zero

content
    .offset(dragOffset)
    .gesture(
        DragGesture()
            .updating($dragOffset) { value, state, _ in
                state = value.translation
            }
    )
    .animation(.interactiveSpring(), value: dragOffset)
```

### Velocity-Based Animation

```swift
@State private var offset: CGFloat = 0

content
    .offset(y: offset)
    .gesture(
        DragGesture()
            .onChanged { offset = $0.translation.height }
            .onEnded { value in
                let velocity = value.predictedEndTranslation.height
                withAnimation(.spring(response: 0.3, dampingFraction: 0.8)) {
                    offset = velocity > 100 ? 300 : 0
                }
            }
    )
```

---

## Reduce Motion Accessibility

### Checking the Setting

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

var animation: Animation? {
    reduceMotion ? nil : .spring()
}

content
    .animation(animation, value: state)
```

### Alternative Animations

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

var body: some View {
    content
        .animation(
            reduceMotion
                ? .linear(duration: 0.1)  // Simple fade
                : .spring(response: 0.4), // Full spring
            value: state
        )
}
```

### What to Reduce

| Full Motion | Reduced Alternative |
|-------------|---------------------|
| Bouncy spring | Simple fade or none |
| Parallax | Static |
| Auto-playing video | Paused with play button |
| Continuous loops | Static or very subtle |
| Z-axis motion | 2D transition |
| Scaling | Opacity only |

---

## Animation Interruptibility

### Ensuring Interruptibility

```swift
// ✅ Interruptible - User can change state mid-animation
@State private var isExpanded = false

content
    .frame(height: isExpanded ? 300 : 100)
    .animation(.spring(), value: isExpanded)
    .onTapGesture {
        isExpanded.toggle()
    }
// Tapping again mid-animation redirects smoothly
```

### Avoiding Blocking Animations

```swift
// ❌ Blocking - Disables interaction during animation
Button("Expand") {
    withAnimation(.easeInOut(duration: 2)) {
        isExpanded = true
    }
}
.disabled(isAnimating)

// ✅ Non-blocking - User controls at all times
Button("Expand") {
    isExpanded.toggle()
}
// Animation is visual polish, not gate
```

---

## Performance Optimization

### Animatable Properties

**Efficient (GPU-accelerated):**
- `opacity`
- `scale` / `scaleEffect`
- `rotation` / `rotationEffect`
- `offset`

**Expensive (triggers layout):**
- `frame` size changes
- Content changes
- Complex path animations

### Optimization Techniques

```swift
// ❌ Expensive - Animating frame
.frame(width: isExpanded ? 300 : 100)
.animation(.spring(), value: isExpanded)

// ✅ Efficient - Animating scale
.scaleEffect(isExpanded ? 1.5 : 1.0)
.animation(.spring(), value: isExpanded)

// Use drawingGroup for complex animations
ComplexView()
    .drawingGroup() // Rasterizes to image
    .opacity(animatedOpacity)
```

---

## Common Animation Patterns

### Button Press Feedback

```swift
Button { action() } label: {
    Text("Button")
}
.scaleEffect(isPressed ? 0.95 : 1.0)
.animation(.easeOut(duration: 0.1), value: isPressed)
```

### Card Expand

```swift
CardView()
    .frame(height: isExpanded ? 300 : 100)
    .animation(.spring(response: 0.35, dampingFraction: 0.8), value: isExpanded)
```

### View Appear

```swift
content
    .opacity(isVisible ? 1 : 0)
    .offset(y: isVisible ? 0 : 20)
    .animation(.easeOut(duration: 0.3), value: isVisible)
    .onAppear { isVisible = true }
```

### Tab Switch

```swift
TabView(selection: $selectedTab) {
    // tabs
}
.animation(.easeInOut(duration: 0.25), value: selectedTab)
```

### Pull to Refresh

```swift
// Built into List with .refreshable
// Custom implementation:
content
    .offset(y: refreshOffset)
    .animation(.spring(response: 0.4), value: isRefreshing)
```

### Loading Pulse

```swift
Circle()
    .opacity(isPulsing ? 0.3 : 1.0)
    .animation(
        .easeInOut(duration: 0.8)
        .repeatForever(autoreverses: true),
        value: isPulsing
    )
    .onAppear { isPulsing = true }
```

---

## Anti-Patterns

```swift
// ❌ Animation on every keystroke
TextField("Search", text: $query)
    .onChange(of: query) {
        withAnimation { updateResults() }
    }

// ✅ No animation for high-frequency updates
TextField("Search", text: $query)
    .onChange(of: query) { updateResults() }

// ❌ Blocking user during animation
.disabled(isAnimating)

// ✅ Allow interruption
// Don't disable controls during animations

// ❌ Ignoring Reduce Motion
.animation(.spring(), value: state)

// ✅ Respecting accessibility
@Environment(\.accessibilityReduceMotion) var reduceMotion
.animation(reduceMotion ? nil : .spring(), value: state)

// ❌ Overly long duration
.animation(.easeInOut(duration: 1.5), value: state)

// ✅ Appropriate duration
.animation(.easeInOut(duration: 0.3), value: state)

// ❌ Gratuitous animation
// Every element bouncing, spinning, etc.

// ✅ Purposeful animation
// Motion serves feedback, orientation, or focus
```

---

## Quick Reference

| Action | Duration | Curve | Notes |
|--------|----------|-------|-------|
| Button tap | 100-150ms | easeOut | Immediate feedback |
| Toggle | 150-200ms | spring | Quick state change |
| Expand/collapse | 250-300ms | spring | Show transformation |
| Screen transition | 300-400ms | spring | Orientation |
| Modal present | 300-400ms | spring | Focus shift |
| Modal dismiss | 250-300ms | easeIn | Exit acceleration |
| Stagger delay | 50-100ms | - | Between items |

---

## SwiftUI Animation Cheatsheet

```swift
// Implicit animation
.animation(.spring(), value: state)

// Explicit animation
withAnimation(.spring()) {
    state = newValue
}

// Transaction for fine control
var transaction = Transaction(animation: .spring())
transaction.disablesAnimations = true
withTransaction(transaction) {
    state = newValue
}

// Phase animator (iOS 17+)
.phaseAnimator([Phase.initial, .middle, .final]) { content, phase in
    content
        .opacity(phase == .initial ? 0 : 1)
}

// Keyframe animator (iOS 17+)
.keyframeAnimator(initialValue: Values(), trigger: trigger) { content, value in
    content.scaleEffect(value.scale)
} keyframes: { _ in
    KeyframeTrack(\.scale) {
        SpringKeyframe(1.2, duration: 0.2)
    }
}
```

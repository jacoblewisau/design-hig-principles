# Materials & Visual Depth Reference

Apple's visual layer system creates hierarchy through materials, blur, and depth. This reference covers the complete visual effects system.

---

## Materials Philosophy

Apple's design language uses materials to:
- Create visual hierarchy without borders
- Allow content to "live" in the interface
- Maintain context while focusing attention
- Adapt to light and dark appearances

**Key principle:** "Materials should recede, allowing content to come forward."

---

## System Materials

### UIKit Material Types

```swift
// Ultra thin - Maximum transparency
UIBlurEffect(style: .systemUltraThinMaterial)

// Thin - High transparency
UIBlurEffect(style: .systemThinMaterial)

// Regular - Balanced
UIBlurEffect(style: .systemMaterial)

// Thick - More opacity
UIBlurEffect(style: .systemThickMaterial)

// Chrome - For toolbars
UIBlurEffect(style: .systemChromeMaterial)
```

### SwiftUI Materials

```swift
// Available materials (thinnest to thickest)
.background(.ultraThinMaterial)
.background(.thinMaterial)
.background(.regularMaterial)
.background(.thickMaterial)
.background(.bar)  // Navigation/tab bar style
```

### Material Selection Guidelines

| Context | Material | Rationale |
|---------|----------|-----------|
| Floating overlays | Ultra thin | Maximum content visibility |
| Action sheets | Thin | Balance visibility + readability |
| Sidebars | Regular | Clear separation |
| Modals | Thick | Focus on modal content |
| Navigation bars | Bar/Chrome | System consistency |

---

## Vibrancy

Vibrancy adapts foreground content to maintain legibility over materials.

### Vibrancy Styles

```swift
// UIKit vibrancy
UIVibrancyEffect(blurEffect: blur, style: .label)
UIVibrancyEffect(blurEffect: blur, style: .secondaryLabel)
UIVibrancyEffect(blurEffect: blur, style: .tertiaryLabel)
UIVibrancyEffect(blurEffect: blur, style: .fill)
UIVibrancyEffect(blurEffect: blur, style: .separator)
```

### SwiftUI Vibrancy

```swift
// Text automatically gains vibrancy over materials
ZStack {
    Color.blue.ignoresSafeArea()

    VStack {
        Text("Vibrant text")
    }
    .padding()
    .background(.thinMaterial)
}
// Text adapts color for legibility
```

---

## Visual Hierarchy Layers

Apple defines distinct visual layers:

### Layer 0: Base Content
The main content layer—photos, documents, primary UI.

### Layer 1: Secondary Content
Cards, grouped content, secondary surfaces.

```swift
VStack {
    content
}
.background(Color(.secondarySystemBackground))
```

### Layer 2: Elevated Content
Sheets, popovers, floating elements.

```swift
.sheet(isPresented: $showSheet) {
    SheetContent()
}
// Automatically uses elevated appearance
```

### Layer 3: Alerts & System UI
Highest priority—alerts, notifications.

```swift
.alert("Title", isPresented: $showAlert) {
    // System handles elevation
}
```

---

## Shadows

Shadows indicate elevation and create depth hierarchy.

### System Shadow Styles

```swift
// Subtle shadow (cards, buttons)
.shadow(color: .black.opacity(0.1), radius: 4, y: 2)

// Medium shadow (floating elements)
.shadow(color: .black.opacity(0.15), radius: 8, y: 4)

// Prominent shadow (modals, popovers)
.shadow(color: .black.opacity(0.2), radius: 16, y: 8)

// System shadow (Material Design aligned)
.shadow(radius: 4) // SwiftUI default
```

### Shadow Guidelines

| Elevation | Shadow | Use Case |
|-----------|--------|----------|
| On surface | None or 2pt | Buttons, toggles |
| Slight raise | 4pt blur, 2pt offset | Cards |
| Floating | 8pt blur, 4pt offset | FABs, toolbars |
| Modal | 16pt blur, 8pt offset | Sheets |
| High priority | 24pt blur, 12pt offset | Alerts |

### Dark Mode Shadows

```swift
// Shadows less effective in dark mode
// Use lighter backgrounds instead
@Environment(\.colorScheme) var colorScheme

var elevation: some View {
    if colorScheme == .dark {
        // Lighter background instead of shadow
        Color(.secondarySystemBackground)
    } else {
        // Shadow in light mode
        Color(.systemBackground)
            .shadow(radius: 8)
    }
}
```

---

## Blur Effects

### Implementation

```swift
// SwiftUI blur
Image("photo")
    .blur(radius: 10)

// Behind content (material)
content
    .background(.ultraThinMaterial)

// UIKit blur view
let blurEffect = UIBlurEffect(style: .systemMaterial)
let blurView = UIVisualEffectView(effect: blurEffect)
```

### Blur Radius Guidelines

| Effect | Radius | Use Case |
|--------|--------|----------|
| Subtle | 2-4pt | Slight softening |
| Standard | 8-12pt | Materials, overlays |
| Heavy | 16-24pt | Full obscuring |
| Extreme | 32pt+ | Loading states, transitions |

---

## Depth in visionOS

visionOS introduces true 3D depth:

### Z-Axis Positioning

```swift
// Depth modifier
Model3D(named: "Object")
    .frame(depth: 100)

// Offset in z-axis
content
    .offset(z: 50)
```

### Depth Guidelines

- Reserve depth for important, interactive content
- Keep comfortable viewing distance (1-3 meters)
- Avoid excessive z-axis variation
- Center important content in field of view

### Glass Material (Liquid Glass)

```swift
// visionOS glass material
content
    .glassBackgroundEffect()

// With shape
content
    .glassBackgroundEffect(
        in: RoundedRectangle(cornerRadius: 32, style: .continuous)
    )
```

---

## Translucency Patterns

### When to Use Translucency

| Context | Use | Don't Use |
|---------|-----|-----------|
| Temporary overlays | ✅ | |
| Navigation chrome | ✅ | |
| Reading content | | ❌ Distracting |
| Data entry | | ❌ Reduces focus |
| Accessibility (Reduce Transparency) | Respect setting | |

### Respecting Reduce Transparency

```swift
@Environment(\.accessibilityReduceTransparency) var reduceTransparency

var background: some ShapeStyle {
    if reduceTransparency {
        return AnyShapeStyle(Color(.systemBackground))
    } else {
        return AnyShapeStyle(.regularMaterial)
    }
}
```

---

## Layered Backgrounds

### Card Over Material

```swift
ZStack {
    // Base content
    Image("background")
        .ignoresSafeArea()

    // Material layer
    VStack {
        // Card on material
        VStack {
            Text("Card content")
        }
        .padding()
        .background(Color(.secondarySystemGroupedBackground))
        .clipShape(RoundedRectangle(cornerRadius: 12, style: .continuous))
    }
    .padding()
    .background(.regularMaterial)
}
```

### Nested Elevation

```swift
// Each layer progressively more opaque
ZStack {
    Color(.systemBackground)        // L0: Base

    VStack {
        content
    }
    .background(Color(.secondarySystemBackground))  // L1: Card

    .overlay {
        VStack {
            Text("Floating")
        }
        .background(Color(.tertiarySystemBackground))  // L2: Elevated
    }
}
```

---

## Animation with Materials

### Animating Material Changes

```swift
@State private var isExpanded = false

VStack {
    content
}
.background(isExpanded ? .thickMaterial : .thinMaterial)
.animation(.easeInOut, value: isExpanded)
```

### Blur Transitions

```swift
Image("photo")
    .blur(radius: isLoading ? 20 : 0)
    .animation(.easeOut(duration: 0.3), value: isLoading)
```

---

## Performance Considerations

### Material Performance

```swift
// ❌ Expensive - nested materials
VStack {
    ForEach(items) { item in
        Text(item.name)
            .background(.thinMaterial) // Each item creates blur
    }
}

// ✅ Efficient - single material
VStack {
    ForEach(items) { item in
        Text(item.name)
    }
}
.background(.thinMaterial) // One blur for container
```

### Reducing GPU Load

- Minimize overlapping blur layers
- Use solid colors when materials aren't necessary
- Limit blur radius
- Prefer system materials (optimized)

---

## Light & Dark Mode Adaptation

Materials automatically adapt:

### Light Mode
- Materials appear translucent white
- Shadows visible
- Higher contrast

### Dark Mode
- Materials appear translucent dark
- Shadows less effective
- Elevation through brightness

```swift
// No special handling needed
content
    .background(.regularMaterial)
// Automatically correct in both modes
```

---

## Common Material Patterns

### Floating Action Button

```swift
Button { action() } label: {
    Image(systemName: "plus")
        .font(.title2)
        .foregroundStyle(.white)
        .padding()
        .background(Color.accentColor)
        .clipShape(Circle())
        .shadow(color: .black.opacity(0.2), radius: 8, y: 4)
}
```

### Overlay Sheet

```swift
VStack {
    // Handle
    Capsule()
        .fill(Color(.tertiaryLabel))
        .frame(width: 36, height: 5)
        .padding(.top, 8)

    // Content
    sheetContent
}
.background(.thickMaterial)
.clipShape(RoundedRectangle(cornerRadius: 20, style: .continuous))
```

### Frosted Toolbar

```swift
.toolbar {
    ToolbarItemGroup(placement: .bottomBar) {
        // System applies .bar material automatically
        toolbarContent
    }
}
.toolbarBackground(.visible, for: .bottomBar)
```

---

## Anti-Patterns

```swift
// ❌ Multiple nested materials (performance)
VStack {
    content
        .background(.thinMaterial)
}
.background(.thinMaterial)

// ✅ Single material layer
VStack {
    content
}
.background(.thinMaterial)

// ❌ Material over material without purpose
ZStack {
    Color.clear.background(.regularMaterial)
    VStack {
        Text("Content")
    }
    .background(.thinMaterial)
}

// ✅ Intentional layering
ZStack {
    backgroundContent
    VStack {
        Text("Content")
    }
    .background(.regularMaterial)
}

// ❌ Ignoring Reduce Transparency
.background(.ultraThinMaterial)

// ✅ Respecting accessibility
@Environment(\.accessibilityReduceTransparency) var reduceTransparency
.background(reduceTransparency ? Color(.systemBackground) : .ultraThinMaterial)
```

---

## Quick Reference

| Need | Solution |
|------|----------|
| Floating overlay | `.ultraThinMaterial` |
| Sheet/modal | `.thickMaterial` |
| Toolbar | `.bar` or `.systemChromeMaterial` |
| Card elevation (light) | Shadow |
| Card elevation (dark) | Lighter background |
| visionOS surfaces | `.glassBackgroundEffect()` |
| Respect accessibility | Check `reduceTransparency` |

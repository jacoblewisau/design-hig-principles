# Dark Mode Design Reference

Complete guide to designing for Dark Mode, color adaptation, and appearance handling.

---

## Dark Mode Philosophy

**Apple's guidance:** "Think of Dark Mode as having the lights dimmed rather than flipping colors inside out."

**Key principles:**
- Colors are redesigned, not inverted
- Maintain readability and contrast
- Preserve visual hierarchy
- Elevation through brightness, not shadows
- Respect user system preference

---

## Understanding Dark Mode Colors

### How Colors Adapt

| Light Mode | Dark Mode | Note |
|------------|-----------|------|
| White background | Near-black | Not pure black |
| Black text | White text | High contrast |
| Gray backgrounds | Darker grays | Layered |
| Shadows | Lighter surfaces | Elevation shift |
| Vibrant colors | Adjusted saturation | Easier on eyes |

### Why Not Pure Black?

```swift
// ❌ Pure black is harsh
Color.black // #000000

// ✅ System uses elevated blacks
Color(.systemBackground) // #000000 base, but elevated surfaces are lighter
```

Pure black creates harsh contrast and OLED "smearing". Apple uses near-black with slight gray tones.

---

## Semantic Color System

### Background Hierarchy

```swift
// Ungrouped (standard lists)
Color(.systemBackground)           // Base - darkest
Color(.secondarySystemBackground)  // Cards/sections
Color(.tertiarySystemBackground)   // Nested content

// Grouped (Settings-style)
Color(.systemGroupedBackground)           // Table background
Color(.secondarySystemGroupedBackground)  // Cell background
Color(.tertiarySystemGroupedBackground)   // Nested content
```

### Text Hierarchy

```swift
.foregroundStyle(.primary)    // Maximum contrast
.foregroundStyle(.secondary)  // 60% opacity
.foregroundStyle(.tertiary)   // 30% opacity
.foregroundStyle(.quaternary) // 18% opacity
```

### Fill Colors

```swift
Color(.systemFill)           // Most visible fill
Color(.secondarySystemFill)  // Medium fill
Color(.tertiarySystemFill)   // Subtle fill
Color(.quaternarySystemFill) // Minimal fill
```

---

## Base vs Elevated Surfaces

Dark Mode has two background contexts:

### Base Level
Used for primary app window, full-screen content.

### Elevated Level
Used for:
- Sheets and modals
- Popovers
- Slide-over apps on iPad
- Floating elements

```swift
// The system automatically applies elevated colors for:
.sheet { }
.popover { }
.fullScreenCover { }
```

---

## Custom Colors

### Asset Catalog Setup

1. Open `Assets.xcassets`
2. Add new Color Set
3. In Attributes Inspector, set Appearances to "Any, Dark"
4. Define both variants

### Defining Custom Colors

```swift
// In Asset Catalog:
// BrandBlue (Any): #0066CC
// BrandBlue (Dark): #4D9EFF (lighter for dark backgrounds)

// Usage
Color("BrandBlue")
    .foregroundStyle(Color("BrandBlue"))
```

### Programmatic Custom Colors

```swift
extension Color {
    static let brandBlue = Color("BrandBlue")

    // Or dynamic color
    static let dynamicBrand = Color(
        UIColor { traitCollection in
            traitCollection.userInterfaceStyle == .dark
                ? UIColor(red: 0.3, green: 0.62, blue: 1, alpha: 1)
                : UIColor(red: 0, green: 0.4, blue: 0.8, alpha: 1)
        }
    )
}
```

---

## Tint Colors

System tints automatically adapt:

```swift
// System colors auto-adjust
.tint(.blue)  // Lighter blue in dark mode

// Custom tints need both variants
.tint(Color("BrandAccent"))
```

### Tint Adjustment Guidelines

| Light Mode | Dark Mode | Why |
|------------|-----------|-----|
| Saturated | Desaturated | Easier on eyes |
| Darker shade | Lighter shade | Visibility |
| Higher contrast | Similar contrast | Maintain hierarchy |

---

## Images and Icons

### SF Symbols

SF Symbols automatically adapt with proper template rendering.

```swift
Image(systemName: "star.fill")
    .foregroundStyle(.yellow) // Adapts automatically
```

### Custom Images

```swift
// Asset catalog with appearance variants
Image("CustomIcon")
// Provide "Any" and "Dark" versions

// Or use template mode
Image("CustomIcon")
    .renderingMode(.template)
    .foregroundStyle(.primary)
```

### Photos and Content

Don't modify user photos or content images for dark mode.

```swift
// ❌ Don't darken photos
Image(photo)
    .brightness(-0.2) // Wrong!

// ✅ Photos display as-is
Image(photo)
```

---

## Shadows in Dark Mode

Shadows are less effective in dark mode. Use elevation instead.

```swift
@Environment(\.colorScheme) var colorScheme

var cardStyle: some View {
    if colorScheme == .dark {
        // Lighter background for elevation
        Color(.secondarySystemBackground)
    } else {
        // Shadow for elevation
        Color(.systemBackground)
            .shadow(color: .black.opacity(0.1), radius: 8, y: 4)
    }
}
```

---

## Materials and Blur

Materials adapt automatically:

```swift
.background(.regularMaterial)
// Appropriate transparency for both modes
```

### Vibrancy

Text over materials automatically gains vibrancy for legibility.

```swift
ZStack {
    Image("background")
    Text("Vibrant")
        .foregroundStyle(.primary)
        .padding()
        .background(.thinMaterial)
}
// Text adapts for visibility
```

---

## Detecting Color Scheme

### Environment Value

```swift
@Environment(\.colorScheme) var colorScheme

var body: some View {
    VStack {
        if colorScheme == .dark {
            // Dark-specific content
        } else {
            // Light-specific content
        }
    }
}
```

### Reacting to Changes

```swift
.onChange(of: colorScheme) { _, newScheme in
    // Respond to appearance change
}
```

---

## Permanent Dark Mode

**Apple's guidance:** Only for media-focused apps.

### Apps That Use Permanent Dark

| App | Rationale |
|-----|-----------|
| Camera | Reduce distraction |
| Photos | Image focus |
| Music | Album art visibility |
| Clock | Nighttime use |
| TV | Video content |

### Implementation

```swift
// Only if appropriate for your app type
@main
struct MediaApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .preferredColorScheme(.dark)
        }
    }
}
```

### When NOT to Force Dark

- Productivity apps
- Reading apps
- Most utility apps
- Social apps
- Anything without media focus

---

## Contrast and Accessibility

### Increase Contrast

```swift
@Environment(\.colorSchemeContrast) var contrast

var textColor: Color {
    contrast == .increased
        ? .primary          // Maximum contrast
        : .secondary        // Normal secondary
}
```

### Testing Contrast

1. Settings > Accessibility > Display & Text Size
2. Enable "Increase Contrast"
3. Test your app in both modes

### Minimum Contrast Ratios

| Element | Light Mode | Dark Mode |
|---------|------------|-----------|
| Body text | 4.5:1 | 4.5:1 |
| Large text | 3:1 | 3:1 |
| UI components | 3:1 | 3:1 |

---

## Reduce Transparency

Some users enable Reduce Transparency for better legibility.

```swift
@Environment(\.accessibilityReduceTransparency) var reduceTransparency

var background: some View {
    if reduceTransparency {
        Color(.systemBackground)
    } else {
        Color.clear.background(.thinMaterial)
    }
}
```

---

## Testing Dark Mode

### Xcode Preview

```swift
#Preview("Light") {
    ContentView()
        .preferredColorScheme(.light)
}

#Preview("Dark") {
    ContentView()
        .preferredColorScheme(.dark)
}
```

### Simulator

Environment Overrides panel: Appearance toggle

### Device

Settings > Developer > Dark Appearance

---

## Common Issues and Fixes

### Hardcoded Colors

```swift
// ❌ Won't adapt
.foregroundStyle(.white)
.background(Color(red: 0.9, green: 0.9, blue: 0.9))

// ✅ Semantic colors
.foregroundStyle(.primary)
.background(Color(.systemBackground))
```

### Images Without Variants

```swift
// ❌ Single variant looks wrong in dark mode
Image("logo") // Same image both modes

// ✅ Provide variants or use template
Image("logo")
    .renderingMode(.template)
    .foregroundStyle(.primary)
```

### Insufficient Contrast

```swift
// ❌ Gray text on dark background
Text("Label")
    .foregroundStyle(.gray) // May fail contrast

// ✅ System colors maintain contrast
Text("Label")
    .foregroundStyle(.secondary)
```

### Hardcoded Shadows

```swift
// ❌ Shadow invisible in dark mode
.shadow(color: .black.opacity(0.2), radius: 8)

// ✅ Elevation through background
.background(
    colorScheme == .dark
        ? Color(.secondarySystemBackground)
        : Color(.systemBackground).shadow(radius: 8)
)
```

---

## Checklist

### Before Release

- [ ] All custom colors have dark variants
- [ ] No hardcoded `.white`, `.black` for UI
- [ ] Images have variants or use template rendering
- [ ] Shadows adapted for dark mode
- [ ] Test with Increase Contrast enabled
- [ ] Test with Reduce Transparency enabled
- [ ] Contrast ratios meet WCAG guidelines
- [ ] Status bar style adapts correctly

---

## Anti-Patterns

```swift
// ❌ Forcing light mode
.preferredColorScheme(.light) // Ignores user preference

// ✅ Respect system setting (default behavior)
// Don't set preferredColorScheme unless media app

// ❌ Custom dark mode toggle
@AppStorage("darkMode") var darkMode = false
.preferredColorScheme(darkMode ? .dark : .light)

// ✅ Let system control appearance

// ❌ Same color both modes
Color("Brand") // No dark variant defined

// ✅ Define both variants in asset catalog

// ❌ Pure black backgrounds
Color.black

// ✅ System backgrounds
Color(.systemBackground)
```

---

## Quick Reference

| Need | Light Mode | Dark Mode |
|------|------------|-----------|
| Background | White | Near-black |
| Secondary bg | Light gray | Dark gray |
| Primary text | Black | White |
| Secondary text | Gray (60%) | Gray (60%) |
| Elevation | Shadow | Lighter surface |
| Accent color | Saturated | Desaturated |
| Icons | Template rendering | Template rendering |
| Photos | As-is | As-is |

# Layout & Spacing System Reference

Apple's layout system is built on systematic spacing, adaptable margins, and respect for safe areas. This reference covers the complete spatial design system.

---

## The 8-Point Grid Foundation

Apple's layouts are built on an 8-point base grid.

**Standard spacing values:**
```swift
// The spacing scale (multiples of 4, anchored on 8)
4pt   // Micro - icon padding, inline spacing
8pt   // Base - standard control padding
12pt  // Compact - tighter list items
16pt  // Standard - default margins
20pt  // Comfortable - section spacing
24pt  // Generous - card padding
32pt  // Major - section breaks
40pt  // Large - major divisions
48pt  // Extra large - hero spacing
64pt  // Maximum - dramatic separation
```

**Why 8-point base?**
- Divides evenly (8→4→2→1)
- Scales cleanly for 2x/3x displays
- Matches iOS baseline grid
- Creates visual harmony

---

## Safe Areas

### Understanding Safe Areas

Safe areas define the portion of the screen not obscured by system UI:
- Status bar
- Home indicator
- Navigation bar
- Tab bar
- Dynamic Island
- Notch (older devices)
- Keyboard

### Safe Area Implementation

```swift
// ✅ Content respects safe areas (default)
VStack {
    Text("Content in safe area")
}

// ✅ Background extends, content safe
ZStack {
    Color.blue
        .ignoresSafeArea() // Background extends
    VStack {
        Text("Content stays safe") // Text respects safe area
    }
}

// ❌ Wrong - content hidden behind system UI
VStack {
    Text("Hidden behind notch!")
}
.ignoresSafeArea()
```

### Safe Area Edge Cases

```swift
// Ignore only specific edges
.ignoresSafeArea(.all, edges: .bottom) // For tab bar overlay
.ignoresSafeArea(.keyboard) // Content shifts for keyboard
.ignoresSafeArea(.container, edges: .horizontal) // Scroll view

// Reading safe area insets
GeometryReader { proxy in
    let safeArea = proxy.safeAreaInsets
    // safeArea.top = Status bar + Dynamic Island
    // safeArea.bottom = Home indicator
}
```

---

## Standard Margins

### iOS Standard Margins

| Context | Margin | Usage |
|---------|--------|-------|
| **readableContentGuide** | Dynamic | Text content (optimal ~70 characters) |
| **layoutMarginsGuide** | 16-20pt | Default content margin |
| **Compact width** | 16pt | iPhone portrait |
| **Regular width** | 20pt | iPad, landscape |

### Implementation

```swift
// Readable content width (adapts to device)
Text(longArticle)
    .frame(maxWidth: .infinity)
    .scenePadding(.horizontal) // Uses system margins

// Or explicit readable guide
ScrollView {
    Text(article)
        .padding(.horizontal)
        .frame(maxWidth: 672) // Max readable width
        .frame(maxWidth: .infinity)
}

// Standard content padding
List { }
    .listStyle(.insetGrouped)
    // System automatically applies correct margins
```

### Content Width Guidelines

| Content Type | Ideal Width | Max Width |
|--------------|-------------|-----------|
| Body text | 45-75 characters | 672pt |
| UI elements | Flexible | Full width |
| Cards/cells | Flexible | Respect margins |
| Full-bleed media | Edge to edge | Screen width |

---

## Adaptive Layout Patterns

### Size Classes

```swift
@Environment(\.horizontalSizeClass) var hSizeClass
@Environment(\.verticalSizeClass) var vSizeClass

var body: some View {
    switch (hSizeClass, vSizeClass) {
    case (.compact, .regular):
        // iPhone portrait - single column
        PhoneLayout()
    case (.compact, .compact):
        // iPhone landscape - constrained
        CompactLandscapeLayout()
    case (.regular, .regular):
        // iPad - multi-column possible
        TabletLayout()
    case (.regular, .compact):
        // iPad in slide-over or iPhone Plus landscape
        RegularCompactLayout()
    default:
        PhoneLayout()
    }
}
```

### ViewThatFits (iOS 16+)

```swift
// System picks first layout that fits
ViewThatFits(in: .horizontal) {
    // Try wide layout first
    HStack(spacing: 16) {
        LargeCard()
        LargeCard()
        LargeCard()
    }

    // Fall back to medium
    HStack(spacing: 12) {
        MediumCard()
        MediumCard()
    }

    // Final fallback - single column
    VStack(spacing: 12) {
        SmallCard()
        SmallCard()
        SmallCard()
    }
}
```

### Adaptive Spacing

```swift
@Environment(\.horizontalSizeClass) var sizeClass

var spacing: CGFloat {
    sizeClass == .compact ? 12 : 20
}

var padding: CGFloat {
    sizeClass == .compact ? 16 : 24
}

VStack(spacing: spacing) {
    content
}
.padding(padding)
```

---

## Grid Systems

### LazyVGrid / LazyHGrid

```swift
// Adaptive columns - system determines count
let adaptiveColumns = [
    GridItem(.adaptive(minimum: 160, maximum: 200))
]

LazyVGrid(columns: adaptiveColumns, spacing: 16) {
    ForEach(items) { item in
        GridCell(item: item)
    }
}

// Fixed column count
let fixedColumns = [
    GridItem(.flexible()),
    GridItem(.flexible()),
    GridItem(.flexible())
]

// Mixed column sizing
let mixedColumns = [
    GridItem(.fixed(100)),      // Fixed width
    GridItem(.flexible()),       // Takes remaining space
    GridItem(.flexible(minimum: 80)) // Flexible with minimum
]
```

### Grid Spacing Guidelines

| Grid Type | Item Spacing | Section Spacing |
|-----------|--------------|-----------------|
| Photo grid | 2-4pt | 16pt |
| Card grid | 12-16pt | 24pt |
| Icon grid | 16-24pt | 32pt |
| App icon grid | 20pt | - |

---

## List Layout Patterns

### Standard List Styles

```swift
// Automatic style (adapts to context)
List { content }
    .listStyle(.automatic)

// Inset grouped (Settings style)
List { content }
    .listStyle(.insetGrouped)
// Rounded corners, inset from edges

// Grouped (older iOS Settings)
List { content }
    .listStyle(.grouped)

// Plain (no separators outside content)
List { content }
    .listStyle(.plain)

// Sidebar (macOS/iPadOS)
List { content }
    .listStyle(.sidebar)
```

### List Row Layout

```swift
// Standard row with system layout
List {
    ForEach(items) { item in
        HStack {
            // Leading - 44pt for accessories
            Image(systemName: item.icon)
                .frame(width: 28, height: 28)

            VStack(alignment: .leading, spacing: 2) {
                Text(item.title)
                    .font(.body)
                Text(item.subtitle)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()

            // Trailing - value/accessory
            Text(item.value)
                .foregroundStyle(.secondary)
        }
        .padding(.vertical, 8)
    }
}
```

### Row Heights

| Content | Height | Notes |
|---------|--------|-------|
| Single line | 44pt minimum | Standard |
| Two lines | ~56pt | Title + subtitle |
| Three lines | ~72pt | Rare, consider alternatives |
| With thumbnail | 60-80pt | Depends on thumbnail |

---

## Card & Container Patterns

### Card Spacing

```swift
// Standard card layout
VStack(alignment: .leading, spacing: 12) {
    // Image/header - full bleed within card
    Image(item.image)
        .aspectRatio(16/9, contentMode: .fill)
        .clipped()

    // Content with padding
    VStack(alignment: .leading, spacing: 8) {
        Text(item.title)
            .font(.headline)

        Text(item.description)
            .font(.subheadline)
            .foregroundStyle(.secondary)
    }
    .padding(.horizontal, 16)
    .padding(.bottom, 16)
}
.background(Color(.secondarySystemBackground))
.clipShape(RoundedRectangle(cornerRadius: 12, style: .continuous))
```

### Container Padding Standards

| Container | Internal Padding | Between Items |
|-----------|------------------|---------------|
| Card | 16pt | 12pt |
| Section | 20pt | 16pt |
| Group | 12pt | 8pt |
| Compact cell | 8-12pt | 4pt |

---

## Corner Radius Standards

**Apple's corner radius hierarchy:**

```swift
// Continuous corners (Apple's style)
.clipShape(RoundedRectangle(cornerRadius: radius, style: .continuous))

// Standard radius values
8pt  - Small elements (toggles, segments)
12pt - Cards, grouped sections
16pt - Larger cards, popovers
20pt - Sheets, large UI elements
24pt - Full-screen overlays (device specific)

// Device corner matching
UIScreen.main.displayCornerRadius // Match device corners
```

**Nesting corners**: Inner radius = outer radius - padding

```swift
// Container with 16pt corner, 12pt padding
// Inner element gets 4pt corner radius
ZStack {
    RoundedRectangle(cornerRadius: 16, style: .continuous)
        .fill(Color(.secondarySystemBackground))

    content
        .clipShape(RoundedRectangle(cornerRadius: 4, style: .continuous))
        .padding(12)
}
```

---

## Content Alignment

### Text Alignment

```swift
// Leading alignment (default for LTR)
Text("Title")
    .multilineTextAlignment(.leading)

// Center - titles, single-line content
Text("Centered Title")
    .multilineTextAlignment(.center)
    .frame(maxWidth: .infinity)

// Trailing - numbers, dates, values
Text("$99.99")
    .multilineTextAlignment(.trailing)
```

### Baseline Alignment

```swift
// Align text baselines in horizontal layouts
HStack(alignment: .firstTextBaseline) {
    Text("Label")
        .font(.body)
    Text("12")
        .font(.title)
}
```

### Visual (Optical) Alignment

Some elements need optical adjustment:

```swift
// Icons often need offset to appear centered
HStack(spacing: 12) {
    Image(systemName: "play.fill")
        .offset(x: 2) // Optical adjustment for play icon
    Text("Play")
}
```

---

## Responsive Breakpoints

### Common Device Widths

| Device | Width (Portrait) | Width (Landscape) |
|--------|------------------|-------------------|
| iPhone SE | 320pt | 568pt |
| iPhone 14 | 390pt | 844pt |
| iPhone 14 Pro Max | 430pt | 932pt |
| iPad Mini | 744pt | 1133pt |
| iPad | 820pt | 1180pt |
| iPad Pro 12.9" | 1024pt | 1366pt |

### Adaptive Design Thresholds

```swift
@Environment(\.horizontalSizeClass) var hSize

// Layout adaptation points
let useSidebar = UIDevice.current.userInterfaceIdiom == .pad
let useMultiColumn = hSize == .regular
let showCompactControls = hSize == .compact

// Content width adaptation
GeometryReader { proxy in
    let columns = max(1, Int(proxy.size.width / 200))
    // Dynamically adjust column count
}
```

---

## Dynamic Type Layout Adaptation

### Responding to Large Text

```swift
@Environment(\.dynamicTypeSize) var typeSize

var body: some View {
    if typeSize >= .accessibility1 {
        // Stack vertically for accessibility sizes
        VStack(alignment: .leading, spacing: 8) {
            icon
            labels
        }
    } else {
        // Horizontal for normal sizes
        HStack(spacing: 12) {
            icon
            labels
        }
    }
}
```

### Maintaining Touch Targets

```swift
// Touch target stays 44pt even when text grows
Button { action() } label: {
    Text("Action")
        .font(.body) // Scales with Dynamic Type
}
.frame(minHeight: 44) // Always tappable
```

---

## Quick Reference: Spacing Cheatsheet

| Use Case | Value | Notes |
|----------|-------|-------|
| Icon padding | 4pt | Inside SF Symbols |
| Inline text spacing | 4pt | Between icon and label |
| Control padding | 8pt | Button internal padding |
| Compact list row | 8-12pt | Vertical |
| Standard list row | 12-16pt | Vertical |
| Section spacing | 20-24pt | Between sections |
| Card padding | 16pt | All sides |
| Screen margins | 16-20pt | Size class dependent |
| Major breaks | 32-40pt | Hero sections |

---

## Anti-Patterns

```swift
// ❌ Arbitrary spacing values
.padding(13)
.spacing(7)

// ✅ Use system scale
.padding(12) // Or 16
.padding(.vertical, 8)

// ❌ Fixed sizes that don't adapt
.frame(width: 375) // iPhone width hardcoded

// ✅ Flexible, adaptive sizing
.frame(maxWidth: .infinity)
.frame(maxWidth: 672) // Readable max

// ❌ Ignoring safe areas for content
.ignoresSafeArea()

// ✅ Only backgrounds ignore safe areas
ZStack {
    Color.blue.ignoresSafeArea()
    content // Content respects safe areas
}
```

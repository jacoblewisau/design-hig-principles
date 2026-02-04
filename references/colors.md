# Color System Reference

Complete guide to Apple's semantic color system with implementation patterns.

---

## Semantic Color Philosophy

**Key insight from WWDC19:** "Think of Dark Mode as having the lights dimmed rather than everything being flipped inside out."

Colors are NOT simply inverted—they're redesigned for each context.

---

## Label Colors (Text Hierarchy)

```swift
// Primary content - most prominent
Text("Title")
    .foregroundStyle(.primary)
// Light Mode: Black | Dark Mode: White

// Secondary content - subtitles
Text("Subtitle")
    .foregroundStyle(.secondary)
// Light Mode: 60% black | Dark Mode: 60% white

// Tertiary content - placeholder
Text("Placeholder")
    .foregroundStyle(.tertiary)
// Light Mode: 30% black | Dark Mode: 30% white

// Quaternary content - disabled
Text("Disabled")
    .foregroundStyle(.quaternary)
// Light Mode: 18% black | Dark Mode: 18% white
```

**Hierarchy rule**: `primary` → `secondary` → `tertiary` → `quaternary`

---

## Background Colors

### Ungrouped Backgrounds (Standard Lists)

```swift
Color(.systemBackground)           // Pure white/black
Color(.secondarySystemBackground)  // Light/dark gray
Color(.tertiarySystemBackground)   // Lighter/darker gray
```

### Grouped Backgrounds (Settings-Style Lists)

```swift
Color(.systemGroupedBackground)           // Table background
Color(.secondarySystemGroupedBackground)  // Cell background
Color(.tertiarySystemGroupedBackground)   // Nested content
```

**Usage pattern**:
```swift
// Standard list
List { ... }
    .listStyle(.plain)
    .background(Color(.systemBackground))

// Grouped list (Settings style)
List { ... }
    .listStyle(.insetGrouped)
    // Grouped backgrounds applied automatically
```

---

## Base vs Elevated Colors

Two background sets exist for layering:

| Context | Color Set | When Used |
|---------|-----------|-----------|
| Base | Darker | Background apps, base layer |
| Elevated | Lighter | Foreground apps, sheets, popovers |

**Why this matters in Dark Mode:**
- Drop shadows don't work well
- Elevated surfaces use lighter colors instead
- iPad multitasking uses both sets

**Example**: iPad Slide Over
- Main app: base colors
- Slide-over app: elevated colors (stands out)

---

## Fill Colors (Semi-Transparent)

For controls and interactive elements:

```swift
Color(.systemFill)           // Most visible
Color(.secondarySystemFill)
Color(.tertiarySystemFill)
Color(.quaternarySystemFill) // Most subtle
```

**Why semi-transparent?** Adapts to any background context while maintaining contrast.

---

## Tint Colors

System tints adapt automatically:

```swift
Button("Action") { }
    .tint(.blue)
// Lighter blue in Dark Mode
// Darker blue in Light Mode
```

**Custom tint colors** require both variants:
1. Define in Assets.xcassets
2. Add Light and Dark appearances
3. Verify **4.5:1 contrast** in both modes

---

## Separator Colors

```swift
Color(.separator)       // Semi-transparent (default)
Color(.opaqueSeparator) // Solid color
```

**Use opaque** when semi-transparent creates optical issues (intersecting grid lines).

---

## When to Use Permanent Dark

**Apple's guidance:** Only for immersive media viewing apps.

| App | Permanent Dark | Rationale |
|-----|----------------|-----------|
| Music | Yes | Album art focus |
| Photos | Yes | Images are hero |
| Camera | Yes | Reduce distraction |
| Clock | Yes | Nighttime use |
| Most apps | No | Respect user preference |

---

## Creating Custom Colors

### Asset Catalog Setup

1. Open `Assets.xcassets`
2. Right-click → New Color Set
3. Name it (e.g., "BrandAccent")
4. In Attributes Inspector:
   - Appearances: Any, Dark
   - (Optional) High Contrast: Add variants

### Using Custom Colors

```swift
// Automatically selects correct variant
Text("Brand")
    .foregroundStyle(Color("BrandAccent"))

// Or define as extension
extension Color {
    static let brandAccent = Color("BrandAccent")
}

Text("Brand")
    .foregroundStyle(.brandAccent)
```

---

## Color Contrast Requirements

| Text Size | Minimum Ratio | WCAG Level |
|-----------|---------------|------------|
| Small (<14pt) | 7:1 | AAA |
| Normal (14pt+) | 4.5:1 | AA |
| Large (18pt+) | 3:1 | AA |

**Testing tools**:
- Xcode Accessibility Inspector
- Online contrast calculators
- Enable "Increase Contrast" in Simulator

---

## Quick Reference: Color Selection

| Use Case | Color to Use |
|----------|--------------|
| Primary text | `.primary` |
| Secondary text | `.secondary` |
| Placeholder text | `.tertiary` |
| Disabled text | `.quaternary` |
| Main background | `Color(.systemBackground)` |
| Card/section background | `Color(.secondarySystemBackground)` |
| Grouped list background | `Color(.systemGroupedBackground)` |
| Cell in grouped list | `Color(.secondarySystemGroupedBackground)` |
| Button/control fill | `Color(.systemFill)` |
| Tappable accent | `.tint` or custom asset |
| Success indicator | `.green` + icon + text |
| Error indicator | `.red` + icon + text |
| Warning indicator | `.yellow` + icon + text |

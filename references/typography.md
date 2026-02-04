# Typography Reference

Complete guide to Apple's typography system with Dynamic Type implementation.

---

## System Fonts

### San Francisco (SF) Family

- **SF Pro**: General UI use
- **SF Pro Rounded**: Softer, friendlier feel
- **SF Compact**: watchOS and constrained layouts
- **SF Mono**: Code and monospaced text

**Weights**: Ultralight, Thin, Light, Regular, Medium, Semibold, Bold, Heavy, Black

### New York (NY) Family

Serif font for editorial content. Same weights as SF.

---

## Text Styles Hierarchy

Built-in styles with automatic Dynamic Type:

```swift
.font(.largeTitle)   // 34pt - Hero content
.font(.title)        // 28pt - Page titles
.font(.title2)       // 22pt - Section headers
.font(.title3)       // 20pt - Sub-sections
.font(.headline)     // 17pt semibold - Emphasized body
.font(.body)         // 17pt - Default body text
.font(.callout)      // 16pt - Slightly emphasized
.font(.subheadline)  // 15pt - Less prominent
.font(.footnote)     // 13pt - Auxiliary info
.font(.caption)      // 12pt - Minimal emphasis
.font(.caption2)     // 11pt - Smallest
```

---

## Dynamic Type Requirements

**Mandatory**: Support at least **200%** scaling (iOS/iPadOS), **140%** (watchOS).

### Correct Implementation

```swift
// ✅ Uses text style - scales automatically
Text("Title")
    .font(.title)

// ✅ Custom font with scaling
Text("Title")
    .font(.custom("Avenir-Medium", size: 24, relativeTo: .title))
```

### Wrong Implementation

```swift
// ❌ Fixed size - won't scale
Text("Title")
    .font(.system(size: 24))

// ❌ Custom font without relativeTo
Text("Title")
    .font(.custom("Avenir-Medium", fixedSize: 24))
```

---

## Font Weight Guidelines

**Apple HIG**: "Avoid light font weights. Prefer Regular, Medium, Semibold, or Bold."

| Weight | Use Case | Recommendation |
|--------|----------|----------------|
| Ultralight | - | Avoid |
| Thin | - | Avoid |
| Light | - | Avoid |
| Regular | Body text | Preferred |
| Medium | Emphasis | Preferred |
| Semibold | Headers | Preferred |
| Bold | Strong emphasis | Preferred |
| Heavy | Display only | Context-dependent |
| Black | Display only | Context-dependent |

**Why avoid light weights?**
- Poor legibility at small sizes
- Difficult to read in bright conditions
- Problematic for users with visual impairments

---

## Weight Implementation

```swift
// Headers - Bold for prominence
Text("Page Title")
    .font(.title.weight(.bold))

// Section headers - Semibold
Text("Section")
    .font(.title2.weight(.semibold))

// Body text - Regular (default)
Text("Body content")
    .font(.body)

// Captions - Regular, never Light
Text("Caption")
    .font(.caption) // Regular weight by default
```

---

## Layout Considerations at Large Sizes

### Adapt Multi-Column to Single Column

```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    if dynamicTypeSize >= .accessibility1 {
        // Stack vertically at large sizes
        VStack { content }
    } else {
        // Side-by-side at normal sizes
        HStack { content }
    }
}
```

### Use ViewThatFits

```swift
ViewThatFits {
    HStack { Column1(); Column2() } // Preferred
    VStack { Column1(); Column2() } // Fallback at large sizes
}
```

### Avoid Truncation

```swift
// ❌ Essential content may truncate
Text(title)
    .lineLimit(1)

// ✅ Allow wrapping for essential content
Text(title)
    .lineLimit(3)
    .minimumScaleFactor(0.8)
```

---

## Custom Fonts

### Setup Requirements

1. Add font files to project
2. Add to Info.plist under "Fonts provided by application"
3. Implement Dynamic Type support

### Implementation

```swift
// ✅ Correct - Scales with Dynamic Type
Text("Custom")
    .font(.custom("BrandFont-Regular", size: 17, relativeTo: .body))

// ❌ Wrong - Fixed size
Text("Custom")
    .font(.custom("BrandFont-Regular", fixedSize: 17))
```

### Responding to Bold Text Setting

```swift
@Environment(\.legibilityWeight) var legibilityWeight

var fontWeight: Font.Weight {
    legibilityWeight == .bold ? .semibold : .regular
}

Text("Adapts to Bold Text setting")
    .fontWeight(fontWeight)
```

---

## Line Spacing (Leading)

```swift
// Loose leading - Wide columns
Text(longContent)
    .lineSpacing(8)

// Default leading - Most contexts
Text(content)
    // System default

// Tight leading - Constrained height (max 2 lines)
Text(shortContent)
    .lineSpacing(2)
    .lineLimit(2)
```

**Rule**: Avoid tight leading for 3+ lines of text.

---

## Text Hierarchy Example

```swift
VStack(alignment: .leading, spacing: 8) {
    // Primary - Bold, largest
    Text("Article Title")
        .font(.title.weight(.bold))
        .foregroundStyle(.primary)

    // Secondary - Semibold, smaller
    Text("Subtitle or Tagline")
        .font(.title3.weight(.semibold))
        .foregroundStyle(.secondary)

    // Body - Regular weight
    Text("Body content with the main information...")
        .font(.body)
        .foregroundStyle(.primary)

    // Tertiary - Smaller, lighter
    Text("Published 2 hours ago")
        .font(.caption)
        .foregroundStyle(.tertiary)
}
```

---

## Quick Reference

| Need | Solution |
|------|----------|
| Page title | `.font(.largeTitle.weight(.bold))` |
| Section header | `.font(.title2.weight(.semibold))` |
| List row title | `.font(.headline)` |
| List row subtitle | `.font(.subheadline).foregroundStyle(.secondary)` |
| Body text | `.font(.body)` |
| Caption | `.font(.caption).foregroundStyle(.secondary)` |
| Timestamp | `.font(.caption2).foregroundStyle(.tertiary)` |

---

## Testing Checklist

- [ ] Test at default text size
- [ ] Test at maximum accessibility size (AX5)
- [ ] Test with Bold Text enabled
- [ ] Verify no truncation of essential content
- [ ] Verify layout adapts at large sizes
- [ ] Custom fonts scale with Dynamic Type

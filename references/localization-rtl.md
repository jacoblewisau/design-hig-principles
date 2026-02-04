# Localization & RTL Reference

Complete guide to internationalization, right-to-left support, and localization-friendly design.

---

## Localization Philosophy

**Apple's guidance:** Design for a global audience from the start. Retrofitting localization is expensive and error-prone.

**Key principles:**
- Text expands in translation (up to 30%+)
- Some languages read right-to-left
- Date, time, number formats vary
- Cultural context matters

---

## Text Expansion

### Expansion Guidelines

| Source Length | Expected Expansion |
|---------------|-------------------|
| 1-10 characters | 200-300% |
| 11-20 characters | 80-100% |
| 21-30 characters | 60-80% |
| 31-50 characters | 40-60% |
| 51-70 characters | 31-40% |
| 70+ characters | 30% |

### Design for Expansion

```swift
// ❌ Fixed width truncates translations
Text("Save")
    .frame(width: 60)

// ✅ Flexible width accommodates "Enregistrer"
Text("Save")
    .fixedSize(horizontal: true, vertical: false)

// ✅ Or set minimum
Text("Save")
    .frame(minWidth: 60)
```

### Testing Expansion

```swift
// Preview with pseudolanguage
#Preview("Expanded") {
    ContentView()
        .environment(\.locale, Locale(identifier: "en"))
    // Use Xcode's pseudolanguage for testing
}
```

---

## Right-to-Left (RTL) Languages

### RTL Languages

- Arabic (ar)
- Hebrew (he)
- Persian/Farsi (fa)
- Urdu (ur)

### Automatic RTL Support

SwiftUI automatically mirrors layouts for RTL:

```swift
// Automatically mirrors in RTL
HStack {
    Image(systemName: "star")
    Text("Favorite")
}
// Leading becomes trailing, etc.
```

### Checking Layout Direction

```swift
@Environment(\.layoutDirection) var layoutDirection

var body: some View {
    HStack {
        if layoutDirection == .rightToLeft {
            trailingContent
            leadingContent
        } else {
            leadingContent
            trailingContent
        }
    }
}
```

### Semantic Leading/Trailing

```swift
// ✅ Use leading/trailing (respects RTL)
.padding(.leading, 16)
.padding(.trailing, 16)
.frame(alignment: .leading)

// ❌ Avoid left/right (doesn't flip)
.padding(.left, 16)  // Won't flip in RTL
```

### Preventing Mirror for Specific Content

```swift
// Disable mirroring for content that shouldn't flip
Image("logo")
    .environment(\.layoutDirection, .leftToRight)

// Or use flipsForRightToLeftLayoutDirection
Image("arrow")
    .flipsForRightToLeftLayoutDirection(false)
```

### When NOT to Mirror

| Content | Mirror? |
|---------|---------|
| Text | Yes (automatic) |
| Back chevron | Yes |
| Play/forward controls | No (universal meaning) |
| Graphs (X axis = time) | No |
| Phone numbers | No |
| Logos/branding | No |
| Clocks | No |

---

## String Localization

### String Catalogs (Xcode 15+)

Preferred approach:

```swift
// Automatic - Text views are localized
Text("Hello, World!")
// Xcode extracts to String Catalog

// With comments for translators
Text("Welcome", comment: "Greeting shown on home screen")

// Interpolation
Text("Hello, \(name)")
```

### LocalizedStringKey

```swift
// For custom components
struct MyLabel: View {
    let title: LocalizedStringKey

    var body: some View {
        Text(title)
    }
}

MyLabel(title: "Settings")
```

### Formatted Strings

```swift
// ❌ String concatenation
Text("You have " + String(count) + " items")

// ✅ Localized interpolation
Text("You have \(count) items")
// Catalog: "You have %lld items"
// Allows: "Items: %lld" in different languages

// Plural rules
Text("item_count \(count)")
// Catalog handles: "1 item", "2 items", etc.
```

### Attributed Strings

```swift
// Localize attributed text
Text("Terms of **Service**")
// Markdown preserved in localization
```

---

## Date & Time Formatting

### Automatic Formatting

```swift
// ✅ Locale-aware formatting
Text(date, format: .dateTime.month().day())
// US: "Jan 15"
// Germany: "15. Jan"
// Japan: "1月15日"

Text(date, format: .relative(presentation: .named))
// "Yesterday", "Morgen", "昨日"
```

### Date Styles

```swift
Text(date, style: .date)    // Full date
Text(date, style: .time)    // Time only
Text(date, style: .relative) // "2 hours ago"
Text(date, style: .timer)   // Countdown
Text(date, style: .offset)  // "+2 hours"
```

### Custom Formats

```swift
// Use format strings carefully
let formatter = DateFormatter()
formatter.dateFormat = "MMMM d, yyyy"
// ⚠️ Not localized!

// ✅ Use styles instead
formatter.dateStyle = .long
formatter.timeStyle = .short
```

---

## Number Formatting

```swift
// ✅ Locale-aware numbers
Text(1234.56, format: .number)
// US: "1,234.56"
// Germany: "1.234,56"
// France: "1 234,56"

// Currency
Text(price, format: .currency(code: "USD"))

// Percentage
Text(ratio, format: .percent)

// Measurement
Text(Measurement(value: 5, unit: UnitLength.miles), format: .measurement(width: .abbreviated))
// US: "5 mi"
// Metric countries: "8 km"
```

---

## Images and Icons

### Localized Images

```swift
// In Asset Catalog:
// - Add localization to image set
// - Provide variants for each locale

Image("OnboardingHero")
// Automatically loads localized variant
```

### SF Symbols RTL

```swift
// SF Symbols auto-mirror where appropriate
Image(systemName: "text.alignleft")
// Flips to right-align icon in RTL

// Control mirroring
Image(systemName: "arrow.right")
    .imageScale(.large)
    .environment(\.layoutDirection, .leftToRight) // Don't mirror
```

---

## App Name Localization

### InfoPlist.strings

```
// InfoPlist.strings (Arabic)
"CFBundleDisplayName" = "تطبيقي";
```

### Different Names per Region

Common for apps with regional branding differences.

---

## Testing Localization

### Xcode Scheme Options

1. Edit Scheme > Run > Options
2. App Language: Choose language
3. App Region: Choose region
4. Enable pseudolanguages for testing

### Pseudolanguages

| Pseudolanguage | Tests |
|----------------|-------|
| Double-Length | Text expansion |
| Right-to-Left | RTL layout |
| Accented | Character rendering |
| Bounded Strings | String boundaries |

### Preview Testing

```swift
#Preview("Arabic") {
    ContentView()
        .environment(\.locale, Locale(identifier: "ar"))
        .environment(\.layoutDirection, .rightToLeft)
}

#Preview("German") {
    ContentView()
        .environment(\.locale, Locale(identifier: "de"))
}
```

---

## Common Issues

### Truncated Translations

```swift
// ❌ Fixed width
.frame(width: 100)

// ✅ Flexible with minimum
.frame(minWidth: 100)
.fixedSize(horizontal: true, vertical: false)
```

### Broken Layouts in RTL

```swift
// ❌ Using left/right
.padding(.left, 20)

// ✅ Using leading/trailing
.padding(.leading, 20)
```

### Concatenated Strings

```swift
// ❌ Word order varies
"Welcome" + " " + name + "!"

// ✅ Single localized string
String(localized: "Welcome, \(name)!")
```

### Hardcoded Formats

```swift
// ❌ US-specific format
"\(month)/\(day)/\(year)"

// ✅ Locale-aware
date.formatted(date: .numeric, time: .omitted)
```

---

## String Catalog Structure

```
Localizable.xcstrings
├── en (Source)
│   └── "Hello" = "Hello"
├── de
│   └── "Hello" = "Hallo"
├── ar
│   └── "Hello" = "مرحبا"
└── ja
    └── "Hello" = "こんにちは"
```

### Pluralization

```json
// In String Catalog
"item_count" = {
    "localizations": {
        "en": {
            "variations": {
                "plural": {
                    "one": "%lld item",
                    "other": "%lld items"
                }
            }
        }
    }
}
```

---

## Accessibility & Localization

### VoiceOver Localization

```swift
// Accessibility labels are localized
Image(systemName: "star")
    .accessibilityLabel("Favorite")
// Localizable.strings: "Favorite" = "Favorit"
```

### RTL VoiceOver

VoiceOver navigation order automatically adapts to RTL.

---

## Best Practices

### Design Phase

1. Allow for 30%+ text expansion
2. Use flexible layouts
3. Use leading/trailing, not left/right
4. Avoid text in images
5. Consider cultural appropriateness

### Development Phase

1. Use LocalizedStringKey for all user-facing text
2. Add translator comments
3. Use formatters for dates/numbers
4. Test with pseudolanguages
5. Test RTL layout thoroughly

### Testing Phase

1. Test all supported languages
2. Verify RTL layout
3. Check for truncation
4. Verify date/number formats
5. Test with different region settings

---

## Anti-Patterns

```swift
// ❌ Hardcoded strings
Text("Save")  // Not extracted

// ✅ Will be extracted
Text("Save")  // Xcode extracts this

// ❌ String concatenation for sentences
Text(greeting + " " + name)

// ✅ Single localizable string
Text("greeting_with_name \(name)")

// ❌ Left/right instead of leading/trailing
.frame(alignment: .left)

// ✅ Semantic alignment
.frame(alignment: .leading)

// ❌ Fixed sizes for text
.frame(width: 100)

// ✅ Flexible sizing
.frame(minWidth: 100)
.fixedSize(horizontal: true, vertical: false)

// ❌ Custom date formatting
"\(month)/\(day)"

// ✅ System formatter
date.formatted(.dateTime.month().day())
```

---

## Quick Reference

| Need | Solution |
|------|----------|
| Text expansion | Flexible layouts, minWidth |
| RTL support | leading/trailing, not left/right |
| Date formatting | `.formatted()` or `Text(date, style:)` |
| Number formatting | `.formatted(.number)` |
| Currency | `.formatted(.currency(code:))` |
| Plurals | String Catalog variations |
| Layout direction | `@Environment(\.layoutDirection)` |
| Test RTL | Xcode scheme pseudolanguage |
| Test expansion | Double-Length pseudolanguage |

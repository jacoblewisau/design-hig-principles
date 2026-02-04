# SF Symbols Reference

Comprehensive guide to Apple's SF Symbols system—the definitive icon language for Apple platforms.

---

## SF Symbols Overview

SF Symbols is Apple's integrated iconography system providing 5,000+ symbols designed for San Francisco, the system font. All symbols:

- Scale with Dynamic Type automatically
- Support all font weights
- Align with text baselines
- Support multiple rendering modes
- Adapt to localization (RTL, etc.)
- Include accessibility support

---

## Symbol Categories

### Navigation & Actions
```
arrow.left, arrow.right, arrow.up, arrow.down
chevron.left, chevron.right, chevron.up, chevron.down
house, house.fill
magnifyingglass
plus, minus, xmark
checkmark
gear, gearshape.fill
ellipsis, ellipsis.circle
```

### Communication
```
envelope, envelope.fill
phone, phone.fill
message, message.fill
bubble.left, bubble.right
bell, bell.fill
```

### Media
```
play, pause, stop
forward, backward
speaker, speaker.wave.3
mic, mic.fill
camera, camera.fill
photo, photo.fill
```

### Objects & Tools
```
doc, doc.fill
folder, folder.fill
paperclip
link
pencil
trash, trash.fill
bookmark, bookmark.fill
```

### Status & Indicators
```
checkmark.circle, checkmark.circle.fill
xmark.circle, xmark.circle.fill
exclamationmark.triangle, exclamationmark.triangle.fill
info.circle, info.circle.fill
questionmark.circle
```

### People & Sharing
```
person, person.fill
person.2, person.3
person.crop.circle
square.and.arrow.up (Share)
square.and.arrow.down (Download)
```

---

## Symbol Weights

Symbols come in 9 weights matching SF Pro:

```swift
// Available weights (lightest to heaviest)
.ultraLight
.thin
.light
.regular    // Default
.medium
.semibold
.bold
.heavy
.black
```

### Implementation

```swift
// Match symbol weight to adjacent text
Image(systemName: "star.fill")
    .font(.body)               // Inherits text weight
    .fontWeight(.semibold)     // Or set explicitly

// Using symbolVariant for weight
Image(systemName: "star")
    .symbolVariant(.fill)
    .fontWeight(.bold)
```

### Weight Matching Guidelines

| Text Weight | Symbol Weight | Use Case |
|-------------|---------------|----------|
| Regular | Regular | Body text icons |
| Medium | Medium | Emphasized labels |
| Semibold | Semibold | Toolbar items |
| Bold | Bold | Navigation titles |

---

## Symbol Scales

Three scale variants optimize symbols for different contexts:

```swift
// Small - Compact UIs, accessories
Image(systemName: "star.fill")
    .imageScale(.small)

// Medium - Default, most uses
Image(systemName: "star.fill")
    .imageScale(.medium)

// Large - Prominent placement
Image(systemName: "star.fill")
    .imageScale(.large)
```

### Scale Guidelines

| Scale | Use Case | Example |
|-------|----------|---------|
| Small | List accessories, badges | Disclosure indicator |
| Medium | Toolbar, tab bars | Navigation items |
| Large | Hero elements, buttons | Primary actions |

---

## Rendering Modes

SF Symbols support four rendering modes:

### 1. Monochrome (Default)
Single color, respects foreground style.

```swift
Image(systemName: "cloud.sun.fill")
    .foregroundStyle(.blue)
    .symbolRenderingMode(.monochrome)
```

### 2. Hierarchical
Single color with depth through opacity layers.

```swift
Image(systemName: "cloud.sun.fill")
    .foregroundStyle(.blue)
    .symbolRenderingMode(.hierarchical)
// Sun is 100% blue, cloud is ~50% blue
```

### 3. Palette
Multiple explicit colors for distinct layers.

```swift
Image(systemName: "cloud.sun.fill")
    .symbolRenderingMode(.palette)
    .foregroundStyle(.cyan, .yellow)
// Cloud is cyan, sun is yellow
```

### 4. Multicolor
System-defined colors for realistic representation.

```swift
Image(systemName: "cloud.sun.fill")
    .symbolRenderingMode(.multicolor)
// Uses Apple's defined colors
```

### Choosing Rendering Mode

| Context | Mode | Rationale |
|---------|------|-----------|
| Toolbar icons | Monochrome | Consistency, follows tint |
| Status indicators | Hierarchical | Depth without multiple colors |
| App icons | Multicolor | Recognizable, distinctive |
| Custom themes | Palette | Brand colors |

---

## Symbol Variants

Symbols have structural variants:

```swift
// Base symbol
Image(systemName: "heart")

// Filled variant
Image(systemName: "heart.fill")
// Or programmatically:
Image(systemName: "heart")
    .symbolVariant(.fill)

// Circle variant
Image(systemName: "heart.circle")
Image(systemName: "heart")
    .symbolVariant(.circle)

// Circle + Fill
Image(systemName: "heart.circle.fill")
Image(systemName: "heart")
    .symbolVariant(.circle.fill)

// Square variant
Image(systemName: "heart.square")

// Slash variant (disabled/off state)
Image(systemName: "bell.slash")
Image(systemName: "bell")
    .symbolVariant(.slash)
```

### Variant Selection Guidelines

| State | Variant | Example |
|-------|---------|---------|
| Default/Inactive | Outline | `heart` |
| Selected/Active | Fill | `heart.fill` |
| Disabled/Off | Slash | `heart.slash` |
| Interactive button | Circle | `heart.circle` |
| Prominent button | Circle + Fill | `heart.circle.fill` |

---

## Symbol Effects (iOS 17+)

Animated symbol effects for feedback:

### Bounce Effect
```swift
Image(systemName: "bell")
    .symbolEffect(.bounce, value: notificationCount)
// Bounces when value changes
```

### Pulse Effect
```swift
Image(systemName: "heart.fill")
    .symbolEffect(.pulse)
// Continuous pulsing
```

### Variable Color
```swift
Image(systemName: "speaker.wave.3.fill")
    .symbolEffect(.variableColor.iterative.reversing)
// Animates through wave layers
```

### Scale Effect
```swift
Image(systemName: "star.fill")
    .symbolEffect(.scale.up, isActive: isHighlighted)
// Scales up when active
```

### Replace Effect
```swift
// Animated symbol swap
Image(systemName: isPlaying ? "pause.fill" : "play.fill")
    .contentTransition(.symbolEffect(.replace))
```

---

## Dynamic Type Integration

Symbols scale automatically with text styles:

```swift
// Symbol inherits Dynamic Type scaling
Label("Favorites", systemImage: "star.fill")
    .font(.body)
// Both text and symbol scale together

// Or explicit text style on image
Image(systemName: "star.fill")
    .font(.title)
// Scales with title text style
```

### Maintaining Alignment

```swift
// Baseline alignment with text
HStack(alignment: .firstTextBaseline) {
    Image(systemName: "star.fill")
    Text("Favorite")
}
.font(.body)
```

---

## Custom Symbol Points

Control symbol sizing precisely:

```swift
// Point size (like font size)
Image(systemName: "star.fill")
    .font(.system(size: 24))

// With specific weight
Image(systemName: "star.fill")
    .font(.system(size: 24, weight: .semibold))

// Relative to text style (scales with Dynamic Type)
Image(systemName: "star.fill")
    .font(.system(size: 24, weight: .semibold, design: .rounded))
```

---

## Button Integration

### Standard Button Styles

```swift
// Icon-only button (requires label)
Button { action() } label: {
    Image(systemName: "square.and.arrow.up")
}
.accessibilityLabel("Share")

// Icon + Text
Button { action() } label: {
    Label("Share", systemImage: "square.and.arrow.up")
}

// Bordered button with icon
Button { action() } label: {
    Label("Add", systemImage: "plus")
}
.buttonStyle(.bordered)
```

### Toolbar Items

```swift
.toolbar {
    ToolbarItem(placement: .primaryAction) {
        Button { } label: {
            Image(systemName: "plus")
        }
    }

    ToolbarItem(placement: .secondaryAction) {
        Button { } label: {
            Image(systemName: "ellipsis.circle")
        }
    }
}
```

---

## Tab Bar Icons

```swift
TabView {
    HomeView()
        .tabItem {
            Label("Home", systemImage: "house")
            // System automatically uses .fill when selected
        }

    SearchView()
        .tabItem {
            Label("Search", systemImage: "magnifyingglass")
        }

    ProfileView()
        .tabItem {
            Label("Profile", systemImage: "person")
        }
}
```

### Tab Icon Guidelines

- Use outline for unselected, fill for selected (automatic)
- Maximum 5 tabs on iPhone
- Don't use multicolor in tab bar
- Ensure distinct silhouettes

---

## Accessibility Considerations

### Labels for Icon Buttons

```swift
// ❌ No label - VoiceOver says "Button"
Button { } label: {
    Image(systemName: "gear")
}

// ✅ Accessible
Button { } label: {
    Image(systemName: "gear")
}
.accessibilityLabel("Settings")

// ✅ Also accessible - Label provides text
Button { } label: {
    Label("Settings", systemImage: "gear")
        .labelStyle(.iconOnly)
}
```

### Decorative Symbols

```swift
// Hide from VoiceOver if purely decorative
Image(systemName: "sparkles")
    .accessibilityHidden(true)
```

### Symbol Descriptions

Some symbols have built-in accessibility descriptions. Override when needed:

```swift
Image(systemName: "star.fill")
    .accessibilityLabel("Favorite") // Override "filled star"
```

---

## Custom SF Symbols

Create custom symbols matching SF Symbols style:

### Requirements
1. Export template from SF Symbols app
2. Design in vector editor (Illustrator, Sketch, Figma)
3. Follow Apple's design guidelines:
   - Use consistent stroke weights
   - Align to template grid
   - Create all 9 weight variants
   - Include 3 scale variants

### Integration

```swift
// Custom symbol in asset catalog
Image("custom.symbol.name")
    .font(.body)
    .foregroundStyle(.blue)
```

### Validation
- Use SF Symbols app to validate
- Check alignment at all weights
- Test with Dynamic Type

---

## Common Symbol Patterns

### Navigation
```swift
// Back
Image(systemName: "chevron.left")
Image(systemName: "arrow.left")

// Close
Image(systemName: "xmark")
Image(systemName: "xmark.circle.fill")

// More options
Image(systemName: "ellipsis")
Image(systemName: "ellipsis.circle")
```

### Editing
```swift
// Edit
Image(systemName: "pencil")
Image(systemName: "square.and.pencil")

// Delete
Image(systemName: "trash")
Image(systemName: "trash.fill")

// Add
Image(systemName: "plus")
Image(systemName: "plus.circle.fill")
```

### Status
```swift
// Success
Image(systemName: "checkmark.circle.fill")
    .foregroundStyle(.green)

// Warning
Image(systemName: "exclamationmark.triangle.fill")
    .foregroundStyle(.yellow)

// Error
Image(systemName: "xmark.circle.fill")
    .foregroundStyle(.red)

// Info
Image(systemName: "info.circle.fill")
    .foregroundStyle(.blue)
```

---

## Anti-Patterns

```swift
// ❌ Hardcoded size without Dynamic Type
Image(systemName: "star")
    .frame(width: 20, height: 20)

// ✅ Use font sizing
Image(systemName: "star")
    .font(.title2)

// ❌ Mismatched weights
HStack {
    Image(systemName: "star.fill")
        .fontWeight(.bold)
    Text("Favorites")
        .fontWeight(.regular)  // Weight mismatch
}

// ✅ Consistent weights
HStack {
    Image(systemName: "star.fill")
    Text("Favorites")
}
.fontWeight(.semibold)

// ❌ Icon button without accessibility
Button { } label: {
    Image(systemName: "gear")
}

// ✅ Labeled button
Button { } label: {
    Image(systemName: "gear")
}
.accessibilityLabel("Settings")
```

---

## Quick Reference

| Need | Symbol | Notes |
|------|--------|-------|
| Back | `chevron.left` | Or `arrow.left` |
| Close | `xmark` | Or `xmark.circle.fill` |
| Share | `square.and.arrow.up` | Standard share |
| Edit | `pencil` | Or `square.and.pencil` |
| Delete | `trash` | Use `.fill` when selected |
| Add | `plus` | Or `plus.circle.fill` |
| Settings | `gear` | Or `gearshape` |
| Search | `magnifyingglass` | |
| Filter | `line.3.horizontal.decrease.circle` | |
| Sort | `arrow.up.arrow.down` | |
| Favorite | `star` / `star.fill` | |
| Like | `heart` / `heart.fill` | |
| Bookmark | `bookmark` / `bookmark.fill` | |
| Download | `arrow.down.circle` | |
| Upload | `arrow.up.circle` | |
| Refresh | `arrow.clockwise` | |
| Home | `house` / `house.fill` | |
| Profile | `person` / `person.fill` | |
| Notifications | `bell` / `bell.fill` | |
| Menu | `line.3.horizontal` | Hamburger menu |
| More | `ellipsis` | Or `ellipsis.circle` |

# Platform-Specific Guidelines

Each Apple platform has unique characteristics and conventions. This reference covers platform-appropriate design patterns.

---

## iOS

### Device Characteristics
- Medium-size, high-resolution display
- One or two-handed interaction
- Portrait/landscape orientation
- Viewing distance: 1-2 feet
- Multi-Touch gestures

### Navigation Patterns

**Primary navigation**: Tab bar for top-level sections

```swift
// ✅ Correct iOS navigation
TabView {
    HomeView()
        .tabItem { Label("Home", systemImage: "house") }
    SearchView()
        .tabItem { Label("Search", systemImage: "magnifyingglass") }
    ProfileView()
        .tabItem { Label("Profile", systemImage: "person") }
}
```

**Drilling down**: NavigationStack

```swift
NavigationStack {
    List(items) { item in
        NavigationLink(item.title, value: item)
    }
    .navigationDestination(for: Item.self) { item in
        ItemDetailView(item: item)
    }
}
```

### Key Conventions
- Edge swipe for back navigation (don't override)
- Pull to refresh on scrollable content
- Swipe actions on list rows
- Context menus on long press
- Share sheet via `ShareLink`

### Anti-Patterns to Avoid
- Hamburger menus (use tab bar)
- Custom back buttons that break swipe
- Bottom sheets for primary navigation
- Gestures without button alternatives

---

## iPadOS

### Device Characteristics
- Large display (more content simultaneously)
- Landscape-first for many use cases
- Pointer/trackpad support
- Keyboard attachment common
- Multitasking (Split View, Slide Over)

### Navigation Patterns

**Primary navigation**: Sidebar for top-level sections

```swift
NavigationSplitView {
    List(sections, selection: $selectedSection) { section in
        Label(section.title, systemImage: section.icon)
    }
} content: {
    if let section = selectedSection {
        SectionContentView(section: section)
    }
} detail: {
    DetailView()
}
```

### Key Conventions
- Sidebar collapses in compact width
- Support pointer hover states
- Keyboard shortcuts for common actions
- Context menus for additional options
- Support Split View multitasking

### Adaptations from iOS
```swift
// Adapt layout based on horizontal size class
@Environment(\.horizontalSizeClass) var sizeClass

var body: some View {
    if sizeClass == .compact {
        // iPhone-style navigation
        TabView { ... }
    } else {
        // iPad-style navigation
        NavigationSplitView { ... }
    }
}
```

---

## macOS

### Device Characteristics
- Large, high-resolution display
- Pointer-first interactions
- Keyboard-centric workflows
- Multiple windows
- Menu bar always visible

### Navigation Patterns

**Primary navigation**: Sidebar + menu bar

```swift
NavigationSplitView {
    List(items, selection: $selection) { item in
        Text(item.title)
    }
} detail: {
    DetailView(item: selection)
}
.commands {
    CommandMenu("Edit") {
        Button("Find...") { showFind() }
            .keyboardShortcut("f")
    }
}
```

### Key Conventions
- Menu bar for all commands
- Keyboard shortcuts essential
- Window chrome and controls
- Contextual menus on right-click
- Dense layouts acceptable

### Control Sizes
```swift
// macOS control size hierarchy
Button("Mini") { }
    .controlSize(.mini)      // Inspector panels
Button("Small") { }
    .controlSize(.small)     // Dense layouts
Button("Regular") { }
    .controlSize(.regular)   // Default
Button("Large") { }
    .controlSize(.large)     // Spacious areas
```

### Anti-Patterns to Avoid
- iOS-style tab bars
- Touch-sized (44pt) controls
- Missing keyboard shortcuts
- No menu bar commands

---

## watchOS

### Device Characteristics
- Very small display
- Glanceable information
- Brief interactions
- Digital Crown input
- Always-On display consideration

### Navigation Patterns

**Primary navigation**: Vertical paging or list

```swift
// Vertical pages
TabView {
    SummaryView()
    DetailView()
    ActionsView()
}
.tabViewStyle(.verticalPage)

// Scrollable list
List {
    ForEach(items) { item in
        NavigationLink(destination: ItemDetail(item: item)) {
            ItemRow(item: item)
        }
    }
}
```

### Key Conventions
- Full-bleed content
- Minimal padding
- Digital Crown for scrolling/selection
- Complications for watch faces
- Quick actions, not lengthy flows

### Design Constraints
- Maximum 2-3 lines of text per screen
- Large, tappable areas
- High contrast colors
- No horizontal scrolling

---

## tvOS

### Device Characteristics
- Large display (10-foot viewing)
- Focus-based navigation
- Siri Remote input
- No direct touch

### Navigation Patterns

**Primary navigation**: Tab bar at top

```swift
TabView {
    MoviesView()
        .tabItem { Text("Movies") }
    TVShowsView()
        .tabItem { Text("TV Shows") }
    SearchView()
        .tabItem { Text("Search") }
}
```

### Key Conventions
- Focus states must be obvious
- Large elements (60pt+ spacing)
- Spatial navigation (up/down/left/right)
- Limited text input
- Background video/images

### Focus Management
```swift
// Ensure focus states are clear
Button("Watch Now") { }
    .buttonStyle(.card)  // Clear focus appearance
    .focusable()
```

---

## visionOS

### Device Characteristics
- Spatial computing
- Eye tracking + hand gestures
- 3D layouts possible
- Glass materials
- Shared space or immersive

### Navigation Patterns

**Primary navigation**: Window with ornaments

```swift
WindowGroup {
    ContentView()
}
.windowStyle(.plain)
.defaultSize(width: 800, height: 600)

// Or volumetric
WindowGroup(id: "model") {
    Model3D(named: "Globe")
}
.windowStyle(.volumetric)
```

### Key Conventions
- Glass materials (Liquid Glass)
- Comfortable viewing depth
- Center important content in FOV
- Horizontal layouts preferred
- Avoid head-anchored content

### Spatial Considerations
```swift
// Use depth deliberately
Model3D(named: "Product")
    .frame(depth: 200)  // Only for large, important elements

// Avoid
// - Content at peripheral vision edges
// - Sustained oscillating motion
// - Virtual world rotation
```

### Anti-Patterns to Avoid
- Head-locked UI (breaks assistive tech)
- Content requiring head movement
- Motion in peripheral vision
- Excessive depth variation

---

## Cross-Platform Summary

| Aspect | iOS | iPadOS | macOS | watchOS | tvOS | visionOS |
|--------|-----|--------|-------|---------|------|----------|
| Navigation | Tab bar | Sidebar | Sidebar + Menu | List/Pages | Top tabs | Windows |
| Primary Input | Touch | Touch + Pointer | Pointer | Crown + Touch | Focus | Eyes + Hands |
| Density | Medium | Medium-High | High | Low | Very Low | Medium |
| Touch Target | 44pt | 44pt | Varies | Large | 60pt+ | Eye target |
| Key Convention | Swipe back | Split view | Keyboard | Glanceable | Focus states | Spatial |

---

## Adaptive Code Patterns

### Detect Platform
```swift
#if os(iOS)
    // iOS-specific code
#elseif os(macOS)
    // macOS-specific code
#elseif os(watchOS)
    // watchOS-specific code
#elseif os(tvOS)
    // tvOS-specific code
#elseif os(visionOS)
    // visionOS-specific code
#endif
```

### Detect Size Class
```swift
@Environment(\.horizontalSizeClass) var hSizeClass
@Environment(\.verticalSizeClass) var vSizeClass

var body: some View {
    if hSizeClass == .compact {
        // Phone-style layout
    } else {
        // Tablet/desktop layout
    }
}
```

### Platform-Appropriate Navigation
```swift
var body: some View {
    #if os(iOS)
    TabView { content }
    #elseif os(macOS)
    NavigationSplitView { sidebar } detail: { content }
    #endif
}
```

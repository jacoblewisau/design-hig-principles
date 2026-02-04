# Navigation Architecture Reference

Deep dive into navigation patterns, state management, and platform-appropriate navigation design.

---

## Navigation Philosophy

**Apple's navigation principles:**
1. Users should always know where they are
2. Users should always know how to get back
3. Navigation should be predictable
4. Navigation should be interruptible

---

## Navigation Hierarchy

### Flat Navigation (Tab-Based)

Best for: Apps with distinct, parallel sections

```swift
TabView(selection: $selectedTab) {
    HomeView()
        .tabItem { Label("Home", systemImage: "house") }
        .tag(Tab.home)

    SearchView()
        .tabItem { Label("Search", systemImage: "magnifyingglass") }
        .tag(Tab.search)

    ProfileView()
        .tabItem { Label("Profile", systemImage: "person") }
        .tag(Tab.profile)
}
```

**Guidelines:**
- Maximum 5 tabs on iPhone
- Each tab is independent navigation hierarchy
- Tab state should persist when switching
- Use outline icons, filled when selected

### Hierarchical Navigation (Drill-Down)

Best for: Content with parent-child relationships

```swift
NavigationStack {
    List(categories) { category in
        NavigationLink(category.name, value: category)
    }
    .navigationTitle("Categories")
    .navigationDestination(for: Category.self) { category in
        CategoryDetailView(category: category)
    }
}
```

### Split View Navigation (Master-Detail)

Best for: iPad apps, content browsing

```swift
NavigationSplitView {
    // Sidebar
    List(folders, selection: $selectedFolder) { folder in
        Label(folder.name, systemImage: "folder")
    }
    .navigationTitle("Folders")
} content: {
    // Content list (optional middle column)
    if let folder = selectedFolder {
        List(folder.items, selection: $selectedItem) { item in
            Text(item.name)
        }
    }
} detail: {
    // Detail view
    if let item = selectedItem {
        ItemDetailView(item: item)
    } else {
        ContentUnavailableView("Select an Item", systemImage: "doc")
    }
}
```

---

## NavigationStack Deep Dive

### Basic Navigation

```swift
NavigationStack {
    List {
        NavigationLink("Item", value: item)
    }
    .navigationDestination(for: Item.self) { item in
        ItemView(item: item)
    }
}
```

### Programmatic Navigation

```swift
@State private var path = NavigationPath()

NavigationStack(path: $path) {
    ContentView()
        .navigationDestination(for: Item.self) { item in
            ItemView(item: item)
        }
        .navigationDestination(for: Settings.self) { settings in
            SettingsView(settings: settings)
        }
}

// Push programmatically
func showItem(_ item: Item) {
    path.append(item)
}

// Pop to root
func popToRoot() {
    path.removeLast(path.count)
}

// Pop one level
func goBack() {
    path.removeLast()
}
```

### Deep Linking

```swift
NavigationStack(path: $path) {
    RootView()
        .navigationDestination(for: Route.self) { route in
            switch route {
            case .category(let id):
                CategoryView(id: id)
            case .item(let id):
                ItemView(id: id)
            case .settings:
                SettingsView()
            }
        }
}

// Handle deep link
func handleDeepLink(_ url: URL) {
    guard let route = Route(url: url) else { return }
    path.append(route)
}
```

---

## NavigationSplitView Deep Dive

### Column Visibility

```swift
@State private var columnVisibility = NavigationSplitViewVisibility.all

NavigationSplitView(columnVisibility: $columnVisibility) {
    Sidebar()
} content: {
    ContentList()
} detail: {
    DetailView()
}
.navigationSplitViewStyle(.balanced)

// Styles
.navigationSplitViewStyle(.automatic)    // Platform default
.navigationSplitViewStyle(.balanced)     // Equal columns
.navigationSplitViewStyle(.prominentDetail) // Large detail
```

### Adaptive Navigation

```swift
@Environment(\.horizontalSizeClass) var sizeClass

var body: some View {
    if sizeClass == .compact {
        // Tab-based for iPhone
        TabView {
            NavigationStack {
                ContentList()
            }
            .tabItem { Label("Browse", systemImage: "list.bullet") }

            NavigationStack {
                SettingsView()
            }
            .tabItem { Label("Settings", systemImage: "gear") }
        }
    } else {
        // Split view for iPad
        NavigationSplitView {
            Sidebar()
        } detail: {
            DetailView()
        }
    }
}
```

---

## Navigation Bar Configuration

### Title Display Modes

```swift
.navigationTitle("Title")
.navigationBarTitleDisplayMode(.automatic) // Large → inline on scroll
.navigationBarTitleDisplayMode(.large)     // Always large
.navigationBarTitleDisplayMode(.inline)    // Always inline
```

### Toolbar Items

```swift
.toolbar {
    // Primary action (trailing)
    ToolbarItem(placement: .primaryAction) {
        Button("Add", systemImage: "plus") { }
    }

    // Secondary action
    ToolbarItem(placement: .secondaryAction) {
        Button("Edit") { }
    }

    // Navigation bar leading
    ToolbarItem(placement: .navigationBarLeading) {
        Button("Cancel") { }
    }

    // Bottom bar
    ToolbarItem(placement: .bottomBar) {
        HStack {
            Button { } label: { Image(systemName: "arrow.left") }
            Spacer()
            Button { } label: { Image(systemName: "arrow.right") }
        }
    }

    // Keyboard toolbar
    ToolbarItem(placement: .keyboard) {
        Button("Done") { focusField = nil }
    }
}
```

### Toolbar Item Groups

```swift
.toolbar {
    ToolbarItemGroup(placement: .primaryAction) {
        Button("Share", systemImage: "square.and.arrow.up") { }
        Button("Edit", systemImage: "pencil") { }
    }
}
```

### Custom Back Button

```swift
// Custom back button action
.navigationBarBackButtonHidden(true)
.toolbar {
    ToolbarItem(placement: .navigationBarLeading) {
        Button {
            // Custom action before dismiss
            saveChanges()
            dismiss()
        } label: {
            HStack(spacing: 4) {
                Image(systemName: "chevron.left")
                Text("Back")
            }
        }
    }
}

// ⚠️ Warning: This breaks swipe-to-go-back gesture
// Only use when absolutely necessary
```

---

## Modal Presentation

### Sheets

```swift
// Full sheet
.sheet(isPresented: $showSheet) {
    SheetContent()
}

// Sheet with item
.sheet(item: $selectedItem) { item in
    ItemDetail(item: item)
}

// Sheet sizing (iOS 16+)
.sheet(isPresented: $showSheet) {
    SheetContent()
        .presentationDetents([.medium, .large])
        .presentationDragIndicator(.visible)
}

// Custom detents
.presentationDetents([
    .height(200),
    .fraction(0.5),
    .large
])
```

### Full Screen Cover

```swift
.fullScreenCover(isPresented: $showFullScreen) {
    FullScreenContent()
}
```

### Popovers

```swift
.popover(isPresented: $showPopover) {
    PopoverContent()
        .frame(width: 300, height: 400)
}

// With arrow edge
.popover(isPresented: $showPopover, arrowEdge: .top) {
    PopoverContent()
}
```

### When to Use Each

| Presentation | Use Case |
|--------------|----------|
| Sheet | User-initiated task, maintains context |
| Full screen | Immersive content, media, login |
| Popover | Quick options, iPad contextual |
| Navigation push | Drill-down, content hierarchy |

---

## Alerts & Confirmations

### Basic Alert

```swift
.alert("Title", isPresented: $showAlert) {
    Button("OK") { }
}

// With message
.alert("Delete Item?", isPresented: $showDelete) {
    Button("Cancel", role: .cancel) { }
    Button("Delete", role: .destructive) { delete() }
} message: {
    Text("This action cannot be undone.")
}
```

### Confirmation Dialog

```swift
.confirmationDialog("Options", isPresented: $showOptions) {
    Button("Share") { share() }
    Button("Duplicate") { duplicate() }
    Button("Delete", role: .destructive) { delete() }
    Button("Cancel", role: .cancel) { }
}
```

---

## State Restoration

### Preserving Navigation State

```swift
// Scene storage for navigation path
@SceneStorage("navigationPath") private var pathData: Data?

@State private var path = NavigationPath()

NavigationStack(path: $path) {
    // content
}
.task {
    if let data = pathData {
        path = try? JSONDecoder().decode(NavigationPath.self, from: data)
    }
}
.onChange(of: path) { _, newPath in
    pathData = try? JSONEncoder().encode(newPath)
}
```

### Tab State Persistence

```swift
@SceneStorage("selectedTab") private var selectedTab = Tab.home

TabView(selection: $selectedTab) {
    // tabs
}
```

---

## Search Integration

### Searchable Modifier

```swift
NavigationStack {
    List(filteredItems) { item in
        ItemRow(item: item)
    }
    .searchable(text: $searchText, prompt: "Search items")
    .navigationTitle("Items")
}

var filteredItems: [Item] {
    if searchText.isEmpty {
        return items
    }
    return items.filter { $0.name.localizedCaseInsensitiveContains(searchText) }
}
```

### Search Suggestions

```swift
.searchable(text: $searchText) {
    ForEach(suggestions, id: \.self) { suggestion in
        Text(suggestion)
            .searchCompletion(suggestion)
    }
}
```

### Search Scopes

```swift
.searchable(text: $searchText)
.searchScopes($scope) {
    Text("All").tag(SearchScope.all)
    Text("Recent").tag(SearchScope.recent)
    Text("Favorites").tag(SearchScope.favorites)
}
```

---

## Gestures in Navigation

### Swipe to Go Back

```swift
// Enabled by default with NavigationStack
// Don't disable unless necessary
// .interactiveDismissDisabled(true) blocks this
```

### Preventing Accidental Dismissal

```swift
.sheet(isPresented: $showEditor) {
    EditorView()
        .interactiveDismissDisabled(hasUnsavedChanges)
}
```

---

## Platform-Specific Navigation

### iOS Navigation

- Tab bar for top-level (max 5 tabs)
- NavigationStack for drill-down
- Edge swipe for back
- Pull to refresh on lists

### iPadOS Navigation

- Sidebar (NavigationSplitView)
- Keyboard shortcuts
- Pointer/trackpad support
- Multi-window support

### macOS Navigation

- Sidebar + detail
- Menu bar commands
- Window management
- Toolbar customization

### watchOS Navigation

- Vertical page-based
- Digital Crown scrolling
- Minimal depth (2-3 levels max)

### visionOS Navigation

- Window-based
- Ornaments for controls
- Tab-like sidebar

---

## Anti-Patterns

```swift
// ❌ Hamburger menu on iOS
Menu { /* navigation items */ } label: {
    Image(systemName: "line.3.horizontal")
}

// ✅ Tab bar for top-level navigation
TabView { /* tabs */ }

// ❌ Custom back button that breaks gestures
.navigationBarBackButtonHidden(true)
.toolbar {
    ToolbarItem(placement: .leading) {
        Button("Back") { dismiss() }
    }
}

// ✅ Keep system back button (or provide compelling reason)
// System back button works with swipe gesture

// ❌ Deep nesting without escape hatch
// 5+ levels deep with no way to jump back

// ✅ Provide "pop to root" or breadcrumbs for deep hierarchies

// ❌ Modal for everything
.sheet { DetailView() }
.sheet { AnotherDetail() }

// ✅ Push for hierarchical content
.navigationDestination(for: Item.self) { item in
    ItemDetail(item: item)
}

// ❌ Non-standard gestures replacing navigation
// Custom swipe that conflicts with system

// ✅ Use standard navigation patterns
NavigationStack { }
```

---

## Quick Reference

| Pattern | When | Implementation |
|---------|------|----------------|
| Flat/Tabs | Parallel sections | `TabView` |
| Hierarchical | Parent-child | `NavigationStack` |
| Master-Detail | Browse & view | `NavigationSplitView` |
| Modal/Sheet | Focus task | `.sheet()` |
| Full screen | Immersive | `.fullScreenCover()` |
| Popover | Contextual (iPad) | `.popover()` |
| Alert | Confirmation | `.alert()` |
| Action sheet | Multiple options | `.confirmationDialog()` |

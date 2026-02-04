# Lists & Collections Reference

Complete guide to list styles, grids, collection views, and content organization patterns.

---

## List Philosophy

**Apple's guidance:** Lists are the primary way to present structured content on iOS. Use them for navigable content, settings, and data displays.

---

## List Styles

### Automatic (Default)

```swift
List {
    content
}
.listStyle(.automatic)
// Platform chooses appropriate style
```

### Inset Grouped (Settings Style)

```swift
List {
    Section("Account") {
        NavigationLink("Profile") { }
        NavigationLink("Notifications") { }
    }

    Section("About") {
        NavigationLink("Help") { }
        NavigationLink("Privacy") { }
    }
}
.listStyle(.insetGrouped)
```

### Grouped

```swift
List {
    Section(header: Text("Section 1")) {
        rows
    }
}
.listStyle(.grouped)
// Classic grouped appearance
```

### Plain

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
}
.listStyle(.plain)
// No section styling, content edge-to-edge
```

### Sidebar

```swift
List(selection: $selection) {
    Section("Folders") {
        ForEach(folders) { folder in
            Label(folder.name, systemImage: "folder")
        }
    }
}
.listStyle(.sidebar)
// For NavigationSplitView sidebars
```

### Style Selection Guide

| Use Case | Style |
|----------|-------|
| Settings/Forms | .insetGrouped |
| Content browsing | .plain |
| Navigation sidebar | .sidebar |
| Classic iOS | .grouped |
| Let system decide | .automatic |

---

## List Rows

### Standard Row Layout

```swift
List {
    ForEach(items) { item in
        HStack {
            // Leading accessory (optional)
            Image(systemName: item.icon)
                .foregroundStyle(.accent)
                .frame(width: 28)

            // Content
            VStack(alignment: .leading, spacing: 2) {
                Text(item.title)
                    .font(.body)
                Text(item.subtitle)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()

            // Trailing (optional)
            Text(item.value)
                .foregroundStyle(.secondary)
        }
        .padding(.vertical, 4)
    }
}
```

### Navigation Link Row

```swift
NavigationLink {
    DetailView(item: item)
} label: {
    ItemRow(item: item)
}
// Automatically adds chevron
```

### Disclosure Indicator Manually

```swift
HStack {
    Text("Item")
    Spacer()
    Image(systemName: "chevron.right")
        .font(.caption)
        .foregroundStyle(.tertiary)
}
```

---

## Row Heights

| Content | Typical Height |
|---------|----------------|
| Single line | 44pt |
| Two lines | 56-60pt |
| Three lines | 72-80pt |
| With thumbnail | 60-80pt |

```swift
// Minimum height for touch targets
.frame(minHeight: 44)
```

---

## List Interactions

### Swipe Actions

```swift
ForEach(items) { item in
    ItemRow(item: item)
        .swipeActions(edge: .trailing, allowsFullSwipe: true) {
            Button("Delete", role: .destructive) {
                delete(item)
            }
            Button("Archive") {
                archive(item)
            }
            .tint(.orange)
        }
        .swipeActions(edge: .leading) {
            Button("Pin") {
                pin(item)
            }
            .tint(.yellow)
        }
}
```

### Context Menu

```swift
ItemRow(item: item)
    .contextMenu {
        Button("Copy") { copy(item) }
        Button("Share") { share(item) }
        Divider()
        Button("Delete", role: .destructive) { delete(item) }
    }
```

### Selection

```swift
// Single selection
@State private var selection: Item.ID?

List(items, selection: $selection) { item in
    ItemRow(item: item)
}

// Multiple selection
@State private var selections: Set<Item.ID> = []

List(items, selection: $selections) { item in
    ItemRow(item: item)
}
.toolbar {
    EditButton()
}
```

### Reordering

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
    .onMove { from, to in
        items.move(fromOffsets: from, toOffset: to)
    }
}
.toolbar {
    EditButton()
}
```

### Deletion

```swift
ForEach(items) { item in
    ItemRow(item: item)
}
.onDelete { indexSet in
    items.remove(atOffsets: indexSet)
}
```

---

## Sections

### Basic Section

```swift
Section {
    rows
}
```

### With Header/Footer

```swift
Section {
    rows
} header: {
    Text("Header")
} footer: {
    Text("Explanatory footer text")
}
```

### Collapsible Section

```swift
Section("Collapsible", isExpanded: $isExpanded) {
    rows
}
// User can collapse/expand
```

---

## Search in Lists

```swift
@State private var searchText = ""

NavigationStack {
    List {
        ForEach(filteredItems) { item in
            ItemRow(item: item)
        }
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
    Text("All").tag(Scope.all)
    Text("Recent").tag(Scope.recent)
    Text("Favorites").tag(Scope.favorites)
}
```

---

## Grids

### LazyVGrid

```swift
let columns = [
    GridItem(.adaptive(minimum: 150, maximum: 200))
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 16) {
        ForEach(items) { item in
            GridCell(item: item)
        }
    }
    .padding()
}
```

### Fixed Columns

```swift
let columns = [
    GridItem(.flexible()),
    GridItem(.flexible()),
    GridItem(.flexible())
]

LazyVGrid(columns: columns, spacing: 16) {
    content
}
```

### Mixed Column Sizes

```swift
let columns = [
    GridItem(.fixed(100)),
    GridItem(.flexible(minimum: 100)),
    GridItem(.flexible())
]
```

### LazyHGrid

```swift
let rows = [
    GridItem(.fixed(100)),
    GridItem(.fixed(100))
]

ScrollView(.horizontal) {
    LazyHGrid(rows: rows, spacing: 16) {
        ForEach(items) { item in
            GridCell(item: item)
        }
    }
}
```

---

## Grid Spacing

| Content Type | Item Spacing | Edge Padding |
|--------------|--------------|--------------|
| Photos | 2-4pt | 0-16pt |
| Cards | 12-16pt | 16-20pt |
| Icons | 16-24pt | 16pt |
| Large tiles | 20-24pt | 20pt |

---

## Lazy Loading

### LazyVStack

```swift
ScrollView {
    LazyVStack(spacing: 0) {
        ForEach(items) { item in
            ItemRow(item: item)
            Divider()
        }
    }
}
```

### Pinned Headers

```swift
ScrollView {
    LazyVStack(pinnedViews: [.sectionHeaders]) {
        ForEach(sections) { section in
            Section {
                ForEach(section.items) { item in
                    ItemRow(item: item)
                }
            } header: {
                SectionHeader(title: section.title)
            }
        }
    }
}
```

---

## Empty States

```swift
List {
    if items.isEmpty {
        ContentUnavailableView(
            "No Items",
            systemImage: "tray",
            description: Text("Add items to see them here.")
        )
        .listRowBackground(Color.clear)
    } else {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}
```

---

## Pull to Refresh

```swift
List {
    content
}
.refreshable {
    await loadData()
}
```

---

## List Customization

### Row Separator

```swift
// Hide separators
.listRowSeparator(.hidden)

// Customize separator
.listRowSeparatorTint(.red)
```

### Row Background

```swift
.listRowBackground(Color.yellow.opacity(0.2))
```

### Row Insets

```swift
.listRowInsets(EdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16))
```

### Section Separator

```swift
Section {
    content
}
.listSectionSeparator(.hidden)
```

---

## Performance

### Identifier Stability

```swift
// ❌ Unstable identifier
ForEach(items.indices, id: \.self) { index in
    ItemRow(item: items[index])
}

// ✅ Stable identifier
ForEach(items) { item in
    ItemRow(item: item)
}
// Where Item: Identifiable
```

### Heavy Content

```swift
// Use lazy containers for large lists
LazyVStack {
    ForEach(manyItems) { item in
        ItemRow(item: item)
    }
}

// Avoid expensive operations in row
ForEach(items) { item in
    // ❌ Don't
    ExpensiveView(item: processItem(item))

    // ✅ Do
    CachedItemView(item: item)
}
```

---

## Accessibility

### Grouping

```swift
HStack {
    Image(systemName: "star")
    VStack(alignment: .leading) {
        Text("Title")
        Text("Subtitle")
    }
}
.accessibilityElement(children: .combine)
.accessibilityLabel("Title, Subtitle")
```

### Custom Actions

```swift
ItemRow(item: item)
    .accessibilityAction(named: "Delete") {
        delete(item)
    }
    .accessibilityAction(named: "Share") {
        share(item)
    }
```

---

## Common Patterns

### Settings List

```swift
Form {
    Section("Account") {
        NavigationLink("Profile", destination: ProfileView())
        NavigationLink("Privacy", destination: PrivacyView())
    }

    Section("Preferences") {
        Toggle("Notifications", isOn: $notifications)
        Picker("Theme", selection: $theme) {
            Text("Light").tag(Theme.light)
            Text("Dark").tag(Theme.dark)
            Text("System").tag(Theme.system)
        }
    }

    Section {
        Button("Sign Out", role: .destructive) { signOut() }
    }
}
```

### Inbox List

```swift
List {
    ForEach(messages) { message in
        MessageRow(message: message)
            .swipeActions {
                Button("Delete", role: .destructive) { delete(message) }
                Button("Archive") { archive(message) }
            }
    }
}
.listStyle(.plain)
.searchable(text: $search)
```

### Photo Grid

```swift
let columns = [GridItem(.adaptive(minimum: 100))]

ScrollView {
    LazyVGrid(columns: columns, spacing: 2) {
        ForEach(photos) { photo in
            AsyncImage(url: photo.thumbnailURL) { image in
                image.resizable().aspectRatio(1, contentMode: .fill)
            } placeholder: {
                Color.gray
            }
            .clipped()
        }
    }
}
```

---

## Anti-Patterns

```swift
// ❌ Non-lazy for large data
VStack {
    ForEach(thousandsOfItems) { item in
        ItemRow(item: item)
    }
}

// ✅ Lazy loading
List {
    ForEach(thousandsOfItems) { item in
        ItemRow(item: item)
    }
}

// ❌ Unstable identifiers
ForEach(0..<items.count, id: \.self) { }

// ✅ Identifiable items
ForEach(items) { item in }

// ❌ Swipe-only actions
.swipeActions {
    Button("Delete") { }
}

// ✅ Swipe + context menu
.swipeActions {
    Button("Delete") { }
}
.contextMenu {
    Button("Delete", role: .destructive) { }
}
```

---

## Quick Reference

| Pattern | Component |
|---------|-----------|
| Settings | `Form` + `.insetGrouped` |
| Content browse | `List` + `.plain` |
| Navigation | `List` + `NavigationLink` |
| Photo grid | `LazyVGrid` + adaptive |
| Card grid | `LazyVGrid` + fixed |
| Infinite scroll | `LazyVStack` + `.onAppear` |
| Sidebar | `List` + `.sidebar` |

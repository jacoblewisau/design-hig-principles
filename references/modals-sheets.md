# Modals & Sheets Reference

Complete guide to modal presentations, sheets, alerts, and interruption patterns.

---

## Modal Philosophy

**When to use modals:**
- Focus user attention on a specific task
- Require user input before continuing
- Display critical information
- Provide a contained experience

**When NOT to use modals:**
- Regular navigation (use push instead)
- Displaying information that doesn't need focus
- Frequent interactions (modal fatigue)

---

## Sheet Presentations

### Basic Sheet

```swift
@State private var showSheet = false

Button("Show Sheet") {
    showSheet = true
}
.sheet(isPresented: $showSheet) {
    SheetContent()
}
```

### Sheet with Item

```swift
@State private var selectedItem: Item?

List(items) { item in
    Button(item.name) {
        selectedItem = item
    }
}
.sheet(item: $selectedItem) { item in
    ItemDetail(item: item)
}
```

### Dismissing Sheets

```swift
struct SheetContent: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            content
                .toolbar {
                    ToolbarItem(placement: .cancellationAction) {
                        Button("Cancel") { dismiss() }
                    }
                    ToolbarItem(placement: .confirmationAction) {
                        Button("Done") {
                            save()
                            dismiss()
                        }
                    }
                }
        }
    }
}
```

---

## Sheet Presentation Detents (iOS 16+)

### Standard Detents

```swift
.sheet(isPresented: $showSheet) {
    SheetContent()
        .presentationDetents([.medium, .large])
}
```

### Available Detents

```swift
.presentationDetents([
    .medium,        // ~50% of screen
    .large,         // Full height (default)
    .height(200),   // Fixed height
    .fraction(0.3), // Fraction of screen
])
```

### Custom Detents

```swift
extension PresentationDetent {
    static let small = Self.height(200)
    static let third = Self.fraction(0.33)
}

.presentationDetents([.small, .medium, .large])
```

### Detent Selection

```swift
@State private var selectedDetent: PresentationDetent = .medium

.presentationDetents([.medium, .large], selection: $selectedDetent)
```

### Drag Indicator

```swift
.presentationDragIndicator(.visible)   // Always show
.presentationDragIndicator(.hidden)    // Never show
.presentationDragIndicator(.automatic) // System decides
```

---

## Sheet Customization

### Background

```swift
.presentationBackground(.ultraThinMaterial)
.presentationBackground(Color.blue.opacity(0.1))
.presentationBackground {
    LinearGradient(colors: [.blue, .purple], startPoint: .top, endPoint: .bottom)
}
```

### Corner Radius

```swift
.presentationCornerRadius(24)
```

### Content Interaction

```swift
// Allow interaction with content behind sheet
.presentationBackgroundInteraction(.enabled)

// Only at specific detent
.presentationBackgroundInteraction(.enabled(upThrough: .medium))
```

### Preventing Dismissal

```swift
// Prevent interactive dismiss (swipe down)
.interactiveDismissDisabled()

// Conditional
.interactiveDismissDisabled(hasUnsavedChanges)
```

### Compact Adaptation

```swift
// How sheet adapts on compact width
.presentationCompactAdaptation(.sheet)        // Stay as sheet
.presentationCompactAdaptation(.fullScreenCover) // Become full screen
.presentationCompactAdaptation(.popover)      // Try popover first
```

---

## Full Screen Cover

For immersive content that needs the full screen.

```swift
.fullScreenCover(isPresented: $showFullScreen) {
    FullScreenContent()
}

// With item
.fullScreenCover(item: $selectedItem) { item in
    ItemFullScreen(item: item)
}
```

### When to Use Full Screen

| Use Case | Presentation |
|----------|--------------|
| Login/Onboarding | Full screen |
| Media playback | Full screen |
| Camera/AR | Full screen |
| Document editor | Full screen or large sheet |
| Quick task | Sheet |
| Settings | Sheet or push |

---

## Popovers

Best for iPad and Mac, falls back to sheet on iPhone.

```swift
.popover(isPresented: $showPopover) {
    PopoverContent()
        .frame(width: 300, height: 200)
}

// With attachment anchor
.popover(isPresented: $showPopover, attachmentAnchor: .point(.top)) {
    PopoverContent()
}

// With arrow edge
.popover(isPresented: $showPopover, arrowEdge: .bottom) {
    PopoverContent()
}
```

### Popover Sizing

```swift
PopoverContent()
    .frame(width: 320, height: 400)
    .frame(minWidth: 200, maxWidth: 400)
```

---

## Alerts

### Basic Alert

```swift
.alert("Title", isPresented: $showAlert) {
    Button("OK") { }
}
```

### Alert with Message

```swift
.alert("Delete Item?", isPresented: $showDelete) {
    Button("Cancel", role: .cancel) { }
    Button("Delete", role: .destructive) {
        deleteItem()
    }
} message: {
    Text("This cannot be undone.")
}
```

### Alert with Text Field

```swift
@State private var inputText = ""

.alert("Enter Name", isPresented: $showInput) {
    TextField("Name", text: $inputText)
    Button("Cancel", role: .cancel) { }
    Button("Save") { save(inputText) }
} message: {
    Text("Enter a name for this item.")
}
```

### Alert with Error

```swift
.alert(isPresented: $showError, error: currentError) { error in
    Button("OK") { }
} message: { error in
    Text(error.recoverySuggestion ?? "Please try again.")
}
```

### Alert Button Ordering

```swift
// Destructive action
.alert("Delete?", isPresented: $show) {
    Button("Cancel", role: .cancel) { }        // Left, default
    Button("Delete", role: .destructive) { }   // Right, red
}

// Confirmation
.alert("Save?", isPresented: $show) {
    Button("Cancel", role: .cancel) { }        // Left
    Button("Save") { }                          // Right, prominent
}
```

---

## Confirmation Dialog (Action Sheet)

```swift
.confirmationDialog("Options", isPresented: $showOptions) {
    Button("Share") { share() }
    Button("Duplicate") { duplicate() }
    Button("Delete", role: .destructive) { delete() }
    Button("Cancel", role: .cancel) { }
}
```

### With Title Visibility

```swift
.confirmationDialog("Choose Action", isPresented: $show, titleVisibility: .visible) {
    // actions
}
```

### Presenting from Item

```swift
.confirmationDialog("Options", isPresented: $showOptions, presenting: selectedItem) { item in
    Button("Edit \(item.name)") { edit(item) }
    Button("Delete \(item.name)", role: .destructive) { delete(item) }
}
```

---

## Inspector (iOS 17+)

Side panel presentation for auxiliary content.

```swift
.inspector(isPresented: $showInspector) {
    InspectorContent()
        .inspectorColumnWidth(min: 200, ideal: 300, max: 400)
}
```

---

## Modal Content Structure

### Standard Sheet Layout

```swift
struct EditSheet: View {
    @Environment(\.dismiss) private var dismiss
    @State private var name = ""

    var body: some View {
        NavigationStack {
            Form {
                TextField("Name", text: $name)
            }
            .navigationTitle("Edit Item")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Save") {
                        save()
                        dismiss()
                    }
                    .disabled(name.isEmpty)
                }
            }
        }
    }
}
```

### Full Screen Modal Layout

```swift
struct FullScreenModal: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            content
                .toolbar {
                    ToolbarItem(placement: .navigationBarLeading) {
                        Button("Close") { dismiss() }
                    }
                }
        }
    }
}
```

---

## Toolbar Placements for Modals

```swift
// Cancel button (leading)
ToolbarItem(placement: .cancellationAction)

// Confirm button (trailing)
ToolbarItem(placement: .confirmationAction)

// Destructive action
ToolbarItem(placement: .destructiveAction)

// Primary action
ToolbarItem(placement: .primaryAction)
```

---

## Modal Transitions

### Default Sheet Transition

System provides slide-up transition automatically.

### Custom Presentation (UIKit Interop)

```swift
// For custom transitions, use UIViewControllerRepresentable
// SwiftUI sheets use system animations
```

---

## Preventing Data Loss

### Unsaved Changes Warning

```swift
struct EditorSheet: View {
    @Environment(\.dismiss) private var dismiss
    @State private var content = ""
    @State private var originalContent = ""
    @State private var showDiscardAlert = false

    var hasChanges: Bool {
        content != originalContent
    }

    var body: some View {
        NavigationStack {
            TextEditor(text: $content)
                .toolbar {
                    ToolbarItem(placement: .cancellationAction) {
                        Button("Cancel") {
                            if hasChanges {
                                showDiscardAlert = true
                            } else {
                                dismiss()
                            }
                        }
                    }
                }
                .alert("Discard Changes?", isPresented: $showDiscardAlert) {
                    Button("Keep Editing", role: .cancel) { }
                    Button("Discard", role: .destructive) { dismiss() }
                }
        }
        .interactiveDismissDisabled(hasChanges)
    }
}
```

---

## Stacking Modals

### Multiple Sheets (Avoid if Possible)

```swift
// ❌ Multiple sheets - confusing
.sheet(isPresented: $showFirst) {
    FirstSheet()
        .sheet(isPresented: $showSecond) {
            SecondSheet()
        }
}

// ✅ Navigation within sheet
.sheet(isPresented: $showSheet) {
    NavigationStack {
        FirstView()
            .navigationDestination(for: Item.self) { item in
                SecondView(item: item)
            }
    }
}
```

### Alert Over Sheet

```swift
// ✅ Alerts can appear over sheets
.sheet(isPresented: $showSheet) {
    SheetContent()
        .alert("Confirm?", isPresented: $showAlert) {
            // alert buttons
        }
}
```

---

## Accessibility

### Focus Management

```swift
.sheet(isPresented: $showSheet) {
    SheetContent()
        .accessibilityAddTraits(.isModal)
}
```

### Escape to Dismiss

Sheets support hardware keyboard Escape to dismiss by default.

### VoiceOver Announcement

```swift
.onChange(of: showSheet) { _, isPresented in
    if isPresented {
        UIAccessibility.post(notification: .screenChanged, argument: nil)
    }
}
```

---

## Platform Considerations

### iOS

- Sheets slide up from bottom
- Popovers become sheets on iPhone
- Support detents for flexible sizing

### iPadOS

- Popovers appear as floating panels
- Sheets can be form-style (centered)
- Support multiple windows

### macOS

- Sheets slide down from title bar
- Popovers are floating panels
- Alerts are centered dialogs

---

## Anti-Patterns

```swift
// ❌ Modal for navigation
.sheet(isPresented: $showDetail) {
    DetailView()
}

// ✅ Push for hierarchical content
NavigationLink("Detail", destination: DetailView())

// ❌ Alert for complex forms
.alert("Enter Details", isPresented: $show) {
    TextField("Name", text: $name)
    TextField("Email", text: $email)
    TextField("Phone", text: $phone)
}

// ✅ Sheet for forms
.sheet(isPresented: $show) {
    FormSheet()
}

// ❌ Full screen for quick actions
.fullScreenCover(isPresented: $showOptions) {
    OptionsView()
}

// ✅ Sheet or action sheet for options
.confirmationDialog("Options", isPresented: $showOptions) {
    // actions
}

// ❌ Multiple nested modals
.sheet { .sheet { .sheet { } } }

// ✅ Navigation within modal
.sheet {
    NavigationStack { }
}

// ❌ Dismissing without saving user work
Button("Cancel") { dismiss() }

// ✅ Warn about unsaved changes
Button("Cancel") {
    if hasChanges { showWarning = true }
    else { dismiss() }
}
```

---

## Quick Reference

| Presentation | Best For | Height |
|--------------|----------|--------|
| Sheet (.medium) | Quick tasks | ~50% |
| Sheet (.large) | Forms, detail | Full |
| Full screen cover | Immersive | Full |
| Popover | Contextual (iPad) | Content |
| Alert | Confirmation | Compact |
| Confirmation dialog | Multiple options | Compact |
| Inspector | Auxiliary panel | Variable |

| Toolbar Placement | Use |
|-------------------|-----|
| .cancellationAction | Cancel button |
| .confirmationAction | Save/Done button |
| .destructiveAction | Delete button |
| .primaryAction | Main action |

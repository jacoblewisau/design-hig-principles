# Controls & Inputs Reference

Complete guide to system controls, form elements, and input patterns on Apple platforms.

---

## Control Philosophy

**Apple's approach to controls:**
- Familiar controls reduce learning curve
- Controls should be obvious affordances
- Customization should preserve usability
- Controls adapt to context (size class, platform)

---

## Buttons

### Button Styles

```swift
// Default (automatic style based on context)
Button("Action") { }

// Bordered - Secondary actions
Button("Learn More") { }
    .buttonStyle(.bordered)

// Bordered Prominent - Primary actions
Button("Continue") { }
    .buttonStyle(.borderedProminent)

// Borderless - Tertiary actions
Button("Cancel") { }
    .buttonStyle(.borderless)

// Plain - Minimal styling
Button("Details") { }
    .buttonStyle(.plain)
```

### Button Roles

```swift
// Destructive - Red tint
Button("Delete", role: .destructive) { delete() }

// Cancel - Special positioning in alerts
Button("Cancel", role: .cancel) { }
```

### Button Sizing

```swift
// Control sizes
Button("Mini") { }
    .controlSize(.mini)      // 22pt height

Button("Small") { }
    .controlSize(.small)     // 28pt height

Button("Regular") { }
    .controlSize(.regular)   // 34pt height (default)

Button("Large") { }
    .controlSize(.large)     // 44pt height

Button("Extra Large") { }
    .controlSize(.extraLarge) // 52pt height (iOS 17+)
```

### Button Touch Targets

```swift
// ✅ Minimum 44x44pt touch target
Button { } label: {
    Image(systemName: "xmark")
        .font(.caption) // Visual can be small
}
.frame(minWidth: 44, minHeight: 44) // Hit area must be large
.contentShape(Rectangle())

// ❌ Too small
Button { } label: {
    Image(systemName: "xmark")
}
.frame(width: 24, height: 24)
```

### Button Best Practices

| Use Case | Style | Size |
|----------|-------|------|
| Primary action | .borderedProminent | .large |
| Secondary action | .bordered | .regular |
| Tertiary action | .borderless | .regular |
| Destructive | role: .destructive | .regular |
| Navigation | .plain | .regular |
| Toolbar | Default | .regular |

---

## Toggle / Switch

```swift
// Standard toggle
Toggle("Airplane Mode", isOn: $isAirplaneMode)

// With system image
Toggle(isOn: $isEnabled) {
    Label("Notifications", systemImage: "bell")
}

// Custom tint
Toggle("Feature", isOn: $isEnabled)
    .tint(.purple)
```

### Toggle Placement

| Context | Placement | Notes |
|---------|-----------|-------|
| Settings row | Trailing | Standard iOS pattern |
| Form section | Trailing | With label leading |
| Toolbar | Avoid | Use button instead |

---

## Picker

### Picker Styles

```swift
// Automatic (context-dependent)
Picker("Selection", selection: $selection) {
    ForEach(options, id: \.self) { Text($0) }
}

// Segmented control
Picker("View", selection: $viewMode) {
    Text("List").tag(ViewMode.list)
    Text("Grid").tag(ViewMode.grid)
}
.pickerStyle(.segmented)

// Menu picker
Picker("Sort", selection: $sortOrder) {
    ForEach(SortOrder.allCases) { order in
        Text(order.title).tag(order)
    }
}
.pickerStyle(.menu)

// Wheel picker
Picker("Count", selection: $count) {
    ForEach(1...10, id: \.self) { Text("\($0)") }
}
.pickerStyle(.wheel)

// Inline picker (expanded in form)
Picker("Category", selection: $category) {
    ForEach(categories) { Text($0.name).tag($0) }
}
.pickerStyle(.inline)

// Navigation link picker
Picker("Country", selection: $country) {
    ForEach(countries) { Text($0.name).tag($0) }
}
.pickerStyle(.navigationLink)
```

### Picker Guidelines

| Options | Style | Context |
|---------|-------|---------|
| 2-4 mutually exclusive | .segmented | Toolbar, header |
| 5-10 options | .menu | Form row |
| Many options | .navigationLink | Settings, selection |
| Numbers/dates | .wheel | Quick selection |

---

## Slider

```swift
// Basic slider
Slider(value: $volume, in: 0...100)

// With labels
Slider(value: $volume, in: 0...100) {
    Text("Volume")
} minimumValueLabel: {
    Image(systemName: "speaker")
} maximumValueLabel: {
    Image(systemName: "speaker.wave.3")
}

// Stepped slider
Slider(value: $rating, in: 1...5, step: 1)

// Accessibility
Slider(value: $brightness)
    .accessibilityLabel("Brightness")
    .accessibilityValue("\(Int(brightness * 100))%")
```

### Slider Guidelines

- Use for continuous values
- Provide visual feedback of current value
- Consider stepped values for discrete options
- Always include accessibility labels

---

## Stepper

```swift
// Basic stepper
Stepper("Quantity: \(quantity)", value: $quantity, in: 1...99)

// Custom increment
Stepper("Temperature: \(temperature)°", value: $temperature, in: 60...85, step: 5)

// Custom labels
Stepper {
    Text("Adults: \(adults)")
} onIncrement: {
    adults += 1
} onDecrement: {
    adults = max(0, adults - 1)
}
```

---

## Text Input

### TextField

```swift
// Basic text field
TextField("Name", text: $name)

// With prompt
TextField("Email", text: $email, prompt: Text("you@example.com"))

// Axis expansion
TextField("Bio", text: $bio, axis: .vertical)
    .lineLimit(3...6)

// Secure field
SecureField("Password", text: $password)
```

### Text Field Styles

```swift
// Default (platform appropriate)
TextField("Search", text: $query)

// Rounded border (iOS)
TextField("Search", text: $query)
    .textFieldStyle(.roundedBorder)

// Plain
TextField("Amount", text: $amount)
    .textFieldStyle(.plain)
```

### Keyboard Types

```swift
TextField("Email", text: $email)
    .keyboardType(.emailAddress)

TextField("Phone", text: $phone)
    .keyboardType(.phonePad)

TextField("URL", text: $url)
    .keyboardType(.URL)

TextField("Number", text: $number)
    .keyboardType(.decimalPad)
```

### Text Content Types

```swift
TextField("Email", text: $email)
    .textContentType(.emailAddress)

TextField("Password", text: $password)
    .textContentType(.password)

TextField("Name", text: $name)
    .textContentType(.name)

TextField("Address", text: $address)
    .textContentType(.fullStreetAddress)
```

### Input Validation

```swift
// Visual feedback for validation
TextField("Email", text: $email)
    .textFieldStyle(.roundedBorder)
    .overlay(
        RoundedRectangle(cornerRadius: 8)
            .stroke(isValidEmail ? Color.clear : Color.red, lineWidth: 1)
    )

// Error message
VStack(alignment: .leading, spacing: 4) {
    TextField("Email", text: $email)
    if !isValidEmail && !email.isEmpty {
        Text("Please enter a valid email")
            .font(.caption)
            .foregroundStyle(.red)
    }
}
```

---

## TextEditor

```swift
// Multi-line text input
TextEditor(text: $notes)
    .frame(minHeight: 100)

// With placeholder (custom)
ZStack(alignment: .topLeading) {
    if notes.isEmpty {
        Text("Enter notes...")
            .foregroundStyle(.tertiary)
            .padding(.top, 8)
            .padding(.leading, 5)
    }
    TextEditor(text: $notes)
}
```

---

## Date & Time Pickers

```swift
// Date picker
DatePicker("Date", selection: $date, displayedComponents: .date)

// Time picker
DatePicker("Time", selection: $time, displayedComponents: .hourAndMinute)

// Date and time
DatePicker("When", selection: $dateTime)

// Date range
DatePicker("Start", selection: $startDate, in: Date()...)

// Compact style
DatePicker("Date", selection: $date)
    .datePickerStyle(.compact)

// Graphical style
DatePicker("Date", selection: $date)
    .datePickerStyle(.graphical)

// Wheel style
DatePicker("Time", selection: $time, displayedComponents: .hourAndMinute)
    .datePickerStyle(.wheel)
```

---

## Color Picker

```swift
// Basic color picker
ColorPicker("Accent Color", selection: $accentColor)

// Without opacity
ColorPicker("Color", selection: $color, supportsOpacity: false)
```

---

## PhotosPicker

```swift
import PhotosUI

PhotosPicker(selection: $selectedItem, matching: .images) {
    Label("Select Photo", systemImage: "photo")
}

// Multiple selection
PhotosPicker(selection: $selectedItems, maxSelectionCount: 5, matching: .images) {
    Text("Select Photos")
}
```

---

## Menu

```swift
// Basic menu
Menu("Options") {
    Button("Edit") { }
    Button("Duplicate") { }
    Divider()
    Button("Delete", role: .destructive) { }
}

// Menu with icon
Menu {
    Button("Edit") { }
    Button("Share") { }
} label: {
    Image(systemName: "ellipsis.circle")
}

// Nested menus
Menu("Sort") {
    Menu("By Date") {
        Button("Newest First") { }
        Button("Oldest First") { }
    }
    Menu("By Name") {
        Button("A to Z") { }
        Button("Z to A") { }
    }
}

// Primary action + menu
Menu {
    Button("Option 1") { }
    Button("Option 2") { }
} label: {
    Label("Add", systemImage: "plus")
} primaryAction: {
    addItem()
}
```

---

## Context Menu

```swift
Text("Long press me")
    .contextMenu {
        Button("Copy") { }
        Button("Share") { }
        Divider()
        Button("Delete", role: .destructive) { }
    }

// With preview
Text("Item")
    .contextMenu {
        Button("View") { }
    } preview: {
        ItemPreview(item: item)
    }
```

---

## Swipe Actions

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
            .swipeActions(edge: .trailing) {
                Button("Delete", role: .destructive) { delete(item) }
                Button("Archive") { archive(item) }
                    .tint(.orange)
            }
            .swipeActions(edge: .leading) {
                Button("Pin") { pin(item) }
                    .tint(.yellow)
            }
    }
}
```

### Swipe Action Guidelines

- Trailing = Destructive/primary actions
- Leading = Secondary/positive actions
- Always provide alternative access (context menu)
- Limit to 3 actions per side

---

## Forms

### Form Structure

```swift
Form {
    Section {
        TextField("Name", text: $name)
        TextField("Email", text: $email)
    } header: {
        Text("Account")
    } footer: {
        Text("Your email will be used for notifications.")
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
        Button("Save", action: save)
            .frame(maxWidth: .infinity)
    }
}
```

### LabeledContent

```swift
Form {
    LabeledContent("Version", value: "1.0.0")
    LabeledContent("Build", value: "42")
    LabeledContent {
        Text(formattedDate)
    } label: {
        Text("Last Updated")
    }
}
```

---

## Disclosure Controls

```swift
// Disclosure group
DisclosureGroup("Advanced Options") {
    Toggle("Option 1", isOn: $option1)
    Toggle("Option 2", isOn: $option2)
}

// Controlled expansion
DisclosureGroup("Details", isExpanded: $isExpanded) {
    Text("Hidden content")
}
```

---

## Progress Indicators

### Determinate Progress

```swift
// Linear progress
ProgressView(value: progress, total: 100)

// With label
ProgressView(value: downloadProgress) {
    Text("Downloading...")
} currentValueLabel: {
    Text("\(Int(downloadProgress * 100))%")
}

// Circular (Gauge)
Gauge(value: progress, in: 0...100) {
    Text("Progress")
}
.gaugeStyle(.accessoryCircular)
```

### Indeterminate Progress

```swift
// Spinner
ProgressView()

// With label
ProgressView("Loading...")
```

---

## Control Accessibility

### Labels

```swift
Button { } label: {
    Image(systemName: "gear")
}
.accessibilityLabel("Settings")

Slider(value: $volume)
    .accessibilityLabel("Volume")
    .accessibilityValue("\(Int(volume))%")
```

### Hints

```swift
Button("Submit") { }
    .accessibilityHint("Double-tap to submit the form")
```

### Traits

```swift
Toggle("Feature", isOn: $isEnabled)
    .accessibilityAddTraits(.isSelected)
```

---

## Platform Variations

### iOS vs macOS

| Control | iOS | macOS |
|---------|-----|-------|
| Toggle | Switch | Checkbox |
| Picker | Wheel/Menu | Popup |
| Button height | 34-44pt | Varies |
| Touch target | 44x44pt | Smaller OK |

### Adapting Controls

```swift
#if os(iOS)
Picker("Option", selection: $option) { }
    .pickerStyle(.menu)
#elseif os(macOS)
Picker("Option", selection: $option) { }
    .pickerStyle(.radioGroup)
#endif
```

---

## Anti-Patterns

```swift
// ❌ Custom control replacing system
CustomToggle(isOn: $value) // Loses accessibility

// ✅ Use system control
Toggle("Feature", isOn: $value)

// ❌ Button without clear affordance
Text("Click here")
    .onTapGesture { action() }

// ✅ Obvious button
Button("Take Action") { action() }

// ❌ Tiny touch target
Button { } label: { Image(systemName: "x") }
    .frame(width: 20, height: 20)

// ✅ Adequate touch target
Button { } label: { Image(systemName: "xmark").font(.caption) }
    .frame(minWidth: 44, minHeight: 44)

// ❌ Color-only state indication
Circle().fill(isActive ? .green : .gray)

// ✅ Multiple indicators
Image(systemName: isActive ? "checkmark.circle.fill" : "circle")
    .foregroundStyle(isActive ? .green : .secondary)
```

---

## Quick Reference

| Need | Control |
|------|---------|
| Single action | Button |
| On/off state | Toggle |
| Select from few | Picker (.segmented) |
| Select from many | Picker (.menu/.navigationLink) |
| Continuous value | Slider |
| Discrete increment | Stepper |
| Text input | TextField |
| Multi-line text | TextEditor |
| Date selection | DatePicker |
| Color selection | ColorPicker |
| Photo selection | PhotosPicker |
| Additional options | Menu |
| Contextual actions | contextMenu |
| Quick row actions | swipeActions |
| Loading | ProgressView |

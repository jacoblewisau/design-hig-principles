# Feedback & Status Reference

Complete guide to communicating system state, progress, and feedback to users.

---

## Feedback Philosophy

**Apple's feedback principles:**
- Acknowledge every user action
- Keep users informed of system state
- Provide immediate feedback, even if action is pending
- Never leave users wondering "did that work?"

---

## Immediate Feedback

### Visual Feedback

```swift
// Button press state
Button { } label: {
    Text("Submit")
}
.buttonStyle(.borderedProminent)
// System automatically provides press state

// Custom press feedback
@GestureState private var isPressed = false

Button { } label: {
    Text("Custom")
        .scaleEffect(isPressed ? 0.95 : 1.0)
        .opacity(isPressed ? 0.8 : 1.0)
}
.simultaneousGesture(
    DragGesture(minimumDistance: 0)
        .updating($isPressed) { _, state, _ in
            state = true
        }
)
```

### Haptic Feedback

```swift
// Impact feedback
let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()

// Selection feedback
let selection = UISelectionFeedbackGenerator()
selection.selectionChanged()

// Notification feedback
let notification = UINotificationFeedbackGenerator()
notification.notificationOccurred(.success)
notification.notificationOccurred(.warning)
notification.notificationOccurred(.error)
```

### Sound Feedback

```swift
import AudioToolbox

// System sounds
AudioServicesPlaySystemSound(1104) // Camera shutter
AudioServicesPlaySystemSound(1057) // Send message
AudioServicesPlaySystemSound(1016) // Tweet

// Custom sound
AudioServicesPlaySystemSound(soundID)
```

---

## Progress Indicators

### Indeterminate Progress (Unknown Duration)

```swift
// Spinner
ProgressView()

// With label
ProgressView("Loading...")

// Custom styling
ProgressView()
    .progressViewStyle(.circular)
    .tint(.blue)
```

### Determinate Progress (Known Duration)

```swift
// Linear progress bar
ProgressView(value: progress, total: 100)

// With label
ProgressView(value: downloadProgress) {
    Text("Downloading")
} currentValueLabel: {
    Text("\(Int(downloadProgress * 100))%")
}

// Circular gauge
Gauge(value: progress, in: 0...100) {
    Text("Progress")
} currentValueLabel: {
    Text("\(Int(progress))%")
}
.gaugeStyle(.accessoryCircularCapacity)
```

### Progress Best Practices

| Duration | Indicator | Notes |
|----------|-----------|-------|
| < 1 second | None or subtle | Don't flash indicator |
| 1-3 seconds | Indeterminate spinner | User knows something is happening |
| > 3 seconds | Determinate if possible | Show actual progress |
| > 10 seconds | Progress + estimate | Consider background processing |

---

## Loading States

### Skeleton Views (Placeholder)

```swift
// Placeholder while loading
VStack(alignment: .leading, spacing: 12) {
    // Title placeholder
    RoundedRectangle(cornerRadius: 4)
        .fill(Color(.systemGray5))
        .frame(width: 200, height: 20)

    // Body placeholder
    RoundedRectangle(cornerRadius: 4)
        .fill(Color(.systemGray5))
        .frame(height: 16)

    RoundedRectangle(cornerRadius: 4)
        .fill(Color(.systemGray5))
        .frame(width: 280, height: 16)
}
.redacted(reason: .placeholder)
```

### Shimmer Effect

```swift
struct ShimmerModifier: ViewModifier {
    @State private var phase: CGFloat = 0

    func body(content: Content) -> some View {
        content
            .overlay(
                LinearGradient(
                    colors: [.clear, .white.opacity(0.3), .clear],
                    startPoint: .leading,
                    endPoint: .trailing
                )
                .offset(x: phase)
            )
            .onAppear {
                withAnimation(.linear(duration: 1.5).repeatForever(autoreverses: false)) {
                    phase = 200
                }
            }
    }
}
```

### Pull to Refresh

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
}
.refreshable {
    await loadData()
}
```

---

## Success States

### Inline Success

```swift
// Checkmark animation
@State private var showSuccess = false

ZStack {
    Circle()
        .fill(Color.green)
        .frame(width: 60, height: 60)

    Image(systemName: "checkmark")
        .font(.title.bold())
        .foregroundStyle(.white)
        .scaleEffect(showSuccess ? 1 : 0)
}
.animation(.spring(response: 0.3, dampingFraction: 0.6), value: showSuccess)
```

### Toast/Banner

```swift
struct SuccessBanner: View {
    let message: String

    var body: some View {
        HStack {
            Image(systemName: "checkmark.circle.fill")
                .foregroundStyle(.green)
            Text(message)
        }
        .padding()
        .background(Color(.secondarySystemBackground))
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .shadow(radius: 4)
    }
}
```

### Button State Transition

```swift
enum ButtonState {
    case idle, loading, success
}

@State private var state: ButtonState = .idle

Button {
    state = .loading
    Task {
        await performAction()
        state = .success
        try? await Task.sleep(for: .seconds(2))
        state = .idle
    }
} label: {
    Group {
        switch state {
        case .idle:
            Text("Submit")
        case .loading:
            ProgressView()
        case .success:
            Image(systemName: "checkmark")
        }
    }
    .frame(width: 80)
}
.disabled(state != .idle)
```

---

## Error States

### Inline Error

```swift
VStack(alignment: .leading, spacing: 4) {
    TextField("Email", text: $email)
        .textFieldStyle(.roundedBorder)
        .overlay(
            RoundedRectangle(cornerRadius: 8)
                .stroke(hasError ? Color.red : Color.clear, lineWidth: 1)
        )

    if hasError {
        Label("Please enter a valid email", systemImage: "exclamationmark.circle")
            .font(.caption)
            .foregroundStyle(.red)
    }
}
```

### Error Alert

```swift
.alert("Error", isPresented: $showError, presenting: error) { _ in
    Button("OK") { }
    Button("Retry") { retry() }
} message: { error in
    Text(error.localizedDescription)
}
```

### Error Banner

```swift
struct ErrorBanner: View {
    let error: Error
    let retry: () -> Void

    var body: some View {
        HStack {
            Image(systemName: "exclamationmark.triangle.fill")
                .foregroundStyle(.orange)

            VStack(alignment: .leading) {
                Text("Something went wrong")
                    .font(.headline)
                Text(error.localizedDescription)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()

            Button("Retry", action: retry)
                .buttonStyle(.bordered)
        }
        .padding()
        .background(Color(.secondarySystemBackground))
    }
}
```

---

## Empty States

### ContentUnavailableView (iOS 17+)

```swift
// Standard empty state
ContentUnavailableView(
    "No Items",
    systemImage: "tray",
    description: Text("Items you add will appear here.")
)

// With action
ContentUnavailableView {
    Label("No Documents", systemImage: "doc")
} description: {
    Text("Create a document to get started.")
} actions: {
    Button("New Document") { createDocument() }
        .buttonStyle(.borderedProminent)
}

// Search empty state
ContentUnavailableView.search(text: searchQuery)
```

### Custom Empty State

```swift
struct EmptyStateView: View {
    let title: String
    let message: String
    let systemImage: String
    let action: (() -> Void)?
    let actionTitle: String?

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: systemImage)
                .font(.system(size: 56))
                .foregroundStyle(.tertiary)

            Text(title)
                .font(.title2.bold())

            Text(message)
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            if let action, let actionTitle {
                Button(actionTitle, action: action)
                    .buttonStyle(.borderedProminent)
                    .padding(.top, 8)
            }
        }
        .padding(32)
    }
}
```

### Empty State Guidelines

| State | Show | Don't Show |
|-------|------|------------|
| Empty list | Helpful illustration + action | Just whitespace |
| No search results | "No results for X" | Generic error |
| No permissions | How to enable + action | Just "Denied" |
| Offline | Status + retry option | Nothing |

---

## Status Indicators

### Badge

```swift
// Tab bar badge
.tabItem { Label("Inbox", systemImage: "envelope") }
.badge(unreadCount)

// Custom badge
Image(systemName: "bell")
    .overlay(alignment: .topTrailing) {
        if unreadCount > 0 {
            Text("\(unreadCount)")
                .font(.caption2.bold())
                .foregroundStyle(.white)
                .padding(4)
                .background(Color.red)
                .clipShape(Circle())
                .offset(x: 8, y: -8)
        }
    }
```

### Status Dot

```swift
HStack {
    Circle()
        .fill(status.color)
        .frame(width: 8, height: 8)
    Text(status.title)
}

// With accessibility
HStack {
    Circle()
        .fill(isOnline ? .green : .gray)
        .frame(width: 8, height: 8)
    Text(isOnline ? "Online" : "Offline")
}
.accessibilityElement(children: .combine)
.accessibilityLabel(isOnline ? "Status: Online" : "Status: Offline")
```

### Connection Status

```swift
struct ConnectionStatusView: View {
    let isConnected: Bool

    var body: some View {
        HStack(spacing: 6) {
            Image(systemName: isConnected ? "wifi" : "wifi.slash")
            Text(isConnected ? "Connected" : "Offline")
        }
        .font(.caption)
        .foregroundStyle(isConnected ? .green : .orange)
        .padding(.horizontal, 12)
        .padding(.vertical, 6)
        .background(
            Capsule()
                .fill(Color(.systemGray6))
        )
    }
}
```

---

## Sync Status

```swift
enum SyncStatus {
    case synced
    case syncing
    case pending
    case error
}

struct SyncStatusView: View {
    let status: SyncStatus

    var body: some View {
        HStack(spacing: 4) {
            switch status {
            case .synced:
                Image(systemName: "checkmark.icloud")
                    .foregroundStyle(.green)
            case .syncing:
                ProgressView()
                    .scaleEffect(0.8)
            case .pending:
                Image(systemName: "icloud")
                    .foregroundStyle(.secondary)
            case .error:
                Image(systemName: "exclamationmark.icloud")
                    .foregroundStyle(.red)
            }

            Text(status.description)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
    }
}
```

---

## Real-Time Updates

### Live Activity Indicator

```swift
// Typing indicator
HStack(spacing: 4) {
    ForEach(0..<3) { index in
        Circle()
            .fill(Color.secondary)
            .frame(width: 6, height: 6)
            .offset(y: isAnimating ? -4 : 0)
            .animation(
                .easeInOut(duration: 0.4)
                .repeatForever()
                .delay(Double(index) * 0.15),
                value: isAnimating
            )
    }
}
```

### Update Banner

```swift
// "New content available" banner
struct UpdateBanner: View {
    let onRefresh: () -> Void

    var body: some View {
        Button(action: onRefresh) {
            HStack {
                Image(systemName: "arrow.up.circle.fill")
                Text("New posts available")
            }
            .font(.subheadline.bold())
            .foregroundStyle(.white)
            .padding(.horizontal, 16)
            .padding(.vertical, 8)
            .background(Color.accentColor)
            .clipShape(Capsule())
        }
    }
}
```

---

## Accessibility for Feedback

### Announcing Status Changes

```swift
// Announce to VoiceOver
UIAccessibility.post(
    notification: .announcement,
    argument: "Upload complete"
)

// Announce layout change
UIAccessibility.post(notification: .layoutChanged, argument: nil)

// Announce screen change
UIAccessibility.post(notification: .screenChanged, argument: element)
```

### Status Accessibility

```swift
ProgressView(value: progress)
    .accessibilityLabel("Download progress")
    .accessibilityValue("\(Int(progress * 100)) percent")

StatusDot(isOnline: isOnline)
    .accessibilityLabel(isOnline ? "Online" : "Offline")
```

---

## Multi-Modal Feedback

### Combining Feedback Types

```swift
func showSuccess() {
    // Visual
    withAnimation { showSuccessIndicator = true }

    // Haptic
    UINotificationFeedbackGenerator().notificationOccurred(.success)

    // Audio (optional)
    AudioServicesPlaySystemSound(1001)

    // Accessibility announcement
    UIAccessibility.post(notification: .announcement, argument: "Action completed")
}
```

### Feedback Intensity Guidelines

| Action | Visual | Haptic | Sound |
|--------|--------|--------|-------|
| Button tap | Highlight | Light impact | None |
| Toggle | State change | Selection | None |
| Success | Checkmark | Success | Optional |
| Error | Red indicator | Error | Optional |
| Delete | Remove animation | Heavy impact | None |
| Pull to refresh | Spinner | None | None |

---

## Anti-Patterns

```swift
// ❌ No feedback on action
Button("Submit") { performAction() }

// ✅ Clear feedback
Button("Submit") {
    isLoading = true
    performAction()
}
.disabled(isLoading)
.overlay(isLoading ? ProgressView() : nil)

// ❌ Flashing progress for instant action
// 100ms spinner is distracting

// ✅ Only show progress after threshold
.task {
    try? await Task.sleep(for: .milliseconds(150))
    if !isComplete { showProgress = true }
}

// ❌ Error without recovery
Text("Error")

// ✅ Error with action
ContentUnavailableView("Failed to Load") {
    Button("Try Again") { retry() }
}

// ❌ Color-only status
Circle().fill(status == .success ? .green : .red)

// ✅ Multiple indicators
HStack {
    Image(systemName: status.icon)
    Text(status.title)
}
.foregroundStyle(status.color)
```

---

## Quick Reference

| Feedback Type | When | Implementation |
|---------------|------|----------------|
| Press highlight | Every tap | System button styles |
| Haptic | Meaningful actions | UIFeedbackGenerator |
| Progress | > 1 second | ProgressView |
| Skeleton | Initial load | .redacted(reason:) |
| Success | Completed action | Checkmark + haptic |
| Error | Failed action | Banner + retry |
| Empty | No content | ContentUnavailableView |
| Badge | Unread count | .badge() |
| Status dot | Live state | Circle + text |

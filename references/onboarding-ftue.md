# Onboarding & First-Time User Experience Reference

Complete guide to onboarding flows, feature discovery, permission requests, and FTUE design.

---

## Onboarding Philosophy

**Apple's guidance:** "Get people into the experience quickly. Avoid lengthy setup sequences."

**Key principles:**
- Let users explore immediately
- Teach through doing, not reading
- Request permissions in context
- Defer non-essential setup
- Remember returning users

---

## When to Use Onboarding

### Required Onboarding

- Account creation/login (if essential)
- Legal agreements (GDPR, terms)
- Critical setup (selecting data source)

### Optional Onboarding

- Feature highlights
- Tips and guidance
- Premium feature promotion

### Skip Onboarding When

- App is self-explanatory
- Users can figure it out
- Standard patterns are used

---

## Onboarding Patterns

### Welcome Screens (Use Sparingly)

```swift
struct WelcomeView: View {
    @Binding var showOnboarding: Bool

    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "sparkles")
                .font(.system(size: 64))
                .foregroundStyle(.accent)

            Text("Welcome to App Name")
                .font(.largeTitle.bold())

            Text("Brief value proposition in one sentence.")
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            Spacer()

            Button("Get Started") {
                showOnboarding = false
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
        }
        .padding(32)
    }
}
```

### Feature Carousel

```swift
struct OnboardingCarousel: View {
    @State private var currentPage = 0
    @Binding var showOnboarding: Bool

    let pages = [
        OnboardingPage(title: "Feature One", description: "...", image: "1.circle"),
        OnboardingPage(title: "Feature Two", description: "...", image: "2.circle"),
        OnboardingPage(title: "Feature Three", description: "...", image: "3.circle"),
    ]

    var body: some View {
        VStack {
            TabView(selection: $currentPage) {
                ForEach(Array(pages.enumerated()), id: \.offset) { index, page in
                    PageView(page: page)
                        .tag(index)
                }
            }
            .tabViewStyle(.page)

            // Page indicator is built-in with .page style

            Button(currentPage == pages.count - 1 ? "Get Started" : "Next") {
                if currentPage < pages.count - 1 {
                    withAnimation { currentPage += 1 }
                } else {
                    showOnboarding = false
                }
            }
            .buttonStyle(.borderedProminent)
            .padding(.bottom, 32)
        }
    }
}
```

### Progressive Disclosure

```swift
// Reveal features as user progresses
struct ProgressiveOnboarding: View {
    @AppStorage("hasCompletedTask1") var task1 = false
    @AppStorage("hasCompletedTask2") var task2 = false

    var body: some View {
        List {
            if !task1 {
                Section("Getting Started") {
                    TipRow(
                        title: "Create your first item",
                        action: "Tap + to add an item"
                    )
                }
            }

            if task1 && !task2 {
                Section("Next Steps") {
                    TipRow(
                        title: "Organize with folders",
                        action: "Long press to move items"
                    )
                }
            }

            // Regular content
            mainContent
        }
    }
}
```

---

## Permission Requests

### Pre-Permission Explanation

```swift
struct LocationPermissionView: View {
    @State private var showSystemPrompt = false

    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "location.fill")
                .font(.system(size: 48))
                .foregroundStyle(.blue)

            Text("Find Nearby Places")
                .font(.title2.bold())

            Text("We use your location to show restaurants, shops, and attractions near you.")
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            Spacer()

            Button("Enable Location") {
                showSystemPrompt = true
            }
            .buttonStyle(.borderedProminent)

            Button("Not Now") {
                // Skip, continue without
            }
            .buttonStyle(.borderless)
        }
        .padding(32)
        .task(id: showSystemPrompt) {
            if showSystemPrompt {
                requestLocationPermission()
            }
        }
    }
}
```

### Contextual Permission Requests

```swift
// ❌ Request all permissions at launch
func applicationDidFinishLaunching() {
    requestPhotos()
    requestCamera()
    requestLocation()
    requestNotifications()
}

// ✅ Request in context
func userTappedAddPhoto() {
    if photoPermission == .notDetermined {
        showPhotoExplanation = true
    } else {
        showPhotoPicker()
    }
}

func userEnabledReminders() {
    if notificationPermission == .notDetermined {
        requestNotifications()
    }
    scheduleReminder()
}
```

### Permission States

```swift
enum PermissionState {
    case notDetermined
    case denied
    case authorized
    case restricted
}

struct PermissionHandler: View {
    let permission: PermissionState

    var body: some View {
        switch permission {
        case .notDetermined:
            RequestPermissionView()
        case .denied:
            DeniedView(openSettings: openSettings)
        case .authorized:
            FeatureView()
        case .restricted:
            RestrictedView()
        }
    }
}
```

### Settings Deep Link for Denied Permissions

```swift
struct PermissionDeniedView: View {
    var body: some View {
        ContentUnavailableView {
            Label("Camera Access Required", systemImage: "camera")
        } description: {
            Text("Allow camera access in Settings to scan QR codes.")
        } actions: {
            Button("Open Settings") {
                if let url = URL(string: UIApplication.openSettingsURLString) {
                    UIApplication.shared.open(url)
                }
            }
        }
    }
}
```

---

## Feature Discovery

### TipKit (iOS 17+)

```swift
import TipKit

struct FavoriteTip: Tip {
    var title: Text {
        Text("Save Your Favorites")
    }

    var message: Text? {
        Text("Tap the heart to save items for quick access.")
    }

    var image: Image? {
        Image(systemName: "heart")
    }
}

struct ItemView: View {
    let tip = FavoriteTip()

    var body: some View {
        VStack {
            content

            Button("Favorite") { }
                .popoverTip(tip)
        }
    }
}

// Configure TipKit at app launch
@main
struct MyApp: App {
    init() {
        try? Tips.configure()
    }
}
```

### Inline Tips

```swift
TipView(tip)

// Dismiss after action
Button("Got it") {
    tip.invalidate(reason: .actionPerformed)
}
```

### Coach Marks (Custom)

```swift
struct CoachMark: View {
    let message: String
    let arrowEdge: Edge

    var body: some View {
        VStack(spacing: 8) {
            if arrowEdge == .top {
                Triangle()
                    .fill(Color(.secondarySystemBackground))
                    .frame(width: 20, height: 10)
            }

            Text(message)
                .font(.callout)
                .padding()
                .background(Color(.secondarySystemBackground))
                .clipShape(RoundedRectangle(cornerRadius: 12))

            if arrowEdge == .bottom {
                Triangle()
                    .fill(Color(.secondarySystemBackground))
                    .frame(width: 20, height: 10)
                    .rotationEffect(.degrees(180))
            }
        }
    }
}
```

---

## Account Creation

### Minimizing Friction

```swift
// ❌ Full form upfront
Form {
    TextField("Name", text: $name)
    TextField("Email", text: $email)
    SecureField("Password", text: $password)
    SecureField("Confirm Password", text: $confirmPassword)
    DatePicker("Birthday", selection: $birthday)
    Picker("Gender", selection: $gender) { }
    TextField("Phone", text: $phone)
}

// ✅ Progressive collection
// Step 1: Just email/Apple Sign In
// Step 2: Collect more as needed
VStack {
    SignInWithAppleButton { }

    Divider()

    TextField("Email", text: $email)
    Button("Continue") { }

    Button("Skip for Now") { }
}
```

### Sign in with Apple

```swift
import AuthenticationServices

SignInWithAppleButton(.signIn) { request in
    request.requestedScopes = [.email, .fullName]
} onCompletion: { result in
    switch result {
    case .success(let auth):
        handleAuth(auth)
    case .failure(let error):
        handleError(error)
    }
}
.signInWithAppleButtonStyle(.black)
.frame(height: 50)
```

---

## What's New Screens

For app updates:

```swift
struct WhatsNewView: View {
    @AppStorage("lastSeenVersion") var lastVersion = ""
    @State private var showWhatsNew = false

    var body: some View {
        content
            .sheet(isPresented: $showWhatsNew) {
                WhatsNewSheet(version: currentVersion)
            }
            .onAppear {
                if lastVersion != currentVersion {
                    showWhatsNew = true
                    lastVersion = currentVersion
                }
            }
    }
}

struct WhatsNewSheet: View {
    let features = [
        Feature(icon: "sparkles", title: "New Feature", description: "..."),
        Feature(icon: "bolt", title: "Improved Performance", description: "..."),
    ]

    var body: some View {
        NavigationStack {
            List(features) { feature in
                Label {
                    VStack(alignment: .leading) {
                        Text(feature.title).font(.headline)
                        Text(feature.description).font(.caption).foregroundStyle(.secondary)
                    }
                } icon: {
                    Image(systemName: feature.icon)
                }
            }
            .navigationTitle("What's New")
            .toolbar {
                Button("Done") { dismiss() }
            }
        }
    }
}
```

---

## Empty State as Onboarding

```swift
struct EmptyStateOnboarding: View {
    var body: some View {
        ContentUnavailableView {
            Label("No Projects Yet", systemImage: "folder")
        } description: {
            Text("Create your first project to get started.")
        } actions: {
            Button("Create Project") { createProject() }
                .buttonStyle(.borderedProminent)

            Button("Watch Tutorial") { showTutorial() }
                .buttonStyle(.bordered)
        }
    }
}
```

---

## Tracking Onboarding State

```swift
class OnboardingManager: ObservableObject {
    @AppStorage("hasCompletedOnboarding") var completed = false
    @AppStorage("onboardingStep") var currentStep = 0
    @AppStorage("permissionsRequested") var permissionsRequested: [String] = []

    func markComplete() {
        completed = true
    }

    func shouldShowOnboarding() -> Bool {
        !completed
    }

    func shouldRequestPermission(_ permission: String) -> Bool {
        !permissionsRequested.contains(permission)
    }
}
```

---

## Anti-Patterns

```swift
// ❌ Lengthy onboarding
TabView {
    Page1() // "Welcome!"
    Page2() // "Feature 1"
    Page3() // "Feature 2"
    Page4() // "Feature 3"
    Page5() // "Feature 4"
    Page6() // "Create Account"
    Page7() // "Notifications?"
    Page8() // "Location?"
}

// ✅ Minimal onboarding
if !hasOnboarded {
    WelcomeView() // Single screen, optional
}

// ❌ All permissions at launch
func onLaunch() {
    requestAllPermissions()
}

// ✅ Contextual permission requests
func onFeatureUse() {
    requestRelevantPermission()
}

// ❌ Blocking content behind account
if !isLoggedIn {
    LoginRequired()
} else {
    Content()
}

// ✅ Allow exploration first
Content()
    .sheet(isPresented: $needsLogin) {
        LoginSheet()
    }

// ❌ Forced tutorial
TutorialOverlay(canDismiss: false)

// ✅ Optional, skippable help
HelpOverlay()
    .overlay(alignment: .topTrailing) {
        Button("Skip") { dismiss() }
    }
```

---

## Checklist

### Before Launching Onboarding

- [ ] Is onboarding necessary?
- [ ] Can users skip it?
- [ ] Is it 3 screens or fewer?
- [ ] Are permissions contextual?
- [ ] Does empty state guide users?
- [ ] Are tips dismissible?
- [ ] Is account creation deferred?
- [ ] Does it work for returning users?

---

## Quick Reference

| Pattern | When to Use |
|---------|-------------|
| Welcome screen | Brand introduction (optional) |
| Feature carousel | Complex apps (max 3-4 pages) |
| Progressive tips | Teach through doing |
| Empty state guidance | First content creation |
| TipKit | Feature discovery |
| Pre-permission screen | Before system prompt |
| What's New | After updates |

| Permission | Request When |
|------------|--------------|
| Location | User taps map/nearby |
| Camera | User taps camera button |
| Photos | User tries to add photo |
| Notifications | User enables reminders |
| Contacts | User uses contact feature |
| Microphone | User taps record |

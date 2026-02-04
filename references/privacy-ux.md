# Privacy UX Reference

Complete guide to privacy-conscious design, permission requests, and Apple's privacy requirements.

---

## Privacy Philosophy

**Apple's guidance:** Privacy is a fundamental human right. Design with privacy in mind from the start.

**Key principles:**
- Request only what you need
- Be transparent about data use
- Give users control
- Minimize data collection
- Secure stored data

---

## Permission Patterns

### Request In Context

```swift
// ❌ Request all permissions at launch
func applicationDidFinishLaunching() {
    requestCamera()
    requestPhotos()
    requestLocation()
    requestNotifications()
}

// ✅ Request when needed
func userTappedCamera() {
    AVCaptureDevice.requestAccess(for: .video) { granted in
        // ...
    }
}
```

### Pre-Permission Explanation

Before the system prompt:

```swift
struct CameraPermissionView: View {
    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "camera.fill")
                .font(.system(size: 48))
                .foregroundStyle(.blue)

            Text("Camera Access")
                .font(.title2.bold())

            Text("We need camera access to scan documents. Your photos are processed on-device and never uploaded.")
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            Button("Enable Camera") {
                requestCameraPermission()
            }
            .buttonStyle(.borderedProminent)

            Button("Not Now") {
                dismiss()
            }
            .buttonStyle(.borderless)
        }
        .padding(32)
    }
}
```

### Permission Request Timing

| Permission | When to Request |
|------------|-----------------|
| Camera | User taps camera button |
| Photos | User tries to add photo |
| Location | User needs location feature |
| Notifications | User enables reminders |
| Contacts | User uses contacts feature |
| Microphone | User taps record |
| Health | User connects health feature |
| Tracking | After demonstrating value |

---

## Privacy Manifest (iOS 17+)

### Required APIs

Apps using certain APIs must declare their purpose in PrivacyInfo.xcprivacy:

```xml
<!-- PrivacyInfo.xcprivacy -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

### APIs Requiring Declaration

- UserDefaults
- File timestamp APIs
- System boot time APIs
- Disk space APIs
- Active keyboard APIs

---

## App Tracking Transparency

### When Required

Required when:
- Linking user data with third-party data for advertising
- Sharing user data with data brokers
- Using IDFA for tracking

### Implementation

```swift
import AppTrackingTransparency

func requestTrackingPermission() {
    ATTrackingManager.requestTrackingAuthorization { status in
        switch status {
        case .authorized:
            // Can use IDFA
            break
        case .denied, .restricted, .notDetermined:
            // Cannot track
            break
        @unknown default:
            break
        }
    }
}
```

### UI Best Practices

```swift
// Show value proposition BEFORE system prompt
struct TrackingExplanationView: View {
    var body: some View {
        VStack(spacing: 20) {
            Text("Personalized Experience")
                .font(.title2.bold())

            Text("Allow tracking to see ads relevant to your interests. Your data is never sold.")
                .multilineTextAlignment(.center)
                .foregroundStyle(.secondary)

            Button("Continue") {
                requestTrackingPermission()
            }

            Button("Skip") {
                // Continue without tracking
            }
        }
    }
}
```

---

## Data Handling Transparency

### App Privacy Labels

Required for App Store:

**Categories:**
- Data Used to Track You
- Data Linked to You
- Data Not Linked to You

### Displaying Privacy Info

```swift
// Link to privacy policy
Link("Privacy Policy", destination: URL(string: "https://example.com/privacy")!)

// In settings
Section("Privacy") {
    NavigationLink("Privacy Policy") {
        PrivacyPolicyView()
    }
    NavigationLink("Data & Privacy") {
        DataPrivacyView()
    }
}
```

---

## Handling Permission Denial

### Graceful Degradation

```swift
enum CameraPermission {
    case authorized
    case denied
    case notDetermined
}

@ViewBuilder
func cameraView(permission: CameraPermission) -> some View {
    switch permission {
    case .authorized:
        CameraPreview()
    case .denied:
        ContentUnavailableView {
            Label("Camera Access Required", systemImage: "camera")
        } description: {
            Text("Enable camera access in Settings to scan documents.")
        } actions: {
            Button("Open Settings") {
                openSettings()
            }
        }
    case .notDetermined:
        CameraPermissionRequestView()
    }
}
```

### Deep Link to Settings

```swift
func openSettings() {
    if let url = URL(string: UIApplication.openSettingsURLString) {
        UIApplication.shared.open(url)
    }
}
```

---

## On-Device Processing

**Apple's preference:** Process data on-device when possible.

### Communicating On-Device Processing

```swift
// Reassure users
HStack {
    Image(systemName: "lock.shield")
    Text("Processed on your device")
        .font(.caption)
        .foregroundStyle(.secondary)
}

// In feature explanation
Text("Your photos are analyzed on-device and never leave your phone.")
```

### Features That Should Be On-Device

| Feature | On-Device Option |
|---------|------------------|
| Photo analysis | Vision framework |
| Text recognition | Live Text |
| Translation | Translate framework |
| Speech recognition | Speech framework |
| Face detection | Vision framework |
| ML inference | Core ML |

---

## Secure Data Storage

### Keychain for Sensitive Data

```swift
// ✅ Store credentials in Keychain
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "userToken",
    kSecValueData as String: tokenData
]
SecItemAdd(query as CFDictionary, nil)

// ❌ Don't store in UserDefaults
UserDefaults.standard.set(token, forKey: "authToken")
```

### File Protection

```swift
// Protect files at rest
try data.write(to: url, options: .completeFileProtection)
```

---

## Location Privacy

### Precision Control

```swift
// Request appropriate precision
let manager = CLLocationManager()

// Only when user chooses "While Using"
manager.requestWhenInUseAuthorization()

// Reduced accuracy when precise isn't needed
manager.desiredAccuracy = kCLLocationAccuracyReduced
```

### Temporary Precise Location

```swift
// For one-time precise location
manager.requestTemporaryFullAccuracyAuthorization(
    withPurposeKey: "FindNearbyStores"
) { error in
    // ...
}
```

---

## Photo Library Access

### Limited Library

```swift
import PhotosUI

// PHPicker doesn't require permission
PhotosPicker(selection: $selectedItem) {
    Label("Select Photo", systemImage: "photo")
}
// User picks from their library
// App only accesses selected photos
```

### When Full Access is Needed

```swift
// Only request if actually needed
PHPhotoLibrary.requestAuthorization(for: .readWrite) { status in
    // .limited - User granted access to select photos
    // .authorized - Full access
    // .denied - No access
}
```

---

## Minimizing Data Collection

### Collect Only What's Needed

```swift
// ❌ Over-collection
struct SignUpForm {
    var email: String
    var password: String
    var fullName: String
    var birthday: Date
    var gender: String
    var phoneNumber: String
    var address: String
}

// ✅ Minimal collection
struct SignUpForm {
    var email: String
    var password: String
}
// Collect additional info only when needed
```

### Anonymous Usage Analytics

```swift
// ✅ Aggregated, anonymized analytics
Analytics.track("feature_used", properties: [
    "feature": "export"
    // No user ID, no personal data
])

// ❌ Personally identifiable analytics
Analytics.track("feature_used", properties: [
    "user_id": userId,
    "email": email,
    "feature": "export"
])
```

---

## Privacy Nutrition Labels

### Data Types

**Identifiers:**
- User ID
- Device ID
- Advertising identifier

**Location:**
- Precise Location
- Coarse Location

**Contact Info:**
- Name
- Email Address
- Phone Number
- Physical Address

**User Content:**
- Photos or Videos
- Audio Data
- Gameplay Content
- Customer Support
- Other User Content

**Browsing History**

**Search History**

**Purchases**

**Usage Data:**
- Product Interaction
- Advertising Data
- Other Usage Data

**Diagnostics:**
- Crash Data
- Performance Data
- Other Diagnostic Data

---

## Anti-Patterns

```swift
// ❌ Request permissions at launch
func applicationDidFinishLaunching() {
    requestAllPermissions()
}

// ✅ Request in context when needed

// ❌ No explanation for why you need data
AVCaptureDevice.requestAccess(for: .video) { _ in }

// ✅ Explain first
showCameraExplanation()
// Then request

// ❌ Store sensitive data insecurely
UserDefaults.standard.set(apiKey, forKey: "key")

// ✅ Use Keychain
storeInKeychain(apiKey)

// ❌ Request full photo library access when limited works
PHPhotoLibrary.requestAuthorization(for: .readWrite)

// ✅ Use PHPicker (no permission needed)
PhotosPicker(selection: $item) { }

// ❌ Request precise location for rough needs
manager.desiredAccuracy = kCLLocationAccuracyBest

// ✅ Request appropriate accuracy
manager.desiredAccuracy = kCLLocationAccuracyKilometer
```

---

## Checklist

### Before Submission

- [ ] Privacy Manifest complete (iOS 17+)
- [ ] App Privacy nutrition labels accurate
- [ ] Permissions requested in context
- [ ] Pre-permission screens explain usage
- [ ] Denied state handled gracefully
- [ ] Settings deep link provided
- [ ] Sensitive data in Keychain
- [ ] On-device processing where possible
- [ ] Privacy policy linked in app
- [ ] Minimal data collection

---

## Quick Reference

| Data Type | Storage |
|-----------|---------|
| Credentials | Keychain |
| API keys | Keychain |
| User preferences | UserDefaults |
| Sensitive files | .completeFileProtection |
| Cache | Caches directory |

| Permission | Best Practice |
|------------|---------------|
| Camera | Request when camera UI shown |
| Location | Request when map shown |
| Photos | Use PHPicker (no permission) |
| Notifications | Request after value shown |
| Tracking | Request last, explain value |

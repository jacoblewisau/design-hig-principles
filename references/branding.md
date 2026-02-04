# Branding Reference

Complete guide to appropriate branding, launch screens, and maintaining platform citizenship while expressing brand identity.

---

## Branding Philosophy

**Apple's guidance:** "Integrate brand content tastefully."

**Key principles:**
- Content should be the hero, not branding
- Use restraint—subtle branding is more sophisticated
- Express brand through quality, not logos everywhere
- Follow platform conventions even with custom styling

---

## Brand Expression Hierarchy

### 1. Accent Color (Primary)

The most appropriate way to express brand:

```swift
// App-wide tint
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .tint(Color("BrandAccent"))
        }
    }
}

// Define in Asset Catalog with Light/Dark variants
Color("BrandAccent")
```

### 2. Typography (Secondary)

Custom fonts as brand expression:

```swift
// Custom font with Dynamic Type support
Text("Headline")
    .font(.custom("BrandFont-Bold", size: 24, relativeTo: .title))

// Register fonts in Info.plist
// "Fonts provided by application": ["BrandFont-Regular.ttf", ...]
```

### 3. Writing Style (Tertiary)

Brand voice in copy:

```swift
// Playful brand voice
Text("Oops! Something went sideways.")

// Professional brand voice
Text("An error occurred. Please try again.")

// Friendly brand voice
Text("Hey! We couldn't load that. Give it another go?")
```

### 4. Logo Placement (Minimal)

Use logos sparingly:

```swift
// ✅ Appropriate places for logo
// - Onboarding flow
// - About screen
// - Sign-in screen
// - Empty states (sometimes)

// ❌ Avoid logos in
// - Navigation bars
// - Tab bars
// - Every screen
// - Launch screen
```

---

## Launch Screens

### Requirements

Apple's guidelines state launch screens should:
- Match the first screen of your app
- Contain no text
- Contain no logos
- Not be used for branding/splash

### Implementation

```swift
// LaunchScreen.storyboard
// - Match your app's background color
// - Keep it simple (system background)
// - No images, no text

// Or UILaunchScreen in Info.plist
/*
<key>UILaunchScreen</key>
<dict>
    <key>UIColorName</key>
    <string>LaunchBackground</string>
</dict>
*/
```

### What Launch Screen IS For

- Instant perceived launch
- Smooth transition to app content
- Making app feel faster

### What Launch Screen IS NOT For

- Branding
- Advertising
- Showing terms/legal
- Loading indicators

---

## App Icon

### Design Guidelines

| Attribute | Guideline |
|-----------|-----------|
| Shape | System applies mask (no transparency needed) |
| Simplicity | Recognizable at small sizes |
| Consistency | Match app experience |
| Background | Solid or simple gradient |
| Content | No text, no photos |

### Sizes Required

```
// iOS
1024x1024 (App Store)
180x180 (@3x)
120x120 (@2x)
60x60 (@1x)

// iPadOS
167x167 (iPad Pro @2x)
152x152 (iPad @2x)
76x76 (iPad @1x)

// Additional
40x40, 58x58, 80x80, 87x87 (Settings, Spotlight)
```

### Icon Best Practices

```swift
// No transparency (system applies rounded rect mask)
// No included rounded corners
// Single focal point
// Works at 29x29 in Settings
```

---

## Navigation Bar Branding

### Don't Do This

```swift
// ❌ Logo in navigation bar
.toolbar {
    ToolbarItem(placement: .principal) {
        Image("logo")
            .resizable()
            .frame(width: 100, height: 30)
    }
}
```

### Do This Instead

```swift
// ✅ Standard navigation title
.navigationTitle("Home")

// ✅ Or use tint color for brand
.tint(Color("BrandAccent"))
```

---

## Onboarding Branding

Onboarding is appropriate for brand expression:

```swift
struct OnboardingView: View {
    var body: some View {
        VStack(spacing: 32) {
            // ✅ Logo appropriate here
            Image("AppLogo")
                .resizable()
                .frame(width: 80, height: 80)

            Text("Welcome to AppName")
                .font(.custom("BrandFont-Bold", size: 28, relativeTo: .title))

            Text("A brief brand-voice description of value.")
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            Spacer()

            Button("Get Started") { }
                .buttonStyle(.borderedProminent)
        }
        .padding(32)
    }
}
```

---

## About Screen

The about screen is prime real estate for branding:

```swift
struct AboutView: View {
    var body: some View {
        List {
            Section {
                VStack(spacing: 16) {
                    Image("AppLogo")
                        .resizable()
                        .frame(width: 80, height: 80)
                        .clipShape(RoundedRectangle(cornerRadius: 18, style: .continuous))

                    Text("AppName")
                        .font(.title2.bold())

                    Text("Version 1.0 (42)")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
                .frame(maxWidth: .infinity)
                .padding(.vertical, 20)
            }

            Section {
                Link("Website", destination: websiteURL)
                Link("Twitter", destination: twitterURL)
                Link("Privacy Policy", destination: privacyURL)
            }

            Section {
                Text("Made with love in City, Country")
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
        }
    }
}
```

---

## Custom Styling Within HIG

### Custom Buttons

```swift
// ✅ Custom style that respects HIG
struct BrandButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(.custom("BrandFont-Semibold", size: 17, relativeTo: .body))
            .foregroundStyle(.white)
            .padding(.horizontal, 24)
            .padding(.vertical, 14)
            .background(Color("BrandAccent"))
            .clipShape(RoundedRectangle(cornerRadius: 12, style: .continuous))
            .scaleEffect(configuration.isPressed ? 0.97 : 1.0)
            .animation(.easeOut(duration: 0.1), value: configuration.isPressed)
    }
}

// ❌ Custom style that breaks HIG
// - Touch target too small
// - No press feedback
// - Custom gestures that conflict
```

### Custom List Rows

```swift
// ✅ Brand elements within standard patterns
List {
    ForEach(items) { item in
        HStack {
            Circle()
                .fill(Color("BrandAccent"))
                .frame(width: 8, height: 8)

            Text(item.title)
                .font(.custom("BrandFont-Regular", size: 17, relativeTo: .body))
        }
    }
}

// ❌ Completely custom list that breaks patterns
// - No swipe actions
// - Custom tap handling
// - Non-standard row heights
```

---

## Color Branding

### Appropriate Brand Color Use

```swift
// ✅ Accent/tint color
.tint(Color("BrandColor"))

// ✅ Status indicators
Circle().fill(isActive ? Color("BrandGreen") : .gray)

// ✅ Highlights and selections
.background(isSelected ? Color("BrandAccent").opacity(0.1) : .clear)
```

### Inappropriate Brand Color Use

```swift
// ❌ Brand color as background everywhere
.background(Color("BrandColor"))

// ❌ Brand color replacing system colors
.foregroundStyle(Color("BrandColor")) // For primary text

// ❌ Brand color in system components
TabView { }
    .background(Color("BrandColor")) // Jarring
```

---

## Dark Mode Branding

### Adapting Brand Colors

```swift
// Asset Catalog
// BrandAccent (Any Appearance): #FF5500
// BrandAccent (Dark): #FF7733 (lighter for dark backgrounds)

// Ensure contrast in both modes
Color("BrandAccent")
```

### Logo Variants

```swift
// Provide light and dark logo variants
Image(colorScheme == .dark ? "logo-dark" : "logo-light")

// Or use template rendering
Image("logo")
    .renderingMode(.template)
    .foregroundStyle(.primary)
```

---

## Maintaining Platform Citizenship

### Do

- Use standard navigation patterns
- Use system controls
- Support system features (Dynamic Type, Dark Mode)
- Follow gesture conventions
- Respect safe areas

### Don't

- Replace tab bar with custom navigation
- Override system gestures
- Ignore accessibility
- Force permanent light/dark mode
- Use non-standard animation timing

---

## Brand vs Platform Balance

| Element | Brand | Platform |
|---------|-------|----------|
| Accent color | Brand color | System application |
| Typography | Custom font | System sizes/Dynamic Type |
| Icons | Custom style | SF Symbols conventions |
| Buttons | Brand styling | System roles/behaviors |
| Navigation | Standard | Tab bar, NavigationStack |
| Copy | Brand voice | Clear communication |

---

## Anti-Patterns

```swift
// ❌ Logo in navigation bar
.toolbar {
    ToolbarItem(placement: .principal) {
        Image("logo")
    }
}

// ❌ Branded launch screen
// LaunchScreen with logo and tagline

// ❌ Brand color overload
VStack {
    content
}
.background(Color("BrandColor"))
.tint(Color("BrandColor"))
.foregroundStyle(Color("BrandColor"))

// ❌ Custom navigation replacing system
// Hamburger menu instead of tab bar

// ❌ Brand font without Dynamic Type
.font(.custom("Brand", fixedSize: 14))

// ✅ Brand font with Dynamic Type
.font(.custom("Brand", size: 14, relativeTo: .body))
```

---

## Checklist

### Brand Implementation

- [ ] Accent color defined with dark variant
- [ ] Custom fonts support Dynamic Type
- [ ] Logo only in appropriate places
- [ ] Launch screen follows guidelines
- [ ] Navigation uses system patterns
- [ ] Standard controls used
- [ ] Brand voice in copy, not decoration
- [ ] About screen has brand presence

---

## Quick Reference

| Brand Element | Where Appropriate |
|---------------|-------------------|
| Logo | Onboarding, About, Sign-in |
| Accent color | Tint, highlights, CTAs |
| Custom font | All text (with Dynamic Type) |
| Brand voice | Copy throughout |
| Custom styling | Within HIG patterns |

| Brand Element | Where NOT Appropriate |
|---------------|----------------------|
| Logo | Navigation bar, Tab bar, Launch screen |
| Brand colors | Backgrounds everywhere, System components |
| Custom font | Fixed sizes |
| Brand patterns | Replacing platform conventions |

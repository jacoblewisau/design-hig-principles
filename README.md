# Design HIG Principles

Expert iOS/macOS UI auditor based on **Apple's Human Interface Guidelines**. Get context-aware, multi-perspective feedback on colors, typography, layout, accessibility, and platform conventions.

## Installation

```bash
npx skills add jacoblewisau/design-hig-principles
```

Works with Claude Code, Cursor, Windsurf, and other AI coding assistants.

## What It Does

This skill audits your SwiftUI/UIKit code through three complementary design perspectives:

| Perspective | Philosophy | Key Question |
|-------------|-----------|--------------|
| **Clarity** | Communication | Does the interface communicate clearly? |
| **Consistency** | Platform Feel | Does this feel like an Apple platform app? |
| **Deference** | Content Priority | Does the UI stay out of the way? |

### Key Features

1. **Context Reconnaissance** — Analyzes your project type (productivity, media, creative, utility) to weight perspectives appropriately

2. **HIG Gap Analysis** — Searches for anti-patterns: hardcoded colors, fixed font sizes, missing accessibility labels, small touch targets, animations without Reduce Motion checks

3. **Three-Perspective Audit** — Evaluates your code through Clarity, Consistency, and Deference lenses with actionable recommendations categorized by severity

## Usage

Once installed, just ask:

```
Audit this SwiftUI code against the HIG
```

The skill will:
1. Do reconnaissance on your project type and platform targets
2. Search for HIG violations and anti-patterns
3. Propose a perspective weighting based on context
4. Wait for your confirmation
5. Provide the full three-perspective audit with code fixes

## Example Output

```
┌─────────────────────────────────────────┐
│  HIG AUDIT SUMMARY                      │
│  Project: PhotoViewer | Platform: iOS   │
│  Critical: 2 | Important: 5 | Minor: 3  │
└─────────────────────────────────────────┘

## Clarity Perspective
### Working Well
- Clear information hierarchy in list views
- Obvious tap targets with appropriate sizing

### Issues Found
- ContentView.swift:45 - Custom icons lack clear meaning
- DetailView.swift:112 - Button labels are ambiguous

## Consistency Perspective
### Working Well
- Standard navigation patterns used

### Issues Found
- Uses non-standard swipe gestures
- Custom back button breaks platform expectations

## Deference Perspective
### Working Well
- Content takes center stage in photo grid

### Issues Found
- Heavy branding competes with user content
- Unnecessary visual elements distract
```

## What's Included

```
design-hig-principles/
├── SKILL.md                          # Main skill with workflow
├── audit-checklist.md                # Structured audit criteria
└── references/
    ├── philosophy.md                 # Clarity, Consistency, Deference
    ├── accessibility.md              # Vision, mobility, cognitive, hearing
    ├── common-mistakes.md            # Anti-patterns to flag
    ├── colors.md                     # Semantic colors, dark mode, contrast
    ├── typography.md                 # Text styles, Dynamic Type
    ├── sf-symbols.md                 # Symbol usage, weights, rendering
    ├── materials-depth.md            # Blur, vibrancy, visual layers
    ├── dark-mode.md                  # Dark mode design, elevation
    ├── layout-spacing.md             # 8pt grid, safe areas, margins
    ├── lists-collections.md          # List styles, grids, lazy loading
    ├── navigation-architecture.md    # Navigation patterns, deep linking
    ├── controls-inputs.md            # System controls, forms, pickers
    ├── gestures-touch.md             # Gesture design, touch targets
    ├── modals-sheets.md              # Sheets, alerts, popovers
    ├── motion-animation.md           # Timing, springs, Reduce Motion
    ├── haptics-feedback.md           # UIFeedbackGenerator patterns
    ├── feedback-status.md            # Progress, loading, error states
    ├── platform-specific.md          # iOS/iPadOS/macOS/watchOS/visionOS
    ├── branding.md                   # Appropriate branding, app icons
    ├── onboarding-ftue.md            # First-run experience, permissions
    ├── localization-rtl.md           # Internationalization, RTL support
    └── privacy-ux.md                 # Privacy Manifest, data handling
```

## Severity Levels

| Level | Examples |
|-------|----------|
| **Critical** | Accessibility violations, Dynamic Type not supported, Reduce Motion ignored |
| **Important** | Hardcoded colors, fixed font sizes, inconsistent spacing |
| **Minor** | Optical alignment, material refinements, platform polish |

## Manual Installation

If you prefer not to use `npx skills add`:

**Global (all projects):**
```bash
git clone https://github.com/jacoblewisau/design-hig-principles.git
cp -r design-hig-principles ~/.claude/skills/
```

**For Cursor:**
```bash
cp -r design-hig-principles ~/.cursor/skills/
```

## Credits

This skill synthesizes design principles from:

- **Apple Human Interface Guidelines** — [developer.apple.com/design/human-interface-guidelines](https://developer.apple.com/design/human-interface-guidelines)

## License

MIT

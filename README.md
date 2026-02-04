# Design HIG Principles

A Claude Code skill for auditing iOS/macOS UI against Apple Human Interface Guidelines using three complementary design perspectives.

## Installation

```bash
npx skills add https://github.com/jacoblewisau/design-hig-principles
```

## Usage

In Claude Code, invoke with:

```
/design-hig-principles
```

Then share your SwiftUI/UIKit code for a structured HIG audit.

## The Three Perspectives

Every HIG audit applies these complementary lenses:

| Perspective | Focus | Key Question |
|-------------|-------|--------------|
| **Clarity** | Communication | Does the interface communicate clearly? |
| **Consistency** | Platform feel | Does this feel like an Apple platform app? |
| **Deference** | Content priority | Does the UI stay out of the way? |

## Core Philosophy

Apple's design philosophy distilled:

- **Content is paramount** — UI defers to content, not the other way around
- **Clarity through hierarchy** — Users know what they can do without instructions
- **Platform conventions matter** — Standard patterns reduce cognitive load
- **Deference makes beauty** — Restrained UI lets content shine

## What You Get

Each audit provides:

1. **Context Reconnaissance** — Project type analysis to weight perspectives
2. **Three-Perspective Analysis** — Clarity, Consistency, and Deference reviews
3. **Severity Classification** — Critical, Important, and Minor issues
4. **Actionable Fixes** — Code examples for every issue found
5. **Accessibility Check** — Always included, never skipped

## Reference Library

The skill includes 21 comprehensive reference files covering:

### Core
- Philosophy, Accessibility, Common Mistakes, Audit Checklist

### Visual Design
- Colors, Typography, SF Symbols, Materials & Depth, Dark Mode

### Layout & Structure
- Layout & Spacing, Lists & Collections, Navigation Architecture

### Interaction
- Controls & Inputs, Gestures & Touch, Modals & Sheets
- Motion & Animation, Haptics, Feedback & Status

### Platform & Polish
- Platform-Specific (iOS/iPadOS/macOS/watchOS/visionOS)
- Branding, Onboarding & FTUE, Localization & RTL, Privacy UX

## Severity Levels

| Level | Examples |
|-------|----------|
| **Critical** | Accessibility violations, Dynamic Type not supported, Reduce Motion ignored |
| **Important** | Hardcoded colors, fixed font sizes, inconsistent spacing |
| **Minor** | Optical alignment, material refinements, platform polish |

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

## License

MIT

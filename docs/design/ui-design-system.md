# Act UI Design System & Brand Identity

> **Version**: 2.0  
> **Last Updated**: 2026-01-20  
> **Source Material**: 59 wireframes in `./wireframe/`

---

## 1. Design Philosophy: "Fluid Intelligence"

The visual language of Act is characterized by **extreme minimalism, high-contrast typography, and soft, organic containment.**

- **Content First**: The interface recedes. Chrome is minimal. Content (maps, lists, images) takes center stage.
- **Soft & Friendly**: We avoid sharp edges. Everything is rounded, approachable, and tactile.
- **Monochrome Foundation**: The app is 90% Black, White, and Grays. Color is used *only* for content or critical actions.
- **Fluid Physics**: Elements shouldn't just "appear"; they should slide and expand with bouncy, spring animations.

---

## 2. Global Design Tokens

### 2.1 Color Palette

> **Reference**: [act-welcome-empty-state.png](./wireframe/act-welcome-empty-state.png)

We use a semantic color system. Do not use hex codes directly in views; use these semantic definitions.

| Semantic Name | Value (Light Mode) | Value (Dark Mode) | Usage |
| :--- | :--- | :--- | :--- |
| **Canvas** | `#FFFFFF` (White) | `#000000` (Black) | Main app background |
| **Surface Primary** | `#FFFFFF` (White) | `#1C1C1E` (Gray 6) | Cards, Modals, Sheets |
| **Surface Secondary** | `#F2F2F7` (System Gray 6) | `#2C2C2E` (Gray 5) | Input fields, User message bubbles, Hover states |
| **Text Primary** | `#000000` (Black) | `#FFFFFF` (White) | Headings, Main Body |
| **Text Secondary** | `#636366` (Gray) | `#AEAEB2` (Gray) | Subtitles, Timestamps, Captions |
| **Text Tertiary** | `#AEAEB2` (Light Gray) | `#636366` (Dark Gray) | Placeholders, Disabled icons |
| **Accent Blue** | `#007AFF` | `#0A84FF` | Primary Actions, Checkboxes, Links |
| **Accent Orange** | `#FF6B00` | `#FF9F0A` | Specific Integrations (Booking, Restaurant CTAs) |
| **Accent Green** | `#1DB954` | `#1DB954` | Spotify, Success states, Toggle active |
| **Accent Red** | `#FF3B30` | `#FF453A` | Destructive actions, Dismiss buttons |

### 2.2 Typography

Font Family: **SF Pro** (System Standard). We rely heavily on weight contrast.

| Style | Spec | Usage |
|:---|:---|:---|
| Display / Page Title | `17pt Semibold` | Nav bar title |
| Card Title (H2) | `18-20pt Bold` | "Little Nona's", recipe names |
| Body (Primary) | `16pt Regular`, 140% line height | Main content text |
| Subtitle | `14pt Regular`, Text Secondary | Metadata, descriptions |
| Caption / Meta | `13pt Regular`, Text Secondary | Timestamps, prices |
| Button Text | `16pt Semibold` | All button labels |

### 2.3 Shapes & Radii

This is critical to the "vibe".

| Element | Radius | Notes |
|:---|:---:|:---|
| Standard Cards | `16pt` | Default for content cards |
| Rich Media Cards | `24pt` or `32pt` | Maps, Images, Modals |
| Pills / Capsules | `999pt` (full round) | Input bars, Buttons, Tags |
| User Message Bubbles | `20pt` | Rounded rectangle, right-aligned |
| Stroke | `1pt` `#E5E5EA` | Use sparingly, prefer surface differentiation |

---

## 3. Core Component Library

### 3.1 NavigationBar

> **Reference**: [Screenshot 2026-01-16 at 23.20.51.png](./wireframe/component-status-indicators.png)

| Variant | Description |
|:---|:---|
| `default` | Hamburger (left), Title + chevron (center-left), Edit + More icons (right) |
| `default/hasDivider` | Same as default with subtle bottom divider line |
| `newThread` | Minimal: only loading spinner on right, no other icons |

**Specs**:
- Height: ~44pt (iOS standard)
- Background: Transparent, becomes translucent on scroll
- Title format: `[Model Name] [Version] >`
- Icons: SF Symbols, 24pt

---

### 3.2 Buttons

> **Reference**: [Screenshot 2026-01-16 at 23.20.27.png](./wireframe/component-input-controls.png)

| Style | Background | Text | Border | Use Case |
|:---|:---|:---|:---|:---|
| `default` (Primary) | Black | White | None | Main CTAs |
| `secondary` | Light Gray (`#F2F2F7`) | Black | None | Secondary actions |
| `destructive` | Red (`#FF3B30`) | White | None | Delete, Cancel |
| `sec-destructive` | Light Red tint | Red text | None | Soft destructive |

**States**: `default`, `isPressed` (darker), `isDisabled` (50% opacity)

**Shape**: Pill (`cornerRadius: 999`) for floating CTAs, or `cornerRadius: 12` for inline buttons.

---

### 3.3 IconButton

> **Reference**: [Screenshot 2026-01-16 at 23.20.27.png](./wireframe/component-input-controls.png)

- Size: 44pt touch target, 24pt icon
- Background: Transparent or Circle (`Surface Secondary`)
- States: `default`, `isPressed`, `isDisabled`

---

### 3.4 Composer (Input Bar)

> **Reference**: [component-cards.png](./wireframe/component-cards.png), [act-composer-keyboard.png](./wireframe/act-composer-keyboard.png)

The composer is the anchor of the experience. It looks like a floating capsule.

| Variant | Description |
|:---|:---|
| `default` | Floating capsule, keyboard hidden |
| `hasKeyboard` | Same, keyboard visible, input focused |

**Structure**:
```
┌─────────────────────────────────────────────────┐
│  [+]  │  Ask anything...  │  [🎤]  │  [((●))]  │
└─────────────────────────────────────────────────┘
```

| Element | Spec |
|:---|:---|
| Container | Capsule shape, `Surface Secondary`, ~56pt height |
| + Button | Circle, Text Tertiary color, opens attachment menu |
| TextField | Placeholder "Ask anything", center-left aligned |
| Mic Icon | Outline style, Text Tertiary |
| Voice Button | **Solid Black Circle** with white waveform icon - strongest visual element |

**Voice Active State**: Voice button shows **white square** (stop icon) when AI is speaking.

---

### 3.5 ConversationMessage

> **Reference**: [component-message-states.png](./wireframe/component-message-states.png), [act-loading-state-1.png](./wireframe/act-loading-state-1.png)

#### User Message (`ConversationMessage/to`)
- Alignment: Right
- Background: `Surface Secondary` (gray pill) - **NOT transparent**
- Corner radius: ~20pt
- Padding: 12pt horizontal, 10pt vertical

#### AI Message (`ConversationMessage/from`)

| State | Visual |
|:---|:---|
| `isLoading` | Black pulsing dot (●), left-aligned |
| `isSearching` | "Searching..." text label |
| `default` | Plain text, left-aligned, no background |
| `hasImage(s)` | Text + image(s) with rounded corners |

#### Feedback Row (AI messages only)
Appears below AI responses:
```
[📋] [🔊] [👍] [👎] [💡] [🔄] [📤]
```
- Icons: Copy, Read Aloud, Like, Dislike, Lightbulb, Retry, Share
- Size: 20pt icons, 12pt spacing
- Color: Text Tertiary

---

### 3.6 Suggestion & SuggestionGroup

> **Reference**: [Screenshot 2026-01-16 at 23.21.52.png](./wireframe/component-badges.png), [Screenshot 2026-01-16 at 23.17.22.png](./wireframe/component-message-bubbles.png)

Used in empty states and onboarding.

| Component | Description |
|:---|:---|
| `Suggestion` | Single pill: Bold label + Regular sublabel |
| `SuggestionGroup` | Horizontal scroll of 2-3 suggestions |

**Specs**:
- Background: `Surface Secondary`
- Corner radius: `16pt`
- Padding: 12pt horizontal, 10pt vertical
- Typography: Label `15pt Semibold`, Sublabel `13pt Regular` Text Secondary

---

### 3.7 Row (Settings/List Row)

> **Reference**: [Screenshot 2026-01-16 at 23.21.07.png](./wireframe/component-toggles.png), [Screenshot 2026-01-16 at 23.17.43.png](./wireframe/component-action-buttons.png)

| Variant | Trailing Element |
|:---|:---|
| `toggle` | iOS Toggle switch |
| `chevron` | Right chevron (navigation) |
| `value` | Secondary text value |
| `none` | No trailing element |

**Specs**:
- Height: ~44pt
- Leading icon: 24pt, optional
- Separator: 1pt line, inset from leading edge

---

### 3.8 Sheet (Modal)

> **Reference**: [Screenshot 2026-01-16 at 23.21.23.png](./wireframe/component-list-rows.png), [Screenshot 2026-01-16 at 23.21.15.png](./wireframe/component-empty-states.png)

| Variant | Description |
|:---|:---|
| `default` | Standard iOS sheet with grab handle, partial height |
| `isScrolled` | Expands to full screen when scrolled |

**SheetHeader**:
- Centered title, `17pt Semibold`
- X button on right (circle background)
- Optional blurred background

---

### 3.9 Input (Text Field)

> **Reference**: [Screenshot 2026-01-16 at 23.21.57.png](./wireframe/component-icons.png)

- Container: Rounded rectangle, `Surface Secondary` background
- Corner radius: `12pt`
- Placeholder: Title text
- Helper text: Below input, Text Secondary

---

### 3.10 Form Controls

> **Reference**: [Screenshot 2026-01-16 at 23.20.27.png](./wireframe/component-input-controls.png)

| Control | Style |
|:---|:---|
| Checkbox | **Blue filled CIRCLE** with white checkmark (NOT square) |
| Radio | Circle outline, filled circle when selected |
| Toggle | iOS standard, green (`#34C759`) when active |
| Segmented Control | Pill group, filled segment = selected |

---

### 3.11 Sidebar

> **Reference**: [Screenshot 2026-01-16 at 23.20.59.png](./wireframe/component-tab-bar.png)

Structure:
1. **Search bar** + New Thread button (top)
2. **Primary nav**: ChatGPT, Library, GPTs (with icons)
3. **Secondary nav**: New Project, Variables
4. **Section list**: Items with highlight state on selection
5. **User avatar**: Bottom, circular

---

## 4. Rich Media Cards

### 4.1 Integration Header

> **Reference**: [act-result-shopping-search.png](./wireframe/act-result-shopping-search.png), [act-result-music-spotify.png](./wireframe/act-result-music-spotify.png)

Pattern for AI responses with external integrations:

```
[Agent Label]              ← "Asked Walmart" or "Pizzazz"
[🏪] [Integration Name]    ← Icon + "Walmart" 
```

- Agent Label: Text Secondary, `13pt Regular`
- Integration: Icon (24pt) + Name `15pt Semibold`

---

### 4.2 Grocery List Card (Walmart)

> **Reference**: [act-result-shopping-search.png](./wireframe/act-result-shopping-search.png), [act-result-shopping-detail.png](./wireframe/act-result-shopping-detail.png)

| Element | Spec |
|:---|:---|
| Section Header | "Fresh", "Protein" - `15pt Semibold`, uppercase |
| Product Row | Checkbox + Image(40pt) + Name + Price + Quantity pill |
| Quantity Stepper | `[-] [count] [+]` + "Save" button |
| Quantity Pill | Blue pill with white text "1 pcs." |
| "Order" Button | Top-right, blue pill |

---

### 4.3 Date/Time Picker Card

> **Reference**: [act-result-order-history.png](./wireframe/act-result-order-history.png)

| Element | Spec |
|:---|:---|
| Date Tabs | Horizontal scroll, day cards showing weekday + date |
| Time Slots | Radio-style rows: time range + price + radio button |
| Selected State | Blue filled radio circle |
| "Confirm" Button | Full-width blue pill at bottom |

---

### 4.4 Recipe/Meal Plan Card

> **Reference**: [act-result-shopping-cart.png](./wireframe/act-result-shopping-cart.png), [act-result-shopping-confirm.png](./wireframe/act-result-shopping-confirm.png)

| Element | Spec |
|:---|:---|
| Day Tabs | Horizontal scroll pills: "Day 1 (Friday)", outline style |
| Recipe Row | Image (60pt square) + Title + metadata |
| Metadata Pills | 🍳 35m, 🔥 350kcal, 🌿 (vegetarian badge) |
| Action Row | "Previous" (outline) + "Refresh" (outline) pills |

---

### 4.5 Flight Card (Expedia)

> **Reference**: [act-result-travel-flights.png](./wireframe/act-result-travel-flights.png)

| Element | Spec |
|:---|:---|
| User Query Bubble | **Black background, white text** (distinct from gray) |
| Card Header | "Roundtrip" + price, dates below |
| Flight Row | Airline logo + Time ——— Time + metadata |
| Timeline | Dashed line connecting departure/arrival times |
| CTA | "Book on Expedia" - full-width black button |

---

### 4.6 Restaurant Detail Card

> **Reference**: [act-detail-card.png](./wireframe/act-detail-card.png), [act-ios-restaurant-detail.png](./wireframe/act-ios-restaurant-detail.png)

| Element | Spec |
|:---|:---|
| Hero Image | Full-width, 16:9 ratio, rounded top corners |
| Image Carousel | Page dots centered below hero |
| Dismiss Button | X in circle, top-left overlay on image |
| Rating Badge | Orange circle with white text "9.2" |
| Info Pills | Price range "$$", Wait time "7 min" |
| CTA Row | "Order now" (orange filled) + "Directions" (gray outline) |

---

### 4.7 Image Gallery

> **Reference**: [act-ios-restaurant-detail.png](./wireframe/act-ios-restaurant-detail.png)

Asymmetric grid layout:
- 1 large image (2/3 width)
- 2 small stacked images (1/3 width)
- All with matching corner radii

---

### 4.8 Interactive Media Card (BeatBot)

> **Reference**: [iOS-5.png](./wireframe/act-ios-variant-5.png)

Custom interactive components embedded in cards:
- Yellow tint background
- Grid of touch targets
- Play/Pause button, BPM display
- Undo/Redo controls

---

## 5. Voice Mode UI

> **Reference**: [Screenshot 2026-01-16 at 23.17.32.png](./wireframe/component-settings.png)

Voice mode is a **completely different visual experience** from chat.

### 5.1 Voice Intro Screen

| Element | Spec |
|:---|:---|
| Background | Pure Black |
| Title | "Chat with voice" - White, 24pt Bold |
| Benefits List | Icon + Text rows (Just start talking, Hands-free, etc.) |
| CTA | "Choose a voice" - Blue pill button |

### 5.2 Voice Selector

| Element | Spec |
|:---|:---|
| Voice Avatars | 4 white circles in a row, animated |
| Voice Names | "Breeze", "Cove", "Sky", "Juniper", "Ember" |
| Selection | Row with checkmark when selected |
| CTA | "Confirm" - Full-width outlined button |

### 5.3 Active Voice Mode

| Element | Spec |
|:---|:---|
| Background | Pure Black (#000000) |
| Animated Blob | White organic morphing shape, center |
| Status Text | "Tap to cancel" - White, Text Secondary |
| Stop Button | White square in circle |
| Dismiss | Red X button (bottom-right) |

---

## 6. Onboarding & Authentication

> **Reference**: [Screenshot 2026-01-16 at 23.16.49.png](./wireframe/component-onboarding.png)

### 6.1 Splash Screens

Rotating branded backgrounds:

| Background | Branding |
|:---|:---|
| Black | "●" (pulsing dot) |
| Dark Teal | "ChatGPT 🟢" |
| Cream | "Let's brainstorm 🧠" |
| Dark Green | "Let's go 🟡" |

### 6.2 Auth Buttons

| Button | Style |
|:---|:---|
| Continue with Apple | Black filled, Apple logo left |
| Continue with Google | White outline, Google logo left |
| Sign up with email | White outline |
| Log in | Text link |

### 6.3 Email Verification

- White background
- Centered email icon (large)
- "Verify your email" title
- Secondary text explaining action
- "I've verified my email" button
- "Sign out" text link

---

## 7. Settings Screen

> **Reference**: [Screenshot 2026-01-16 at 23.17.43.png](./wireframe/component-action-buttons.png)

### Section Structure

| Section | Items |
|:---|:---|
| ACCOUNT | Email, Subscription, Restore purchases, Data Controls, etc. |
| APP | Color Scheme (System/Light/Dark), Haptic Feedback (toggle) |
| SPEECH | Voice, Main Language |
| ABOUT | Help Center, Terms, Privacy, Version |

---

## 8. Model Selector

> **Reference**: [Screenshot 2026-01-16 at 23.17.43.png](./wireframe/component-action-buttons.png)

Floating dropdown card:
- Appears below nav title
- Radio-style selection with checkmarks
- Model names: "GPT-3.5", "GPT-4" with sparkle icons
- Rounded corners, shadow elevation

---

## 9. Loading & Empty States

### 9.1 Loading States

> **Reference**: [act-loading-state-1.png](./wireframe/act-loading-state-1.png)

| State | Visual |
|:---|:---|
| Thinking | Black pulsing dot (●), left-aligned |
| Searching | "Searching..." text |
| Stop Button | Voice button transforms to white square icon |

### 9.2 Empty State

> **Reference**: [Screenshot 2026-01-16 at 23.17.22.png](./wireframe/component-message-bubbles.png)

- Large centered ChatGPT logo (gray outline)
- SuggestionGroup below

### 9.3 Scroll-to-Bottom

> **Reference**: [act-loading-state-2.png](./wireframe/act-loading-state-2.png)

- Down arrow in subtle circle
- Appears when new content below viewport

---

## 10. Liquid Glass (iOS 26+)

> **Canonical Guide**: [liquid-glass-implementation.md](./liquid-glass-implementation.md)  
> **Apple Reference**: [SwiftUI-Implementing-Liquid-Glass-Design.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md)

Liquid Glass is Apple's dynamic material design for iOS 26+ that combines optical glass properties with fluid behavior. It blurs content behind it, reflects surrounding colors, and reacts to touch/pointer interactions.

### 10.1 When to Use Liquid Glass in Act

| Component | Liquid Glass? | Notes |
|:---|:---:|:---|
| NavigationBar | ✅ Yes | System will apply automatically on iOS 26+ |
| Composer Input | ✅ Yes | Use `.glassEffect()` for floating feel |
| Action Buttons | ✅ Yes | Use `.buttonStyle(.glass)` |
| Rich Cards | ⚠️ Selective | Only for interactive cards, not all |
| Message Bubbles | ❌ No | Keep flat for readability |
| Modals/Sheets | ✅ Yes | System handles via sheet presentation |

### 10.2 Basic SwiftUI Implementation

```swift
// Simple glass effect (capsule by default)
Text("Action")
    .padding()
    .glassEffect()

// Custom shape
Button("Send") { }
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))

// With tint and interactivity
Image(systemName: "mic.fill")
    .frame(width: 44, height: 44)
    .glassEffect(.regular.tint(.blue).interactive())
```

### 10.3 Glass Button Styles

```swift
// Standard glass button
Button("Submit") { }
    .buttonStyle(.glass)

// Prominent glass button (for primary CTAs)
Button("Order Now") { }
    .buttonStyle(.glassProminent)
```

### 10.4 Multiple Glass Elements (Morphing)

When using multiple glass elements that should interact/merge:

```swift
@Namespace private var namespace

GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(actions.indices, id: \.self) { index in
            Image(systemName: actions[index].icon)
                .frame(width: 60, height: 60)
                .glassEffect()
                .glassEffectID(actions[index].id, in: namespace)
        }
    }
}
```

**Key Parameters**:
- `spacing`: Distance at which glass effects begin to merge
- `glassEffectID`: Enables morphing transitions when view hierarchy changes
- `glassEffectUnion`: Combines multiple views into single glass effect

### 10.5 Act Composer with Liquid Glass

```swift
struct ActComposer: View {
    @State private var prompt = ""
    @Namespace private var composerNamespace
    
    var body: some View {
        GlassEffectContainer(spacing: 12) {
            HStack(spacing: 12) {
                // Plus button
                Button(action: { }) {
                    Image(systemName: "plus")
                        .frame(width: 36, height: 36)
                }
                .glassEffect()
                .glassEffectID("plus", in: composerNamespace)
                
                // Text field area
                TextField("Ask anything...", text: $prompt)
                    .padding(.horizontal, 16)
                    .frame(height: 44)
                    .glassEffect(in: .capsule)
                    .glassEffectID("input", in: composerNamespace)
                
                // Voice button
                Button(action: { }) {
                    Image(systemName: "waveform")
                        .frame(width: 44, height: 44)
                }
                .glassEffect(.regular.tint(.primary).interactive())
                .glassEffectID("voice", in: composerNamespace)
            }
        }
    }
}
```

### 10.6 Interactive States

```swift
// Make glass respond to touch/pointer
.glassEffect(.regular.interactive())

// Or explicitly
.glassEffect(.regular.interactive(true))
```

### 10.7 Best Practices for Act

1. **Container First**: Always wrap multiple glass elements in `GlassEffectContainer`
2. **Namespace for Transitions**: Use `@Namespace` + `glassEffectID` for morphing
3. **Subtle Tints Only**: Use `tintColor.withAlphaComponent(0.2)` max
4. **Performance**: Limit to 4-5 glass elements on screen simultaneously
5. **Fallback**: Check `@available(iOS 26.0, *)` and provide solid backgrounds for older OS
6. **Readability**: Avoid glass behind dense text content

### 10.8 Fallback for iOS 25 and Earlier

```swift
extension View {
    @ViewBuilder
    func adaptiveGlass() -> some View {
        if #available(iOS 26.0, *) {
            self.glassEffect()
        } else {
            self.background(Color(.systemGray6))
                .clipShape(Capsule())
        }
    }
    
    @ViewBuilder
    func adaptiveGlassRect(cornerRadius: CGFloat) -> some View {
        if #available(iOS 26.0, *) {
            self.glassEffect(in: .rect(cornerRadius: cornerRadius))
        } else {
            self.background(Color(.systemGray6))
                .clipShape(RoundedRectangle(cornerRadius: cornerRadius))
        }
    }
}
```

---

## 11. "The Vibe" Guidelines

### DO's ✅

- Use whitespace generously - if you think there's too much, add more
- Use system font with deliberate weight contrast (Bold vs Regular)
- Make every interaction feel "bouncy" with spring animations
- Use `Surface Secondary` to separate layers, not shadows
- Keep the "Thinking" state minimal (pulsing black dot)

### DON'Ts ❌

- Heavy drop shadows - use colored surfaces instead
- Gradients for backgrounds - keep it flat and clean
- Sharp corners - everything is rounded
- Cluttered loading states - simplicity is key
- Borders everywhere - prefer background differentiation

---

## 12. Animation Guidelines

| Interaction | Animation |
|:---|:---|
| View Transitions | Spring animation, 0.3s |
| Button Press | Scale to 0.96, 0.1s |
| Modal Present | Slide up with spring |
| Loading Dot | Pulse (fade in/out), 0.8s loop |
| Voice Blob | Organic morphing, continuous |

---

## 13. Reference Index

Quick lookup for source wireframes:

| Component | Primary Reference |
|:---|:---|
| Navigation | `Screenshot 2026-01-16 at 23.20.51.png` |
| Buttons | `Screenshot 2026-01-16 at 23.20.27.png` |
| Composer | `Screenshot 2026-01-16 at 23.21.43.png` |
| Messages | `component-message-states.png` |
| Sheets | `component-list-rows.png` |
| Suggestions | `component-badges.png` |
| Settings | `component-action-buttons.png` |
| Voice Mode | `component-settings.png` |
| Onboarding | `component-onboarding.png` |
| Shopping Flow | `act-result-shopping-*.png` |
| Restaurant | `act-detail-card.png`, `act-ios-restaurant-detail.png` |

---

> **Note**: All wireframe paths are relative to `./wireframe/` (from the docs directory). This spec should be treated as the **Single Source of Truth** for styling the Act iOS app.

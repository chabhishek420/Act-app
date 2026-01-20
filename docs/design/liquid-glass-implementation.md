# Liquid Glass Implementation Guide for Act

> **Target**: iOS 26+ (with fallbacks for iOS 25)  
> **Last Updated**: 2026-01-20  
> **Apple Documentation**: [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)

---

## Overview

Liquid Glass is Apple's next-generation material design that creates dynamic, fluid interfaces. This guide covers how to implement Liquid Glass effects in the Act iOS app, following Apple's latest patterns.

### Key Characteristics

- **Blurs** content behind the material
- **Reflects** color and light from surrounding content
- **Reacts** to touch and pointer interactions in real time
- **Morphs** between shapes during transitions
- **Merges** when glass elements are placed close together

---

## Quick Reference API

### SwiftUI

| API | Purpose |
|:---|:---|
| `.glassEffect()` | Apply glass to a view (capsule default) |
| `.glassEffect(in: Shape)` | Apply glass with custom shape |
| `.glassEffect(_:in:isEnabled:)` | Full signature with conditional enabling |
| `.glassEffect(.regular.tint(Color))` | Glass with color tint |
| `.glassEffect(.regular.interactive())` | Respond to touch/pointer |
| `GlassEffectContainer(spacing:)` | Container for multiple glass views |
| `.glassEffectID(_:in:)` | Enable morphing transitions |
| `.glassEffectUnion(id:namespace:)` | Combine views into single effect |
| `.buttonStyle(.glass)` | Standard glass button (`GlassButtonStyle`) |
| `.buttonStyle(.glassProminent)` | Prominent glass button |

### UIKit

| API | Purpose |
|:---|:---|
| `UIGlassEffect()` | Glass effect object |
| `UIVisualEffectView(effect:)` | Apply glass to view |
| `UIGlassContainerEffect()` | Container with spacing |
| `glassEffect.tintColor` | Apply tint |
| `glassEffect.isInteractive` | Enable interaction response |
| `UIScrollEdgeEffect` | Scroll edge glass effects |

---

## Implementation Patterns

### 1. Basic Glass Effect

The simplest implementation:

```swift
import SwiftUI

struct GlassButton: View {
    let title: String
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(title)
                .font(.body.weight(.semibold))
                .foregroundStyle(.primary)
                .padding(.horizontal, 20)
                .padding(.vertical, 12)
        }
        .glassEffect() // Capsule shape by default
    }
}
```

### 2. Custom Shape Glass

```swift
struct GlassCard: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("Card Title")
                .font(.headline)
            Text("Card description goes here")
                .font(.subheadline)
                .foregroundStyle(.secondary)
        }
        .padding(16)
        .glassEffect(in: .rect(cornerRadius: 16))
    }
}
```

### 3. Interactive Glass with Tint

For buttons and interactive elements:

```swift
struct VoiceButton: View {
    @State private var isActive = false
    
    var body: some View {
        Button(action: { isActive.toggle() }) {
            Image(systemName: isActive ? "waveform" : "mic.fill")
                .font(.system(size: 20))
                .frame(width: 44, height: 44)
        }
        .glassEffect(
            .regular
                .tint(isActive ? .blue : .clear)
                .interactive()
        )
    }
}
```

### 4. Multiple Glass Elements with Container

When using multiple glass elements that should interact/merge:

```swift
struct ActionBar: View {
    @Namespace private var namespace
    
    let actions = [
        ("copy", "doc.on.doc"),
        ("share", "square.and.arrow.up"),
        ("delete", "trash")
    ]
    
    var body: some View {
        GlassEffectContainer(spacing: 20.0) {
            HStack(spacing: 20.0) {
                ForEach(actions, id: \.0) { action in
                    Button(action: { print(action.0) }) {
                        Image(systemName: action.1)
                            .frame(width: 50, height: 50)
                    }
                    .glassEffect()
                    .glassEffectID(action.0, in: namespace)
                }
            }
        }
    }
}
```

### 5. Morphing Transitions

Glass elements can morph when view hierarchy changes:

```swift
struct ExpandableToolbar: View {
    @State private var isExpanded = false
    @Namespace private var toolbarNamespace
    
    var body: some View {
        GlassEffectContainer(spacing: 16) {
            HStack(spacing: 16) {
                // Always visible
                Button(action: {}) {
                    Image(systemName: "pencil")
                        .frame(width: 50, height: 50)
                }
                .glassEffect()
                .glassEffectID("pencil", in: toolbarNamespace)
                
                // Conditionally visible
                if isExpanded {
                    Button(action: {}) {
                        Image(systemName: "eraser")
                            .frame(width: 50, height: 50)
                    }
                    .glassEffect()
                    .glassEffectID("eraser", in: toolbarNamespace)
                    
                    Button(action: {}) {
                        Image(systemName: "paintbrush")
                            .frame(width: 50, height: 50)
                    }
                    .glassEffect()
                    .glassEffectID("brush", in: toolbarNamespace)
                }
                
                // Toggle button
                Button(action: {
                    withAnimation(.spring(duration: 0.3)) {
                        isExpanded.toggle()
                    }
                }) {
                    Image(systemName: isExpanded ? "chevron.left" : "chevron.right")
                        .frame(width: 50, height: 50)
                }
                .glassEffect()
                .glassEffectID("toggle", in: toolbarNamespace)
            }
        }
    }
}
```

### 6. Uniting Multiple Views

Combine separate views into one glass effect:

```swift
struct UnitedGlassGroup: View {
    @Namespace private var namespace
    
    let items = ["star", "heart", "moon", "sun"]
    
    var body: some View {
        GlassEffectContainer(spacing: 20.0) {
            HStack(spacing: 8) {
                ForEach(items.indices, id: \.self) { index in
                    Image(systemName: "\(items[index]).fill")
                        .frame(width: 40, height: 40)
                        .glassEffect()
                        // First two items form group "A", rest form group "B"
                        .glassEffectUnion(
                            id: index < 2 ? "groupA" : "groupB",
                            namespace: namespace
                        )
                }
            }
        }
    }
}
```

---

## Act-Specific Components

### Composer Input Bar

The main input bar using Liquid Glass:

```swift
struct ActComposerBar: View {
    @Binding var text: String
    @Namespace private var composerNamespace
    @State private var isVoiceActive = false
    
    var body: some View {
        GlassEffectContainer(spacing: 8) {
            HStack(spacing: 8) {
                // Attach button
                Button(action: { /* Show attachment menu */ }) {
                    Image(systemName: "plus")
                        .font(.system(size: 16, weight: .medium))
                        .frame(width: 36, height: 36)
                }
                .glassEffect()
                .glassEffectID("attach", in: composerNamespace)
                
                // Text input
                TextField("Ask anything...", text: $text)
                    .padding(.horizontal, 16)
                    .frame(height: 44)
                    .glassEffect(in: .capsule)
                    .glassEffectID("input", in: composerNamespace)
                
                // Microphone button
                Button(action: { /* Toggle voice input */ }) {
                    Image(systemName: "mic")
                        .font(.system(size: 16))
                        .frame(width: 36, height: 36)
                }
                .glassEffect()
                .glassEffectID("mic", in: composerNamespace)
                
                // Voice mode button
                Button(action: { isVoiceActive.toggle() }) {
                    Image(systemName: isVoiceActive ? "stop.fill" : "waveform")
                        .font(.system(size: 16, weight: .medium))
                        .foregroundStyle(.white)
                        .frame(width: 44, height: 44)
                        .background(Circle().fill(.black))
                }
                .glassEffect(.regular.interactive())
                .glassEffectID("voice", in: composerNamespace)
            }
            .padding(.horizontal, 16)
            .padding(.vertical, 8)
        }
    }
}
```

### Suggestion Pills

```swift
struct SuggestionPills: View {
    let suggestions: [Suggestion]
    
    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            GlassEffectContainer(spacing: 12) {
                HStack(spacing: 12) {
                    ForEach(suggestions) { suggestion in
                        SuggestionPill(suggestion: suggestion)
                            .glassEffect(in: .rect(cornerRadius: 16))
                    }
                }
                .padding(.horizontal, 16)
            }
        }
    }
}

struct SuggestionPill: View {
    let suggestion: Suggestion
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(suggestion.title)
                .font(.subheadline.weight(.semibold))
            Text(suggestion.subtitle)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .padding(.horizontal, 16)
        .padding(.vertical, 12)
    }
}
```

### AI Feedback Row

```swift
struct MessageFeedbackRow: View {
    @Namespace private var feedbackNamespace
    
    let actions: [(id: String, icon: String)] = [
        ("copy", "doc.on.doc"),
        ("speak", "speaker.wave.2"),
        ("like", "hand.thumbsup"),
        ("dislike", "hand.thumbsdown"),
        ("retry", "arrow.clockwise"),
        ("share", "square.and.arrow.up")
    ]
    
    var body: some View {
        GlassEffectContainer(spacing: 8) {
            HStack(spacing: 8) {
                ForEach(actions, id: \.id) { action in
                    Button(action: { handleAction(action.id) }) {
                        Image(systemName: action.icon)
                            .font(.system(size: 14))
                            .frame(width: 32, height: 32)
                            .foregroundStyle(.secondary)
                    }
                    .glassEffect()
                    .glassEffectID(action.id, in: feedbackNamespace)
                }
            }
        }
    }
    
    private func handleAction(_ id: String) {
        // Handle feedback action
    }
}
```

---

## UIKit Implementation

For UIKit-based components:

```swift
import UIKit

class GlassCardView: UIView {
    private let visualEffectView: UIVisualEffectView
    
    /// Access this to add your content - it's inside the glass effect
    var contentView: UIView {
        visualEffectView.contentView
    }
    
    init(tintColor: UIColor? = nil, isInteractive: Bool = false) {
        let glassEffect = UIGlassEffect()
        glassEffect.tintColor = tintColor
        glassEffect.isInteractive = isInteractive
        
        visualEffectView = UIVisualEffectView(effect: glassEffect)
        
        super.init(frame: .zero)
        setupViews()
    }
    
    required init?(coder: NSCoder) {
        let glassEffect = UIGlassEffect()
        visualEffectView = UIVisualEffectView(effect: glassEffect)
        super.init(coder: coder)
        setupViews()
    }
    
    private func setupViews() {
        visualEffectView.translatesAutoresizingMaskIntoConstraints = false
        visualEffectView.layer.cornerRadius = 16
        visualEffectView.clipsToBounds = true
        addSubview(visualEffectView)
        
        // Content goes INSIDE visualEffectView.contentView, not as a sibling
        NSLayoutConstraint.activate([
            visualEffectView.topAnchor.constraint(equalTo: topAnchor),
            visualEffectView.leadingAnchor.constraint(equalTo: leadingAnchor),
            visualEffectView.trailingAnchor.constraint(equalTo: trailingAnchor),
            visualEffectView.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }
}

// Usage:
let cardView = GlassCardView(tintColor: .systemBlue.withAlphaComponent(0.2))
let label = UILabel()
label.text = "Glass Card"
cardView.contentView.addSubview(label) // Add to contentView, not the card itself
```

### UIKit Container Effect

```swift
class GlassContainerView: UIView {
    private let containerEffect: UIGlassContainerEffect
    private let containerView: UIVisualEffectView
    
    init(spacing: CGFloat = 40.0) {
        containerEffect = UIGlassContainerEffect()
        containerEffect.spacing = spacing
        containerView = UIVisualEffectView(effect: containerEffect)
        
        super.init(frame: .zero)
        
        containerView.translatesAutoresizingMaskIntoConstraints = false
        addSubview(containerView)
        
        NSLayoutConstraint.activate([
            containerView.topAnchor.constraint(equalTo: topAnchor),
            containerView.leadingAnchor.constraint(equalTo: leadingAnchor),
            containerView.trailingAnchor.constraint(equalTo: trailingAnchor),
            containerView.bottomAnchor.constraint(equalTo: bottomAnchor)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func addGlassView(_ view: UIView) {
        containerView.contentView.addSubview(view)
    }
}
```

---

## Fallback for Older iOS Versions

Always provide graceful fallbacks:

```swift
extension View {
    /// Glass effect with capsule shape, falls back to gray background on older OS
    @ViewBuilder
    func adaptiveGlass() -> some View {
        if #available(iOS 26.0, *) {
            self.glassEffect()
        } else {
            self
                .background(Color(.systemGray6))
                .clipShape(Capsule())
        }
    }
    
    /// Glass effect with rounded rect, falls back to gray background
    @ViewBuilder
    func adaptiveGlassRect(cornerRadius: CGFloat) -> some View {
        if #available(iOS 26.0, *) {
            self.glassEffect(in: .rect(cornerRadius: cornerRadius))
        } else {
            self
                .background(Color(.systemGray6))
                .clipShape(RoundedRectangle(cornerRadius: cornerRadius))
        }
    }
    
    /// Glass button style, falls back to bordered on older OS
    @ViewBuilder
    func adaptiveGlassButton() -> some View {
        if #available(iOS 26.0, *) {
            self.buttonStyle(.glass)
        } else {
            self.buttonStyle(.bordered)
        }
    }
}

// Usage
Button("Action") { }
    .adaptiveGlassButton()

Text("Label")
    .padding()
    .adaptiveGlass()

Text("Card")
    .padding()
    .adaptiveGlassRect(cornerRadius: 16)
```

---

## Performance Guidelines

### DO ✅

1. **Use `GlassEffectContainer`** for multiple glass elements
2. **Be mindful of count** - glass effects are GPU-intensive; start with few and test
3. **Apply after** other appearance modifiers
4. **Use `@Namespace`** to enable efficient morphing
5. **Test on older devices** for performance impact

### DON'T ❌

1. **Animate glass properties directly** - use view transitions instead
2. **Use on full-screen backgrounds** - too resource intensive
3. **Apply to dense text content** - reduces readability
4. **Nest glass effects** - unpredictable z-order behavior
5. **Use without fallbacks** - breaks on iOS 25 and earlier

---

## Testing Checklist

- [ ] Works on iOS 26 Simulator
- [ ] Fallback works on iOS 25
- [ ] Glass morphing animates smoothly
- [ ] Interactive states respond to touch
- [ ] Performance acceptable on iPhone 12 (baseline)
- [ ] Text remains readable over glass
- [ ] Dark mode appearance correct
- [ ] VoiceOver accessibility maintained

---

## References

- [Apple: Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)
- [Apple: Landmarks - Building an app with Liquid Glass](https://developer.apple.com/documentation/SwiftUI/Landmarks-Building-an-app-with-Liquid-Glass)
- [SwiftUI GlassEffect](https://developer.apple.com/documentation/SwiftUI/View/glassEffect(_:in:isEnabled:))
- [SwiftUI GlassEffectContainer](https://developer.apple.com/documentation/SwiftUI/GlassEffectContainer)
- [UIKit UIGlassEffect](https://developer.apple.com/documentation/UIKit/UIGlassEffect)

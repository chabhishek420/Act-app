# Developer Guide: SwiftUI Liquid Glass

## Introduction
Introduced in iOS 26 (WWDC 2025), **Liquid Glass** is the new design aesthetic for Apple platforms. It moves beyond standard frosted glass to dynamic, morphing materials that react to touch, light, and motion.

## Key Modifiers

### 1. `.glassEffect()`
The core modifier to apply the Liquid Glass material.
```swift
Text("Hello Liquid Glass")
    .glassEffect()
```

### 2. `.glassEffect(in: ...)`
Specify the shape of the glass container.
```swift
RoundedRectangle(cornerRadius: 20)
    .glassEffect(in: .rect(cornerRadius: 20))
```

### 3. `.interactive()`
Enables touch-reactive ripples and light reflections on the glass surface.
```swift
Button("Press Me") {
    // Action
}
.glassEffect(.regular.interactive())
```

## Morphing Transitions
One of the most powerful features of Liquid Glass is its ability to "melt" or morph between states when combined with `GlassEffectContainer`.

```swift
GlassEffectContainer(spacing: 20) {
    HStack {
        Circle()
            .glassEffect()
            .glassEffectID("circle", in: namespace)
        
        if isExpanded {
            Rectangle()
                .glassEffect()
                .glassEffectID("rect", in: namespace)
        }
    }
}
```

## Best Practices (January 2026)
- **Hierarchy**: Use `GlassEffectContainer` for multiple elements to enable shared light reflections.
- **Contrast**: Use the `.shadow()` or `.stroke()` sparingly; let the Liquid Glass's native refraction handle depth.
- **Performance**: While GPU-accelerated, avoid many overlapping morphing containers in complex lists; prefer static glass for background elements.

---
*Information compiled from January 2026 developer documentation updates.*

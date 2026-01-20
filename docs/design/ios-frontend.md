# iOS Frontend Documentation

## UI Framework
**Choice:** SwiftUI  
**Justification:** Fast iteration, streaming-friendly UI, modern state patterns for chat and tool execution feedback.

## Architecture
**Pattern:** MVVM with @Observable (iOS 17+)

**Data Flow:**
```
View → ViewModel → Services → Composio Swift SDK / OpenAI
     ← @Observable state updates ←
```

## Navigation
- **Primary:** Chat-first (single main screen) with slide-out sidebar
- **Sidebar:** History, Connections, Settings (triggered by hamburger menu)
- **Secondary:**
  - Modal/sheet for destructive tool call confirmations
  - OAuth connect flow via `ASWebAuthenticationSession`
- **No Tab Bar:** Following ChatGPT iOS UI pattern

## Screen Inventory

| Screen | Purpose | Key Components |
|--------|---------|----------------|
| Chat | Intent → plan → execute | Message list, streaming text, tool call chips |
| Confirmation | Approve/deny actions | Diff-style summary, "Approve", "Cancel", args preview |
| Connections | Connect toolkits | Toolkit list, status pill, "Connect" button |
| Settings | Configure app | API key entry, confirmation mode picker |
| History | Review runs | Conversation list, execution log |

## State Management
- **Local State:** `@State` for input text, toggles, sheet visibility
- **Shared State:** `@Observable` app state (user, session IDs, confirmation mode)
- **Server State:** async/await requests; represent as `Loading / Error / Data`

## Reusable Components

### MessageRow
```swift
struct MessageRow: View {
    let message: Message
    
    var body: some View {
        HStack(alignment: .top, spacing: 12) {
            // Role indicator
            Image(systemName: message.role == .user ? "person.fill" : "brain.head.profile")
                .foregroundStyle(message.role == .user ? .blue : .green)
            
            VStack(alignment: .leading, spacing: 8) {
                // Message content with markdown
                Text(LocalizedStringKey(message.content))
                
                // Tool call badges
                if let toolCalls = message.toolCalls {
                    ForEach(toolCalls) { call in
                        ToolCallBadge(call: call)
                    }
                }
            }
        }
        .padding(.vertical, 8)
    }
}
```

### ToolCallBadge
```swift
struct ToolCallBadge: View {
    let call: ToolCall
    
    var body: some View {
        Label(call.toolSlug, systemImage: call.status.icon)
            .font(.caption)
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .background(call.status.color.opacity(0.2))
            .clipShape(Capsule())
    }
}

extension ToolCall.Status {
    var icon: String {
        switch self {
        case .pending: return "clock"
        case .running: return "arrow.triangle.2.circlepath"
        case .success: return "checkmark.circle.fill"
        case .failure: return "xmark.circle.fill"
        }
    }
    
    var color: Color {
        switch self {
        case .pending: return .gray
        case .running: return .blue
        case .success: return .green
        case .failure: return .red
        }
    }
}
```

### StreamingTextView
```swift
struct StreamingTextView: View {
    let text: String
    
    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                Text(text)
                    .textSelection(.enabled)
                    .id("bottom")
            }
            .onChange(of: text) {
                withAnimation {
                    proxy.scrollTo("bottom", anchor: .bottom)
                }
            }
        }
    }
}
```

## Theming

### Standard (iOS 17+)
- Light/Dark mode with system colors
- Use semantic roles (primary/secondary/error)
- Emphasize "action" affordances

### iOS 26+ Enhancement: Liquid Glass

> **Canonical Implementation Guide**: [liquid-glass-implementation.md](./liquid-glass-implementation.md)  
> **Apple Design Resource**: [SwiftUI-Implementing-Liquid-Glass-Design.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md)
> **App Intents Support**: [AppIntents-Updates.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/AppIntents-Updates.md)
> **Design Spec**: [ui-design-system.md § 10](./ui-design-system.md#10-liquid-glass-ios-26)

#### When to Use Liquid Glass in Act

| Component | Liquid Glass? | Notes |
|:---|:---:|:---|
| NavigationBar | ✅ Yes | System applies automatically |
| Composer Input | ✅ Yes | Use `.glassEffect()` |
| Action Buttons | ✅ Yes | Use `.buttonStyle(.glass)` |
| Tool Status Badges | ✅ Yes | Interactive glass effect |
| Message Bubbles | ❌ No | Keep flat for readability |
| Confirmation Sheet | ✅ Yes | System-provided |

#### Glass Tool Badge (iOS 26+)
```swift
@available(iOS 26, *)
struct GlassToolBadge: View {
    let call: ToolCall
    @Namespace private var badgeNamespace
    
    var body: some View {
        Label(call.toolSlug, systemImage: call.status.icon)
            .font(.caption)
            .padding(.horizontal, 10)
            .padding(.vertical, 6)
            .foregroundStyle(call.status.color)
            .glassEffect(.regular.tint(call.status.color).interactive())
            .glassEffectID(call.id, in: badgeNamespace)
    }
}
```

#### Adaptive Helper for iOS 17-25 Fallback
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
}
```

#### Glass Composer (iOS 26+)
```swift
@available(iOS 26, *)
struct GlassComposer: View {
    @Binding var text: String
    @Namespace private var composerNamespace
    
    var body: some View {
        GlassEffectContainer(spacing: 8) {
            HStack(spacing: 8) {
                Button(action: {}) {
                    Image(systemName: "plus")
                        .frame(width: 36, height: 36)
                }
                .glassEffect()
                .glassEffectID("attach", in: composerNamespace)
                
                TextField("Ask anything...", text: $text)
                    .padding(.horizontal, 16)
                    .frame(height: 44)
                    .glassEffect(in: .capsule)
                    .glassEffectID("input", in: composerNamespace)
                
                Button(action: {}) {
                    Image(systemName: "arrow.up")
                        .frame(width: 36, height: 36)
                }
                .glassEffect(.regular.interactive())
                .glassEffectID("send", in: composerNamespace)
            }
        }
    }
}
```

## Accessibility
- VoiceOver labels for messages and tool badges
- Dynamic Type for chat text
- Minimum tap target 44x44

## Localization
- English-first
- Structure strings in `.strings` files
- Use String Catalogs for iOS 17+

## Confirmation Mode UI

### Settings Screen
```swift
Picker("Confirmation Mode", selection: $confirmationMode) {
    Text("Always Ask").tag(ConfirmationMode.alwaysAsk)
    Text("Adaptive").tag(ConfirmationMode.adaptive)
    Text("YOLO (Auto)").tag(ConfirmationMode.yolo)
}
.pickerStyle(.segmented)
```

### Confirmation Sheet
```swift
.sheet(item: $pendingConfirmation) { toolCall in
    ConfirmationSheet(
        toolSlug: toolCall.slug,
        description: toolCall.summary,
        arguments: toolCall.sanitizedArgs,
        onApprove: { await execute(toolCall) },
        onCancel: { pendingConfirmation = nil }
    )
}
```

## API Key Entry

```swift
struct APIKeyEntryView: View {
    @State private var apiKey = ""
    @State private var isSecure = true
    
    var body: some View {
        HStack {
            if isSecure {
                SecureField("sk-proj-...", text: $apiKey)
            } else {
                TextField("sk-proj-...", text: $apiKey)
            }
            
            Button {
                isSecure.toggle()
            } label: {
                Image(systemName: isSecure ? "eye" : "eye.slash")
            }
        }
        .textFieldStyle(.roundedBorder)
    }
}
```

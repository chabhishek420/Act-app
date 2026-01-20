# UI Design Guide

## Design Principles

Based on ChatGPT iOS wireframes and iOS 26 design system:

1. **Chat-centric**: Messages are the primary UI element
2. **Minimal chrome**: Focus on content, not navigation
3. **Clear status**: Tool execution states must be instantly visible
4. **Progressive disclosure**: Details on demand, summaries first

---

## iOS 26+ Liquid Glass (Optional Enhancement)

> **Canonical Reference**: [liquid-glass-implementation.md](./liquid-glass-implementation.md)  
> **Design Spec**: [ui-design-system.md § 10](./ui-design-system.md#10-liquid-glass-ios-26)

Liquid Glass is Apple's dynamic material for iOS 26+ that blurs content, reflects surrounding colors, and reacts to touch.

### When to Use

| Component | Use Glass? | Rationale |
|:---|:---:|:---|
| Tool execution badges | ✅ | Interactive, prominent |
| Composer input bar | ✅ | Floating, tactile |
| Action buttons | ✅ | Responsive feel |
| Message bubbles | ❌ | Readability priority |
| Settings rows | ❌ | Too subtle for lists |

### Basic Glass Effect

```swift
@available(iOS 26, *)
struct GlassCard: View {
    let title: String
    let subtitle: String
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(title).font(.headline)
            Text(subtitle).font(.subheadline).foregroundStyle(.secondary)
        }
        .padding()
        .glassEffect(.regular, in: .rect(cornerRadius: 16))
    }
}
```

### Interactive Glass Button

```swift
@available(iOS 26, *)
struct GlassToolButton: View {
    let toolSlug: String
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Label(toolSlug, systemImage: "wrench.fill")
                .padding()
        }
        .glassEffect(.regular.interactive())
    }
}
```

### Glass Effect Container (Multiple Elements)

When using multiple glass effects that should merge/morph:

```swift
@available(iOS 26, *)
struct GlassToolbar: View {
    @Namespace private var toolbarNamespace
    
    var body: some View {
        GlassEffectContainer(spacing: 12) {
            HStack(spacing: 12) {
                ForEach(["message.fill", "gear", "magnifyingglass"], id: \.self) { icon in
                    Image(systemName: icon)
                        .frame(width: 44, height: 44)
                        .glassEffect()
                        .glassEffectID(icon, in: toolbarNamespace)
                }
            }
        }
    }
}
```

### Fallback for iOS 17-25

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

---

## Authentication Screens

From ChatGPT wireframe analysis:

### Welcome Screen

```swift
struct WelcomeView: View {
    var body: some View {
        VStack(spacing: 24) {
            Spacer()
            
            // App icon
            Image("AppIcon")
                .resizable()
                .frame(width: 80, height: 80)
                .clipShape(RoundedRectangle(cornerRadius: 16))
            
            Text("Act")
                .font(.largeTitle.bold())
            
            Text("Action-first AI assistant")
                .foregroundStyle(.secondary)
            
            Spacer()
            
            // Auth buttons
            VStack(spacing: 12) {
                SignInWithAppleButton()
                    .frame(height: 50)
                
                Button("Continue with Google") {
                    // Google OAuth
                }
                .buttonStyle(.borderedProminent)
                .frame(maxWidth: .infinity)
                .frame(height: 50)
                
                Button("Sign up with email") {
                    // Email flow
                }
                .buttonStyle(.bordered)
                .frame(maxWidth: .infinity)
            }
            .padding(.horizontal)
        }
        .padding()
    }
}
```

---

## Chat Interface

### Message Composer

```swift
struct ComposerView: View {
    @Binding var text: String
    @FocusState private var isFocused: Bool
    let onSend: () -> Void
    
    var body: some View {
        HStack(spacing: 12) {
            TextField("Ask anything...", text: $text, axis: .vertical)
                .textFieldStyle(.plain)
                .focused($isFocused)
                .lineLimit(1...5)
            
            Button(action: onSend) {
                Image(systemName: "arrow.up.circle.fill")
                    .font(.title)
            }
            .disabled(text.trimmingCharacters(in: .whitespaces).isEmpty)
        }
        .padding()
        .background(.ultraThinMaterial)
    }
}
```

### Message List

```swift
struct MessageListView: View {
    let messages: [Message]
    
    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                LazyVStack(spacing: 0) {
                    ForEach(messages) { message in
                        MessageRow(message: message)
                            .id(message.id)
                    }
                }
                .padding()
            }
            .onChange(of: messages.count) {
                if let last = messages.last {
                    withAnimation {
                        proxy.scrollTo(last.id, anchor: .bottom)
                    }
                }
            }
        }
    }
}
```

---

## Tool Execution States

### Status Badge

```swift
struct ToolStatusBadge: View {
    let status: ToolCallStatus
    let toolSlug: String
    
    var body: some View {
        HStack(spacing: 6) {
            statusIcon
            Text(toolSlug)
                .font(.caption.monospaced())
        }
        .padding(.horizontal, 10)
        .padding(.vertical, 6)
        .background(statusColor.opacity(0.15))
        .clipShape(Capsule())
    }
    
    @ViewBuilder
    private var statusIcon: some View {
        switch status {
        case .pending:
            Image(systemName: "clock")
                .foregroundStyle(.gray)
        case .running:
            ProgressView()
                .controlSize(.small)
        case .success:
            Image(systemName: "checkmark.circle.fill")
                .foregroundStyle(.green)
        case .failure:
            Image(systemName: "xmark.circle.fill")
                .foregroundStyle(.red)
        }
    }
    
    private var statusColor: Color {
        switch status {
        case .pending: return .gray
        case .running: return .blue
        case .success: return .green
        case .failure: return .red
        }
    }
}
```

---

## Confirmation Sheet

```swift
struct ConfirmationSheet: View {
    let toolSlug: String
    let description: String
    let arguments: [String: String]
    let onApprove: () -> Void
    let onCancel: () -> Void
    
    var body: some View {
        NavigationStack {
            VStack(alignment: .leading, spacing: 16) {
                // Tool info
                Label(toolSlug, systemImage: "wrench.fill")
                    .font(.headline)
                
                Text(description)
                    .foregroundStyle(.secondary)
                
                // Arguments preview
                if !arguments.isEmpty {
                    VStack(alignment: .leading, spacing: 8) {
                        Text("Parameters")
                            .font(.subheadline.bold())
                        
                        ForEach(Array(arguments.keys.sorted()), id: \.self) { key in
                            HStack {
                                Text(key)
                                    .foregroundStyle(.secondary)
                                Spacer()
                                Text(arguments[key] ?? "")
                                    .lineLimit(1)
                            }
                            .font(.caption.monospaced())
                        }
                    }
                    .padding()
                    .background(Color(.secondarySystemBackground))
                    .clipShape(RoundedRectangle(cornerRadius: 8))
                }
                
                Spacer()
            }
            .padding()
            .navigationTitle("Confirm Action")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel", action: onCancel)
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Approve", action: onApprove)
                        .bold()
                }
            }
        }
        .presentationDetents([.medium])
    }
}
```

---

## Connection Status

```swift
struct ConnectionStatusView: View {
    let toolkit: String
    let isConnected: Bool
    let onConnect: () -> Void
    
    var body: some View {
        HStack {
            // Toolkit icon (placeholder)
            Circle()
                .fill(.gray.opacity(0.3))
                .frame(width: 40, height: 40)
                .overlay {
                    Text(toolkit.prefix(1).uppercased())
                        .font(.headline)
                }
            
            VStack(alignment: .leading) {
                Text(toolkit.capitalized)
                    .font(.headline)
                
                Text(isConnected ? "Connected" : "Not connected")
                    .font(.caption)
                    .foregroundStyle(isConnected ? .green : .secondary)
            }
            
            Spacer()
            
            if isConnected {
                Image(systemName: "checkmark.circle.fill")
                    .foregroundStyle(.green)
            } else {
                Button("Connect", action: onConnect)
                    .buttonStyle(.borderedProminent)
                    .controlSize(.small)
            }
        }
        .padding()
    }
}
```

---

## Color Palette

| Role | Light | Dark | Usage |
|------|-------|------|-------|
| Primary | System Blue | System Blue | Buttons, links |
| Success | System Green | System Green | Tool success |
| Warning | System Orange | System Orange | Caution states |
| Error | System Red | System Red | Failures |
| Background | System Background | System Background | Main bg |
| Secondary BG | Secondary System BG | Secondary System BG | Cards |

---

## Typography

| Element | Style | Example |
|---------|-------|---------|
| Message | `.body` | User/assistant text |
| Tool slug | `.caption.monospaced()` | `GMAIL_SEND_EMAIL` |
| Section header | `.headline` | "Parameters" |
| Metadata | `.caption` | Timestamps |

---

## Accessibility

- All interactive elements: `accessibilityLabel`
- Tool status changes: `accessibilityValue`
- Minimum tap target: 44x44 points
- VoiceOver focus order: logical reading order

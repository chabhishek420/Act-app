# User Flow Documentation

## Core User Journey

```mermaid
flowchart TD
    A[Launch Act] --> B{First Launch?}
    B -->|Yes| C[Welcome Screen]
    B -->|No| D[Chat Screen]
    
    C --> E[Sign In Options]
    E --> F[Enter API Keys]
    F --> G[Setup Complete]
    G --> D
    
    D --> H[User Types Message]
    H --> I[Create/Get Session]
    I --> J[Search Tools]
    J --> K{Toolkit Connected?}
    
    K -->|No| L[Show Connect CTA]
    L --> M[OAuth Flow]
    M --> K
    
    K -->|Yes| N[Send to LLM]
    N --> O[LLM Returns Tool Calls]
    O --> P{Needs Confirmation?}
    
    P -->|Yes| Q[Show Confirmation Sheet]
    Q -->|Approve| R[Execute Tools]
    Q -->|Cancel| S[Cancel & Update Message]
    
    P -->|No| R
    R --> T[Display Results]
    T --> H
```

---

## Session Lifecycle

```mermaid
sequenceDiagram
    participant U as User
    participant V as ChatViewModel
    participant SM as SessionManager
    participant C as Composio SDK
    
    U->>V: Send message
    V->>SM: getSession(userId, convId)
    
    alt Session cached & valid
        SM-->>V: Return cached session
    else No session or expired
        SM->>C: createSession(for: userId, toolkits: [...])
        C-->>SM: ToolRouterSession
        SM-->>V: Return new session
    end
    
    V->>C: executeMeta("COMPOSIO_SEARCH_TOOLS", ...)
    C-->>V: Tool search results
    
    V->>V: Send to OpenAI with tools
    V->>V: Parse tool calls from response
    
    loop For each tool call
        V->>C: execute(toolSlug, in: sessionId, arguments: ...)
        C-->>V: ToolRouterExecuteResponse
    end
    
    V-->>U: Display results
```

---

## OAuth Flow

```mermaid
sequenceDiagram
    participant U as User
    participant A as Act App
    participant O as OAuthManager
    participant C as Composio API
    participant B as OAuth Provider (GitHub, Gmail, etc.)
    
    U->>A: Tap "Connect Gmail"
    A->>O: authenticate(userId, toolkit: "gmail")
    O->>C: createLink(for: "gmail", in: sessionId)
    C-->>O: redirectUrl
    
    O->>B: Open ASWebAuthenticationSession(url)
    B-->>U: OAuth consent screen
    U->>B: Approve
    B-->>O: Callback URL
    
    O->>C: Poll waitForConnection(id, timeout: 60)
    C-->>O: ConnectedAccount (status: active)
    O-->>A: Return account
    A-->>U: Show "Connected" status
```

---

## Confirmation Logic

### Decision Tree

```mermaid
flowchart TD
    A[Tool Call] --> B{Confirmation Mode?}
    
    B -->|Always Ask| C[Show Confirmation]
    B -->|YOLO| D{Is DELETE_* ?}
    B -->|Adaptive| E{Risk Category?}
    
    D -->|Yes| C
    D -->|No| F[Auto Execute]
    
    E -->|High| C
    E -->|Medium| G{User Setting?}
    E -->|Low| F
    
    G -->|Cautious| C
    G -->|Normal| F
    
    C -->|Approve| F
    C -->|Cancel| H[Cancel Action]
```

### Risk Categories

| Category | Examples | Default Action |
|----------|----------|----------------|
| Low | FETCH, GET, SEARCH, LIST | Auto-execute |
| Medium | CREATE_DRAFT, UPDATE (private) | Depends on mode |
| High | SEND, DELETE, POST (public) | Always confirm |

---

## Error Recovery Flows

### Connection Required

```mermaid
flowchart TD
    A[Tool Execution] --> B{Connected?}
    B -->|No| C[Display "Connect Gmail" button]
    C --> D[User taps Connect]
    D --> E[OAuth Flow]
    E --> F{Success?}
    F -->|Yes| G[Retry original request]
    F -->|No| H[Show error + retry option]
```

### Rate Limited

```mermaid
flowchart TD
    A[Tool Execution] --> B{Rate Limited?}
    B -->|Yes| C[Show countdown timer]
    C --> D[Wait retryAfter seconds]
    D --> E[Auto-retry]
    E --> A
```

### API Key Missing

```mermaid
flowchart TD
    A[Send Message] --> B{API Key Set?}
    B -->|No| C[Show Settings Link]
    C --> D[Navigate to Settings]
    D --> E[User enters key]
    E --> F[Save to Keychain]
    F --> G[Return to Chat]
    G --> A
```

---

## Screen Flow Summary

| Screen | Entry Points | Exit Points |
|--------|--------------|-------------|
| Welcome | App launch (first time) | Settings complete → Chat |
| Chat | App launch (always visible) | Confirmation sheet, Sidebar |
| Confirmation | Tool needs approval | Approve → Chat, Cancel → Chat |
| Connections | Sidebar → Connections | OAuth success → Chat |
| Settings | Sidebar → Profile/Settings | Save → Chat |

---

## Streaming Response Flow

```mermaid
sequenceDiagram
    participant U as User
    participant V as ChatViewModel  
    participant O as OpenAI
    participant UI as ChatView

    U->>V: sendMessage(text)
    V->>V: ensureSession()
    V->>O: streamChat(messages)
    
    loop Streaming
        O-->>V: AsyncThrowingStream chunk
        V->>UI: streamingContent += chunk
        UI->>UI: Re-render with partial content
    end
    
    O-->>V: Stream complete
    V->>V: Parse tool calls from response
    V->>UI: Update messages array
```

### Implementation Pattern

```swift
for try await chunk in streamChat(message: text) {
    await MainActor.run { 
        streamingContent += chunk 
    }
}
```

---

## Deep Links

| URL Scheme | Purpose | Handler |
|------------|---------|---------|
| `act://oauth-callback` | OAuth completion | OAuthManager |
| `act://chat?prompt=...` | Open chat with prompt | ChatView |
| `act://settings` | Open settings | SettingsView |

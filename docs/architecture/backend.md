# Backend Documentation

## Architecture Overview

Act uses a **direct client-side architecture** for MVP, with the Composio Swift SDK handling tool execution and the user's own LLM API key for inference.

```
iOS App (Act)
  ├─ OpenAI Swift SDK (user's API key in Keychain)
  │    └─ LLM inference (streaming chat completions)
  ├─ Composio Swift SDK (local package)
  │    ├─ Tool Router sessions
  │    ├─ OAuth (ASWebAuthenticationSession)
  │    ├─ Meta-tool execution (COMPOSIO_SEARCH_TOOLS)
  │    └─ Tool execution
  └─ SwiftData (local persistence)
       ├─ Conversations
       ├─ Messages
       └─ Tool execution logs
```

---

## LLM Integration (MVP: Direct API)

### Why Direct API for MVP?
- **Ships faster**: No backend to build/deploy
- **Simpler ops**: No server to monitor
- **Transparent costs**: User pays OpenAI directly
- **Privacy**: Data stays on device + OpenAI only

### User API Key Flow
1. User enters OpenAI key in Settings
2. Key stored in iOS Keychain (encrypted)
3. Displayed as masked: `sk-proj-xxxx...`
4. Used directly for Chat Completions API

### Implementation

```swift
import Keychain

// Store key securely
try Keychain.save(apiKey, key: "openai_api_key")

// Retrieve for requests
guard let apiKey = Keychain.get("openai_api_key") else {
    throw AppError.missingAPIKey
}

// Initialize OpenAI client
let openAI = OpenAI(apiToken: apiKey)
```

### Streaming Responses

```swift
let stream = openAI.chatsStream(query: ChatQuery(
    model: .gpt4o,
    messages: messages
))

for try await result in stream {
    if let content = result.choices.first?.delta.content {
        await MainActor.run {
            self.currentResponse += content
        }
    }
}
```

---

## Composio Integration

### SDK Source
The Composio Swift SDK is a **custom implementation** located at `../composio-swift/`. It is not an official Composio SDK.

### Key Methods (from actual SDK source)

```swift
import Composio

// Initialize
let composio = try Composio(apiKey: composioApiKey)

// Create Tool Router session (official pattern)
let session = try await composio.create(
    userId: userId,
    toolkits: ["gmail", "slack", "github"]
)

// Search tools with execution guidance
let response = try await composio.toolRouter.executeMeta(
    "COMPOSIO_SEARCH_TOOLS",
    in: session.sessionId,
    arguments: ["queries": [["use_case": "send email via gmail"]]]
)

// Execute tool
let result = try await composio.toolRouter.execute(
    "GMAIL_SEND_EMAIL",
    in: session.sessionId,
    arguments: ["to": "user@example.com", "subject": "Hello"]
)

// OAuth flow (create link via Tool Router)
let link = try await composio.toolRouter.createLink(
    for: "gmail",
    in: session.sessionId
)
// Then open link.redirectUrl in ASWebAuthenticationSession
```

---

## Error Recovery (Production Pattern)

Implement retry logic for transient failures:

```swift
func executeWithRecovery(_ tool: String, in session: String, args: [String: Any], attempt: Int = 1) async throws -> ToolRouterExecuteResponse {
    do {
        return try await composio.toolRouter.execute(tool, in: session, arguments: args)
    } catch let error as ComposioError {
        switch error {
        case .notAuthenticated(let toolkit):
            // Toolkit auth failure - reconnect
            guard attempt < 3 else { throw error }
            try await reconnectToolkit(toolkit, in: session)
            return try await executeWithRecovery(tool, in: session, args: args, attempt: attempt + 1)
        case .rateLimited(let retry):
            try await Task.sleep(nanoseconds: UInt64(retry ?? 5) * 1_000_000_000)
            return try await executeWithRecovery(tool, in: session, args: args, attempt: attempt + 1)
        case .networkError:
            guard attempt < 2 else { throw error }
            try await Task.sleep(nanoseconds: 2_000_000_000)
            return try await executeWithRecovery(tool, in: session, args: args, attempt: attempt + 1)
        default: throw error
        }
    }
}
```

### Recovery Rules
| Error | Action | Max Attempts |
|-------|--------|--------------|
| `.notAuthenticated(toolkit:)` | Re-authenticate specific toolkit | 3 |
| 429 / `.rateLimited` | Backoff using `retryAfter` | 5 |
| `.networkError` | Retry with 2s delay | 2 |
| `.unauthorized` / `.forbidden` | Cannot recover - rethrow | 0 |

---

## Local Persistence (SwiftData)

### Why SwiftData for MVP?
- Built-in to iOS 17+
- No external dependencies
- Automatic iCloud sync (optional)
- Type-safe Swift models

### Models

```swift
@Model
class Conversation {
    var id: UUID
    var title: String
    var toolRouterSessionId: String?
    var createdAt: Date
    var messages: [Message]
}

@Model
class Message {
    var id: UUID
    var role: MessageRole
    var content: String
    var toolCalls: [ToolCallData]?
    var createdAt: Date
}
```

---

## Phase 2+: Alternative Architectures

### Option B: Cloud Backend
When to consider:
- Need to hide API keys from users
- Want central logging/analytics
- Multi-user billing

Trade-offs:
- +4-6 weeks to build
- Ongoing ops cost
- Backend expertise required

### Option C: Apple Foundation Models (iOS 26+)

> **Full Documentation**: See `.temp/xcode-26-system-prompts/AdditionalDocumentation/FoundationModels-Using-on-device-LLM-in-your-app.md`

When to consider:
- Privacy-critical use cases
- Offline functionality required
- Target devices: iPhone 15 Pro+, M-series Macs

#### Basic Usage

```swift
import FoundationModels

// Check availability
guard SystemLanguageModel.default.availability == .available else {
    // Fall back to cloud LLM
    return
}

// Create session with instructions
let session = LanguageModelSession(instructions: """
    You are an action-first AI assistant. Execute tasks immediately.
    Only use tools returned by COMPOSIO_SEARCH_TOOLS.
""")

// Generate response
let response = try await session.respond(to: userMessage)
print(response.content)
```

#### Structured Output (Guided Generation)

```swift
import FoundationModels

@Generable
struct ToolCallDecision {
    @Guide(description: "The tool slug to execute")
    var toolSlug: String
    
    @Guide(description: "Arguments for the tool")
    var arguments: [String: String]
    
    @Guide(description: "Whether user confirmation is needed")
    var requiresConfirmation: Bool
}

let decision: ToolCallDecision = try await session.respond(
    to: "Send an email to john@example.com",
    generating: ToolCallDecision.self
)
```

#### Limitations
- 4,096 token context limit
- No streaming for structured output
- Device restrictions (A17+, M-series)
- Not for complex multi-step reasoning

### Option D: Appwrite BaaS (Cross-Device Sync)
When to consider:
- Multi-device sync required
- Team collaboration features
- User account management

---

## Operational Notes

- **Session TTL**: Tool Router sessions expire after 1 hour - cache and reuse
- **API Version**: Composio v3 endpoints only (v1 deprecated)
- **Error Handling**: Log `log_id` from Composio responses for debugging
- **Rate Limits**: Handle 429 with exponential backoff

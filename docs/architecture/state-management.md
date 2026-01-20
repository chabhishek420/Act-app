# State Management

## Approach
**Pattern:** MVVM with `@Observable` (iOS 17+) and async/await services  
**Goal:** Keep UI reactive while tool execution runs concurrently and streams output.

## State Categories

### Local State
- Purpose: input field text, sheet presentation, focus state
- Implementation: `@State` in views

### Shared State
- Purpose: current user ID, active conversation, confirmation mode
- Implementation: `@Observable` model injected via `@Environment`

```swift
@Observable
final class AppState {
    var userId: String = ""
    var confirmationMode: ConfirmationMode = .adaptive
    var activeConversationId: UUID?
    var isOnboarded: Bool = false
    var composioApiKey: String? = nil
    var openAIApiKey: String? = nil
}
```

### Server State
- Purpose: Composio sessions, tool schemas, connection statuses, tool execution results
- Implementation: ViewModels hold `loading/error/data` for each async request

```swift
@Observable
final class ChatViewModel {
    // Loading states
    private(set) var isLoading = false
    private(set) var isStreaming = false
    private(set) var error: Error?
    
    // Data
    private(set) var messages: [Message] = []
    private(set) var activeToolCalls: [ToolCall] = []
    
    // Confirmation
    var pendingConfirmation: ToolCall?
    
    // Services
    private let sessionManager: SessionManager
    private let toolService: ToolExecutionService
    private let llmService: LLMService
}
```

## Data Flow

```
ChatView
  → user submits text
ChatViewModel
  → ensureSession()           // SessionManager.getSession()
  → searchTools()             // executeMeta("COMPOSIO_SEARCH_TOOLS", ...)
  → checkConnections()        // From toolkit_connection_statuses in response
  → connectIfNeeded()         // OAuthManager.authenticate()
  → sendToLLM()               // OpenAI streaming chat completion
  → confirmIfNeeded()         // Check confirmation mode + risk level
  → executeToolCalls()        // toolRouter.execute(_:in:arguments:)
  → appendMessages()          // Update UI
```

## SessionManager (Actor)

Session management uses Swift actor for thread safety. Matches actual SDK API:

```swift
import Composio

actor SessionManager {
    private var cache: [String: CachedSession] = [:]
    private let ttl: TimeInterval = 3600  // 1 hour
    private let composio: Composio
    
    struct CachedSession {
        let session: ToolRouterSession
        let createdAt: Date
        
        var isExpired: Bool {
            Date().timeIntervalSince(createdAt) >= 3600
        }
    }
    
    init(composio: Composio) {
        self.composio = composio
    }
    
    func getSession(
        for userId: String,
        conversationId: String,
        toolkits: [String] = []
    ) async throws -> ToolRouterSession {
        let key = "\(userId)-\(conversationId)"
        
        if let cached = cache[key], !cached.isExpired {
            return cached.session
        }
        
        // Actual SDK method signature
        let session = try await composio.toolRouter.createSession(
            for: userId,
            toolkits: toolkits.isEmpty ? nil : toolkits
        )
        
        cache[key] = CachedSession(session: session, createdAt: Date())
        return session
    }
    
    func invalidate(for conversationId: String) {
        cache = cache.filter { !$0.key.hasSuffix("-\(conversationId)") }
    }
}
```

## Error Handling

Handle Composio SDK error types from `ComposioError`:

```swift
do {
    let result = try await toolService.execute(toolSlug, in: sessionId, arguments: args)
    // Success
} catch ComposioError.notAuthenticated(let toolkit) {
    // Show connect button for toolkit
    pendingConnection = toolkit
} catch ComposioError.rateLimited(let retryAfter) {
    // Show retry countdown
    self.retryAfter = retryAfter ?? 5
} catch ComposioError.networkError(let underlying) {
    // Generic network error
    self.error = underlying
} catch {
    self.error = error
}

## State Management Pattern

Act uses **@Observable** (iOS 17+) with SwiftUI for reactive state management.

### ChatViewModel

```swift
import Observation

@Observable
@MainActor
class ChatViewModel {
    var messages: [Message] = []
    var isTyping: Bool = false
    
    func sendMessage(_ text: String) {
        // State updates automatically trigger UI refresh
    }
}
```

### SwiftUI View Binding

```swift
struct ChatView: View {
    @State private var viewModel = ChatViewModel()
    
    var body: some View {
        // Automatically observes viewModel.messages
        ForEach(viewModel.messages) { message in
            MessageRow(message: message)
        }
    }
}
```
- **Loading:** show inline spinner in chat + "Executing…" badge per tool
- **Error:** map to user-actionable messages (connect, retry, open settings)
- **Empty:** no conversations yet → show sample prompts and "Connect toolkits" CTA

```swift
struct ChatView: View {
    @Environment(ChatViewModel.self) var viewModel
    
    var body: some View {
        Group {
            if viewModel.isLoading && viewModel.messages.isEmpty {
                ProgressView("Loading...")
            } else if let error = viewModel.error {
                ErrorView(error: error, retryAction: viewModel.retry)
            } else if viewModel.messages.isEmpty {
                EmptyStateView(onSampleTap: viewModel.sendSample)
            } else {
                MessageListView(messages: viewModel.messages)
            }
        }
    }
}
```

## Confirmation Modeling

Represent pending confirmation as a strongly typed model:

```swift
struct PendingToolExecution: Identifiable {
    let id: UUID
    let toolSlug: String
    let summarizedIntent: String
    let riskCategory: RiskCategory
    let sanitizedArgs: [String: String]
    let targetDescription: String
}

enum RiskCategory {
    case low      // Read-only (FETCH, GET, SEARCH)
    case medium   // Create private (CREATE_DRAFT)
    case high     // Send, delete, public (SEND, DELETE, POST)
}

enum ConfirmationMode: String, Codable {
    case alwaysAsk  // Confirm everything
    case adaptive   // Confirm high-risk only
    case yolo       // Auto-execute all (still confirms DELETE_*)
}
```

## Memory System (Per-Toolkit Facts)

Store ID mappings and user preferences per toolkit. **Must use array of strings, not nested objects:**

```swift
struct ToolkitMemory: Codable {
    var facts: [String]
}

// ✅ Correct format (array of strings):
let memory: [String: [String]] = [
    "slack": [
        "Channel general has ID C1234567",
        "User John has ID U9876"
    ],
    "gmail": [
        "John's email is john@example.com"
    ]
]

// ❌ Wrong format (nested objects - will break):
// "slack": ["channel_id": "C1234567"]
```

This matches Rube's memory format from the production system prompt.

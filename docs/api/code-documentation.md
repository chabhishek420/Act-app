# Code Documentation Standards

## Inline Comments
- Use comments only for non-obvious rules: confirmation heuristics, session TTL decisions, redaction logic
- Avoid comments that restate SwiftUI syntax or trivial property assignments
- Reference Rube principles when implementing core patterns

## DocC Documentation
**Scope:** Public service protocols + core models used across features

**Must document:**
- Session lifecycle (`createSession`, TTL rules, invalidation)
- Tool execution pipeline (meta-tools vs tool slugs)
- Confirmation modes (auto/always/adaptive)
- OAuth flow with ASWebAuthenticationSession

## Folder Structure
```
Act/
├── App/
│   ├── ActApp.swift              # @main entry, dependency injection
│   └── RootView.swift            # Chat + Sidebar container (ChatGPT-style)
├── Features/
│   ├── Chat/
│   │   ├── Views/
│   │   │   ├── ChatView.swift
│   │   │   ├── MessageRow.swift
│   │   │   └── ToolCallBadge.swift
│   │   ├── ViewModels/
│   │   │   └── ChatViewModel.swift
│   │   └── Models/
│   │       └── Message.swift
│   ├── Connections/
│   │   ├── Views/
│   │   │   └── ConnectionsView.swift
│   │   └── ViewModels/
│   │       └── ConnectionsViewModel.swift
│   └── Settings/
│       ├── Views/
│       │   └── SettingsView.swift
│       └── ViewModels/
│           └── SettingsViewModel.swift
├── Services/
│   ├── SessionManager.swift      # Tool Router session caching (1hr TTL)
│   ├── ToolConnectionService.swift
│   ├── ToolExecutionService.swift
│   ├── ConfirmationService.swift
│   └── LLMService.swift          # OpenAI chat completions
├── Data/
│   ├── Persistence.swift         # SwiftData container
│   └── SwiftDataModels/
│       ├── ConversationModel.swift
│       ├── MessageModel.swift
│       └── ToolExecutionModel.swift
├── Utilities/
│   ├── Keychain.swift            # Secure API key storage
│   ├── Redaction.swift           # Sanitize logs
│   └── Logging.swift
└── Tests/
    ├── ChatViewModelTests.swift
    └── SessionManagerTests.swift
```

## Key Service Patterns

### SessionManager (Actor)

Matches actual Composio SDK API from `ToolRouterClient.swift`:

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
        toolkits: [String] = ["gmail", "slack", "github"]
    ) async throws -> ToolRouterSession {
        let key = "\(userId)-\(conversationId)"
        
        if let cached = cache[key], !cached.isExpired {
            return cached.session
        }
        
        // SDK method: createSession(for:toolkits:)
        let session = try await composio.toolRouter.createSession(
            for: userId,
            toolkits: toolkits
        )
        
        cache[key] = CachedSession(session: session, createdAt: Date())
        return session
    }
    
    func invalidate(for conversationId: String) {
        cache = cache.filter { !$0.key.hasSuffix("-\(conversationId)") }
    }
    
    func clearAll() {
        cache.removeAll()
    }
}
```

### ToolExecutionService

Uses actual SDK methods from `ToolRouterClient.swift`:

```swift
import Composio

struct ToolExecutionService {
    private let composio: Composio
    
    init(composio: Composio) {
        self.composio = composio
    }
    
    func execute(
        _ toolSlug: String,
        in sessionId: String,
        arguments: [String: Any]
    ) async throws -> [String: AnyCodable] {
        // SDK method: execute(_:in:arguments:)
        let response = try await composio.toolRouter.execute(
            toolSlug,
            in: sessionId,
            arguments: arguments
        )
        
        // Response is ToolRouterExecuteResponse with data/error/logId
        if let error = response.error {
            throw ToolExecutionError.serverError(error, logId: response.logId)
        }
        
        return response.data
    }
    
    func searchTools(
        in sessionId: String,
        query: String
    ) async throws -> ToolRouterExecuteResponse {
        // SDK method: executeMeta(_:in:arguments:)
        return try await composio.toolRouter.executeMeta(
            "COMPOSIO_SEARCH_TOOLS",
            in: sessionId,
            arguments: ["queries": [["use_case": query]]]
        )
    }
}
```

### OAuthService

Uses actual SDK methods from `OAuthManager.swift`:

```swift
import Composio

@MainActor
final class OAuthService: ObservableObject {
    @Published private(set) var isAuthenticating = false
    
    private let oauthManager: OAuthManager
    
    init(composio: Composio) {
        self.oauthManager = OAuthManager(composio: composio)
    }
    
    func connect(
        userId: String,
        toolkit: String
    ) async throws -> ConnectedAccount {
        isAuthenticating = true
        defer { isAuthenticating = false }
        
        // SDK method: authenticate(userId:toolkit:callbackURLScheme:)
        return try await oauthManager.authenticate(
            userId: userId,
            toolkit: toolkit,
            callbackURLScheme: "act"  // Must match Info.plist URL scheme
        )
    }
}
```

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Types | PascalCase | `ToolExecutionService` |
| Functions/vars | camelCase | `executeToolCalls` |
| Async APIs | Descriptive verb | `fetchTools`, `createSession`, `executeMeta` |
| SDK methods | Positional first param | `execute(_:in:arguments:)` |

## Logging Policy

- Include `log_id` from Composio responses for debugging
- Include tool slug and high-level state transitions
- Never log secrets or full tool arguments
- Use Composio SDK's built-in logging when DEBUG enabled

```swift
func logExecution(_ toolSlug: String, logId: String?) {
    #if DEBUG
    print("[Act] Executed \(toolSlug), logId: \(logId ?? "nil")")
    #endif
}
```

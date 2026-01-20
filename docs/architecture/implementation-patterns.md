# Implementation Patterns

Production patterns extracted from Rube's architecture for the Act iOS app.

---

## 1. Session Management

Sessions are **conversation-scoped**, not app-scoped.

```swift
actor ToolRouterSessionManager {
    private var cache: [String: CachedSession] = [:]
    
    struct CachedSession {
        let sessionId: String
        let mcpUrl: String
        let createdAt: Date
        var isExpired: Bool { Date().timeIntervalSince(createdAt) >= 86400 }
    }
    
    func getOrCreateSession(
        for userId: String,
        conversationId: String,
        toolkits: [String] = []
    ) async throws -> ToolRouterSession {
        let key = "\(userId)-\(conversationId)"
        
        if let cached = cache[key], !cached.isExpired {
            return ToolRouterSession(sessionId: cached.sessionId)
        }
        
        let session = try await composio.create(userId: userId, toolkits: toolkits.isEmpty ? nil : toolkits)
        cache[key] = CachedSession(sessionId: session.sessionId, mcpUrl: session.mcp?.url ?? "", createdAt: Date())
        return session
    }
}
```

---

## 2. Tool Discovery First

**Always** call `COMPOSIO_SEARCH_TOOLS` before executing any tool.

```swift
func fetchExecutionPlan(for intent: String, in sessionId: String) async throws -> ToolExecutionPlan {
    let response = try await composio.toolRouter.executeMeta(
        "COMPOSIO_SEARCH_TOOLS",
        in: sessionId,
        arguments: ["queries": [["use_case": intent]], "session": ["generate_id": true]]
    )
    
    // Extract from response.data
    return ToolExecutionPlan(
        primaryTools: response.data["primary_tool_slugs"] ?? [],
        pitfalls: response.data["known_pitfalls"] ?? [],
        steps: response.data["recommended_plan_steps"] ?? []
    )
}
```

**Never invent tool slugs** - only use slugs returned by search.

---

## 3. Error Recovery with Retry

```swift
func executeToolWithRecovery(
    _ toolSlug: String,
    in sessionId: String,
    arguments: [String: Any],
    attempt: Int = 1
) async throws -> ToolRouterExecuteResponse {
    do {
        return try await composio.toolRouter.execute(toolSlug, in: sessionId, arguments: arguments)
    } catch let error as ComposioError {
        switch error {
        case .notAuthenticated(let toolkit):
            // Toolkit-specific auth failure - reconnect this toolkit
            guard attempt < 3 else { throw error }
            try await reconnectToolkit(toolkit, in: sessionId)
            return try await executeToolWithRecovery(toolSlug, in: sessionId, arguments: arguments, attempt: attempt + 1)
            
        case .unauthorized, .forbidden:
            // API-level auth failure - cannot recover automatically
            throw error
            
        case .rateLimited(let retryAfter):
            let delay = UInt64(retryAfter ?? 5) * 1_000_000_000
            try await Task.sleep(nanoseconds: delay)
            return try await executeToolWithRecovery(toolSlug, in: sessionId, arguments: arguments, attempt: attempt + 1)
            
        case .networkError(let underlying):
            // Network error with underlying cause
            guard attempt < 2 else { throw error }
            try await Task.sleep(nanoseconds: 2_000_000_000) // 2s delay
            return try await executeToolWithRecovery(toolSlug, in: sessionId, arguments: arguments, attempt: attempt + 1)
            
        default:
            throw error
        }
    }
}

private func extractToolkit(from toolSlug: String) -> String {
    toolSlug.components(separatedBy: "_").first?.lowercased() ?? ""
}
```

---

## 4. Confirmation Rules

| Requires Confirmation | Auto-Execute |
|-----------------------|--------------|
| SEND_* (email, slack) | GET_*, FETCH_*, LIST_* |
| DELETE_* | SEARCH_* |
| SHARE_*, POST_* | CREATE_DRAFT |

```swift
func requiresConfirmation(for toolSlug: String) -> Bool {
    let destructive = ["SEND", "DELETE", "SHARE", "POST", "REMOVE", "UPDATE"]
    let readOnly = ["GET", "FETCH", "LIST", "SEARCH", "FIND"]
    
    if readOnly.contains(where: { toolSlug.contains($0) }) { return false }
    if destructive.contains(where: { toolSlug.contains($0) }) { return true }
    return true // Default: confirm unknown actions
}
```

**After confirmation request**: Pause all tool calls until user responds.

---

## 5. Streaming Responses

```swift
func streamChat(message: String) -> AsyncThrowingStream<String, Error> {
    AsyncThrowingStream { continuation in
        Task {
            do {
                let (bytes, _) = try await URLSession.shared.bytes(for: request)
                var buffer = ""
                
                for try await byte in bytes {
                    buffer.append(Character(UnicodeScalar(byte)))
                    if buffer.hasSuffix("\n") {
                        continuation.yield(buffer)
                        buffer = ""
                    }
                }
                if !buffer.isEmpty { continuation.yield(buffer) }
                continuation.finish()
            } catch {
                continuation.finish(throwing: error)
            }
        }
    }
}

// Usage
for try await chunk in streamChat(message: text) {
    await MainActor.run { streamingContent += chunk }
}
```

---

## 6. Pagination

### Parallel (Page-Based)
```swift
func fetchAllPages<T>(initial: PagedResponse<T>, fetcher: (Int) async throws -> PagedResponse<T>) async throws -> [T] {
    guard let totalPages = initial.totalPages, totalPages > 1 else { return initial.items }
    
    return try await withThrowingTaskGroup(of: PagedResponse<T>.self) { group in
        var items = initial.items
        for page in 2...totalPages {
            group.addTask { try await fetcher(page) }
        }
        for try await response in group {
            items.append(contentsOf: response.items)
        }
        return items
    }
}
```

### Sequential (Cursor-Based)
```swift
func fetchAllCursor<T>(initial: CursorResponse<T>, fetcher: (String) async throws -> CursorResponse<T>) async throws -> [T] {
    var items = initial.items
    var cursor = initial.nextCursor
    while let c = cursor {
        let response = try await fetcher(c)
        items.append(contentsOf: response.items)
        cursor = response.nextCursor
    }
    return items
}
```

---

## 7. Memory Storage (SwiftData)

Store **entity mappings** and **preferences**, not action logs.

```swift
@Model
class ToolkitMemory {
    @Attribute(.unique) var id: String
    var toolkit: String
    var userId: String
    var entries: [String]
    var lastUpdated: Date
    
    func addEntry(_ entry: String) {
        guard !entries.contains(entry) else { return }
        entries.append(entry)
        lastUpdated = Date()
    }
}

// ✅ Good entries
memory.addEntry("Channel #general has ID C1234567")
memory.addEntry("User prefers markdown without emojis")

// ❌ Bad entries (don't store)
memory.addEntry("Successfully sent email at 3:00 PM")
```

---

## 8. System Prompt Rules

Include these in your LLM system prompt:

```
CRITICAL RULES:
1. Action First: Execute immediately unless user asks for plan only
2. Tool Discovery: Call COMPOSIO_SEARCH_TOOLS before any tool execution
3. Never Invent: Only use tool slugs returned by search
4. Confirm Destructive: MUST confirm send/delete/share actions
5. Read-Only Free: Execute list/fetch/search without asking
6. Format: Use markdown with deeplinks: "[View](gmail://inbox)"
```

---

## Priority Implementation Order

| Priority | Pattern | Impact |
|----------|---------|--------|
| P0 | Session caching | Reduces API calls by ~80% |
| P0 | Confirmation UI | Prevents destructive mistakes |
| P0 | Tool discovery | Ensures correct tool selection |
| P1 | Error recovery | Improves reliability |
| P1 | Streaming | Better perceived performance |
| P2 | Pagination | Handles large datasets |
| P2 | Memory storage | Reduces repeated lookups |

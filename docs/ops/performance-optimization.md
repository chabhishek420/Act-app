# Performance Optimization

## Performance Targets

| Metric | Target | Critical | Measurement |
|--------|--------|----------|-------------|
| App launch | < 1s | < 2s | Time to first frame |
| Session creation | < 2s | < 3s | Composio API call |
| First token | < 1s | < 2s | OpenAI streaming start |
| Tool execution | < 5s | < 8s | Excluding external API |
| Memory (idle) | < 50MB | < 100MB | Instruments |
| Scroll FPS | 60fps | 30fps | Core Animation |

---

## Session Caching

### SessionManager Actor

```swift
actor SessionManager {
    private var cache: [String: CachedSession] = [:]
    private let ttl: TimeInterval = 3600  // 1 hour
    
    func getSession(for userId: String, conversationId: String) async throws -> ToolRouterSession {
        cleanExpired()  // Prune on access
        
        let key = "\(userId)-\(conversationId)"
        if let cached = cache[key], !cached.isExpired {
            return cached.session
        }
        
        let session = try await createNewSession(userId: userId)
        cache[key] = CachedSession(session: session, createdAt: Date())
        return session
    }
    
    private func cleanExpired() {
        let now = Date()
        cache = cache.filter { now.timeIntervalSince($0.value.createdAt) < ttl }
    }
}
```

### Benefits
- Avoid redundant session creation
- Reduce API calls
- Faster message sending after first

---

## Network Optimization

### URLSession Configuration

```swift
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest = 30
config.timeoutIntervalForResource = 120
config.waitsForConnectivity = true
config.httpMaximumConnectionsPerHost = 4
```

### Request Prioritization
- Session creation: `.userInitiated`
- Tool search: `.userInitiated`
- Tool execution: `.userInitiated`
- Analytics: `.background`

---

## UI Rendering

### Lazy Loading

```swift
LazyVStack(spacing: 0) {
    ForEach(messages) { message in
        MessageRow(message: message)
            .id(message.id)
    }
}
```

### Efficient Updates

```swift
@Observable
final class ChatViewModel {
    // Use @Observable instead of ObservableObject
    // Provides finer-grained updates
}
```

### List Identity
- Use stable `id` values (UUIDs)
- Avoid re-creating models on each update

---

## Memory Management

### Conversation Limits
- Keep last 100 messages in memory
- Load older messages on scroll (pagination)
- Clear session cache on memory warning

```swift
NotificationCenter.default.addObserver(
    forName: UIApplication.didReceiveMemoryWarningNotification,
    object: nil,
    queue: .main
) { _ in
    Task {
        await sessionManager.clearAll()
    }
}
```

### Image Handling
- Downscale attachments before display
- Use `AsyncImage` with placeholder
- Clear image cache on background

---

## Startup Optimization

### Launch Sequence
1. Show cached user state immediately
2. Initialize Composio SDK (background)
3. Prewarm session if user logged in
4. Load first conversation

```swift
@main
struct ActApp: App {
    init() {
        // Prewarm in background
        Task.detached(priority: .utility) {
            if let userId = UserDefaults.standard.string(forKey: "lastUserId") {
                try? await sessionManager.prewarm(userId: userId)
            }
        }
    }
}
```

---

## Streaming Optimization

### Token Accumulation

```swift
// Batch UI updates for streaming
@MainActor
class StreamingBuffer {
    private var buffer = ""
    private var updateTask: Task<Void, Never>?
    
    func append(_ text: String, onUpdate: @escaping (String) -> Void) {
        buffer += text
        
        // Debounce updates to ~30fps
        updateTask?.cancel()
        updateTask = Task {
            try? await Task.sleep(for: .milliseconds(33))
            if !Task.isCancelled {
                onUpdate(buffer)
            }
        }
    }
}
```

---

## Monitoring

### Instruments Profiles
- Time Profiler: Identify slow code paths
- Allocations: Track memory growth
- Network: Monitor API latency

### Custom Signposts

```swift
import os.signpost

let perfLog = OSLog(subsystem: "com.app.act", category: .pointsOfInterest)

func executeWithTiming(_ toolSlug: String) async throws -> ToolResult {
    let signpostID = OSSignpostID(log: perfLog)
    os_signpost(.begin, log: perfLog, name: "Tool Execution", signpostID: signpostID, "%{public}s", toolSlug)
    
    defer {
        os_signpost(.end, log: perfLog, name: "Tool Execution", signpostID: signpostID)
    }
    
    return try await toolService.execute(toolSlug)
}
```

---

## Performance Checklist

### Pre-Release
- [ ] Profile with Time Profiler
- [ ] Check memory with Allocations
- [ ] Verify scroll performance
- [ ] Test on oldest supported device (iPhone X)
- [ ] Measure cold launch time

### Ongoing
- [ ] Monitor crash-free rate
- [ ] Track session creation latency
- [ ] Alert on p95 > 3s for tool execution

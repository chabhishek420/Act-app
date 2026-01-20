# Database Schema

## Storage Strategy

**MVP:** SwiftData (local only)  
**Phase 2+:** Optional Appwrite for cross-device sync

---

## SwiftData Models (iOS 17+)

### Conversation

```swift
import SwiftData

@Model
final class Conversation {
    @Attribute(.unique) var id: UUID
    var title: String
    var toolRouterSessionId: String?
    var createdAt: Date
    var updatedAt: Date
    
    @Relationship(deleteRule: .cascade, inverse: \Message.conversation)
    var messages: [Message] = []
    
    init(title: String = "New Chat") {
        self.id = UUID()
        self.title = title
        self.createdAt = Date()
        self.updatedAt = Date()
    }
}
```

### Message

```swift
@Model
final class Message {
    @Attribute(.unique) var id: UUID
    var role: MessageRole
    var content: String
    var createdAt: Date
    
    // Tool calls (stored as JSON)
    var toolCallsData: Data?
    
    // Parent relationship
    var conversation: Conversation?
    
    init(role: MessageRole, content: String) {
        self.id = UUID()
        self.role = role
        self.content = content
        self.createdAt = Date()
    }
    
    // Computed property for tool calls
    var toolCalls: [ToolCallData]? {
        get {
            guard let data = toolCallsData else { return nil }
            return try? JSONDecoder().decode([ToolCallData].self, from: data)
        }
        set {
            toolCallsData = try? JSONEncoder().encode(newValue)
        }
    }
}

enum MessageRole: String, Codable {
    case user
    case assistant
    case tool
    case system
}
```

### ToolCallData

```swift
struct ToolCallData: Codable, Identifiable {
    let id: String
    let toolSlug: String
    var status: ToolCallStatus
    var logId: String?
    var resultSummary: String?
    var createdAt: Date
    
    // Sanitized arguments (sensitive data redacted)
    var sanitizedArgs: [String: String]
}

enum ToolCallStatus: String, Codable {
    case pending
    case running
    case success
    case failure
}
```

### ToolkitMemory

Store per-toolkit facts for context:

```swift
@Model
final class ToolkitMemory {
    @Attribute(.unique) var toolkit: String
    var facts: [String]  // Array of strings, NOT nested objects
    var updatedAt: Date
    
    init(toolkit: String, facts: [String] = []) {
        self.toolkit = toolkit
        self.facts = facts
        self.updatedAt = Date()
    }
}

// Usage:
// memory.facts = ["Channel general has ID C1234567", "John's email is john@example.com"]
```

---

## SwiftData Container

### Setup

```swift
import SwiftData

@main
struct ActApp: App {
    var sharedModelContainer: ModelContainer = {
        let schema = Schema([
            Conversation.self,
            Message.self,
            ToolkitMemory.self
        ])
        
        let config = ModelConfiguration(
            schema: schema,
            isStoredInMemoryOnly: false,
            cloudKitDatabase: .none  // No iCloud for MVP
        )
        
        return try! ModelContainer(for: schema, configurations: [config])
    }()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(sharedModelContainer)
    }
}
```

### Queries

```swift
struct ConversationListView: View {
    @Query(sort: \Conversation.updatedAt, order: .reverse)
    private var conversations: [Conversation]
    
    var body: some View {
        List(conversations) { conversation in
            NavigationLink(value: conversation) {
                Text(conversation.title)
            }
        }
    }
}

struct ChatView: View {
    let conversation: Conversation
    
    @Query private var messages: [Message]
    
    init(conversation: Conversation) {
        self.conversation = conversation
        _messages = Query(
            filter: #Predicate<Message> { $0.conversation?.id == conversation.id },
            sort: \.createdAt
        )
    }
}
```

---

## Secure Storage (Keychain)

API keys are NOT stored in SwiftData. Use Keychain:

```swift
struct SecureKeys {
    static let openAIKey = "openai_api_key"
    static let composioKey = "composio_api_key"
}

// Store
try Keychain.save(apiKey, key: SecureKeys.openAIKey)

// Retrieve
let apiKey = Keychain.get(SecureKeys.openAIKey)

// Delete
Keychain.delete(SecureKeys.openAIKey)
```

---

## Migration Notes

### From MVP to Phase 2 (Appwrite)

If adding cloud sync later:
1. Keep SwiftData as local cache
2. Add Appwrite collections mirroring SwiftData models
3. Sync on app launch and background refresh
4. Handle conflicts with "last write wins"

### Appwrite Collections (Phase 2)

```
Database: act
├── Collection: conversations
│   ├── $id: string (UUID)
│   ├── userId: string
│   ├── title: string
│   ├── toolRouterSessionId: string?
│   ├── createdAt: datetime
│   └── updatedAt: datetime
├── Collection: messages
│   ├── $id: string (UUID)
│   ├── conversationId: string
│   ├── role: string (enum)
│   ├── content: string
│   ├── toolCallsJson: string
│   └── createdAt: datetime
└── Collection: toolkitMemories
    ├── $id: string
    ├── userId: string
    ├── toolkit: string
    └── factsJson: string
```

---

## Indexes

For SwiftData, indexes are automatic for `@Attribute(.unique)` properties.

Recommended indexes for large datasets:
- `Message.conversation.id` + `Message.createdAt`
- `Conversation.updatedAt`
- `ToolkitMemory.toolkit`

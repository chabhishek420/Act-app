# Third-Party Libraries

## Dependency Manager
**Choice:** Swift Package Manager (SPM)

---

## MVP Dependencies (Minimal)

| Name | Purpose | Version | License |
|------|---------|---------|---------|
| **Composio Swift SDK** | Tool Router, OAuth, tool execution | Local | MIT |
| **MacPaw/OpenAI** | LLM chat completions with streaming | 0.3.0+ | MIT |

That's it. **Two dependencies** for MVP.

---

## Package.swift (MVP)

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "Act",
    platforms: [.iOS(.v17)],
    dependencies: [
        // Local Composio Swift SDK
        .package(path: "../composio-swift"),
        
        // OpenAI with streaming support
        .package(url: "https://github.com/MacPaw/OpenAI", from: "0.3.0"),
    ],
    targets: [
        .target(
            name: "Act",
            dependencies: [
                .product(name: "Composio", package: "composio-swift"),
                .product(name: "OpenAI", package: "OpenAI"),
            ]
        ),
    ]
)
```

---

## Composio Swift SDK

> **Note**: This is a custom SDK, not an official Composio package. Located at `../composio-swift/`.

### Features
- Tool Router session management
- Meta-tool execution (COMPOSIO_SEARCH_TOOLS, COMPOSIO_MANAGE_CONNECTIONS)
- Tool execution with modifiers
- OAuth via ASWebAuthenticationSession
- Swift 6.0 with strict concurrency
- iOS 15+, macOS 12+, visionOS 1+

### Initialization

```swift
import Composio

// Option 1: Direct initialization
let composio = Composio(apiKey: apiKey)

// Option 2: Shared instance (configure at app launch)
Composio.configure(configuration: ComposioConfiguration(apiKey: apiKey))
let composio = Composio.shared
```

### Key Methods (from ToolRouterClient.swift)

```swift
// Create session
let session = try await composio.toolRouter.createSession(
    for: userId,
    toolkits: ["gmail", "slack"]
)

// Execute meta-tool
let response = try await composio.toolRouter.executeMeta(
    "COMPOSIO_SEARCH_TOOLS",
    in: session.sessionId,
    arguments: ["queries": [["use_case": "fetch emails"]]]
)

// Execute tool
let result = try await composio.toolRouter.execute(
    "GMAIL_FETCH_EMAILS",
    in: session.sessionId,
    arguments: ["query": "is:unread", "max_results": 25]
)

// Create auth link
let link = try await composio.toolRouter.createLink(
    for: "gmail",
    in: session.sessionId
)
```

### OAuth Flow (from OAuthManager.swift)

```swift
let oauthManager = OAuthManager(composio: composio)
let account = try await oauthManager.authenticate(
    userId: userId,
    toolkit: "gmail",
    callbackURLScheme: "act"  // Register in Info.plist
)
```

---

## MacPaw/OpenAI

### Streaming Chat Completions

```swift
import OpenAI

let openAI = OpenAI(apiToken: userApiKey)

let query = ChatQuery(
    model: .gpt4o,
    messages: [
        .init(role: .system, content: systemPrompt),
        .init(role: .user, content: userMessage)
    ]
)

// Streaming
let stream = openAI.chatsStream(query: query)
for try await result in stream {
    if let content = result.choices.first?.delta.content {
        // Append to UI
    }
}

// Non-streaming
let response = try await openAI.chats(query: query)
```

---

## Security

### API Key Storage

```swift
import Security

struct Keychain {
    static func save(_ value: String, key: String) throws {
        let data = value.data(using: .utf8)!
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }
    
    static func get(_ key: String) -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true
        ]
        var result: AnyObject?
        SecItemCopyMatching(query as CFDictionary, &result)
        guard let data = result as? Data else { return nil }
        return String(data: data, encoding: .utf8)
    }
}
```

### Key Storage Locations

| Key | Storage | Notes |
|-----|---------|-------|
| OpenAI API key | Keychain | User-provided |
| Composio API key | Keychain | App-provided or user-provided |

---

## Phase 2+ Dependencies

| Name | Purpose | When to Add |
|------|---------|-------------|
| **Appwrite SDK** | Cross-device sync, user auth | Multi-device support |
| **Deepgram SDK** | Voice transcription | Voice input feature |
| **SwiftOpenAI** | Alternative OpenAI client | If MacPaw/OpenAI lacks features |

---

## Platform Requirements

| Component | Minimum | Notes |
|-----------|---------|-------|
| iOS | 17.0 | SwiftData, @Observable |
| iOS (Liquid Glass) | 26.0 | Optional enhancement |
| Composio SDK | iOS 15.0 | |
| OpenAI SDK | iOS 13.0 | |
| Xcode | 15.0+ | |
| Swift | 5.9+ | |

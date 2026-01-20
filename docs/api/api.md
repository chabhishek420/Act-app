# API Documentation

## API Style
**Type:** RESTful JSON (Composio v3)  
**Base URL:** `https://backend.composio.dev/api/v3`  
**Auth Header:** `x-api-key: <COMPOSIO_PROJECT_API_KEY>`

---

## Composio Swift SDK Methods

Use the SDK instead of raw HTTP. All methods from actual SDK source code:

### [ToolRouterClient](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Core/ToolRouterClient.swift)

> **Reference Documentation**: [tool_router-session.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/Swift/api-reference-md/tool_router/POST-tool_router-session.md)

```swift
import Composio

let composio = Composio(apiKey: apiKey)

// Create session
// Method: createSession(for:toolkits:)
let session = try await composio.toolRouter.createSession(
    for: userId,
    toolkits: ["gmail", "slack", "github"]
)
// Returns: ToolRouterSession

// Fetch session tools
// Method: fetchTools(in:)
let tools = try await composio.toolRouter.fetchTools(in: session.sessionId)
// Returns: [Tool]

// Execute meta-tool
// Method: executeMeta(_:in:arguments:)
let response = try await composio.toolRouter.executeMeta(
    "COMPOSIO_SEARCH_TOOLS",
    in: session.sessionId,
    arguments: ["queries": [["use_case": "fetch unread emails"]]]
)
// Returns: ToolRouterExecuteResponse

// Execute tool
// Method: execute(_:in:arguments:)
let result = try await composio.toolRouter.execute(
    "GMAIL_FETCH_EMAILS",
    in: session.sessionId,
    arguments: ["query": "is:unread", "max_results": 25]
)
// Returns: ToolRouterExecuteResponse

// Create auth link
// Method: createLink(for:in:)
let link = try await composio.toolRouter.createLink(
    for: "gmail",
    in: session.sessionId
)
// Returns: ToolRouterLinkResponse
```

### [OAuthManager](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Authentication/OAuthManager.swift)

> **Reference Documentation**: [POST-connected_accounts-link.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/Swift/api-reference-md/connected_accounts/POST-connected_accounts-link.md)

```swift
// OAuth flow with ASWebAuthenticationSession
let oauthManager = OAuthManager(composio: composio)

// Method: authenticate(userId:toolkit:callbackURLScheme:)
let account = try await oauthManager.authenticate(
    userId: "user_123",
    toolkit: "gmail",
    callbackURLScheme: "act"
)
// Returns: ConnectedAccount
// Properties: isAuthenticating, error (published)
```

### [ToolsClient](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Tools/ToolsClient.swift)

> **Reference Documentation**: [GET-tools.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/Swift/api-reference-md/tools/GET-tools.md)

```swift
// Fetch tools for user
// Method: fetch(for:options:modifiers:localModifiers:)
let tools = try await composio.tools.fetch(
    for: userId,
    options: ToolsGetOptions(toolkits: ["gmail"])
)

// Execute tool directly
// Method: execute(_:for:connectedAccountId:parameters:)
let result = try await composio.tools.execute(
    "GMAIL_SEND_EMAIL",
    for: userId,
    parameters: ["to": "user@example.com", "subject": "Hello"]
)

// Search tools
// Method: search(for:)
let tools = try await composio.tools.search(for: "send email")
```

---

## Core REST Endpoints

| Method | Path | Purpose | SDK Method |
|--------|------|---------|------------|
| POST | `/tool_router/session` | Create session | `toolRouter.createSession(for:toolkits:)` |
| GET | `/tool_router/session/{id}/tools` | List meta-tools | `toolRouter.fetchTools(in:)` |
| POST | `/tool_router/session/{id}/execute_meta` | Execute meta-tool | `toolRouter.executeMeta(_:in:arguments:)` |
| POST | `/tool_router/session/{id}/execute` | Execute tool | `toolRouter.execute(_:in:arguments:)` |
| POST | `/tool_router/session/{id}/link` | Create auth link | `toolRouter.createLink(for:in:)` |

---

## COMPOSIO_SEARCH_TOOLS Response

Meta-tool that returns execution guidance with step-by-step plans:

```json
{
  "data": {
    "results": [{
      "index": 1,
      "use_case": "send an email via gmail",
      "execution_guidance": "IMPORTANT: Follow the recommended plan...",
      "recommended_plan_steps": [
        "Required Step 1: Use GMAIL_SEND_EMAIL with specific arguments..."
      ],
      "known_pitfalls": [],
      "difficulty": "easy",
      "primary_tool_slugs": ["GMAIL_SEND_EMAIL"],
      "related_tool_slugs": ["GMAIL_CREATE_DRAFT"],
      "toolkits": ["gmail"],
      "is_empty": false,
      "toolkit_connection_statuses": [{
        "toolkit": "gmail",
        "has_active_connection": true,
        "status_message": "Gmail is connected"
      }]
    }],
    "session": {
      "id": "load",
      "instructions": "REQUIRED: Pass session_id 'load' in ALL subsequent meta tool calls."
    },
    "next_steps_guidance": ["1) CALL COMPOSIO_MANAGE_CONNECTIONS...", "2) Extract validated plan..."],
    "success": true
  },
  "error": null,
  "log_id": "log_B6kA3mUuao_R"
}
```

## COMPOSIO_MANAGE_CONNECTIONS Response

Meta-tool used to initiate connections for missing toolkits:

```json
{
  "data": {
    "message": "All connections have been initiated and are pending completion",
    "results": {
      "gmail": {
        "toolkit": "gmail",
        "status": "initiated",
        "redirect_url": "https://connect.composio.dev/link/lk_...",
        "instruction": "Action required: Share the following authentication link with the user...\n",
        "was_reinitiated": false
      }
    }
  },
  "error": null,
  "log_id": "log_U8XcBESfjrXF"
}
```


> **Warning**: Returns empty results if queried toolkit wasn't enabled during session creation.

---

## Response Types (SDK Models)

### ToolRouterSession
```swift
struct ToolRouterSession: Codable, Sendable {
    let sessionId: String
    let mcp: ToolRouterMcp?
    let toolRouterTools: [String]?
}

struct ToolRouterMcp: Codable, Sendable {
    let type: String?
    let url: String
    let headers: [String: String]?
}
```

### ToolRouterExecuteResponse
```swift
struct ToolRouterExecuteResponse: Codable, Sendable {
    let data: [String: AnyCodable]
    let error: String?
    let logId: String
}
```

### ToolRouterLinkResponse
```swift
struct ToolRouterLinkResponse: Codable, Sendable {
    let connectedAccountId: String
    let linkToken: String
    let redirectUrl: String // Verified from live API: returns "redirect_url"

    enum CodingKeys: String, CodingKey {
        case connectedAccountId = "connected_account_id"
        case linkToken = "link_token"
        case redirectUrl = "redirect_url"
    }
}
```

### ConnectedAccount
```swift
struct ConnectedAccount: Codable, Sendable {
    let id: String
    let userId: String
    let status: AccountStatus
    let createdAt: Date?
    let updatedAt: Date?
    let toolkit: String
}

enum AccountStatus: String, Codable {
    case active = "active"
    case inactive = "inactive"
    case expired = "expired"
    case pending = "pending"
    case initiated = "initiated"
    case failed = "failed"
}
```

---

## Error Handling

SDK throws `ComposioError`:

```swift
enum ComposioError: Error {
    case missingAPIKey
    case invalidConfiguration(String)
    case networkError(Error)
    case invalidResponse(String)
    case notAuthenticated(toolkit: String)
    case rateLimited(retryAfter: Int?)
    case serverError(statusCode: Int, message: String?)
    case decodingError(Error)
    case authenticationCancelled
}
```

Handle in app:
```swift
catch ComposioError.notAuthenticated(let toolkit) {
    // Show "Connect [toolkit]" button
}
catch ComposioError.rateLimited(let retryAfter) {
    // Wait and retry
    try await Task.sleep(for: .seconds(retryAfter ?? 5))
}
```

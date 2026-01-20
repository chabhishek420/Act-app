# Act Documentation

> **Act** is an iOS-native AI assistant that executes authenticated actions across 500+ productivity tools via natural language.

## Quick Links

| Document | Purpose |
|----------|---------|
| **[BUILD_ORCHESTRATOR.md](./BUILD_ORCHESTRATOR.md)** | 🚦 Primary entry point for autonomous agents |
| **[CHATGPT_UI_VERIFICATION.md](./CHATGPT_UI_VERIFICATION.md)** | ✅ ChatGPT clone UI verification & checklist |
| [SPEC.md](./SPEC.md) | Technical specification (v4.0 FINAL) |
| [prd.md](./requirements/prd.md) | Product requirements |
| [navigation-spec.md](./design/navigation-spec.md) | Chat-first navigation (no tabs) |
| [wireframe/README.md](./wireframe/README.md) | 60 wireframes catalog |
| [architecture/](./architecture/) | Backend, State, Dependencies |
| [api/](./api/) | API, Code Docs |
| [design/](./design/) | Frontend, UI, Flows |
| [data/](./data/) | Database, Security |
| [ops/](./ops/) | DevOps, Testing, Perf |

---

## Document Index

### 📋 Requirements
- [prd.md](./requirements/prd.md) - Product Requirements Document
- [use-cases.md](./requirements/use-cases.md) - Real-world use cases & workflows

### 🏗️ Architecture
- [backend.md](./architecture/backend.md) - Direct API architecture (MVP)
- [state-management.md](./architecture/state-management.md) - MVVM with @Observable
- [implementation-patterns.md](./architecture/implementation-patterns.md) - Production patterns from Rube
- [third-party-libraries.md](./architecture/third-party-libraries.md) - Dependencies (Composio SDK, OpenAI)

### 🔌 API Integration
- [api.md](./api/api.md) - Composio API contracts (verified)
- [code-documentation.md](./api/code-documentation.md) - SDK usage patterns

### 🎨 Design
- **[navigation-spec.md](./design/navigation-spec.md)** - ChatGPT clone navigation (chat-first, no tabs) ⚠️
- [ios-frontend.md](./design/ios-frontend.md) - SwiftUI components
- [ui-design-system.md](./design/ui-design-system.md) - Complete design system (v2.0, 400+ lines)
- [ui-design-guide.md](./design/ui-design-guide.md) - iOS 26 Liquid Glass patterns
- [liquid-glass-implementation.md](./design/liquid-glass-implementation.md) - Liquid Glass implementation guide
- [user-flow.md](./design/user-flow.md) - User journey diagrams
- **[wireframe/README.md](./wireframe/README.md)** - Wireframe catalog (60 assets, indexed)

### 💾 Data & Security
- [database-schema.md](./data/database-schema.md) - SwiftData models
- [security.md](./data/security.md) - Keychain, OAuth, Privacy

### ⚙️ Operations
- [devops.md](./ops/devops.md) - CI/CD with GitHub Actions
- [testing-plan.md](./ops/testing-plan.md) - Swift Testing strategy
- [performance-optimization.md](./ops/performance-optimization.md) - Performance targets

---

## 📚 External Reference Index

To ensure developers and AI agents find the right implementation details, use the following links:

### 🧩 Composio Swift SDK (Local)
Built-in SDK powering all tool-calling capabilities.
- **Base Client**: [Composio.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Core/Composio.swift)
- **Tool Router**: [ToolRouterClient.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Core/ToolRouterClient.swift)
- **OAuth/Accounts**: [OAuthManager.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Authentication/OAuthManager.swift)
- **Direct Tools**: [ToolsClient.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Tools/ToolsClient.swift)
- **Entities Search**: [EntitiesClient.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Core/EntitiesClient.swift)
- **Wait For Connection**: [ConnectionRequest+Polling.swift](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/composio-swift/Sources/Composio/Extensions/ConnectionRequest+Polling.swift)

###  iOS 26 & Apple Design
Cutting-edge Apple features used for the "Liquid Glass" and "Foundation" modes.
- **Liquid Glass (SwiftUI)**: [SwiftUI-Implementing-Liquid-Glass-Design.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md)
- **Liquid Glass (UIKit)**: [UIKit-Implementing-Liquid-Glass-Design.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/UIKit-Implementing-Liquid-Glass-Design.md)
- **On-Device LLM**: [FoundationModels-Using-on-device-LLM-in-your-app.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/FoundationModels-Using-on-device-LLM-in-your-app.md)
- **App Intents**: [AppIntents-Updates.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/xcode-26-system-prompts/AdditionalDocumentation/AppIntents-Updates.md)

### 🔌 Composio API Specs
Original API documentation for reference on endpoint behavior.
- **API Overview**: [README.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/README.md)
- **Tool Router API**: [POST-tool_router-session.md](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/Swift/api-reference-md/tool_router/POST-tool_router-session.md)
- **Search Metadata**: [SEARCH_TOOLS](file:///Users/roshansharma/Desktop/Vscode/Rube-ios/.temp/composio-docs/Swift/api-reference-md/tool_router/POST-tool_router-session-session_id-execute_meta.md)

---

## Verification Status

All documentation verified against **live Composio v3 API** on 2026-01-19.

| Component | Status |
|-----------|--------|
| SDK Signatures | ✅ Verified |
| JSON Contracts | ✅ Verified |
| Auth Flows | ✅ Verified |
| Liquid Glass (iOS 26) | ✅ Documented |
| Foundation Models | ✅ Documented |

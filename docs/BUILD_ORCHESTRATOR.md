# Act Project Build Orchestrator

> **MISSION**: Build a production-ready, action-first iOS AI assistant that executes 500+ tool integrations via Composio.

This document is the **ONLY** starting point for autonomous agents. It provides a seamless, entry-to-end path for implementing the entire Act application. Follow this guide sequentially to ensure architectural integrity, design consistency, and functional correctness.

---

## 🏗️ Project Architecture & Tech Stack

- **Framework**: SwiftUI (utilizing @Observable and SwiftData)
- **Architecture**: MVVM-S (Model-View-ViewModel-Service)
- **Minimum OS**: iOS 17.0 (with iOS 26 "Liquid Glass" enhancements)
- **Integrations**: 
  - [Composio Swift SDK](../../composio-swift/) (Local Package)
  - OpenAI API (via LLMService)
- **Data Persistence**: SwiftData + iOS Keychain
- **UI System**: Liquid Glass Design System (Glassmorphism + Interactive Materials)

---

## 📁 Full Project Directory Tree (Target State)

Use this tree to navigate and create files. Most files currently exist as empty stubs.

```text
Act-app/Act/Act/
├── App/
│   ├── ActApp.swift                # App entry point & SDK config
│   ├── AppState.swift              # Global observable state (auth, sidebar)
│   └── RootView.swift              # Chat + Sidebar container (ChatGPT-style)
├── Core/
│   ├── Models/
│   │   ├── Message.swift           # Chat message model (Role, Content, ToolCalls)
│   │   ├── Conversation.swift      # Thread model (ID, Metadata, History)
│   │   ├── ToolCall.swift          # Execution tracking (Status, Arguments, Results)
│   │   └── ToolkitMemory.swift     # Per-toolkit ID mapping (JSON/String storage)
│   └── Utilities/
│       ├── Keychain.swift          # Secure API key storage wrapper
│       └── AnyCodable.swift        # JSON serialization helper
├── Services/
│   ├── SessionManager.swift        # Tool Router session caching (1-hour TTL)
│   ├── ToolConnectionService.swift # OAuth management & Pre-flight checks
│   ├── ToolExecutionService.swift  # execute_tool logic with error recovery
│   ├── LLMService.swift            # OpenAI/Anthropic integration & Streaming
│   ├── ConfirmationService.swift   # Mode-based confirmation logic (Auto/Adaptive)
│   └── OAuthService.swift          # ASWebAuthenticationSession coordination
├── Data/
│   ├── SwiftDataModels/
│   │   ├── ConversationEntity.swift# Persistent conversation storage
│   │   └── MessageEntity.swift     # Persistent message storage
│   └── Persistence.swift           # ModelContainer configuration
├── Features/
│   ├── Chat/
│   │   ├── Views/
│   │   │   ├── ChatView.swift      # Main conversational UI
│   │   │   ├── MessageRow.swift    # Individual message bubble logic
│   │   │   └── ToolStatusCard.swift# Execution feedback component (Wireframe: Rich Layout)
│   │   ├── ViewModels/
│   │   │   └── ChatViewModel.swift # Core business logic (streaming, plan execution)
│   │   └── Components/
│   │       ├── ComposerView.swift  # Floating input bar (Glass)
│   │       └── ChatListHeader.swift# Brand/Status header
│   ├── Connections/
│   │   ├── Views/
│   │   │   ├── ConnectionsView.swift# Toolkit management list
│   │   │   ├── ConnectionCard.swift # Individual tool status UI
│   │   │   └── OAuthSheet.swift     # Auth web-view wrapper
│   │   └── ViewModels/
│   │       └── ConnectionsViewModel.swift
│   └── Onboarding/
│       ├── Views/
│       │   ├── WelcomeView.swift    # Entry screen (Wireframe: Main)
│       │   ├── APISetupView.swift   # Initial API key configuration
│       │   └── FirstConnectionView.swift # Tool connection guide
│       └── ViewModels/
│           └── OnboardingViewModel.swift
└── Resources/
    ├── Assets.xcassets             # Colors, Icons (AccentColor, AppIcon)
    └── PrivacyInfo.xcprivacy       # App Store compliance
```

---

## 🚀 Implementation Roadmap (Step-by-Step)

Follow this order to build the app from ground up.

### Phase 1: Foundation (Core & Utilities)
**Goal**: Build the skeletal structure and data layer.
- **Implementation**:
  1. `Core/Utilities/Keychain.swift`: Implement secure storage for OpenAI/Composio keys.
  2. `Core/Models/*.swift`: Define all data structures. (Ref: [Data Spec](./data/database-schema.md))
  3. `Data/Persistence.swift`: Configure SwiftData `ModelContainer`.
  4. `App/AppState.swift`: Create the global `@Observable` state manager.
  5. `Resources/PrivacyInfo.xcprivacy`: Update as required by [security.md](./data/security.md).

### Phase 2: Service Layer (Infrastructure)
**Goal**: Connect to external APIs and SDKs.
- **Reference**: [SPEC.md](./SPEC.md) (Architecture section)
- **Implementation**:
  1. `Services/SessionManager.swift`: Implement session caching (Rube Architecture).
  2. `Services/LLMService.swift`: Implement OpenAI streaming and Tool Router context.
  3. `Services/ToolConnectionService.swift`: Implement `COMPOSIO_MANAGE_CONNECTIONS`.
  4. `Services/ToolExecutionService.swift`: Implement multi-step execution with error recovery.
  5. `Services/ConfirmationService.swift`: Implement "Adaptive" mode logic.

### Phase 3: UI Implementation (Features)
**Goal**: Build the interactive "Liquid Glass" frontend.
- **Reference**: [ui-design-system.md](./design/ui-design-system.md), [ui-design-guide.md](./design/ui-design-guide.md)
- **Wireframe Catalog**: [wireframe/README.md](./wireframe/README.md) (60 assets, verified mappings)
- **Wireframe Mapping** (✅ Verified via visual inspection):

  | Component | Wireframe | What It Shows |
  |-----------|-----------|---------------|
  | `WelcomeView.swift` | [act-welcome-empty-state.png](./wireframe/act-welcome-empty-state.png) | Empty state with suggestion chips, floating composer |
  | `ComposerView.swift` | [act-composer-keyboard.png](./wireframe/act-composer-keyboard.png) | "+" button, "Ask anything" text field, mic icon, voice waves |
  | `ChatView.swift` | [act-chat-message-basic.png](./wireframe/act-chat-message-basic.png) | User bubble (dark), assistant response, action icons |
  | `MessageRow.swift` | [act-chat-message-multi-turn.png](./wireframe/act-chat-message-multi-turn.png) | Multi-turn conversation with longer responses |
  | `ToolStatusCard.swift` | [act-loading-state.png](./wireframe/act-loading-state.png) | Pulsing black dot indicator, stop button |
  | `ResultCard.swift` (music) | [act-result-music-spotify.png](./wireframe/act-result-music-spotify.png) | "Asked Spotify" badge, playlist card, song list |
  | `ResultCard.swift` (shopping) | [act-result-shopping-grocery.png](./wireframe/act-result-shopping-grocery.png) | Grocery list by category, product cards with prices |
  | `ResultCard.swift` (travel) | [act-result-travel-hotels.png](./wireframe/act-result-travel-hotels.png) | Hotel cards with ratings, "View on Booking.com" CTA |
  | `MapResultView.swift` | [act-map-restaurant-finder.png](./wireframe/act-map-restaurant-finder.png) | Full-screen map with pins, bottom sheet with venue |

- **Implementation**:
  1. `Features/Onboarding/`: Build the entry flow.
  2. `Features/Chat/Components/`: Build reusable message and input UI.
  3. `Features/Chat/Views/ChatView.swift`: Assemble the main assistant interface.
  4. `App/RootView.swift`: Chat-first container with slide-out sidebar (no tabs).

### Phase 4: Integration & Optimization
**Goal**: Polish interactions and ensure performance.
- **Implementation**:
  1. `App/ActApp.swift`: Initialize SDK and services.
  2. Implement Liquid Glass transitions (Ref: [liquid-glass-implementation.md](./design/liquid-glass-implementation.md)).
  3. Run performance checks (Ref: [ops/performance-optimization.md](./ops/performance-optimization.md)).
  4. Verify with [testing-plan.md](./ops/testing-plan.md).

---

## 🔗 Documentation Map

| For this Task... | Use this Document |
|:---|:---|
| **Architectural logic** | [SPEC.md](./SPEC.md) |
| **Product constraints** | [requirements/prd.md](./requirements/prd.md) |
| **Composio SDK usage** | [api/code-documentation.md](./api/code-documentation.md) |
| **Glass Design/Aesthetics** | [design/ui-design-system.md](./design/ui-design-system.md) |
| **Wireframe Reference** | [wireframe/README.md](./wireframe/README.md) |
| **Security/Keychain** | [data/security.md](./data/security.md) |
| **SwiftData Schema** | [data/database-schema.md](./data/database-schema.md) |

---

## 🏁 Final Verification (Definition of Done)

The implementation is complete when:
1.  **Auth Flow**: User can provide an OpenAI key and connect Gmail/Slack/GitHub toolkits.
2.  **Execution**: User can say "Send an email to John" and the app discovers the tool, asks for confirmation (if needed), and executes via Composio.
3.  **UI**: The app uses Liquid Glass effects on interactive components and matches the high-premium aesthetics of the Act wireframes.
4.  **Persistence**: Conversations and messages are saved locally via SwiftData and persist across app reboots.
5.  **Build**: The project builds for iOS 17+ and iOS 26 (Simulators) without errors.

---

**START HERE**: Begin by reading `Phase 1: Foundation (Core & Utilities)` and implementing `Core/Utilities/Keychain.swift`.

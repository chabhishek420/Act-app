# Act App Navigation & UI Specification

> **Version**: 3.0 - ChatGPT Clone  
> **Last Updated**: 2026-01-20  
> **Design Goal**: **Exact ChatGPT iOS UI clone, adapted for tool execution**

---

## ⚠️ CRITICAL: Chat-First, No Tab Bar

Act is a **ChatGPT clone** with tool/app integrations. The UI must match ChatGPT iOS exactly:

| ❌ NOT This | ✅ This (ChatGPT) |
|-------------|-------------------|
| Tab bar at bottom | **No tab bar** |
| Multiple tabs (Chat, Settings, etc.) | **Single chat screen + slide-out sidebar** |
| Complex navigation | **Minimal chrome, content-first** |

---

## 1. App Structure (ChatGPT Clone)

### 1.1 Screen Hierarchy

```
┌─────────────────────────────────────────────┐
│                   Act App                    │
│                                             │
│  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Sidebar   │  │     Chat Screen     │  │
│  │  (Hidden)   │←→│    (Main View)      │  │
│  │             │  │                     │  │
│  │  • History  │  │  ┌───────────────┐  │  │
│  │  • Projects │  │  │  Nav Header   │  │  │
│  │  • Settings │  │  ├───────────────┤  │  │
│  │             │  │  │               │  │  │
│  │             │  │  │   Messages    │  │  │
│  │             │  │  │               │  │  │
│  │             │  │  ├───────────────┤  │  │
│  │  [Avatar]   │  │  │   Composer    │  │  │
│  └─────────────┘  │  └───────────────┘  │  │
│                   └─────────────────────┘  │
└─────────────────────────────────────────────┘
```

### 1.2 Navigation Pattern

| Pattern | Description |
|---------|-------------|
| **Main View** | Chat screen (always visible, full screen) |
| **Sidebar** | Slide-out drawer (hamburger menu trigger) |
| **Modals** | Bottom sheet for tool confirmation, settings |
| **No Tab Bar** | ❌ Never use TabView or tab-based navigation |

---

## 2. Chat Screen Layout (Exact ChatGPT Match)

### 2.1 Header Bar
> **Reference**: [act-welcome-empty-state.png](./wireframe/act-welcome-empty-state.png)

```
┌──────────────────────────────────────────────────┐
│  ☰   Act 1 >                            🔄  ⋯   │
│                                                  │
│  Hamburger  Model Name  Chevron    Compose More │
└──────────────────────────────────────────────────┘
```

| Element | Position | Action |
|---------|----------|--------|
| Hamburger `☰` | Left | Opens sidebar |
| Model Name + `>` | Center-left | Model selector (future) |
| Refresh `🔄` | Right | New/clear chat |
| More `⋯` | Right | Context menu |

### 2.2 Message Area
> **Reference**: [act-chat-message-basic.png](./wireframe/act-chat-message-basic.png)

- Full height scrollable area
- Messages grow from bottom
- User messages: Right-aligned, dark/filled bubble
- Assistant messages: Left-aligned, no bubble (just text)

### 2.3 Composer Bar (Bottom)
> **Reference**: [act-composer-keyboard.png](./wireframe/act-composer-keyboard.png)

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  Suggestion Chips (horizontal scroll)            │
│  ┌────────────────┐  ┌────────────────┐          │
│  │ Website content │  │ Weekly perform  │          │
│  │    audit        │  │    ance         │          │
│  └────────────────┘  └────────────────┘          │
│                                                  │
│  ┌──────────────────────────────────────────┐   │
│  │ +  │ Ask anything                  🎤 🎵│   │
│  └──────────────────────────────────────────┘   │
│                                                  │
│           Floating Composer Bar                  │
└──────────────────────────────────────────────────┘
```

| Element | Function |
|---------|----------|
| `+` button | Attach files, access tools |
| Text field | "Ask anything" placeholder |
| 🎤 (mic) | Voice input |
| 🎵 (waves) | Voice mode toggle |

---

## 3. Sidebar (Drawer)

### 3.1 Structure
> **Reference**: [component-tab-bar.png](./wireframe/component-tab-bar.png)

```
┌─────────────────────────┐
│  ┌─────────────────────┐│
│  │ 🔍 Search...    [+] ││  ← Search + New Chat
│  └─────────────────────┘│
│                         │
│  📱 Act              →  │  ← Main chat (Act = ChatGPT)
│  📚 Library          →  │  ← Conversation history
│  🔧 Connections      →  │  ← Connected tools (replaces GPTs)
│                         │
│  ─────────────────────  │
│                         │
│  📁 Projects            │  ← Organized conversations
│  ⚙️ Variables           │  ← Settings/preferences
│                         │
│  ─────────────────────  │
│                         │
│  Today                  │
│    • Send email to...   │  ← Recent conversations
│    • Check weather...   │
│                         │
│  Yesterday              │
│    • Book flight...     │
│                         │
│  ─────────────────────  │
│                         │
│  ┌─────┐                │
│  │ 👤  │ User Name   →  │  ← Profile/settings
│  └─────┘                │
└─────────────────────────┘
```

### 3.2 Sidebar Actions

| Section | Act Equivalent | Notes |
|---------|----------------|-------|
| ChatGPT | **Act** | Main assistant |
| Library | **History** | Past conversations |
| GPTs | **Connections** | Connected toolkits (Gmail, Slack, etc.) |
| Projects | **Projects** | Grouped conversations |
| User Avatar | **Settings** | Account, API keys, preferences |

---

## 4. SwiftUI View Hierarchy

### 4.1 Corrected Structure (No TabView)

```swift
// ❌ WRONG - Old approach with tabs
TabView {
    ChatView().tabItem { ... }
    SettingsView().tabItem { ... }
}

// ✅ CORRECT - ChatGPT clone approach
struct RootView: View {
    @State private var isSidebarOpen = false
    
    var body: some View {
        ZStack {
            // Main chat view (always visible)
            ChatView()
            
            // Sidebar overlay (slides in)
            if isSidebarOpen {
                SidebarView(isOpen: $isSidebarOpen)
                    .transition(.move(edge: .leading))
            }
        }
    }
}
```

### 4.2 View Files

| File | Purpose | Notes |
|------|---------|-------|
| `RootView.swift` | Main container | Chat + optional sidebar overlay |
| `ChatView.swift` | Full chat interface | Header + Messages + Composer |
| `SidebarView.swift` | Drawer navigation | History, Connections, Settings |
| `ComposerView.swift` | Floating input bar | Reusable bottom component |
| `MessageRow.swift` | Single message | User or assistant bubble |

---

## 5. Screen Inventory (Revised)

| Screen | Trigger | UI Pattern |
|--------|---------|------------|
| **Chat** (main) | App launch | Full screen, always visible |
| **Sidebar** | Hamburger tap | Slide-in drawer (left edge) |
| **Connections** | Sidebar → Connections | Push view or sheet |
| **Settings** | Sidebar → Avatar | Push view or sheet |
| **Tool Confirmation** | Destructive action | Bottom sheet modal |
| **Model Selector** | Header title tap | Dropdown or sheet (future) |

---

## 6. ChatGPT UI Elements Checklist

### Must Match Exactly

| Element | ChatGPT | Act Implementation |
|---------|---------|-------------------|
| ☑️ No tab bar | ✅ | No TabView, single screen |
| ☑️ Hamburger menu | ✅ | Opens sidebar drawer |
| ☑️ Model name in header | ✅ | "Act 1 >" with chevron |
| ☑️ Floating composer | ✅ | Bottom bar with + 🎤 🎵 |
| ☑️ User bubbles (dark, right) | ✅ | Rounded, filled, right-aligned |
| ☑️ Assistant text (no bubble) | ✅ | Left-aligned, plain text |
| ☑️ Action icons row | ✅ | Copy, speak, like, dislike, retry, share |
| ☑️ Suggestion chips | ✅ | Horizontal scroll above composer |
| ☑️ Tool badge | ✅ | "Asked [Service]" pattern |
| ☑️ Loading indicator | ✅ | Pulsing black dot |

### Not In ChatGPT (Don't Add)

| Element | Reason |
|---------|--------|
| ❌ Bottom tab bar | ChatGPT doesn't have this |
| ❌ Multiple screens at once | Single focus on chat |
| ❌ Complex navigation stack | Keep it flat |
| ❌ Separate connections screen | Put in sidebar |

---

## 7. Wireframe-to-Component Mapping (Final)

| Component | Primary Wireframe | Notes |
|-----------|-------------------|-------|
| `RootView` | `act-welcome-empty-state.png` | Main container, no tabs |
| `ChatView` | `act-chat-message-basic.png` | Full chat interface |
| `SidebarView` | `component-tab-bar.png` | Drawer, NOT tab bar |
| `ComposerView` | `act-composer-keyboard.png` | Floating bottom bar |
| `MessageRow` | `act-chat-message-multi-turn.png` | User/assistant messages |
| `ToolStatusCard` | `act-loading-state.png` | Loading/executing state |
| `ResultCard` | `act-result-music-spotify.png` | Tool result display |

---

## 8. Changes Required to Match ChatGPT

### 8.1 Documentation to Update

| Document | Change Required |
|----------|-----------------|
| `ios-frontend.md` | Remove "Tab bar" reference, add sidebar |
| `BUILD_ORCHESTRATOR.md` | Remove "tab/container navigation" |
| `api/code-documentation.md` | Remove "Tab-based navigation" |
| `SPEC.md` | Verify no tab references |
| `wireframe/README.md` | Clarify sidebar not tab bar |

### 8.2 Code Structure Change

```
OLD (Wrong):
├── RootView.swift        # TabView container
├── ChatTab/
├── SettingsTab/
└── ConnectionsTab/

NEW (ChatGPT Clone):
├── RootView.swift        # ZStack: Chat + Sidebar overlay
├── Chat/
│   ├── ChatView.swift    # Main chat (always visible)
│   ├── ComposerView.swift
│   └── MessageRow.swift
├── Sidebar/
│   ├── SidebarView.swift # Drawer (slide-in)
│   ├── HistoryList.swift
│   └── ConnectionsList.swift
└── Shared/
    ├── SettingsSheet.swift
    └── ConfirmationSheet.swift
```

---

## 9. Summary: Chat-First Principles

1. **Chat is the app** - The entire screen is the chat interface
2. **Everything else is secondary** - Sidebar, settings, connections are overlays
3. **No persistent navigation** - No tabs, no bottom bar, no nav stack
4. **Minimal chrome** - Header and composer only
5. **Content-first** - Messages and results take all the space

---

**This document supersedes any previous navigation references in other docs.**

**Act = ChatGPT clone for tool execution. Match it exactly.**

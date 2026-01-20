# Product Requirements Document

## App Overview
**Name:** Act  
**Tagline:** Action-first AI assistant for your tools  
**Description:** Act is an iOS-native AI assistant that executes authenticated actions across 500+ integrations through Composio Tool Router. Users connect toolkits (Gmail, Slack, GitHub, etc.), issue natural-language commands, and Act plans, confirms risky actions, and executes tool calls with streaming feedback.

---

## Core Principles (from Rube Production System)

> "Your PRIMARY PURPOSE is to EXECUTE tasks using external tools - you are NOT a chat assistant."

Act embodies 5 fundamental principles:

1. **Action-First Mandate**  
   Execute immediately unless user explicitly asks for explanation only. No "Would you like me to..." - just do it.

2. **Never Invent Tool Slugs**  
   Only use tools returned by COMPOSIO_SEARCH_TOOLS. No hardcoded tool lists.

3. **Connection Check Before Execute**  
   Always verify toolkit connections before tool execution. Show auth link if needed.

4. **Session Persistence**  
   Reuse session IDs across conversation turns (1-hour TTL). Cache sessions per user+conversation.

5. **Confirm Destructive, Skip Read-Only**  
   Only confirm: sends, deletes, public-facing actions. Auto-execute: reads, searches, fetches.

---

## Target Audience

### Primary Persona: Busy Operator
- Demographics: 25-45, founder/PM/ops lead, heavy SaaS usage
- Goals: "Do it for me" automation (email, tickets, reports) from mobile
- Pain Points: context-switching, manual copy/paste, credential friction

### Secondary Persona: Power User
- Demographics: 20-50, engineer/analyst, wants deterministic execution
- Goals: fast reads, safe writes, repeatable workflows
- Pain Points: unreliable slugs, hidden auth requirements, unclear tool output

---

## Problem & Solution
Modern work spans dozens of tools, each with its own UI and auth. Act reduces time-to-action by:
1. Turning user intent into Composio Tool Router sessions
2. Using COMPOSIO_SEARCH_TOOLS to discover tools with execution plans
3. Following recommended_plan_steps and avoiding known_pitfalls
4. Executing tools with proper confirmation flow

---

## Features & Timeline

### Phase 1: MVP (6 weeks)
**Goal:** Prove core loop works - user message → tool discovery → execution

| Feature | Description |
|---------|-------------|
| Chat UI | Streaming responses, message history |
| LLM Integration | Direct OpenAI API (user provides key) |
| Tool Router | Session creation + 1-hour caching |
| Tool Discovery | COMPOSIO_SEARCH_TOOLS with execution guidance |
| OAuth | Connect toolkits via ASWebAuthenticationSession |
| Confirmation | Three modes: Always Ask / Adaptive / YOLO |
| Persistence | SwiftData for local conversations |
| Settings | API key management, confirmation mode |

### Phase 2: Polish (4 weeks)
| Feature | Description |
|---------|-------------|
| Apple Foundation Models | On-device LLM option (iOS 26+) |
| Multi-toolkit | Enable 5+ toolkits per session |
| Memory system | Per-toolkit ID mappings |
| Offline queue | Retry non-destructive actions |

### Phase 3: Expansion
| Feature | Description |
|---------|-------------|
| Voice input | Deepgram transcription |
| Recipe system | Save/execute workflows |
| Trigger support | Webhook-based automation |
| Cloud sync | Appwrite for cross-device |

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| UI response | < 300ms to user input |
| First token | < 1s for streaming response |
| Tool execution | < 5s (excluding external API) |
| Offline | Read-only local history |
| Security | Keys in Keychain, redacted logs |

---

## Platform Requirements

| Requirement | Value |
|-------------|-------|
| Minimum iOS | 17.0 (SwiftData, @Observable) |
| Composio SDK | iOS 15.0+ |
| Target devices | iPhone (MVP), iPad (nice-to-have) |
| Optional iOS 26 | Liquid Glass, Foundation Models |

> **iOS 26 Details**: See [liquid-glass-implementation.md](./design/liquid-glass-implementation.md)

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Time to first successful action | < 5 minutes after install |
| Sessions with tool execution | ≥ 30% |
| Tool execution success rate | > 95% |
| Connection failure rate | < 5% |
| Crash-free sessions | > 99.5% |

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| COMPOSIO_SEARCH_TOOLS returns empty | Warn if toolkit not in session; dynamically add |
| OAuth flow abandoned | Persist "initiated" state; show retry CTA |
| API key exposure | Keychain storage; never log keys |
| High LLM costs | User-owned keys; show token estimates |
| Composio SDK changes | Custom SDK; version lock; test suite |

---

## Out of Scope (MVP)

- Cloud backend / server deployment
- Cross-device sync (Phase 3)
- Voice input (Phase 3)
- Recipe/workflow system (Phase 3)
- iPad-optimized UI
- watchOS/visionOS

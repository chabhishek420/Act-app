# Act - iOS AI Assistant

> Execute authenticated actions across 500+ productivity tools via natural language.

## Overview

**Act** is an iOS-native AI assistant built on Composio's Tool Router architecture. It enables users to:

- 📧 Send emails via Gmail
- 💬 Post messages to Slack
- 📋 Create issues in Linear/GitHub
- 📝 Update Notion pages
- ...and 500+ more integrations

## Quick Start

1. **Open the project** in Xcode:
   ```bash
   open Act.xcodeproj
   ```

2. **Select an iOS Simulator** (e.g., iPhone 15) from the destination picker in the Xcode toolbar.

3. **Build and run** (⌘R).

*Note: The Composio SDK is already linked as a local package.*

## Architecture

```
Act iOS App
    ├── ChatView (SwiftUI)
    ├── ChatViewModel (@Observable)
    ├── SessionManager (actor)
    └── Composio Swift SDK
            └── Composio Cloud API (v3)
```

## Documentation

All documentation is in the [`docs/`](./docs/) directory. **For autonomous implementation, start with the [Master Plan](./docs/BUILD_ORCHESTRATOR.md).**

| Category | Purpose |
|----------|---------|
| [**BUILD_ORCHESTRATOR.md**](./docs/BUILD_ORCHESTRATOR.md) | **Master Implementation Plan (Start Here)** |
| [SPEC.md](./docs/SPEC.md) | Technical specification |

| [architecture/](./docs/architecture/) | System design |
| [api/](./docs/api/) | Composio SDK integration |
| [design/](./docs/design/) | SwiftUI and UX |
| [data/](./docs/data/) | Database and security |
| [ops/](./docs/ops/) | DevOps and testing |

## Requirements

- **Xcode** 16.0+ (iOS 26 SDK for Liquid Glass)
- **iOS** 17.0+ deployment target
- **Swift** 5.9+

## Dependencies

| Package | Purpose |
|---------|---------|
| [composio-swift](../composio-swift/) | Composio API client |
| [MacPaw/OpenAI](https://github.com/MacPaw/OpenAI) | OpenAI API client |

## License

Proprietary - All rights reserved.

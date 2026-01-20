# Testing Plan

## Framework
**Unit Tests:** Swift Testing (Xcode 16+)  
**UI Tests:** XCUITest  
**Coverage Target:** 80% of service + ViewModel logic

---

## Unit Tests

### Priority Areas
- SessionManager cache logic (TTL, composite key)
- ConfirmationService decision tree
- OAuth error handling
- Keychain operations
- Tool execution error mapping

### SessionManager Tests

```swift
import Testing
@testable import Act

@Suite("SessionManager Tests")
struct SessionManagerTests {
    
    @Test func testSessionCaching() async throws {
        let manager = SessionManager(composio: MockComposio())
        
        let session1 = try await manager.getSession(
            for: "user1", 
            conversationId: "conv1"
        )
        let session2 = try await manager.getSession(
            for: "user1", 
            conversationId: "conv1"
        )
        
        #expect(session1.sessionId == session2.sessionId)
    }
    
    @Test func testDifferentConversations() async throws {
        let manager = SessionManager(composio: MockComposio())
        
        let session1 = try await manager.getSession(
            for: "user1", 
            conversationId: "conv1"
        )
        let session2 = try await manager.getSession(
            for: "user1", 
            conversationId: "conv2"
        )
        
        #expect(session1.sessionId != session2.sessionId)
    }
}
```

### ConfirmationService Tests

```swift
@Suite("Confirmation Logic Tests")
struct ConfirmationTests {
    
    @Test(arguments: [
        ("GMAIL_FETCH_EMAILS", ConfirmationMode.adaptive, false),
        ("GMAIL_SEND_EMAIL", ConfirmationMode.adaptive, true),
        ("GMAIL_SEND_EMAIL", ConfirmationMode.yolo, false),
        ("GITHUB_DELETE_REPOSITORY", ConfirmationMode.yolo, true),
        ("SLACK_GET_MESSAGES", ConfirmationMode.alwaysAsk, true),
    ])
    func testShouldConfirm(slug: String, mode: ConfirmationMode, expected: Bool) {
        let service = ConfirmationService()
        let result = service.shouldConfirm(slug, mode: mode)
        #expect(result == expected)
    }
}
```

---

## Integration Tests

### API Contract Tests

```swift
@Suite("API Contract Tests")
struct APIContractTests {
    
    @Test func testToolRouterSessionDecoding() throws {
        let json = """
        {
            "session_id": "sess_123",
            "mcp": {"url": "wss://..."},
            "tool_router_tools": ["COMPOSIO_SEARCH_TOOLS"]
        }
        """
        
        let session = try JSONDecoder().decode(
            ToolRouterSession.self, 
            from: json.data(using: .utf8)!
        )
        
        #expect(session.sessionId == "sess_123")
        #expect(session.toolRouterTools.contains("COMPOSIO_SEARCH_TOOLS"))
    }
    
    @Test func testExecuteResponseDecoding() throws {
        let json = """
        {"data": {"messages": []}, "error": null, "log_id": "log_abc"}
        """
        
        let response = try JSONDecoder().decode(
            ToolRouterExecuteResponse.self,
            from: json.data(using: .utf8)!
        )
        
        #expect(response.error == nil)
        #expect(response.logId == "log_abc")
    }
}
```

---

## UI Tests

### Core Flows

```swift
final class ChatUITests: XCTestCase {
    
    func testSendMessage() throws {
        let app = XCUIApplication()
        app.launch()
        
        // Navigate to chat
        // Verify Chat is default screen
        XCTAssertTrue(app.staticTexts["Act 1 >"].exists)
        
        // Type and send
        let textField = app.textFields["Ask anything..."]
        textField.tap()
        textField.typeText("Hello")
        app.buttons["Send"].tap()
        
        // Verify message appears
        XCTAssertTrue(app.staticTexts["Hello"].exists)
    }
    
    func testConfirmationFlow() throws {
        let app = XCUIApplication()
        app.launchArguments = ["--mock-tool-call"]
        app.launch()
        
        // Trigger destructive action
        // ...
        
        // Verify confirmation sheet
        XCTAssertTrue(app.sheets.firstMatch.waitForExistence(timeout: 5))
        
        // Approve
        app.buttons["Approve"].tap()
        
        // Verify execution
        XCTAssertTrue(app.staticTexts.matching(
            NSPredicate(format: "label CONTAINS 'success'")
        ).firstMatch.exists)
    }
}
```

---

## Manual QA Checklist

### Authentication
- [ ] OpenAI key entry and validation
- [ ] Composio key entry and validation
- [ ] Key persistence across app restart
- [ ] Key deletion on logout

### OAuth
- [ ] Gmail connect success
- [ ] OAuth cancel handling
- [ ] OAuth timeout handling
- [ ] Reconnect after expiry

### Tool Execution
- [ ] Read-only tools auto-execute
- [ ] Destructive tools show confirmation
- [ ] YOLO mode still confirms DELETE_*
- [ ] Error states display correctly

### Streaming
- [ ] Tokens appear in real-time
- [ ] Scroll follows new content
- [ ] Can scroll up during streaming

### iOS 26 Liquid Glass (if targeting iOS 26+)
- [ ] Glass effects render on iOS 26 Simulator
- [ ] Fallback renders correctly on iOS 25
- [ ] Morphing transitions animate smoothly
- [ ] Interactive glass responds to touch
- [ ] Text remains readable over glass
- [ ] Dark mode appearance correct
- [ ] VoiceOver accessibility maintained

---

## Performance Benchmarks

| Metric | Target | Critical |
|--------|--------|----------|
| App launch | < 1s | < 2s |
| Session creation | < 2s | < 3s |
| Tool execution | < 5s | < 8s |
| Memory (idle) | < 50MB | < 100MB |
| Scroll FPS | 60fps | 30fps |

---

## Beta Exit Criteria

Before advancing TestFlight stages:

- [ ] Crash-free rate > 99.5%
- [ ] OAuth success rate > 95%
- [ ] Tool execution success rate > 90%
- [ ] No P0 bugs open
- [ ] Performance benchmarks met

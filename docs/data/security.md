# Security Documentation

## API Key Management

### Storage: iOS Keychain

All sensitive keys stored in Keychain with hardware-backed encryption:

```swift
import Security

struct Keychain {
    enum KeychainError: Error {
        case saveFailed(OSStatus)
        case deleteFailed(OSStatus)
        case notFound
    }
    
    static func save(_ value: String, key: String) throws {
        let data = value.data(using: .utf8)!
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        // Delete existing item
        SecItemDelete(query as CFDictionary)
        
        // Add new item
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }
    
    static func get(_ key: String) -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        guard status == errSecSuccess, let data = result as? Data else {
            return nil
        }
        return String(data: data, encoding: .utf8)
    }
    
    static func delete(_ key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.deleteFailed(status)
        }
    }
}
```

### Key Types

| Key | Provider | Storage |
|-----|----------|---------|
| OpenAI API key | User | Keychain |
| Composio API key | App or User | Keychain |

### Display Rules
- Mask after 8 characters: `sk-proj-xxxx...`
- Never log full keys
- Clear from memory after use

---

## OAuth Security

### ASWebAuthenticationSession
- Uses system Safari (sandboxed)
- No credential sharing with app webviews
- Callback URL validation automatic

```swift
// Register in Info.plist
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>act</string>
    </array>
  </dict>
</array>

// Callback: act://oauth-callback
```

### Token Handling
- OAuth tokens managed by Composio Cloud
- Never stored locally
- Automatic refresh handled server-side

---

## Data Protection

### In-Transit
- All traffic over HTTPS
- Composio: `https://backend.composio.dev`
- OpenAI: `https://api.openai.com`
- No ATS exceptions required

### At-Rest
- SwiftData: Default iOS data protection
- Keychain: Hardware encryption on Secure Enclave devices
- Tool arguments: Redacted before persistence

---

## Sensitive Data Redaction

### Log Sanitization

```swift
struct Redaction {
    private static let sensitiveKeys = [
        "email", "password", "token", "key", "secret", 
        "body", "content", "message", "to", "cc", "bcc"
    ]
    
    static func sanitize(_ args: [String: Any]) -> [String: String] {
        var redacted: [String: String] = [:]
        
        for (key, value) in args {
            if sensitiveKeys.contains(where: { key.lowercased().contains($0) }) {
                redacted[key] = "[REDACTED]"
            } else if let str = value as? String, str.count > 100 {
                redacted[key] = String(str.prefix(50)) + "...[truncated]"
            } else {
                redacted[key] = String(describing: value)
            }
        }
        return redacted
    }
}
```

### Usage
- Before logging tool execution
- Before analytics events
- Before displaying in confirmation sheet

---

## Error Handling Security

### Safe Error Messages

```swift
func userFacingError(_ error: Error) -> String {
    switch error {
    case ComposioError.notAuthenticated:
        return "Please connect your account to continue."
    case ComposioError.rateLimited:
        return "Too many requests. Please wait."
    case ComposioError.networkError:
        return "Network error. Check your connection."
    default:
        // Never expose internal error details
        return "Something went wrong. Please try again."
    }
}
```

---

## Privacy Manifest

Required for App Store submission. Create `PrivacyInfo.xcprivacy`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyCollectedDataTypes</key>
    <array/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyTracking</key>
    <false/>
</dict>
</plist>
```

---

## Security Checklist

### Pre-Submission
- [ ] All keys in Keychain (not UserDefaults)
- [ ] No API keys in source code
- [ ] Privacy manifest complete
- [ ] Log redaction verified
- [ ] Error messages sanitized

### Runtime
- [ ] OAuth uses ASWebAuthenticationSession
- [ ] HTTPS only (no ATS exceptions)
- [ ] Session tokens expire and refresh
- [ ] Confirmation for destructive actions

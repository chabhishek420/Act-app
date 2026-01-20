# DevOps Documentation

## Build & Distribution

### Requirements
- Xcode 15.0+
- iOS 17.0+ deployment target
- Swift 5.9+

### Build Commands

```bash
# Build for simulator
xcodebuild -scheme Act -destination 'platform=iOS Simulator,name=iPhone 15'

# Build for device
xcodebuild -scheme Act -destination 'generic/platform=iOS'

# Archive for distribution
xcodebuild archive -scheme Act -archivePath ./build/Act.xcarchive
```

---

## CI/CD (GitHub Actions)

### Build & Test Workflow

```yaml
name: Build and Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: macos-14
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Select Xcode
      run: sudo xcode-select -s /Applications/Xcode_15.2.app
    
    - name: Build
      run: |
        xcodebuild build \
          -scheme Act \
          -destination 'platform=iOS Simulator,name=iPhone 15' \
          -skipPackagePluginValidation
    
    - name: Test
      run: |
        xcodebuild test \
          -scheme Act \
          -destination 'platform=iOS Simulator,name=iPhone 15' \
          -skipPackagePluginValidation
```

---

## TestFlight Distribution

### Manual
1. Archive in Xcode (Product → Archive)
2. Distribute App → TestFlight
3. Wait for processing
4. Add testers in App Store Connect

### Automated (Fastlane)

```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  lane :beta do
    build_app(scheme: "Act")
    upload_to_testflight
  end
end
```

---

## Environment Configuration

### Debug vs Release

```swift
#if DEBUG
let composioBaseURL = "https://backend.composio.dev"  // Same for now
let logLevel = LogLevel.debug
#else
let composioBaseURL = "https://backend.composio.dev"
let logLevel = LogLevel.error
#endif
```

### API Keys

For CI, use secrets:
```yaml
env:
  COMPOSIO_API_KEY: ${{ secrets.COMPOSIO_API_KEY }}
```

Never commit keys to repo.

---

## Crash Reporting

### Sentry (Recommended)

```swift
import Sentry

SentrySDK.start { options in
    options.dsn = "https://xxx@sentry.io/xxx"
    options.enableTracing = true
    options.tracesSampleRate = 0.1
}
```

### Firebase Crashlytics (Alternative)

```swift
import FirebaseCrashlytics

Crashlytics.crashlytics().log("Tool execution: \(toolSlug)")
```

---

## Monitoring

### Key Metrics (Phase 2)
- Session creation latency
- Tool execution success rate
- OAuth completion rate
- Crash-free sessions

### Alerting Thresholds
- Crash rate > 0.5%: P0
- Tool success < 90%: P1
- Session latency p95 > 5s: P2

---

## Release Checklist

### Pre-Submission
- [ ] Version bump in Xcode
- [ ] Update changelog
- [ ] All tests pass
- [ ] Profile on device
- [ ] Privacy manifest complete
- [ ] App Store screenshots updated

### Post-Release
- [ ] Monitor crash rate (24h)
- [ ] Check user feedback
- [ ] Respond to reviews
- [ ] Document hotfix if needed

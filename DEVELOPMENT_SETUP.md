## Act iOS App - Local Development Setup

This project uses a **local, embedded copy** of the Composio Swift SDK located in `Packages/composio-swift/`.

### Structure
```
Act-app/
├── Act/                    # Main iOS application
├── Packages/
│   └── composio-swift/    # Local Swift Package (our SDK)
└── docs/
```

### Why This Setup?
- **Instant iteration**: Changes to the SDK are immediately available in the app
- **Single source of truth**: The SDK lives within the Act project boundary
- **Self-contained**: No external path dependencies
- **Git-friendly**: Can be tracked as a submodule or regular folder

### Adding the SDK to Your Xcode Project

1. **Open Act.xcodeproj in Xcode**
2. **File → Add Package Dependencies**
3. **Click "Add Local..."**
4. **Navigate to**: `Act-app/Packages/composio-swift`
5. **Select it and click "Add Package"**

### For Package.swift Projects
```swift
dependencies: [
    .package(path: "./Packages/composio-swift")
]
```

### Remote Sync (Optional)
The SDK also exists as a standalone repo at:
- **GitHub**: https://github.com/chabhishek420/composio-swifft.git

To sync local changes to remote:
```bash
cd Packages/composio-swift
git add .
git commit -m "Update SDK"
git push origin main
```

### gitignore Recommendations
Already configured in `.gitignore`:
```
# Build artifacts
.build/
.swiftpm/
Packages/composio-swift/.build/

# Don't commit package resolution with local paths
Package.resolved
```

---
**Current SDK Version**: Development (tracking main branch)

# Coding Standards

## Working Style

1. **Read before writing** — Understand existing code before modifying it
2. **Build frequently** — Use `build_sim` after changes to catch errors early
3. **Verify visually** — Use `screenshot` after UI changes to confirm correctness

## Core Philosophy

- SwiftUI is the default UI paradigm — embrace its declarative nature
- Avoid legacy UIKit patterns and unnecessary abstractions
- Focus on simplicity, clarity, and native data flow
- Let SwiftUI handle the complexity — don't fight the framework

## State Management

Use SwiftUI's built-in property wrappers appropriately:
- `@State` — Local, ephemeral view state
- `@Binding` — Two-way data flow between views
- `@Observable` — Shared state (iOS 17+)
- `@Environment` — Dependency injection for app-wide concerns

### State Ownership Principles

- Views own their local state unless sharing is required
- State flows down, actions flow up
- Keep state as close to where it's used as possible
- Extract shared state only when multiple views need it

## Async Patterns

- Use `async/await` as the default for asynchronous operations
- Leverage `.task` modifier for lifecycle-aware async work
- Avoid Combine unless absolutely necessary
- Handle errors gracefully with try/catch

## Common Mistakes

### @ViewBuilder Rules

```swift
// BAD — variable assignment in @ViewBuilder
@ViewBuilder
var content: some View {
  let x = computeValue() // ERROR: not allowed
  Text("\(x)")
}

// GOOD — extract to computed property
@ViewBuilder
var content: some View {
  Text("\(computedValue)")
}

private var computedValue: Int {
  // computation here
}
```

### SwiftUI Mistake Table

| Mistake | Correct |
|---------|---------|
| Using CGFloat bindings with Slider | Use Binding<Double> |
| let assignments in @ViewBuilder | Extract to computed properties or helper functions |
| Nested NavigationStack inside a NavigationLink destination | Only use one root NavigationStack |
| Using .task { } for synchronous setup | Use .onAppear { } for sync, .task { } for async |
| Force-unwrapping optionals in views | Use if let or provide defaults |
| Unnecessary ViewModels for simple views | Use @State for local state, @Observable for shared state |
| Creating ViewModels for every view | Only extract @Observable when state is truly shared |
| Using Combine for simple async operations | Use async/await with .task modifier |
| Moving state out of views unnecessarily | Keep state local unless sharing is required |

## Swift Conventions

- Use `private` for internal state and helpers
- Prefer computed properties over methods with no parameters
- Use `MARK: -` comments to organize view sections (State, Body, Subviews, Helpers)
- Keep views small and compose them together
- Feature-based file organization (not by type)
- Use extensions to organize large files
- Follow Swift naming conventions consistently

## Patterns

### @Observable with Environment

```swift
@Observable
class UserSession {
    var isAuthenticated = false
    var currentUser: User?

    func signIn(user: User) {
        currentUser = user
        isAuthenticated = true
    }
}

struct MyApp: App {
    @State private var session = UserSession()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(session)
        }
    }
}
```

### Async Data Loading

```swift
struct ProfileView: View {
    @State private var profile: Profile?
    @State private var isLoading = false
    @State private var error: Error?

    var body: some View {
        Group {
            if isLoading {
                ProgressView()
            } else if let profile {
                ProfileContent(profile: profile)
            } else if let error {
                ErrorView(error: error)
            }
        }
        .task {
            await loadProfile()
        }
    }

    private func loadProfile() async {
        isLoading = true
        defer { isLoading = false }

        do {
            profile = try await ProfileService.fetch()
        } catch {
            self.error = error
        }
    }
}
```

## DO / DON'T

### DO:
- Write self-contained views when possible
- Use property wrappers as intended by Apple
- Test logic in isolation, preview UI visually
- Handle loading and error states explicitly
- Keep views focused on presentation
- Use Swift's type system for safety
- Use Swift Concurrency (async/await, actors)

### DON'T:
- Create ViewModels for every view
- Move state out of views unnecessarily
- Add abstraction layers without clear benefit
- Use Combine for simple async operations
- Fight SwiftUI's update mechanism
- Overcomplicate simple features

## Testing Strategy

### What to Test Where

| Layer | What to Test | How |
|-------|-------------|-----|
| **Unit tests** | @Observable classes, business logic, data transformations | XCTestCase — instantiate, call methods, assert state |
| **PreviewRenderer** | Layout, styling, visual verification | Render to PNG and inspect (see mcp-tools.md) |
| **Simulator** | Navigation flows, gestures, full integration | build_run_sim + describe_ui + tap |

### Unit Testing @Observable Classes

Test @Observable classes as plain Swift classes — no special setup needed:

```swift
import XCTest
@testable import YourApp

final class UserSessionTests: XCTestCase {
    func testSignIn() {
        let session = UserSession()
        let user = User(name: "Test")

        session.signIn(user: user)

        XCTAssertTrue(session.isAuthenticated)
        XCTAssertEqual(session.currentUser?.name, "Test")
    }

    func testInitialState() {
        let session = UserSession()

        XCTAssertFalse(session.isAuthenticated)
        XCTAssertNil(session.currentUser)
    }
}
```

### View Testing with PreviewRenderer

Use the PreviewRenderer utility (defined in mcp-tools.md) to render any view to a PNG for visual inspection:

1. Edit the test to render the target view
2. Run with `test_sim(extraArgs: ["-only-testing:AppTests/PreviewRenderTests/testRenderPreview"])`
3. Read `/tmp/swiftui_preview.png` to inspect the result

This workflow catches layout issues, missing content, and styling problems without navigating through the full app.

### Test Organization

- Mirror the feature folder structure in the test target
- Naming: `{FeatureName}Tests.swift`
- One test class per @Observable class or complex utility
- Keep tests simple and focused — don't sacrifice code clarity for testability

## Asset Catalogs

No special tooling needed. Asset catalogs are directories with Contents.json files — create with mkdir + Write.

- Color set: `Assets.xcassets/ColorName.colorset/Contents.json`
- Image set: `Assets.xcassets/ImageName.imageset/Contents.json`

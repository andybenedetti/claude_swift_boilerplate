# MCP Tools

## Available Servers

Server: xcodebuild
Key Tools: build_sim, build_run_sim, test_sim, screenshot, tap, gesture, describe_ui, type_text
Use When: Building, running, testing, interacting with the simulator
────────────────────────────────────────
Server: xcodeproj
Key Tools: add_file, add_target, create_group, set_build_setting, add_swift_package
Use When: Modifying the Xcode project structure
────────────────────────────────────────
Server: apple-docs
Key Tools: search_framework_symbols, get_apple_doc_content, search_apple_docs
Use When: Looking up SwiftUI APIs and documentation
────────────────────────────────────────
Server: swiftlens
Key Tools: swift_validate_file
Use When: Checking Swift code for errors before building
────────────────────────────────────────
Server: xcode-diagnostics
Key Tools: get_project_diagnostics
Use When: Getting structured error details after a build failure

## Session Setup

At start of session, discover the project and set defaults:

mcp__xcodebuild__discover_projs(workspaceRoot: "/path/to/project")
mcp__xcodebuild__list_sims()

mcp__xcodebuild__session-set-defaults(
projectPath: "/path/to/Project.xcodeproj",
scheme: "YourScheme",
simulatorId: "SIMULATOR-UUID"
)

Important: Use simulatorId (UUID) from list_sims, not simulatorName + useLatestOS. Name-based matching fails when multiple OS
versions are installed.

## Core Workflows

### Build

mcp__xcodebuild__build_sim()

### Build and Run

mcp__xcodebuild__build_run_sim()

### Screenshot

mcp__xcodebuild__screenshot()

### Simulator Interaction

Always call describe_ui before tap to get accurate coordinates. Never guess from screenshots.

mcp__xcodebuild__describe_ui()
mcp__xcodebuild__tap(x: 196, y: 400)
mcp__xcodebuild__gesture(preset: "scroll-down")
mcp__xcodebuild__type_text(text: "Hello world")

tap(label:) works when the accessibility label matches exactly (including commas and counts).

### Run Tests

mcp__xcodebuild__test_sim()

# Run a specific test
mcp__xcodebuild__test_sim(
extraArgs: ["-only-testing:TargetName/TestClass/testMethod"]
)

### Debug Build Failures

# Build output usually has enough info. For structured diagnostics:
mcp__xcode-diagnostics__get_project_diagnostics()

### Build Performance

- The xcodebuild MCP uses incremental builds automatically.
- Use preferXcodebuild: true if the incremental system has issues.

## Xcode Project Management (xcodeproj)

Never edit project.pbxproj by hand. Use the xcodeproj MCP server.

### Add a File

mcp__xcodeproj__add_file(
project_path: "Project.xcodeproj",
file_path: "Sources/NewFile.swift",
target_name: "MyApp",
group_name: "Sources"
)

### Create a Group

mcp__xcodeproj__create_group(
project_path: "Project.xcodeproj",
group_name: "NewFolder",
path: "NewFolder",
parent_group: "Sources"
)

### Add a Target

mcp__xcodeproj__add_target(
project_path: "Project.xcodeproj",
target_name: "MyAppTests",
product_type: "unitTestBundle",
bundle_identifier: "com.example.app.tests"
)

### Set Build Settings

mcp__xcodeproj__set_build_setting(
project_path: "Project.xcodeproj",
target_name: "MyAppTests",
configuration: "All",
setting_name: "TEST_HOST",
setting_value: "$(BUILT_PRODUCTS_DIR)/MyApp.app/$(BUNDLE_EXECUTABLE_FOLDER_PATH)/MyApp"
)

### Add Swift Package Dependencies

mcp__xcodeproj__add_swift_package(
project_path: "Project.xcodeproj",
package_url: "https://github.com/org/package",
requirement: "from: 1.0.0",
target_name: "MyApp"
)

### Known xcodeproj Limitations

add_target doesn't create a productReference. For unit test targets, manually add to project.pbxproj:
1. A PBXFileReference for the .xctest product
2. Add it to the Products PBXGroup's children array
3. Add productReference = ID; to the PBXNativeTarget entry

add_target creates a bad INFOPLIST_FILE setting. Set GENERATE_INFOPLIST_FILE = YES and remove the INFOPLIST_FILE entry.

New test targets aren't added to the scheme. Edit the .xcscheme XML to add a <TestableReference> with the target's
BlueprintIdentifier.

Always verify with list_targets after adding a new target.

## Apple Docs Integration

# Search for symbols in a framework
mcp__apple-docs__search_framework_symbols(framework: "swiftui", namePattern: "*View")

# Read full documentation
mcp__apple-docs__get_apple_doc_content(
url: "https://developer.apple.com/documentation/swiftui/button"
)

# Search broadly
mcp__apple-docs__search_apple_docs(query: "NavigationStack")

# Check platform availability
mcp__apple-docs__get_platform_compatibility(
apiUrl: "https://developer.apple.com/documentation/swiftui/list"
)

URL pattern: https://developer.apple.com/documentation/{framework}/{typename}

## Preview Rendering with ImageRenderer

For verifying SwiftUI views without navigating through the full app, set up an ImageRenderer-based test target.

### PreviewRenderer Utility

```swift
import SwiftUI
import UIKit

@MainActor
enum PreviewRenderer {
    nonisolated static let defaultSize = CGSize(width: 393, height: 852)
    nonisolated static let defaultScale: CGFloat = 2.0
    nonisolated static let defaultOutputPath = "/tmp/swiftui_preview.png"

    @discardableResult
    static func renderToFile<V: View>(
        _ view: V,
        size: CGSize = defaultSize,
        scale: CGFloat = defaultScale,
        outputPath: String = defaultOutputPath
    ) -> Bool {
        let renderer = ImageRenderer(content: view.frame(width: size.width, height: size.height))
        renderer.scale = scale
        guard let data = renderer.uiImage?.pngData() else { return false }
        do {
            try data.write(to: URL(fileURLWithPath: outputPath))
            return true
        } catch { return false }
    }
}
```

### Test File

```swift
import XCTest
import SwiftUI
@testable import YourApp

final class PreviewRenderTests: XCTestCase {
    @MainActor
    func testRenderPreview() throws {
        let view = NavigationStack { SomeView() }
        let success = PreviewRenderer.renderToFile(view)
        XCTAssertTrue(success)
    }
}
```

### Workflow

# 1. Edit the test to render any view
# 2. Run the test
mcp__xcodebuild__test_sim(
extraArgs: ["-only-testing:AppTests/PreviewRenderTests/testRenderPreview"]
)
# 3. Read the rendered image
Read("/tmp/swiftui_preview.png")

### Notes
- ImageRenderer requires iOS 16+. Use @MainActor on the enum and test method.
- Mark static let constants nonisolated to avoid Swift 6 warnings as default parameters.
- Size 393x852 = iPhone 16 in points; scale 2.0 = retina.
- Views using async loading (AsyncImage, Maps) render their immediate state only.
- Wrap views in NavigationStack if they expect a navigation context.

## Test Target Setup

1. Create a test target via xcodeproj MCP (see "Add a Target" above)
2. Set TEST_HOST, BUNDLE_LOADER, GENERATE_INFOPLIST_FILE = YES
3. Add dependency on the app target
4. Add to the scheme's <Testables> (see Known Limitations)

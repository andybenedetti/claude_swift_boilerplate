# Claude Code SwiftUI Boilerplate

A ready-to-use `.claude/` configuration for iOS development with [Claude Code](https://claude.ai/claude-code). Clone this into any SwiftUI project to get an optimized AI-assisted development environment with build tooling, simulator control, and Apple documentation — all wired up.

## Prerequisites

Install these on your Mac before using the MCP servers:

| Dependency                     | Why                                                            |
|--------------------------------|----------------------------------------------------------------|
| **Xcode** (with iOS Simulator) | Building and running the app in the simulator                  |
| **Node.js / npm**              | For the xcodebuild and apple-docs MCP servers (they use `npx`) |
| **Docker**                     | For the xcodeproj MCP server (runs in a container)             |
| **Python 3 + pip**             | For the xcode-diagnostics MCP server                           |
| **uv** (`brew install uv`)     | For the swiftlens MCP server                                   |

## MCP Server Installation

Ask Claude to install these for you (or you can run each command once per machine):

```bash
# xcodebuild — Build, run, test, simulator control, UI interaction
claude mcp add xcodebuild -- npx -y xcodebuildmcp@latest

# xcodeproj — Xcode project file manipulation
claude mcp add xcodeproj -- docker run --pull=always --rm -i \
  -v "$PWD:/workspace" ghcr.io/giginet/xcodeproj-mcp-server:latest /workspace

# apple-docs — Apple Developer Documentation
claude mcp add apple-docs -- npx -y @kimsungwhee/apple-docs-mcp@latest

# swiftlens — Swift code analysis
claude mcp add swiftlens -- uvx swiftlens

# xcode-diagnostics — Structured build errors
pip3 install git+https://github.com/leftspin/mcp-xcode-diagnostics.git
claude mcp add xcode-diagnostics -- python3 -m mcp_xcode_diagnostics
```

## Getting Started

1. Clone this repo (or copy the `.claude/` directory into your existing project)
2. Ask Claude to install the MCP servers needed for this project
3. Open Claude Code in your project directory
4. Start building — Claude will automatically discover the project and set up the simulator

## Why This Boilerplate

- **All rules auto-loaded** — Everything in `rules/` and `CLAUDE.md` is injected into context automatically. No invisible instructions, no manual prompting.
- **Concise by design** — Anthropic's docs warn that bloated CLAUDE.md causes instructions to be ignored. Each rule file is focused and minimal.
- **5 MCP servers** covering the full Xcode lifecycle — build, run, test, screenshot, UI interaction, project file management, Apple documentation, Swift analysis, and structured diagnostics.
- **PreviewRenderer** for visual verification — Render any SwiftUI view to PNG without navigating through the app. Anthropic calls verification "the single highest-leverage thing you can do."
- **Coding standards** with SwiftUI-specific pitfall tables, state management guidance, and async patterns.
- **Testing strategy** across three layers — unit tests for logic, PreviewRenderer for layout, simulator for integration.
- **pbxproj protection hook** — A PreToolUse hook deterministically blocks direct edits to `project.pbxproj`, enforcing use of the xcodeproj MCP server.
- **Compaction instructions** — Tells Claude what to preserve when auto-summarizing long conversations.
- **Clean separation** — Human-facing setup lives in README; Claude-facing runtime rules live in `.claude/`.

## What's Included

```
.gitignore                           # Xcode, macOS, and Claude Code ignores
.claude/
  CLAUDE.md                          # Session startup instructions, compaction rules, memory file references
  PROGRESS.md                        # Cross-session progress tracking (update after each session)
  settings.json                      # Hooks (pbxproj protection)
  rules/
    coding-standards.md              # SwiftUI conventions, state management, async patterns, testing strategy
    mcp-tools.md                     # How to use each MCP server (build, test, screenshot, xcodeproj, docs)
    project-architecture.md          # Project structure, architecture decisions, file locations
    file-maintenance.md              # When and how to update these configuration files
```

All files in `rules/` are automatically loaded into Claude's context. `CLAUDE.md` is also auto-loaded. This means every instruction is visible to Claude without manual prompting.

## Additional Capabilities to Consider

Ideas that were evaluated during the design of this boilerplate and intentionally left out. You can always ask Claude to install them for you if you think one of them might be useful.

### SwiftLint PostToolUse hook

- **What it does**: Automatically runs SwiftLint after every file edit to enforce style rules.
- **Why it might be useful**: Catches style violations immediately, before they accumulate.
- **Why rejected**: Requires SwiftLint to be installed on the user's machine. Too opinionated for a boilerplate — users who want it can add it themselves.

### Deep link navigation (`simctl openurl`)

- **What it does**: Uses `xcrun simctl openurl` to navigate directly to specific screens in the app via URL schemes.
- **Why it might be useful**: Speeds up testing by skipping navigation to reach a target screen.
- **Why rejected**: Requires an app-specific URL scheme to be implemented first. Not useful in a blank template.

### Console output capture patterns

- **What it does**: Adds rules for capturing and filtering `os_log` / `print` output from the running simulator app.
- **Why it might be useful**: Lets Claude read runtime logs to debug issues without the user copying output manually.
- **Why rejected**: Requires project-specific values (bundle ID, app scheme). Better documented in a project's own CLAUDE.md once those values exist.

### DerivedData-in-project

- **What it does**: Configures Xcode to place DerivedData inside the project directory instead of `~/Library/Developer/Xcode/DerivedData`.
- **Why it might be useful**: Makes build artifacts project-local, which can simplify cleanup and avoid cross-project cache conflicts.
- **Why rejected**: Requires changing Xcode preferences, which is too invasive for a template. Better suited to a specific project's setup.

### Subagents (security-reviewer, etc.)

- **What it does**: Defines specialized Claude Code subagents for tasks like security review, accessibility auditing, or performance analysis.
- **Why it might be useful**: Provides focused, expert-level analysis for specific concerns.
- **Why rejected**: Too project-specific. Users should define subagents tailored to their actual codebase and requirements.

### PRD-driven workflows

- **What it does**: Establishes a structured workflow where Claude generates a PRD, then a technical spec, then a task list before writing code.
- **Why it might be useful**: Enforces disciplined planning for large features, reducing wasted iteration.
- **Why rejected**: Over-engineered for a boilerplate. Great for large apps, overkill for a starting template.


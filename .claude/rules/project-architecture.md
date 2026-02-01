# Project Architecture

## Overview

<!-- Describe the app's purpose in 1-2 sentences -->

## Architecture Decisions

1. **Navigation**: NavigationStack-based
2. **State**: @State for local, @Observable for shared
3. **Organization**: Feature-based folder structure
4. **Minimum iOS**: 17.0

## File Structure

<!-- Update this tree as the project evolves -->

ProjectName/
  ├── ProjectNameApp.swift        # App entry point
  ├── Info.plist
  ├── Assets.xcassets/
  └── Features/
      └── Home/
          └── HomeView.swift

## Frameworks Used

```swift
import SwiftUI
```

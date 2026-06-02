# Project Structure

Use this reference when deciding how an Apple app repository should be laid out before feature code starts sprawling across unrelated folders.

## Preferred App-First Layout

For non-trivial apps, organize by feature first and keep genuinely shared code in a focused library layer:

```text
Project/
├── Features/
│   ├── Home/
│   ├── Settings/
│   ├── Main/
│   └── [Feature]/
├── Library/
│   ├── Routes/
│   ├── Extensions/
│   ├── Views/
│   ├── Models/
│   ├── Services/
│   ├── Database/
│   └── Core/
└── Resources/
    ├── AppAssets.xcassets
    └── App.xcstrings
```

For teams that want a stronger engineering split inside each feature, use a generated shape like this:

```text
Project/
├── App/
│   ├── AppDelegate/
│   ├── Scene/
│   ├── Root/
│   └── Bootstrap/
├── Features/
│   ├── Home/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Routing/
│   │   └── Resources/
│   │       ├── HomeAssets.xcassets
│   │       └── Home.xcstrings
│   ├── Settings/
│   └── [Feature]/
├── Library/
│   ├── Routes/
│   ├── Extensions/
│   ├── Views/
│   ├── Models/
│   ├── Services/
│   ├── Database/
│   ├── Localization/
│   ├── Resources/
│   └── Core/
└── Resources/
    ├── AppAssets.xcassets
    └── App.xcstrings
```

## Clean Architecture Layout

For medium-to-large projects with strict layer separation:

```text
Project/
├── App/
│   ├── AppEntry.swift
│   ├── DIContainer.swift
│   └── AppCoordinator.swift
├── Domain/                        (NO framework imports — pure Swift)
│   ├── Entities/
│   │   ├── User.swift
│   │   └── Product.swift
│   ├── UseCases/
│   │   ├── FetchUserUseCase.swift
│   │   └── PlaceOrderUseCase.swift
│   ├── Repositories/              (protocols only)
│   │   ├── UserRepositoryProtocol.swift
│   │   └── ProductRepositoryProtocol.swift
│   └── Errors/
│       └── DomainError.swift
├── Data/
│   ├── Network/
│   │   ├── APIClient.swift
│   │   ├── Endpoints/
│   │   └── DTOs/
│   ├── Persistence/
│   │   ├── CoreDataStack.swift
│   │   └── UserDAO.swift
│   ├── Repositories/              (implementations)
│   │   ├── UserRepository.swift
│   │   └── ProductRepository.swift
│   └── Mappers/
│       ├── UserMapper.swift
│       └── ProductMapper.swift
├── Presentation/
│   ├── Features/
│   │   ├── Home/
│   │   │   ├── HomeView.swift
│   │   │   └── HomeViewModel.swift
│   │   └── Profile/
│   │       ├── ProfileView.swift
│   │       └── ProfileViewModel.swift
│   ├── Navigation/
│   │   └── Router.swift
│   └── DesignSystem/
│       ├── Components/
│       └── Theme.swift
└── Resources/
```

## SPM Modular Layout

For large projects (50k+ LOC, 3+ developers), split into SPM local packages:

```text
MyApp.xcodeproj
├── Packages/
│   ├── CoreKit/           (models, protocols, utilities shared by all)
│   ├── NetworkKit/        (API client, endpoints, DTOs)
│   ├── DesignSystem/      (UI components, colors, fonts, tokens)
│   ├── Features/
│   │   ├── HomeFeature/   (Home screen: view, viewmodel, local models)
│   │   ├── ProfileFeature/
│   │   ├── OrderFeature/
│   │   └── AuthFeature/
│   └── SharedServices/    (analytics, logging, feature flags)
└── App/                   (main target — ties everything together)
    ├── MyAppApp.swift
    └── DIContainer.swift
```

### Module Dependency Rules

```
                    ┌─────────┐
                    │   App   │  (imports everything)
                    └────┬────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼────┐    ┌──────▼──────┐   ┌────▼────┐
   │HomeFeature│   │ProfileFeature│  │OrderFeature│
   └────┬────┘    └──────┬──────┘   └────┬────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼────┐    ┌──────▼──────┐   ┌────▼────────┐
   │NetworkKit│   │DesignSystem │   │SharedServices│
   └────┬────┘    └──────┬──────┘   └────┬────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
                    ┌────▼────┐
                    │ CoreKit │  (no dependencies)
                    └─────────┘
```

Critical rules:
1. **Feature modules NEVER import each other.** HomeFeature cannot import ProfileFeature.
2. **Dependencies flow downward only.** CoreKit never imports NetworkKit.
3. **The App target is the only place that imports all modules** and wires them together.
4. **Circular dependencies are impossible** if you follow these rules.

### Feature Module Package.swift Example

```swift
// Packages/Features/HomeFeature/Package.swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "HomeFeature",
    platforms: [.iOS(.v17)],
    products: [
        .library(name: "HomeFeature", targets: ["HomeFeature"]),
    ],
    dependencies: [
        .package(path: "../../CoreKit"),
        .package(path: "../../NetworkKit"),
        .package(path: "../../DesignSystem"),
    ],
    targets: [
        .target(
            name: "HomeFeature",
            dependencies: ["CoreKit", "NetworkKit", "DesignSystem"]
        ),
        .testTarget(
            name: "HomeFeatureTests",
            dependencies: ["HomeFeature"]
        ),
    ]
)
```

### Feature Entry Point Pattern

Each feature module exposes a single public entry point. The App target creates it with injected dependencies.

```swift
// In HomeFeature module
public struct HomeView: View {
    @State private var viewModel: HomeViewModel

    public init(userRepository: UserRepositoryProtocol) {
        _viewModel = State(initialValue: HomeViewModel(userRepository: userRepository))
    }

    public var body: some View { /* ... */ }
}

// In App target — wires all features
@main
struct MyApp: App {
    let apiClient = APIClient(baseURL: URL(string: "https://api.example.com")!)

    var body: some Scene {
        WindowGroup {
            TabView {
                HomeView(userRepository: UserRepository(apiClient: apiClient))
                    .tabItem { Label("Home", systemImage: "house") }
                ProfileView(userRepository: UserRepository(apiClient: apiClient))
                    .tabItem { Label("Profile", systemImage: "person") }
            }
        }
    }
}
```

### When to Modularize

Modularize when:
- Build times exceed 30 seconds for incremental changes
- 3+ developers frequently cause merge conflicts in the same files
- You want to enforce architectural boundaries at compile time
- The project has 50k+ lines of code
- You need to share code between app and app extensions (widgets, watch)

Do NOT modularize when:
- Solo developer, small project
- Rapid prototyping phase (modularize later when architecture stabilizes)
- The team is unfamiliar with SPM

Incremental modularization strategy:
1. Extract `CoreKit` first (models, protocols) — smallest risk
2. Extract `DesignSystem` next (shared UI) — high reuse value
3. Extract `NetworkKit` — isolates all API concerns
4. Extract features one at a time, starting with the most independent one
5. Keep the monolithic app target shrinking until it only contains DI + navigation

## Structure Rules

- Prefer `Features/` for user-facing product areas that evolve independently.
- Keep `Library/` or an equivalent shared layer small and intentional; it should hold code used across multiple features, not code that simply has no owner.
- Keep `Resources/` separate from executable code so asset, localization, and generated resource management stay predictable.
- Prefer `.xcstrings` for localization catalogs rather than older loose string files in new Apple app projects.
- Keep each feature or module resource set in its own bundle when modularity matters; do not push every asset into the main bundle by default.
- If a screen is SwiftUI-first but depends on one UIKit bridge, keep the UIKit host or adapter close to that feature rather than burying it deep in generic shared folders.

## Feature-First vs Layer-First

Choose feature-first when:
- the app has more than a couple of user-facing flows
- multiple developers work on different product areas
- state, views, services, and models change together within a feature

Keep global layer-first folders only for code that is truly shared:
- route infrastructure
- design-system views
- cross-feature services
- persistence utilities
- low-level helpers
- localization helpers used across many modules
- shared resource-loading utilities for non-main bundles

## UIKit, SwiftUI, and Objective-C Coexistence

- UIKit is the primary UI layer; SwiftUI is used for new standalone screens when deployment target allows.
- Keep SwiftUI views behind explicit hosting boundaries (`UIHostingController`) when embedded in UIKit flows.
- Objective-C modules should be self-contained with clear Swift interop boundaries.
- Do not mix SwiftUI views and UIKit controllers in the same feature folder without making the ownership relationship obvious.

Good examples:
- `Features/Profile/ProfileView.swift`
- `Features/Profile/ProfileViewModel.swift`
- `Features/Profile/ProfileHostingController.swift`

Avoid:
- placing unrelated UIKit adapters in a generic catch-all `Controllers/` folder
- moving all SwiftUI code into one app-wide `Views/` folder once the app grows
- flattening all feature assets into one global asset catalog when modules should be independently owned

## File Naming Conventions

| Type | Convention | Example |
|---|---|---|
| View | `<Name>View.swift` | `HomeView.swift` |
| ViewModel | `<Name>ViewModel.swift` | `HomeViewModel.swift` |
| Model | `<Name>.swift` | `User.swift` |
| Protocol | `<Name>Protocol.swift` | `UserRepositoryProtocol.swift` |
| Extension | `<Type>+<Context>.swift` | `String+Validation.swift` |
| DTO | `<Name>DTO.swift` | `UserDTO.swift` |
| Mapper | `<Name>Mapper.swift` | `UserMapper.swift` |
| Mock | `Mock<Name>.swift` | `MockUserRepository.swift` |
| Use Case | `<Verb><Noun>UseCase.swift` | `FetchUserUseCase.swift` |

## File Splitting Rules

- Prefer one primary type or entry point per file.
- Split large SwiftUI views into focused files such as `HomeView+hero.swift` or `HomeView+list.swift` when sections are substantial.
- Split by responsibility, not by arbitrary line count.
- If a file mixes view layout, networking, persistence, and navigation policy, the structure is already too broad.

## Review Heuristics

- If a folder name does not tell you which feature owns the code, the structure is probably too generic.
- If shared folders keep growing while features stay thin, the library layer is becoming a dumping ground.
- If UIKit legacy files are scattered across the app shell, add explicit feature or bridge boundaries before adding more code.
- If a module cannot load its own assets or strings without reaching into the main bundle, the resource ownership boundary is too weak.

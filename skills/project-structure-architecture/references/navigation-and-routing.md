# Navigation And Routing

Use this reference when choosing how an Apple app should own navigation state and route transitions.

For UIKit-first projects, the primary navigation pattern is the **Coordinator** (see UIKit Coordinator Pattern section below). For SwiftUI screens, use the **Router** pattern.

## SwiftUI Router Pattern

For SwiftUI screens, prefer stack-based routing with:
- a route enum
- a router object that owns the path
- `NavigationStack` for per-flow navigation

This keeps routing explicit and testable without burying transitions in ad hoc button handlers.

## SwiftUI Router with Full Navigation Support

```swift
import SwiftUI

@Observable
final class Router {
    var path = NavigationPath()
    var sheet: Sheet?
    var fullScreenCover: FullScreenCover?

    // MARK: - Destinations

    enum Destination: Hashable {
        case profile(userId: UUID)
        case productDetail(productId: UUID)
        case orderHistory
        case settings
    }

    enum Sheet: Identifiable {
        case createProduct
        case editProfile
        case filter(current: FilterOptions)

        var id: String { String(describing: self) }
    }

    enum FullScreenCover: Identifiable {
        case onboarding
        case imageViewer(url: URL)

        var id: String { String(describing: self) }
    }

    // MARK: - Actions

    func push(_ destination: Destination) {
        path.append(destination)
    }

    func pop() {
        guard !path.isEmpty else { return }
        path.removeLast()
    }

    func popToRoot() {
        path = NavigationPath()
    }

    func present(_ sheet: Sheet) {
        self.sheet = sheet
    }

    func presentFullScreen(_ cover: FullScreenCover) {
        self.fullScreenCover = cover
    }

    func dismiss() {
        sheet = nil
        fullScreenCover = nil
    }
}
```

## Router Usage in Views

```swift
struct ContentView: View {
    @State private var router = Router()

    var body: some View {
        NavigationStack(path: $router.path) {
            HomeView()
                .navigationDestination(for: Router.Destination.self) { destination in
                    switch destination {
                    case .profile(let userId):
                        ProfileView(userId: userId)
                    case .productDetail(let productId):
                        ProductDetailView(productId: productId)
                    case .orderHistory:
                        OrderHistoryView()
                    case .settings:
                        SettingsView()
                    }
                }
        }
        .sheet(item: $router.sheet) { sheet in
            switch sheet {
            case .createProduct:
                CreateProductView()
            case .editProfile:
                EditProfileView()
            case .filter(let current):
                FilterView(options: current)
            }
        }
        .fullScreenCover(item: $router.fullScreenCover) { cover in
            switch cover {
            case .onboarding:
                OnboardingView()
            case .imageViewer(let url):
                ImageViewerView(url: url)
            }
        }
        .environment(router)
    }
}

// Child view triggers navigation
struct HomeView: View {
    @Environment(Router.self) private var router

    var body: some View {
        Button("Profile") {
            router.push(.profile(userId: UUID()))
        }
    }
}
```

## `NavigationStack` Ownership

- Give each tab or major app area its own router when stacks should stay independent.
- Keep navigation state close to the shell that owns it.
- Do not let leaf views create hidden global routing behavior if the app shell should own the stack.

Example:

```swift
struct MainView: View {
    @State private var homeRouter = Router()
    @State private var settingsRouter = Router()

    var body: some View {
        TabView {
            NavigationStack(path: $homeRouter.path) {
                HomeView()
            }
            .tabItem { Label("Home", systemImage: "house") }

            NavigationStack(path: $settingsRouter.path) {
                SettingsView()
            }
            .tabItem { Label("Settings", systemImage: "gear") }
        }
    }
}
```

## UIKit Coordinator Pattern

For UIKit-heavy projects or hybrid apps where navigation flows are complex, branching, or involve deep linking.

### Coordinator Protocol

```swift
@MainActor
protocol Coordinator: AnyObject {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get }
    func start()
}
```

### App Coordinator Example

```swift
@MainActor
final class AppCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        let homeCoordinator = HomeCoordinator(navigationController: navigationController)
        homeCoordinator.delegate = self
        childCoordinators.append(homeCoordinator)
        homeCoordinator.start()
    }
}

extension AppCoordinator: HomeCoordinatorDelegate {
    func homeDidSelectUser(_ userId: UUID) {
        let profileCoordinator = ProfileCoordinator(
            navigationController: navigationController,
            userId: userId
        )
        childCoordinators.append(profileCoordinator)
        profileCoordinator.start()
    }
}
```

### Feature Coordinator Example

```swift
@MainActor
protocol HomeCoordinatorDelegate: AnyObject {
    func homeDidSelectUser(_ userId: UUID)
}

@MainActor
final class HomeCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController
    weak var delegate: HomeCoordinatorDelegate?
    private let repository: UserRepository

    init(navigationController: UINavigationController, repository: UserRepository = .init()) {
        self.navigationController = navigationController
        self.repository = repository
    }

    func start() {
        let vm = HomeViewModel(repository: repository)
        vm.onSelectUser = { [weak self] userId in
            self?.delegate?.homeDidSelectUser(userId)
        }
        let vc = HomeViewController(viewModel: vm)
        navigationController.pushViewController(vc, animated: false)
    }
}
```

## Cross-Feature Communication (Modular Projects)

Since feature modules cannot import each other, use one of these patterns:

### 1. Closure-Based Callbacks

```swift
// Feature exposes a callback; App target handles navigation
public struct HomeView: View {
    let onUserSelected: (UUID) -> Void

    public init(userRepository: UserRepositoryProtocol, onUserSelected: @escaping (UUID) -> Void) {
        self.onUserSelected = onUserSelected
    }
}

// App target wires it
HomeView(userRepository: repo) { userId in
    router.push(.profile(userId))
}
```

### 2. Protocol-Based Navigator (defined in CoreKit)

```swift
// CoreKit/Protocols/AppNavigator.swift
public protocol AppNavigator: AnyObject {
    func showProfile(userId: UUID)
    func showOrder(orderId: UUID)
    func showSettings()
}
```

## When UIKit Coordinators Still Make Sense

UIKit coordinators remain reasonable when:
- the flow is still UIKit-heavy
- multiple legacy view controllers coordinate presentation and dismissal
- the app integrates with frameworks that still expect UIKit navigation ownership
- deep linking requires programmatic stack construction

Even then:
- keep coordinator boundaries explicit
- avoid giant coordinator objects that also become service locators
- prefer typed route surfaces where possible

## Review Heuristics

- If navigation can only be understood by reading many unrelated button actions, introduce a route surface.
- If one router owns unrelated stacks, split by app area.
- If SwiftUI and UIKit transitions interleave, make the bridge or host boundary explicit rather than letting navigation leak across layers.
- Navigation logic belongs in Router/Coordinator, not in ViewModel — ViewModel reports intent, Router/Coordinator acts on it.

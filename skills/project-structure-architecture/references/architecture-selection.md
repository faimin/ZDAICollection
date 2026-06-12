# Architecture Selection

Use this reference when choosing how much architectural structure an Apple app or feature actually needs.

## Core Principle

Choose the smallest architecture that keeps state ownership, effects, and navigation understandable. Do not pay TCA or Clean Architecture complexity for screens that only need straightforward local state.

## Decision Framework

```
START
  |
  v
Is the module written in Objective-C?
  YES -> MVP (Presenter owns logic, View is passive UIViewController)
  |
  v
Is the module written in Swift with UIKit?
  YES -> MVI (Intent enum + Combine state pipeline)
  |
  v
Is the module SwiftUI-first with simple local state?
  YES -> MVVM with @Observable
  |
  v
Do multiple features share state or trigger effects in each other?
  YES -> TCA (app-level)
  |
  v
Does the project require strict domain/data/presentation separation?
  YES -> Clean Architecture
  |
  v
Is navigation complex with deep linking and conditional flows?
  YES -> Add Coordinator pattern on top of the chosen architecture
```

## Selection Matrix

| Pattern | Best For | Language | Complexity | Testability | Scope |
|---------|----------|---------|-----------|-------------|-------|
| **MVP** | Objective-C modules, passive views | Objective-C | Low | High | Per-feature |
| **MVI** | Swift UIKit, complex state transitions | Swift | Medium | High | Per-page |
| **MVVM** | SwiftUI screens, simple features | Swift/SwiftUI | Low | High | Per-feature |
| **TCA** | Large apps, composable features, strong testing | Swift | High | Very High | App-wide |
| **Clean Architecture** | Enterprise apps, strict layer separation | Swift | High | Very High | App-wide |
| **Coordinator** | Complex navigation flows (UIKit or hybrid) | Both | Medium | High | Cross-feature |

## MVP (Objective-C Default)

MVP (Model-View-Presenter) is the default architecture for Objective-C modules. The View (UIViewController) is passive — it forwards user actions to the Presenter and renders state it receives back. The Presenter owns all logic and talks to the Model layer.

Prefer MVP when:
- the module is written in Objective-C
- you want clear testability by testing the Presenter in isolation
- the View should contain zero business logic

```objc
// MARK: - Presenter Protocol
@protocol ArticleListPresenterProtocol <NSObject>
- (void)viewDidLoad;
- (void)didSelectItemAtIndex:(NSInteger)index;
- (void)didPullToRefresh;
@end

// MARK: - View Protocol (Presenter -> View)
@protocol ArticleListViewProtocol <NSObject>
- (void)showLoading;
- (void)hideLoading;
- (void)reloadWithItems:(NSArray<ArticleViewModel *> *)items;
- (void)showError:(NSString *)message;
@end

// MARK: - Presenter
@interface ArticleListPresenter : NSObject <ArticleListPresenterProtocol>
@property (nonatomic, weak) id<ArticleListViewProtocol> view;
- (instancetype)initWithService:(id<ArticleServiceProtocol>)service
                     coordinator:(id<ArticleCoordinatorProtocol>)coordinator;
@end

@implementation ArticleListPresenter {
    id<ArticleServiceProtocol> _service;
    id<ArticleCoordinatorProtocol> _coordinator;
    NSArray<Article *> *_articles;
}

- (instancetype)initWithService:(id<ArticleServiceProtocol>)service
                     coordinator:(id<ArticleCoordinatorProtocol>)coordinator {
    if (self = [super init]) {
        _service = service;
        _coordinator = coordinator;
    }
    return self;
}

- (void)viewDidLoad {
    [self loadArticles];
}

- (void)didPullToRefresh {
    [self loadArticles];
}

- (void)didSelectItemAtIndex:(NSInteger)index {
    if (index < (NSInteger)_articles.count) {
        [_coordinator showArticleDetail:_articles[index]];
    }
}

- (void)loadArticles {
    [self.view showLoading];
    __weak typeof(self) weakSelf = self;
    [_service fetchArticlesWithCompletion:^(NSArray<Article *> *articles, NSError *error) {
        __strong typeof(weakSelf) self = weakSelf;
        if (!self) return;
        [self.view hideLoading];
        if (error) {
            [self.view showError:error.localizedDescription];
            return;
        }
        self->_articles = articles;
        NSArray *viewModels = [articles xx_map:^(Article *a) {
            return [[ArticleViewModel alloc] initWithArticle:a];
        }];
        [self.view reloadWithItems:viewModels];
    }];
}

@end

// MARK: - ViewController (passive View)
@interface ArticleListViewController () <ArticleListViewProtocol>
@property (nonatomic, strong) id<ArticleListPresenterProtocol> presenter;
@end

@implementation ArticleListViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self.presenter viewDidLoad];
}

- (void)showLoading {
    // 显示加载指示器
}

- (void)hideLoading {
    // 隐藏加载指示器
}

- (void)reloadWithItems:(NSArray<ArticleViewModel *> *)items {
    // 刷新列表数据
}

- (void)showError:(NSString *)message {
    // 显示错误提示
}

@end
```

Key rules:
- ViewController contains ZERO business logic — only UI rendering and forwarding user actions
- Presenter does not import UIKit — it works through the View protocol
- Navigation is delegated to a Coordinator, not owned by the Presenter
- View protocol methods describe UI state changes, not implementation details

### MVP in Swift (for legacy UIKit modules migrating to Swift)

When an Objective-C MVP module is rewritten in Swift but remains UIKit-based, keep the same MVP structure:

```swift
// MARK: - View Protocol
protocol WeatherPresenterView: AnyObject {
    func showLoading()
    func hideLoading()
    func display(_ viewData: WeatherViewData)
    func displayError(_ message: String)
}

// MARK: - Presenter
final class WeatherPresenter {
    private weak var view: WeatherPresenterView?
    private let service: WeatherServiceProtocol

    init(view: WeatherPresenterView, service: WeatherServiceProtocol) {
        self.view = view
        self.service = service
    }

    func viewDidLoad() {
        loadWeather(for: "Beijing")
    }

    func didSearchCity(_ city: String) {
        loadWeather(for: city)
    }

    private func loadWeather(for city: String) {
        view?.showLoading()
        service.fetchWeather(for: city) { [weak self] result in
            guard let self else { return }
            self.view?.hideLoading()
            switch result {
            case .success(let weather):
                self.view?.display(WeatherViewData(from: weather))
            case .failure(let error):
                self.view?.displayError(error.localizedDescription)
            }
        }
    }
}

// MARK: - ViewController (passive)
final class WeatherViewController: UIViewController, WeatherPresenterView {
    private var presenter: WeatherPresenter!

    override func viewDidLoad() {
        super.viewDidLoad()
        presenter = WeatherPresenter(view: self, service: WeatherService())
        presenter.viewDidLoad()
    }

    func showLoading() { /* 显示加载 */ }
    func hideLoading() { /* 隐藏加载 */ }
    func display(_ viewData: WeatherViewData) { /* 渲染数据 */ }
    func displayError(_ message: String) { /* 显示错误 */ }
}
```

### Unit Testing the Presenter

The core advantage of MVP is that Presenter can be separated from UIKit independent testing:

```swift
final class MockWeatherView: WeatherPresenterView {
    var isLoadingShown = false
    var displayedViewData: WeatherViewData?
    var displayedError: String?

    func showLoading() { isLoadingShown = true }
    func hideLoading() { isLoadingShown = false }
    func display(_ viewData: WeatherViewData) { displayedViewData = viewData }
    func displayError(_ message: String) { displayedError = message }
}

final class StubWeatherService: WeatherServiceProtocol {
    var stubbedResult: Result<Weather, Error> = .failure(WeatherError.noData)

    func fetchWeather(for city: String, completion: @escaping (Result<Weather, Error>) -> Void) {
        completion(stubbedResult)
    }
}

final class WeatherPresenterTests: XCTestCase {
    func test_success_displaysWeather() {
        let view = MockWeatherView()
        let service = StubWeatherService()
        service.stubbedResult = .success(Weather(city: "Paris", celsius: 22, condition: "sunny"))
        let presenter = WeatherPresenter(view: view, service: service)

        presenter.viewDidLoad()

        XCTAssertNotNil(view.displayedViewData)
        XCTAssertEqual(view.displayedViewData?.title, "Paris")
        XCTAssertNil(view.displayedError)
    }

    func test_failure_displaysError() {
        let view = MockWeatherView()
        let service = StubWeatherService()
        service.stubbedResult = .failure(WeatherError.noData)
        let presenter = WeatherPresenter(view: view, service: service)

        presenter.viewDidLoad()

        XCTAssertNotNil(view.displayedError)
        XCTAssertNil(view.displayedViewData)
    }
}
```

## Communication Patterns: Delegate vs Closure

When choosing how components communicate (especially in MVP and MVI), use this comparison:

| Aspect | Protocol-Delegate | Closures |
|---|---|---|
| **Coupling** | Low — communicates through a protocol contract | Slightly higher — closure captures call-site context |
| **Number of callbacks** | Scales well for many callbacks (multiple delegate methods) | Best for 1-2 callbacks; many closures clutter the API |
| **Relationship** | Single delegate at a time (1:1) | Each call site can supply a different closure |
| **Optional methods** | Via default protocol extension implementations | Via optional closure properties |
| **Memory** | `weak var delegate` prevents retain cycles | `[weak self]` in capture list required |
| **Best for** | Sustained, multi-event communication (table view, long-lived connections) | One-shot async operations (network, animations) |

### Rule of Thumb

```
Use DELEGATE when:
  - The object calls back multiple times over its lifetime
  - You need many distinct callback methods
  - Examples: UITableViewDelegate, Presenter -> View, custom services with progress

Use CLOSURES when:
  - You need a single, one-shot callback
  - The callback is tightly coupled to the call site
  - Examples: network completion handlers, animation completions, alert actions
```

## Page Communication

Cross-page (cross-ViewController) communication follows different patterns depending on the language boundary:

### Swift ↔ Swift: Combine Subject (MVI-style)

Use `PassthroughSubject` or `CurrentValueSubject` with enum associated values to communicate between Swift pages. This keeps the event flow typed, traceable, and consistent with the MVI intent pattern.

```swift
// 定义页面间通信事件
enum ProfileEvent {
    case avatarUpdated(URL)
    case nicknameChanged(String)
    case logout
}

// 发送方持有 subject，创建时传给接收方
final class EditProfileViewController: UIViewController {
    let event = PassthroughSubject<ProfileEvent, Never>()

    func didFinishEditing(newName: String) {
        event.send(.nicknameChanged(newName))
    }
}

// 接收方订阅
let editVC = EditProfileViewController()
editVC.event
    .receive(on: DispatchQueue.main)
    .sink { [weak self] event in
        switch event {
        case .nicknameChanged(let name):
            self?.updateNickname(name)
        case .avatarUpdated(let url):
            self?.updateAvatar(url)
        case .logout:
            self?.handleLogout()
        }
    }
    .store(in: &cancellables)
```

Use `CurrentValueSubject` when the receiver needs the latest value immediately upon subscription (state-like). Use `PassthroughSubject` for transient events that only matter when observed in real time.

### Swift ↔ Objective-C / Objective-C ↔ Objective-C: Closure/Block Callbacks

When crossing the Swift/ObjC boundary or communicating between two ObjC pages, use closure (block) callbacks. Combine subjects are not available in Objective-C, and block callbacks are the natural inter-page pattern for these scenarios.

```objc
// ObjC 页面定义回调 block
typedef void(^SettingsDidChangeHandler)(NSDictionary *changedSettings);

@interface SettingsViewController : UIViewController
@property (nonatomic, copy) SettingsDidChangeHandler onSettingsChanged;
@end

// 调用方（ObjC 或 Swift）创建时注入
SettingsViewController *vc = [[SettingsViewController alloc] init];
vc.onSettingsChanged = ^(NSDictionary *changed) {
    [self applySettings:changed];
};
```

```swift
// Swift 页面向 ObjC 页面传递回调
let settingsVC = SettingsViewController()
settingsVC.onSettingsChanged = { [weak self] changed in
    self?.applySettings(changed)
}
```

### Communication Pattern Selection

| Scenario | Pattern | Reason |
|---|---|---|
| Swift page → Swift page | Combine Subject + enum | 类型安全、可追溯、与 MVI 一致 |
| Swift page → ObjC page | Closure/Block | ObjC 无法消费 Combine |
| ObjC page → ObjC page | Block callback | 原生支持、简洁 |
| ObjC page → Swift page | Block callback | 跨语言最小公约数 |
| 1:N broadcast (same language) | NotificationCenter 或 Combine Subject | 多个监听者场景 |

## MVVM

Prefer MVVM when:
- the module uses SwiftUI
- state is mostly local to the feature
- async work is present but manageable
- the team wants a familiar default with clear test seams

```swift
@Observable
class TripListViewModel {
    private(set) var trips: [TripRowItem] = []
    private(set) var isLoading = false
    var searchText = ""

    var filteredTrips: [TripRowItem] {
        guard !searchText.isEmpty else { return trips }
        return trips.filter { $0.name.localizedStandardContains(searchText) }
    }

    private let repository: TripRepository

    init(repository: TripRepository) {
        self.repository = repository
    }

    func loadTrips() async {
        isLoading = true
        defer { isLoading = false }
        let models = (try? await repository.fetchAll()) ?? []
        trips = models.map { TripRowItem(from: $0) }
    }
}
```

Core rules:
- ViewModels must NOT import SwiftUI — `import Foundation` only
- One ViewModel per screen, not per view
- Dependencies injected via init, not created internally
- Use `@Observable` for iOS 17+ targets; fall back to `ObservableObject` only for iOS 16

Risks:
- view models can become dumping grounds if state, navigation, and service orchestration are not split carefully

## MVI (Page-Level)

MVI applies at the **page or feature level**. Use it when a single screen or flow benefits from explicit, traceable intent -> state transitions, but the complexity does not yet span multiple features.

Prefer MVI when:
- a screen has non-trivial state transitions (forms, wizards, multi-step flows)
- state derivation should be easy to trace and unit-test
- unidirectional flow is desirable without committing to a whole-app store

```swift
enum FeedIntent {
    case viewDidLoad
    case refresh
    case searchQueryChanged(String)
    case itemTapped(id: String)
}

struct FeedState {
    var items: [FeedItem] = []
    var isLoading = false
    var searchQuery = ""
    var error: String?
}

final class FeedViewModel {
    @Published private(set) var state = FeedState()

    private let service: FeedService
    private var cancellables = Set<AnyCancellable>()

    init(service: FeedService) {
        self.service = service
    }

    func send(_ intent: FeedIntent) {
        switch intent {
        case .viewDidLoad, .refresh:
            loadFeed()
        case .searchQueryChanged(let query):
            state.searchQuery = query
        case .itemTapped:
            break
        }
    }

    private func loadFeed() {
        state.isLoading = true
        state.error = nil
        service.fetchFeed(query: state.searchQuery)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.state.isLoading = false
                    if case .failure(let error) = completion {
                        self?.state.error = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] items in self?.state.items = items }
            )
            .store(in: &cancellables)
    }
}
```

Strengths:
- Combine publishers and Swift enum associated values make the pattern natural
- page-scoped: no global store, no cross-feature coupling
- easy to log/replay intents

Risks:
- ceremony overhead for screens that only need local state

## TCA (App-Level)

TCA applies at the **app or cross-feature level**. Use it when state and effects must be coordinated across multiple features and high testability with explicit dependency injection is a core requirement.

Prefer TCA when:
- many features share state or trigger effects in each other
- reducer-driven behavior and explicit action logging are requirements
- the team is comfortable with the TCA learning curve

```swift
import ComposableArchitecture

@Reducer
struct TripList {
    @ObservableState
    struct State: Equatable {
        var trips: IdentifiedArrayOf<Trip> = []
        var isLoading = false
    }

    enum Action {
        case onAppear
        case tripsLoaded([Trip])
        case deleteTrip(Trip.ID)
    }

    @Dependency(\.tripClient) var tripClient

    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .onAppear:
                state.isLoading = true
                return .run { send in
                    let trips = try await tripClient.fetchAll()
                    await send(.tripsLoaded(trips))
                }
            case .tripsLoaded(let trips):
                state.trips = IdentifiedArray(uniqueElements: trips)
                state.isLoading = false
                return .none
            case .deleteTrip(let id):
                state.trips.remove(id: id)
                return .run { _ in try await tripClient.delete(id) }
            }
        }
    }
}
```

Strengths:
- strong consistency for large apps
- explicit state and side-effect modeling
- exhaustive testing via TestStore

Risks:
- heavier learning curve
- too much ceremony for small apps or isolated screens

## Clean Architecture

Layers: **Domain** (entities, use cases, repository protocols) -> **Data** (repository implementations, network, persistence) -> **Presentation** (views, view models). Dependencies point inward.

Prefer Clean Architecture when:
- strict separation is required (enterprise, regulated domains)
- the domain layer must be testable without any framework dependencies
- multiple presentation targets share the same business logic

```swift
// Domain layer — pure Swift, no framework imports
protocol TripRepository: Sendable {
    func fetchAll() async throws -> [Trip]
    func save(_ trip: Trip) async throws
    func delete(id: UUID) async throws
}

struct FetchUpcomingTripsUseCase: Sendable {
    private let repository: TripRepository

    init(repository: TripRepository) {
        self.repository = repository
    }

    func execute() async throws -> [Trip] {
        try await repository.fetchAll()
            .filter { $0.startDate > .now }
            .sorted { $0.startDate < $1.startDate }
    }
}

// Data layer
struct RemoteTripRepository: TripRepository {
    private let client: APIClient

    func fetchAll() async throws -> [Trip] {
        try await client.request(.get, "/trips")
    }
}

// Presentation layer
@Observable
class UpcomingTripsViewModel {
    private(set) var trips: [Trip] = []
    private let useCase: FetchUpcomingTripsUseCase

    init(useCase: FetchUpcomingTripsUseCase) {
        self.useCase = useCase
    }

    func load() async {
        trips = (try? await useCase.execute()) ?? []
    }
}
```

Key rule: Domain layer must NOT import any framework (no UIKit, no SwiftUI, no Combine as reactive primitive unless it is a defined boundary contract).

## Coordinator Pattern

Separates navigation logic from views. Especially useful in UIKit or hybrid apps with complex navigation flows. In pure SwiftUI apps, `NavigationStack` with path-based routing often replaces the Coordinator.

```swift
@MainActor
protocol Coordinator: AnyObject {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get }
    func start()
}

@MainActor
final class TripCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    let navigationController: UINavigationController
    private let repository: TripRepository

    init(navigationController: UINavigationController, repository: TripRepository) {
        self.navigationController = navigationController
        self.repository = repository
    }

    func start() {
        let vm = TripListViewModel(repository: repository)
        vm.onSelectTrip = { [weak self] trip in
            self?.showDetail(for: trip)
        }
        let vc = TripListViewController(viewModel: vm)
        navigationController.pushViewController(vc, animated: false)
    }

    private func showDetail(for trip: Trip) {
        let detailCoordinator = TripDetailCoordinator(
            navigationController: navigationController,
            trip: trip,
            repository: repository
        )
        childCoordinators.append(detailCoordinator)
        detailCoordinator.start()
    }
}
```

Use Coordinators when:
- the app is UIKit-heavy
- navigation flows are conditional, branching, or involve deep linking
- multiple view controllers coordinate presentation and dismissal

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Massive ViewController/View | Unreadable, untestable | Extract ViewModel + services |
| ViewModel imports SwiftUI | Couples logic to UI framework | Import Foundation only |
| Singletons for everything | Hidden dependencies, hard to test | Protocol + DI |
| Feature A imports Feature B | Tight coupling, circular deps | Shared module or coordinator |
| Network calls in ViewModel | ViewModel does too much | Repository/Service layer |
| God model (one huge struct) | Hard to maintain | Split into domain entities |
| Choosing TCA for a two-screen app | Over-engineering | Start with MVVM; escalate when needed |
| Mixing patterns within a module | Inconsistent, confusing | One pattern per feature module |

## Testing Strategy

| Architecture | Unit Test Target | What to Test |
|---|---|---|
| MVVM | ViewModels | State transitions, service calls, error handling |
| MVI | ViewModel (store) | Intent -> state mapping, side effects |
| TCA | Reducers via TestStore | State changes, effects, action sequences |
| Clean | UseCases + ViewModels | Business logic in isolation, correct layer interaction |
| All | Repositories (with mocks) | Data mapping, caching logic, error propagation |

## Practical Defaults

- Objective-C module: default to MVP (Presenter owns logic, ViewController is passive).
- Swift (UIKit) module: default to MVI (Intent enum + Combine state pipeline).
- SwiftUI module with simple state: MVVM with `@Observable`.
- Large app with shared state, effects, and deep testing requirements: use TCA.
- Enterprise or multi-target shared domain: use Clean Architecture.
- Complex navigation in UIKit-heavy project: add Coordinator pattern.

## Avoid Over-Architecture

Warning signs:
- adding reducers, actions, and effect plumbing for a static screen
- introducing a global store before feature boundaries are understood
- choosing a pattern because it is fashionable rather than because ownership is difficult
- protocol-heavy Clean Architecture for a simple feature

If a feature only needs a view, a small amount of local state, one service call, and straightforward navigation, MVVM is usually enough.

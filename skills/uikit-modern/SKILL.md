---
name: uikit-modern
description: Use when building, reviewing, or modernizing UIKit screens for iOS 13 through iOS 26 — including adding new features to existing pages. Especially for collection views, diffable data sources, compositional layout, keyboard-safe layout, scenes, and modern presentation patterns.
---

# UIKit Modern

## When to Use
- Building or refactoring UIKit screens that show lists, grids, sidebars, settings, or mixed-content dashboards.
- Replacing `reloadData`, manual index-path bookkeeping, or large delegate trees with a safer state-driven collection view setup.
- Migrating from custom cells that mutate subviews directly toward registrations, content configurations, and list configurations when platform support allows.
- Fixing keyboard overlap, sheet behavior, scene lifecycle bugs, or presentation flows that rely on outdated assumptions.
- Reviewing UIKit code for modern API adoption, especially when `UICollectionView` can unify list and grid behavior.

Do not treat `UITableView` as obsolete. It remains acceptable when list behavior is simple, the code is already stable, and collection-view-only features are not needed.

## Operating Rules
- Prefer `UICollectionView` for new list, grid, and mixed-content work unless a stable `UITableView` is clearly the simpler fit.
- For multi-section lists, grids, dashboards, sidebars, and expandable content, default to `UICollectionView` with diffable data source and compositional layout.
- Prefer `UICollectionViewDiffableDataSource` and `NSDiffableDataSourceSnapshot` over manual `dataSource` bookkeeping for modern state-driven lists.
- Prefer `UICollectionView.CellRegistration`, `UICollectionView.SupplementaryRegistration`, `UIListContentConfiguration`, and `UIBackgroundConfiguration` over manual subview mutation when available. For older deployment targets or untouched legacy screens, keep fallback registration and reuse code isolated.
- Use `UICollectionLayoutListConfiguration` for standard list surfaces before building custom row chrome when the deployment target and screen requirements allow it.
- Model diffable identity explicitly. Item identifiers must be stable across updates and must not depend on display order.
- Prefer `UIControl`-based tappable surfaces or built-in collection view selection over attaching `UITapGestureRecognizer` to plain container views.
- Prefer directional constraints such as `leadingAnchor` and `trailingAnchor` over `leftAnchor` and `rightAnchor`.
- Prefer `UILayoutGuide` over spacer `UIView` instances for auxiliary layout structure.
- Use `UIKeyboardLayoutGuide` and safe-area-aware constraints when available, then fall back to keyboard notifications for older targets or legacy container hierarchies.
- Treat scene activation and view controller containment as first-class lifecycle boundaries. Do not assume a single window or a single foreground scene.
- Treat newer sheet APIs and other post-iOS-13 conveniences as preferred upgrades, not baseline assumptions. Keep fallback paths small and isolated when modern APIs cannot be adopted in one pass.

## Task Workflow
1. Classify the screen: simple stable table, modern list, mixed list/grid, or fully custom layout.
2. Route to the relevant reference files before editing implementation details.
3. Decide whether the modern path is `UICollectionView` list, custom compositional layout, or a contained legacy table view.
4. Define section and item identity before wiring data source updates.
5. Choose registrations and content configurations before writing custom cell subclasses.
6. Add keyboard, lifecycle, and presentation handling with scene-safe and containment-safe boundaries.
7. Run the modernization checklist and quick diagnostics before shipping.

## Topic Router
Use the matching reference for the active problem:

| Topic | Reference |
| --- | --- |
| Collection-view modernization strategy | `references/collection-view-modernization.md` |
| Diffable snapshots and update patterns | `references/diffable-data-source.md` |
| Item/group/section layout composition | `references/compositional-layout.md` |
| List APIs, accessories, and row configurations | `references/list-configurations.md` |
| Keyboard-safe layout and text input flow | `references/keyboard-and-input.md` |
| Scene model, lifecycle entry points, containment | `references/view-controller-lifecycle-and-scenes.md` |
| Navigation stacks, sheets, popovers | `references/navigation-and-presentation.md` |
| Symptom-driven debugging for modern UIKit | `references/diagnostics.md` |

For version-specific adoption and compatibility questions, also consult `references/ios-version-features/ios-13.md` through `references/ios-version-features/ios-26.md`.

## Modernization Checklist
- [ ] New complex list or grid work uses `UICollectionView` by default.
- [ ] Diffable identifiers are stable, unique, and separate from row position.
- [ ] Modern list state flows through diffable snapshots instead of manual index-path mutation where practical.
- [ ] Snapshot updates are targeted; no blanket `reloadData` for ordinary state changes.
- [ ] Layout is expressed with compositional sections or list configuration instead of manual size math where possible.
- [ ] Cells use registrations and content/background configurations unless a custom view hierarchy is truly required.
- [ ] Tappable surfaces use `UIControl` or built-in list selection semantics instead of gesture recognizers on passive container views.
- [ ] Auto Layout uses directional anchors and `UILayoutGuide` helpers where spacer views would otherwise be empty structure.
- [ ] Keyboard handling uses `UIKeyboardLayoutGuide` when available, with an explicit fallback for older targets if the screen edits text.
- [ ] Scene activation, foreground/background assumptions, and multi-window behavior were checked.
- [ ] Navigation choice matches intent: push for drill-down, sheet for focused tasks, popover for anchored secondary work, with pre-detent sheet fallback where needed.
- [ ] Table views kept in place only when the simpler surface is still the right tradeoff.

## Quick Diagnostics
| Symptom | Likely issue | Reference |
| --- | --- | --- |
| Animations look wrong or rows jump | Item identity is unstable or snapshots are rebuilding too much | `references/diffable-data-source.md` |
| Cell state leaks between rows | Manual mutation is not reset, or configuration boundaries are mixed | `references/list-configurations.md` |
| Grid clips or self-sizing is erratic | Layout dimensions or estimated sizing assumptions are wrong | `references/compositional-layout.md` |
| Keyboard covers controls | Constraints ignore `UIKeyboardLayoutGuide` or scroll coordination is incomplete | `references/keyboard-and-input.md` |
| Screen logic breaks after scene changes | Lifecycle work lives in the wrong entry point or ignores activation state | `references/view-controller-lifecycle-and-scenes.md` |
| Navigation stack duplicates screens or dismisses oddly | Push and sheet responsibilities are mixed or presentation context is wrong | `references/navigation-and-presentation.md` |

## Common Mistakes
- Treating `UICollectionView` modernization as a pure view-layer rewrite without first defining section and item identity.
- Rebuilding snapshots from scratch for every keystroke when a targeted item or section update would be clearer and cheaper.
- Mixing content, background, selection, and accessory state in custom cell code when configuration APIs already separate those concerns.
- Using keyboard notifications alone to move frames manually even though constraint-based keyboard guides are available.
- Assuming `viewDidLoad` or app delegate callbacks fully describe runtime lifecycle in a scene-based app.
- Replacing every `UITableView` on principle instead of only where collection-view-based modernization materially improves correctness or flexibility.
- Reading `UIScreen.main.bounds` for layout sizing when `UIView` geometry, safe area layout guides, or trait collections should drive measurements.
- Hard-coding point or pixel size constants instead of respecting Dynamic Type, content size categories, and system layout metrics.

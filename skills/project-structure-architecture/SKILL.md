---
name: project-structure-architecture
description: Use when organizing or standardizing an Apple app repository, choosing project structure or architecture, deciding package strategy, evaluating third-party libraries, or determining page communication patterns — for UIKit-first, Swift/Objective-C mixed, or SwiftUI apps. Applies to new pages and modifications to existing pages alike.
---

# Project Structure Architecture

## When to Use
- New Apple app setup or major repository reorganization.
- UIKit-first apps with Swift/Objective-C mixed codebases.
- Unclear architecture choice across MVP, MVI, MVVM, TCA, Clean Architecture, Coordinator, or modular boundaries.
- SPM vs CocoaPods decisions, package splitting, or third-party library selection.
- Choosing how Objective-C-heavy modules should stay consistent with the rest of the app.

## Operating Rules
- Prefer feature-first structure over layer-first folders once the app has multiple product flows.
- For Objective-C modules, prefer MVP (Model-View-Presenter) as the default architecture.
- For Swift (UIKit) modules, prefer MVI (Model-View-Intent) with Combine as the default architecture.
- SwiftUI is a secondary UI layer; use it for new standalone screens when deployment target allows, but do not assume it as the primary stack.
- Prefer `async`/`await` and structured concurrency for new Swift APIs; do not introduce new closure-first surfaces unless bridging older SDKs or Objective-C code.
- Prefer SPM for first-party modules and third-party dependencies; use CocoaPods deliberately and record the constraint that justifies it.
- Choose the smallest architecture that preserves state clarity, testability, and ownership boundaries.
- Prefer Apple frameworks or existing repo utilities before adding a new third-party dependency.
- Keep guidance app-first and open-source generic; do not assume proprietary backend, analytics, or internal platform layers.

## Topic Router
| Need | Reference |
| --- | --- |
| Repository layout, feature boundaries, SPM modular layout, Clean Architecture layout | `references/project-structure.md` |
| MVP vs MVI vs MVVM vs TCA vs Clean Architecture vs Coordinator pattern, page communication | `references/architecture-selection.md` |
| UIKit Coordinator, SwiftUI Router, sheets, cross-feature communication | `references/navigation-and-routing.md` |
| Naming, file organization, comments, previews, async-first conventions | `references/development-conventions.md` |
| SPM vs CocoaPods, module boundaries, package policy | `references/package-management.md` |
| Swift and Objective-C library choices by category | `references/library-selection.md` |

## Quick Decision Matrix
- Objective-C module: default to MVP (Presenter owns logic, View is passive).
- Swift (UIKit) module: default to MVI (Intent enum + Combine state pipeline).
- SwiftUI module: MVVM with `@Observable` or MVI depending on state complexity.
- Cross-feature state, effects, and high testability pressure: consider TCA (app-level).
- Enterprise or multi-target shared domain with strict layer separation: use Clean Architecture.
- Complex navigation flows in UIKit-heavy project: add Coordinator pattern.
- Large project (50k+ LOC, 3+ devs, slow incremental builds): modularize with SPM.
- New Swift dependencies: default to SPM.
- Objective-C-heavy or pod-centric legacy environment: CocoaPods can be pragmatic, but keep the reason explicit.
- New Swift service APIs: default to `async`/`await`, then add callback or Combine adapters only when required.

## Quick Review Checklist
- Is the structure feature-first for user-facing flows?
- Are SwiftUI and UIKit boundaries explicit instead of mixed ad hoc?
- Is the chosen architecture justified by actual state, navigation, and effect complexity?
- Do new Swift APIs use `async`/`await` rather than introducing fresh callback surfaces?
- Is SPM the default, with any CocoaPods use explained by a concrete constraint?
- Does each new library fill a real gap, avoid overlap, and look maintainable?

## Common Mistakes
- Keeping everything layer-first in a growing app, which hides ownership and slows feature work.
- Mixing SwiftUI views, UIKit controllers, and adapters in the same folder without boundary markers.
- Adopting TCA or Redux for simple screens that only need local state and clear navigation.
- Wrapping modern Swift work in new closure-first facades instead of exposing async APIs.
- Reaching for CocoaPods by habit instead of proving SPM is insufficient.
- Adding broad third-party libraries before exhausting Apple APIs or the repo's existing utilities.

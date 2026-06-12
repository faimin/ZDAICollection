---
name: swift-modern
description: Use when writing, reviewing, or migrating Swift application code across Swift 4.2 through 6.3 — including adding features to existing Swift pages. Especially for concurrency, Combine, Codable, observation, Swift Testing, interoperability, and version-sensitive language decisions.
---

# Swift Modern

## When to Use
- Writing or reviewing modern Swift application code for Apple platforms.
- Migrating legacy Swift code from Swift 4.2 or early Swift 5 releases to current language and library patterns.
- Choosing between structured concurrency, Combine, Codable, `@Observable`, and Objective-C interoperability techniques.
- Writing or reviewing tests with the modern Swift Testing framework.
- Investigating compiler diagnostics related to `Sendable`, actor isolation, decoding failures, or publisher threading.
- Answering version-sensitive Swift questions where availability or behavior changed across releases.

## Operating Rules
- Prefer modern Swift over legacy escape-hatch patterns when platform support allows it.
- Route version-sensitive questions to the exact `references/whats-new-swift/` file before answering from memory.
- Prefer namespaced extension surfaces, similar to `kf`, when extending foreign or widely shared types.
- Prefer structured concurrency over detached work unless isolation boundaries require otherwise.
- Prefer structured concurrency over GCD queues, locks, and ad hoc thread management unless a lower-level primitive is required.
- Prefer Combine over KVO for new reactive observation in app code when Objective-C runtime observation is not required.
- Treat `Sendable`, actor isolation, and cancellation as correctness constraints, not style advice.
- Prefer explicit ownership and typed models over `Any`, mutable globals, or stringly typed state.
- Wrap multi-property object initialization in an immediately-invoked closure `{ ... }()` to co-locate creation and configuration.
- Mark classes `final` when they are not designed for inheritance.
- Split large types into focused `extension` blocks by responsibility instead of keeping every method in one body.
- Prefer targeted `@objc` annotations over `@objcMembers` unless nearly the entire type must be Objective-C-visible.
- Avoid `dynamic` unless Objective-C runtime dispatch is explicitly required.
- For modern Swift tests, prefer Swift Testing conventions, parameterized coverage, and async-aware assertions when the toolchain supports them.
- Use the core references for stable patterns and the `whats-new-swift` notes for release-specific behavior.

## Topic Router
- Language evolution, migration order, and release-era priorities: `references/language-evolution-overview.md`
- `async`/`await`, task groups, cancellation, actors, and isolation rules: `references/concurrency.md`
- Combine lifecycle, schedulers, ownership, and async bridging: `references/combine.md`
- `Codable` modeling, strategies, resilience, and mixed payloads: `references/codable.md`
- `@Observable`, state ownership, and boundaries with tasks or pipelines: `references/observation-and-state.md`
- Swift and Objective-C boundaries, nullability, selectors, and callback bridging: `references/interoperability.md`
- Swift Testing suites, expectations, traits, parameterization, and async test patterns: `references/swift-testing.md`
- Common diagnostics for concurrency, decoding, and publisher mistakes: `references/diagnostics.md`

## Version Matrix
- Swift 4.2 to 5.0 migration baseline: open `references/whats-new-swift/swift-4.2.md`, then `references/language-evolution-overview.md`.
- Swift 5.0 ABI stability, `Result`, and string-era modernization: open `references/whats-new-swift/swift-5.0.md` first.
- Swift 5.1 to 5.4 modernization work such as property wrappers, result builders, and language polish: open the matching `swift-5.x.md` file first.
- Swift 5.5 to 5.10 adoption work for concurrency, `any`, existential cleanup, and modern libraries: open the matching `swift-5.x.md` file first.
- Swift 6.0 strictness and current concurrency rules: open the matching `swift-6.x.md` file first.
- If the target release is unknown, use `references/language-evolution-overview.md` to find the likely era, then route to the exact version note before final guidance.

## Correctness Checklist
- Confirm the Swift toolchain version, language mode, and deployment target before suggesting syntax or migration steps.
- Check whether the design should use value types, actors, or main-actor isolation instead of shared mutable classes.
- Verify that values crossing task or actor boundaries are actually `Sendable`, or isolate them so they do not need to cross.
- Preserve cancellation through task hierarchies, continuations, and callback bridges.
- For Combine code, verify scheduler expectations, subscription lifetime, and capture ownership.
- For `Codable`, verify key mapping, date or data strategies, optionality, `String` or `Int` compatibility for unstable fields, and failure behavior for malformed input.
- For observation and UI state, verify who owns the state and which isolation domain mutates it.
- For Objective-C interop, verify nullability, selector exposure, execution context, and bridging semantics.
- For modern Swift tests, verify that expectations, parameterization, async flow, and actor access match Swift Testing conventions.
- Bounds-check collection access unless the calling context proves the index is valid.

## Quick Diagnostics
- `Sending 'x' risks causing data races`: check `references/concurrency.md` and `references/diagnostics.md`.
- UI state updates from background work or mixed isolation: check `references/concurrency.md` and `references/observation-and-state.md`.
- Combine output arrives on the wrong thread or stops early: check `references/combine.md` and `references/diagnostics.md`.
- ViewModel or object using `assign(to:on:)` never deallocates: check `references/combine.md` "assign(to:on:) vs assign(to: &$property)" section.
- `DecodingError.keyNotFound`, `typeMismatch`, or inconsistent payloads: check `references/codable.md` and `references/diagnostics.md`.
- Unsure whether to use `@objc`, `dynamic`, or a Swift wrapper: check `references/interoperability.md`.
- Unsure whether a new Swift test should use `@Test`, `#expect`, parameterization, or async actor-aware patterns: check `references/swift-testing.md`.
- Unsure whether a feature exists in a particular Swift release: open the exact `references/whats-new-swift/` file before answering.

## Common Mistakes
- Using `DispatchQueue.main.async` as a default fix when `@MainActor` or actor-aware structure should express the boundary.
- Reaching for `Task.detached` to avoid designing ownership or isolation correctly.
- Adding `@unchecked Sendable` without a concrete thread-safety argument.
- Keeping new state models on `ObservableObject` and `@Published` without checking whether `@Observable` is the better fit.
- Decoding complex payloads into `[String: Any]` before modeling a resilient `Codable` enum or wrapper.
- Bridging Objective-C callbacks into async code without single-resume guarantees, cancellation handling, or queue expectations.
- Marking whole types `@objcMembers` when only a few selectors must cross the Objective-C boundary.
- Using `assign(to:on: self)` + `store(in: &cancellables)` when `assign(to: &$property)` avoids the retain cycle entirely.
- Leaving non-polymorphic classes open and indexing directly into collections without making the safety contract explicit.
- Wrapping view expressions in `AnyView` when `@ViewBuilder` or a concrete return type avoids the type erasure.
- Using `GeometryReader` as a first resort instead of trying `containerRelativeFrame` or layout-aware modifiers first.
- Defining reusable view pieces as computed `var` properties on `View` instead of dedicated `View` structs.
- Using the single-argument `onChange(of:perform:)` form instead of the two-parameter closure variant that receives old and new values.


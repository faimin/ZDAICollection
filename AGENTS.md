# Repository AGENTS

## 1. Scope

This file defines repository-local operating rules.

Reusable Apple and iOS engineering guidance lives in `skills/` and should not be duplicated here unless the rule is genuinely repository-specific.

## 2. Repository Workflow

- When modifying code, only make changes directly related to the request. Do not auto-format, restyle, or refactor surrounding code that was not asked to be changed.
- All new or modified code must include Chinese comments explaining the implementation approach and key logic so other team members can maintain the code. This applies to every code change without exception.
- Prefer checking the existing repository structure and active patterns before editing.
- For UI-related work, provide a concrete plan first. Use ASCII sketches when a layout or flow benefits from visual structure.
- Use `Git worktree` when isolation is helpful for multi-step feature work.
- Use commit messages in the format `<type>(<scope>): <subject>`.
- Before handing off significant changes, summarize what changed, how it was verified, and any remaining risk.

## 3. Project-Specific Constraints

- Prefer existing project patterns over introducing a new local convention in one file.
- Keep build, resource, and module ownership explicit; do not assume everything belongs in a global bucket by default.
- Network-facing code must surface failure paths explicitly rather than silently ignoring errors.
- New business logic should include unit-test coverage when the repository has an established place for that coverage.
- Critical user flows should keep or gain UI-test coverage when the repository already exercises that path.
- If the task depends on product design, prefer the existing Figma or design-source direction and do not hardcode ad hoc visual values when a design token or existing asset should be used.
- When using CocoaPods with KuaiLiao spec sources, use `https://github.com/KuaiLiao/KLSpecs.git` as the pod source.
- Page communication: Swift↔Swift uses Combine `PassthroughSubject`/`CurrentValueSubject` with enum associated values (MVI-style event bus); Swift↔ObjC or ObjC↔ObjC uses closure/block callbacks.

## 4. Security And Safety

- Do not hardcode secrets, passwords, tokens, or private service endpoints.
- When a feature touches camera, microphone, photo library, location, or similar sensitive capabilities, verify the required privacy descriptions exist.
- Before adding a new third-party dependency, verify that its license is acceptable for the project.
- Avoid destructive repository actions unless they are explicitly requested.
- Follow App Store Review Guidelines when implementing features that interact with payments, user data, or platform capabilities.

## 5. Skill Routing

This project is UIKit-first with Swift/Objective-C mixed codebases and SwiftUI used as a secondary UI layer.

Route by the dominant problem shape:

| Problem shape | Skill |
| --- | --- |
| UIKit screens, collection views, diffable data source, compositional layout, keyboard-safe layout, navigation, scenes, presentation | `uikit-modern` |
| Objective-C ownership, blocks, runtime behavior, KVC/KVO, swizzling, forwarding, mixed-language legacy concerns | `objective-c-patterns` |
| Swift language behavior, concurrency, Combine, Codable, observation, Swift Testing, SwiftUI screens, version-sensitive decisions | `swift-modern` |
| Project structure, architecture selection, package strategy, dependency selection, modular boundaries | `project-structure-architecture` |

Triage rules:

1. Skills apply to ALL code changes in the relevant domain — not only new pages/screens, but also adding features, modifying logic, or fixing bugs in existing pages.
2. Identify the dominant layer: UIKit UI, Objective-C behavior, Swift language/SwiftUI UI, or project architecture.
3. If multiple layers are involved, name the dominant one before loading a second skill.
4. Read the matching specialist skill before answering from memory.
5. Load a second specialist skill only when the task genuinely spans two domains.
6. Keep one specialist skill as the primary owner so guidance stays coherent.
7. Keep reusable language, UIKit, Objective-C, architecture, and library-selection guidance inside the skill system rather than expanding this file.

## 6. Platform Capabilities Quick Reference

Key APIs introduced per iOS version. Use this to determine what is available at the project's deployment target.

| iOS | Key additions |
| --- | --- |
| 17 | `@Observable` macro (replaces `ObservableObject`), `@Bindable`, SwiftData, TipKit |
| 18 | SwiftData `#Index` and `#Unique`, Control Center widgets, `@Previewable` preview macro |
| 26 | Liquid Glass design system, new translucent materials, Modern Tab API |

For full per-release details, see `skills/swift-modern/references/whats-new-swift/`.

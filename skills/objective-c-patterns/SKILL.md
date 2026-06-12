---
name: objective-c-patterns
description: Use when writing or reviewing Objective-C — including adding features to existing ObjC pages — debugging retain cycles, reasoning about blocks or ARC, using runtime features, or bridging Objective-C with Swift.
---

# Objective-C Patterns

## When to Use

Use this skill when the task involves Objective-C language behavior rather than app-specific product logic:

- Writing or reviewing Objective-C classes, categories, or protocols
- Debugging ownership bugs, leaks, or `EXC_BAD_ACCESS`
- Choosing between delegates, blocks, target-action, KVO, or runtime hooks
- Evaluating swizzling, forwarding, associated objects, or other dynamic behavior
- Designing Objective-C and Swift boundaries for mixed-language codebases

Start with the router, then open only the references needed for the current question.

## Operating Rules

- Prefer simple Objective-C language features before reaching for runtime mutation.
- Nil-check optional blocks before invocation, for example `if (completion) { completion(...); }`.
- Treat retain cycles as ownership design bugs, not cleanup chores.
- Prefer inheritance, composition, or explicit integration points before any hook or swizzle.
- Use swizzling only when a direct override, wrapper, delegate, or explicit integration point is unavailable.
- Prefix category APIs with a project-specific abbreviation (e.g. `xx_`), keep categories lightweight, and move complex stateful logic into real classes.
- Prefix private methods with `_`.
- In getter methods, build the object into a local variable and assign `_ivar` as the final statement; see `references/coding-conventions.md`.
- Wrap multi-property object initialization in a `({ })` statement expression to group creation and configuration; see `references/coding-conventions.md`.
- Check `respondsToSelector:` before invoking any optional delegate method; see `references/coding-conventions.md`.
- Prefer `RACObserve` over raw KVO when the project depends on `ReactiveObjC`; see `references/kvc-kvo-and-associations.md`.
- Group methods with `#pragma mark -` and split very large implementations into focused categories when it improves readability.
- Place `- (void)dealloc` near the top of `@implementation` so teardown stays visible during reviews.
- Guard uncertain collection indexing before subscripting, and prefer `firstObject` or `lastObject` for boundary access.
- Use early returns for nil, invalid state, or failed preconditions instead of nesting conditionals.
- Route bridging questions to the interop reference before proposing mixed-language APIs.
- Keep dynamic behavior local and easy to remove. If a pattern cannot be explained in a few sentences, it is probably the wrong default.
- For crash or leak triage, confirm the symptom first, then narrow to ownership, threading, or runtime mutation before editing code.

## Topic Router

Consult the matching reference before giving implementation advice:

| Topic | Reference |
| --- | --- |
| Blocks, captures, ARC ownership | `references/blocks-and-memory.md` |
| Runtime APIs and swizzling | `references/runtime-and-swizzling.md` |
| Forwarding and dynamic dispatch | `references/message-forwarding.md` |
| KVC, KVO, RAC observation | `references/kvc-kvo-and-associations.md` |
| Coding conventions, getter patterns, delegate guards | `references/coding-conventions.md` |
| Queues, locks, legacy async work | `references/threading-and-gcd.md` |
| Swift bridging boundaries | `references/objc-swift-interop.md` |
| Crash and leak triage | `references/diagnostics.md` |

## Memory Checklist

Use this checklist when ownership or lifecycle is unclear:

- Identify who owns the object now, and who should own it after the change.
- Check whether the relationship is one-shot, repeated, or long-lived.
- Keep delegates `weak` unless the platform contract says otherwise.
- Treat stored blocks as owners of the objects they capture.
- Break cycles by redesigning the relationship first, not by sprinkling `nil` assignments later.
- Verify notification, KVO, timer, display link, and dispatch source teardown paths.
- For bridged Swift APIs, confirm nullability and collection element types are explicit.

## Retain-Cycle Workflow

1. Find the strong ownership loop.
2. Name the expected owner for each node in the loop.
3. Decide which edge should become non-owning or shorter-lived.
4. Replace the pattern with a better boundary:
   - delegate for long-lived callbacks
   - block for one-shot completion
   - injected coordinator or service for shared orchestration
5. Use `weak` capture only where the ownership model truly requires it.
6. Re-check deallocation with a breakpoint, Instruments, or a temporary `dealloc` log.

Typical loop shapes:

- controller -> view model -> completion block -> controller
- owner -> timer or display link -> target -> owner
- object A -> object B -> delegate-like property -> object A
- object -> associated object -> block -> object

## Quick Diagnostics

| Symptom | Likely cause | Reference |
| --- | --- | --- |
| Object never deallocates | Block capture or delegate ownership bug | `references/blocks-and-memory.md` |
| `EXC_BAD_ACCESS` after async callback | Object deallocated earlier than expected | `references/diagnostics.md` |
| Strange behavior after adding category hooks | Swizzle collision or wrong method exchange timing | `references/runtime-and-swizzling.md` |
| Message sent to wrong helper object | Forwarding chain too broad or incomplete | `references/message-forwarding.md` |
| Observer crash on property change | KVO lifecycle mismatch | `references/kvc-kvo-and-associations.md` |
| UI state updates from background queue | Missing main-thread hop | `references/threading-and-gcd.md` |
| Swift sees awkward Objective-C API | Missing nullability or poor naming surface | `references/objc-swift-interop.md` |

## Common Mistakes

- Using associated objects to avoid creating a real subclass or stored property owner.
- Swizzling framework methods without collision guards, original implementation checks, or a rollback plan.
- Assuming `[weak self]` is always correct. Weak capture is a tool, not the ownership model.
- Using blocks for long-lived relationships that should be delegates or explicit collaborators.
- Invoking optional blocks without checking whether they are non-nil.
- Hiding complex behavior in broad categories instead of a subclass or helper object.
- Using raw subscripting on uncertain arrays or sets when `firstObject`, `lastObject`, or bounds checks would make the contract explicit.
- Forwarding selectors broadly enough to hide missing APIs or typos.
- Mixing KVO, manual ivar mutation, and direct setter bypasses without understanding notification behavior.
- Exposing Swift generics, structs, or async-only APIs directly to Objective-C callers that cannot consume them safely.
- Treating intermittent crashes as "ARC bugs" before checking lifetime, queueing, and swizzle interactions.

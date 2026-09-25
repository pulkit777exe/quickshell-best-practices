---
name: quickshell-best-practices
description: Create, change, debug, review, or verify portable Quickshell QML configurations. Use for shell.qml, Quickshell imports, windows, screens, services, IPC, configuration, processes, plugins, reloads, performance, and runtime errors; route generic QtQuick work to Qt/QML guidance. Check the version-matched Quickshell documentation before relying on an API.
---

# Quickshell best practices

## Rule

Keep a portable core and put unavoidable differences behind small adapters. The core may depend on Qt Quick and documented Quickshell types; compositor, desktop-service, distribution, and hardware integrations are optional capabilities. Portability means no undeclared assumptions, not that every Quickshell build supports every backend.

Before relying on any type, property, signal, import, or command, check the Quickshell documentation for the installed version. Do not infer an API from another shell or from memory.

## Work in this order

1. **Inspect.** Find `shell.qml`, the launch path, imports, configuration, entry points, and tests. Record the Quickshell and Qt versions and runtime. If no launch command exists, inspect `qs --help` and use the path form supported by the installed version. Done when affected surfaces and assumptions are named.
2. **Classify.** For a bug, capture the failing transition and expected result. For performance work, record a baseline. For a review or question, stay read-only unless a change is requested. Done when the branch has one observable target.
3. **Own.** Assign every window, model, process, timer, socket, watcher, and state object to one owner with a destruction or cancellation path. Done when no resource has two owners.
4. **Implement.** Keep state and integration code out of visual leaves. Add the smallest complete UI and data path. Done when relevant optional integrations degrade without taking down the shell.
5. **Self-review and repeat.** Re-read changed files in three passes: semantics (bindings, scopes, models), edges (failure, security, capabilities), and scope (diff, ownership, unrelated churn). List unmet or unverified claims, make the smallest correction, and rerun affected checks. Repeat until no material gap remains. Done when every critical changed branch is owned and verified.
6. **Verify.** Define a configuration-specific readiness signal, run repository checks, then exercise failure, reload, input, and multi-screen paths. Done when each claim has evidence or a named limitation.

Load `reference/qml.md` only for QML structure, delegates, imports, or loading. Load `reference/edges.md` only for processes, services, configuration, IPC, plugins, or platform boundaries. Always load `reference/verification.md` before completion; do not load unrelated references.

## Core rules

- **Ownership.** Use one root per configuration. Use `ShellRoot` for configuration lifecycle, `Scope` for composition and reload scope, and `Singleton` only for shared state. `Variants` is reload-aware. Keep a configuration in one Quickshell process.
- **Inputs.** Expose component inputs as typed `required` properties and derived values as `readonly`. Keep IDs file-local. Pass delegate data through `modelData` or explicit properties; shared code must not reach into a child instance.
- **Reactivity.** Bind derived state. Assign only for intentional state transitions; assignment removes a binding. Keep commands, process writes, and other side effects in handlers or functions. Replace nested `var` objects or use a model when observers must see a mutation.
- **State.** Represent asynchronous data as loading, empty, ready, stale, or failed when applicable. A missing value and a failed read are different results. Make pure conversion and validation functions side-effect free.
- **Reload.** For each long-lived owner, state whether a live reload reuses or recreates it. Stop timers, terminate owned processes, close sockets and watchers, and verify signal cleanup. Persist only intentionally reloadable state; give critical services and authenticated flows an explicit recovery policy.
- **Imports.** Use the import form documented for the installed version. A QML module version such as `import Module Major.Minor` is separate from the Quickshell package version; add one only when the installed module declares it. On versions that support `qs.<path>`, prefer it over legacy `root:/` imports. Keep optional modules out of the portable core.
- **Screens.** Treat `Quickshell.screens` as reactive. Use `Variants` for windows and other non-`Item` objects, `Repeater` for small visual collections, and a real model with `ListView` for large or unbounded lists. Handle zero, one, and many screens; never retain a disconnected screen object.
- **Loading.** Use `Loader` for an `Item` when synchronous creation is acceptable. Use `LazyLoader` for deferred or asynchronous `Item` and non-`Item` trees. Check the installed version's `active`, `activeAsync`, `loading`, and `item` semantics: `active: true` forces a foreground load; set `loading: true` or assign `activeAsync: true` once, then wait for the readiness signal; reading `activeAsync` or `item` before ready can force or block a load. `visible: false` does not unload an object.
- **Windows.** Configure anchors, exclusion, focus, transparency, and input masks deliberately. If a window may later become transparent, set `surfaceFormat.opaque: false` in its initial definition. Anchor popups to their source and available screen bounds.
- **Edges.** Prefer a documented Quickshell service. Put `Process`, `Socket`, D-Bus, native, and backend calls behind a normalized adapter with explicit capabilities. A missing QML module is a load-time failure; keep optional imports in separately loaded files.
- **Processes.** Use one process owner per logical source. Runs may overlap across separate owners, never within one owner; cap total concurrency and queue or coalesce excess requests. Pass argv separately. Use `StdioCollector` for finite output and `SplitParser` for delimited streams. For finite output, enforce a hard byte cap and reject or terminate above it; for streams, cap frame size and bound retention. Handle stderr, malformed or partial data, timeouts, exits, bounded retries, stale results, and cancellation. Use `sh -c` only for deliberate shell code.
- **Configuration and IPC.** Validate a versioned configuration schema, keep defaults and user data separate, use safe fallback and last-known-good state, write atomically, and watch only for live reload. Use stable typed IPC targets, validate arguments, return useful status, and keep arbitrary command execution out of the interface.
- **Security.** Treat same-process plugins as trusted code; a capability facade is not a sandbox. Escape rich text, constrain paths, allowlist user-controlled or privileged executables, avoid interpolated shell input, and never log secrets or full sensitive payloads.
- **Performance.** Measure on target hardware. Prefer events to polling, one shared producer to repeated work, bounded timers to permanent polling, and lazy loading to large idle trees. Avoid hot-path allocation, parsing, deep copies, and layout work. Treat renderer and effect pragmas as capability-specific. Do not promise resource numbers.

## Definition of done

- The exact configuration reaches a defined readiness signal and remains alive with no unexplained runtime warnings. A launcher return, spawned process, or quiet log is not readiness. If the environment prevents launch, record the blocker and do not mark runtime verification complete.
- Clean start, missing dependency, invalid configuration, process failure, stream disconnect, reload, and adapter recovery have been exercised where relevant.
- Zero, one, and multiple screens; hotplug; scale or rotation; popup bounds; keyboard and pointer input have been exercised where relevant.
- The portable core contains no undeclared compositor command, desktop-specific path, or distribution package assumption.
- IPC input, file paths, shell commands, rich text, plugin boundaries, and sensitive logging have been reviewed.
- Tests cover changed behavior; the final diff has no unrelated churn, stale callbacks, unowned resources, or silent fallback. Untested conditions are recorded plainly.

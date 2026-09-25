# Verification reference

A review is complete when the evidence matches the claim.

## Static checks

- Run the repository's formatter, linter, and tests.
- If available, run the QML language server or `qmllint` with the project's import paths.
- Test pure JavaScript or model helpers separately: normalization, validation, state transitions, command construction, and escaping.
- Launch the exact configuration through its documented command. Define a configuration-specific readiness signal before launch—an IPC health method, a test marker, or an expected window state—then require the process to remain alive through it. Check the startup log for warnings, missing types, invalid bindings, and repeated reloads.
- Confirm the installed Quickshell version and read its documentation for every newly used API. If static tooling cannot resolve a Quickshell type, record that limitation and verify it with a runtime launch.

## Runtime matrix

Exercise the changed paths, not the whole product by default:

| Area | Check |
| --- | --- |
| Startup | Clean config, missing config, invalid config, missing optional dependency, readiness signal, process liveness |
| State | Empty data, malformed data, repeated triggers, rapid updates, reload, stale completion |
| Reload | Live reload, failed reload, reused or destroyed resources, signal cleanup |
| Processes | Success, non-zero exit, stderr, timeout, stream disconnect, concurrency cap, overflow, restart |
| Output | Below, at, and above byte/frame caps on stdout and stderr; chosen overflow behavior |
| Adapters | Missing module, transient unavailability, reconnect, hotplug |
| Loaders | `active` versus deferred loading, reading `activeAsync` before ready, `item` while loading, synchronous live reload |
| Screens | Zero, one, many, hotplug, removal, scale, rotation, changed geometry |
| Windows | Opaque initial surface, later transparency, anchors, focus, mask, popup edges, hide and show |
| Input | Pointer, keyboard, focus transfer, empty targets, repeated activation |
| IPC | Valid call, invalid argument, unavailable capability, duplicate call |
| Resources | Image failure, large data, cache growth, lazy load and unload |
| Security | Executable allowlist, untrusted text, option-shaped arguments, path traversal, symlinks, rich-text injection, oversized output, plugin capability violations, sensitive logs |

Use an isolated fixture with a temporary config and data directory when possible. Exercise missing binaries, malformed output, hangs, repeated triggers, concurrency overflow, stale completions, output below/at/above cap on both streams, reading `activeAsync` before ready, loader access during loading, synchronous reload, and an opaque-to-transparent transition without waiting on real hardware.

## Portability review

Search imports, environment variables, executable names, absolute paths, and compositor-specific properties. Each hit must be either:

1. inside an optional adapter;
2. a documented capability probe; or
3. a deliberate platform-specific mode.

Run the core with the adapter disabled. The shell should start, show a truthful unavailable state, and avoid repeated failed probes.

## Performance review

Measure idle and active behavior on the target hardware. Record process count, memory, CPU, GPU activity, frame or interaction latency, and reload time. Compare against the pre-change baseline when possible. Investigate new timers, per-screen instances, unbounded lists, repeated parsing, and hidden objects before accepting a result.

## Completion record

Record the Quickshell and Qt versions, QML module versions, platform, exact launch command, readiness signal, import paths, version-matched API assumptions, commands and checks run, expected versus observed results, and conditions not covered. Do not turn an unrun check into a pass.

# Edge reference

## Adapter shape

Keep an edge small and explicit:

- `capabilities`: which operations are available now.
- `state`: normalized values for the UI, with loading, empty, ready, stale, and failed states where relevant.
- `subscribe`: a live or polled update path.
- `command`: validated operations with a result status.

A missing runtime capability is a state, not an exception. Probe at startup and on relevant availability events, or use bounded retries and a cache timeout; cache the result and expose `unavailable` or `retrying`. A missing QML module is a load-time failure, not an unavailable capability: keep optional imports in separately loaded adapter files. A backend process is justified when a native API is needed; give it a versioned, narrow protocol rather than letting QML depend on its implementation.

Each adapter should expose an availability state, a redacted error category, a last-attempt or last-success marker when useful, and a retrying state. Record state transitions and process or reload outcomes without logging payloads or secrets.

## Processes

`Process.command` is an argument list, not a shell command. Keep each argument separate. Use a shell only when the operation is intentionally shell syntax, and treat the complete string as code.

Choose the parser by data shape:

- `StdioCollector`: one finite stdout result; parse after the stream finishes when the response is atomic.
- `SplitParser`: one update per newline or another delimiter for incremental data.
- An application or protocol size limit, plus a timeout: external commands that may hang.

`StdioCollector` accumulates output; it is not a memory bound. For finite stdout or stderr, enforce a hard byte cap and reject or terminate above it. For framed streams, cap each frame and bound retained data; do not describe bounded retention as a total-output cap.

Every process path needs:
- one owner and a global concurrency cap;
- no overlapping runs within that owner; overlap across owners is allowed only up to the cap;
- an exit handler and stderr path;
- stale-result protection: attach a request generation or identifier and ignore older completions;
- a bounded reconnect or retry policy; and
- a failure state.

Queue or coalesce excess requests when the cap is reached. Make the overflow policy explicit.

Treat initial loaders and watchers as possibly concurrent; make initialization idempotent. Use detached processes only when the child must outlive Quickshell.

For commands that accept options, place `--` before operands and test option-shaped paths and names. Pass dynamic values as positional arguments to a fixed script rather than interpolating them into shell source.

## Configuration

Keep built-in defaults in code and user data in a versioned file or adapter. Validate types and ranges at the boundary. On a missing or malformed file, use safe defaults and report the reason. On a transient read failure, keep the last known good value and mark it stale.

Use atomic writes, debounce bursts, and handle save failure. Watch the file only when live reload is part of the contract; do not create a watcher for data that changes once at startup. Reserve blocking reads for configuration needed before the first window; use the documented asynchronous file and loader paths elsewhere.

Choose merge semantics deliberately. A user document may replace defaults, inherit sparse overrides, or merge by section. Make that choice explicit and test missing, partial, and unknown keys. Keep one configuration owner and pass detached snapshots to views and plugins.

Never use a user-facing text command, path, or JSON field as executable code without validation. Keep generated files separate from user-owned files and make ownership explicit.

## IPC and plugins

Use stable target names and typed operations. Follow the installed handler's supported argument types and arity. Give every mutating operation a clear success, failure, and unavailable result. Validate identifiers, bounds, and state preconditions.

Use the platform's authenticated service for privileged work; never pass credentials through a command line or a broadly readable IPC property. Catch plugin callback failures so one extension cannot terminate the host. Keep authentication and other sensitive services out of broad plugin-facing maps. A capability-shaped object reduces accidental access; same-process QML plugins are still trusted code, not an isolation boundary.

For rich text, escape data before adding markup. For paths, resolve them, constrain them to the intended directory, and reject traversal. Use the documented Quickshell data, cache, and state path helpers for generated files. For logs, record the component, operation, stage, and safe error category rather than secrets or full payloads.

## Portability boundary

The portable core may use Qt Quick, documented Quickshell types, normalized state, and capability checks. Compositor modules, desktop services, distribution commands, and hardware probes belong in optional adapters. Keep optional imports in files loaded only after a capability check; a missing module must not prevent the core from parsing. Do not make a feature silently disappear because a module or executable is missing; make its unavailable state visible and keep the shell running.

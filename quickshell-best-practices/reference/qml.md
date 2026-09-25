# QML reference

## Ownership

Keep the root small enough to read. Put long-lived data sources beside the root or in a named `Scope`/`Singleton`, then pass their state into visual components. `Scope` propagates a reload scope; `Variants` is reload-aware. A `Singleton` needs `pragma Singleton` and a `Singleton` root. A repeated component should receive values, not reach into another instance's object tree.

```qml
import Quickshell
import QtQuick

ShellRoot {
    id: root

    readonly property string clockText:
        Qt.formatDateTime(clock.date, "HH:mm")

    SystemClock {
        id: clock
        precision: SystemClock.Minutes
    }

    Variants {
        model: Quickshell.screens

        PanelWindow {
            required property var modelData
            screen: modelData

            anchors {
                top: true
                left: true
                right: true
            }

            implicitHeight: 32

            Text {
                anchors.centerIn: parent
                text: root.clockText
            }
        }
    }
}
```

The clock exists once. Each screen window binds to its value. Use a more specific property type instead of `var` when the installed Quickshell version exposes a suitable type.

## Reactivity

- Bind a property when it is a function of other state.
- Assign a property when an event changes state. Do not mix the two accidentally.
- Use `readonly` for values that callers should not mutate.
- Replace an object when its nested fields change, or expose a model with change notifications. A plain nested `var` object has no useful field-level notification.
- Keep conversion and formatting functions pure. Return a new value; do not quietly change another object.
- Use `Connections` for signals that cannot be expressed as a binding. Prefer a typed readonly property over a mutable `var` bridge.
- Use `Qt.binding` only when a binding must be created at runtime; an ordinary assignment removes an existing binding.
- On reload, keep only state that has an explicit persistence owner. Stop old producers before replacing them; do not assume an object survives because it was a singleton.

## Collections and loading

| Need | Use |
| --- | --- |
| Visual items | `Repeater` |
| Long or unbounded visual lists | `ListView` with a real model |
| Windows or other non-`Item` objects | `Variants` |
| Conditional `Item` with synchronous creation acceptable | `Loader` |
| Deferred or asynchronous `Item` or non-`Item` tree | `LazyLoader` |
| Shared, process-wide state | `Singleton` |

Do not place a `Process`, `Timer`, or expensive model in a screen or list delegate when several delegates can share it. Let a `ListView` recycle visible items instead of creating the entire data set as objects. `visible: false` hides an object; it does not release it. `LazyLoader` can release a subtree, but `active: true` forces a foreground load and reading `item` while loading can block the interface. Set `loading: true` or assign `activeAsync: true` once, then wait for `activeChanged`; reading `activeAsync` before completion behaves like `active` and can force a foreground load. During a live reload, lazy loaders may load synchronously so their windows can be reused; account for that lifecycle in tests.

## Geometry and input

Use layouts for content, anchors for window relationships, and implicit sizes for preferred dimensions. Keep a fallback for null geometry. Use logical pixels, then read `devicePixelRatio` only where physical-pixel work is required. Format dates, numbers, and text through the active locale and translation catalog.

Treat a transparent window as a deliberate surface choice. If it may later become transparent, set `surfaceFormat.opaque: false` in the initial window definition; do not rely on changing the format after creation. Define focus, keyboard exclusivity, click masks, margins, and exclusion zones as part of the window contract, not as incidental styling. Anchor popups to their source window and available screen bounds; do not place them at fixed global coordinates.

A `delegate` may be created zero or many times. Declare `required property var modelData` (or a more specific type), and use that value for identity. Prefer named model roles over index arithmetic when the model has stable fields. Do not use a delegate-local ID as shared state.

## Imports

Use the import form documented for the installed Quickshell version. A QML module version such as `import Module Major.Minor` is independent of the Quickshell package version; add it only when the installed module declares that version. On versions that support it, prefer `qs.<path>` for shell modules because it is clearer to tooling than legacy root imports. Keep optional modules in separately loaded files.

The example above leaves module versions implicit. Match the versions required by the installed modules rather than copying a version from an unrelated project.

## Appearance

Keep colors, spacing, radii, and type sizes in semantic tokens. Bind controls to the same interaction states for pointer, focus, and selection. This keeps a shell coherent when its scale or theme changes.

## Files and resources

Use `Qt.resolvedUrl()` for paths relative to a QML file. Use `Quickshell.env()`, Qt standard paths, or the documented cache/data/state helpers for runtime paths. Do not assume a particular home directory or configuration root. For images, set a sensible `sourceSize` when the display size is known and provide a fallback for decode failure.

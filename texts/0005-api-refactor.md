- Feature Name: v2_api_refactor
- Start Date: 2023-03-24
- RFC PR: [tauri-apps/rfcs#0005](https://github.com/tauri-apps/rfcs/pull/10)

# Summary

With the plugin interface being capable of integrating with mobile-specific functionalities (see [RFC #0005](https://github.com/tauri-apps/rfcs/pull/9)), it makes sense for each existing API module to be rewritten as a plugin in order to use this new interface to implement current desktop API for Android and iOS. For instance, APIs like dialog, which uses [`rfd`](https://github.com/PolyMeilex/rfd), a desktop-only crate, can use the new mobile plugin structure to implement file and message dialogs for mobile.

# Motivation

The main motivation is actually implementing the existing APIs for mobile targets. This gets complicated with the current codebase, which already includes a lot of conditional compilation checks for both allowlist and platform detection. Separating each API module as its own plugin enhances maintainability as each package has a limited scope.

Additionally, this change simplifies future audits since the scope of changes are much more restricted and we can easily check which functionalities were actually changed, instead of inspecting changes in a monolith core crate that is always evolving.

# Guide-level explanation

Two types of plugins will be defined: core plugins and regular (optional? external?) plugins. Core plugins are actually used by the `tauri` crate so they are defined in the crate itself and automatically configured and enabled, users will see these plugins as regular Rust modules that just export types and functions to be directly used. That is the case for the `path` and `file_system` plugins. The other APIs (`cli`,  `dialog`, `http`, `notification` and `process`) are optional and will be defined in the [plugins-workspace repo](https://github.com/tauri-apps/plugins-workspace) along with new plugins introduced in v2 like `camera` and `geolocation`.
For the `path` plugin implementation, see [the PR](https://github.com/tauri-apps/tauri/pull/6339).
  
# Reference-level explanation

Each API will follow the existing Tauri plugin structure, with the new mobile capabilities defined in the [RFC #0005](https://github.com/tauri-apps/rfcs/pull/9). Like any plugin in the plugins-workspace repository, they will have their own Rust crate and NPM package defining the JavaScript interface to consume its API, along with the Android and Swift packages implementing the functionality on mobile if needed.

## Platform Considerations

We can improve the existing desktop APIs, but it is not required to do so. The main goal of this refactor is to actually implement existing functionality on mobile, though improvements are always available and are being discussed for instance on the [capability-based API issue](https://github.com/tauri-apps/tauri/issues/6107).

## Security Considerations

With each API as a separate package, it will be easier to keep the security guarantees from previous audits. Changes will be easier to track and audit.

## Performance Considerations

Performance is not impacted by this change. Existing desktop code runs the same way, and mobile APIs are executed in the native layer so it can leverage the most performant way of doing things.

## Community Considerations

This refactor introduces a massive breaking change. All existing code using the `tauri::api` module or the `@tauri-apps/api` package would break. To mitigate this and simplify the migration path to v2, I propose we move all existing functionality (both the `tauri::api` module and the current Tauri endpoints) to a `legacy` plugin that can be enabled with a Cargo feature. With this plugin, an existing Tauri v1 application could start using the v2 functionalities by just enabling the Cargo feature and changing the `tauri::api` imports to `tauri::legacy`. This solution can also be used for other breaking changes for v2 and beyond, as existing functions that have incompatible new signatures could be moved to a Rust trait that can be imported by the developers to migrate major versions without touching their existing codebase.

# Drawbacks

This change introduces a lot of new packages. Each plugin must have a crate and an NPM package exposing the webview API usage. Keeping both versions in sync is necessary for compatibility, so it introduces complexity for end users. To help users, we are proposing a [`tauri add` CLI command](https://github.com/tauri-apps/tauri/issues/6505).

# Rationale and alternatives

- This design is good for maintainability as it separates application APIs from the core crate. Additionally, it simplifies the mess associated with target detection for the huge list of platforms we must support: macOS, Linux, Windows, Android and iOS.
- It reduces the current feature flags hell with the allowlist Cargo features. Currently, the tauri crate has over 70 Cargo features related to the allowlist. These flags would be moved to each API plugin crate.

# Prior art

For possible mobile-specific plugins, we can learn a lot from the official Capacitor [plugin collection](https://github.com/ionic-team/capacitor-plugins).

# Unresolved questions

How should we handle versioning? Should we keep minors in sync with the core crate so it is easier to determine incompatibilities? Syncing can be problematic when we have 20+ plugins and we only want to minor bump one of them...

# API list

Here is the complete list of APIs we need to rewrite as plugins. Some of them are core plugins. Not all of them actually need dedicated Android or iOS packages, for instance the `path` APIs on iOS can be written in Rust as it only relies on environment variables and work the same as on macOS.

| API              | Kind    | Desktop | Mobile | Needs Android project | Needs Swift project |
| ---------------- | ------- | ------- | ------ | --------------------- | ------------------- |
| path             | core    | ✓       | ✓      | ✓                     | x                   |
| file-system      | core    | ✓       | ✓      | x                     | x                   |
| window           | core    | ✓       | ✓      | x                     | x                   |
| event            | core    | ✓       | ✓      | x                     | x                   |
| tauri            | core    | ✓       | ✓      | x                     | x                   |
| app              | core    | ✓       | ✓      | x                     | x                   |
| testing (mocks)  | regular | ✓       | ✓      | x                     | x                   |
| clipboard        | regular | ✓       | ✓      | ✓                     | ?                   |
| dialog           | regular | ✓       | ✓      | ✓                     | ?                   |
| http             | regular | ✓       | ✓      | x                     | x                   |
| notification     | regular | ✓       | ✓      | ✓                     | ?                   |
| shell            | regular | ✓       | ✓      | ✓                     | ?                   |
| operating-system | regular | ✓       | ✓      | x                     | x                   |
| cli              | regular | ✓       | x      |                       |                     |
| process          | regular | ✓       | x      |                       |                     |
| global-shortcut  | regular | ✓       | x      |                       |                     |
| camera           | regular | x       | ✓      | ✓                     | ✓                   |
| geolocation      | regular | x       | ✓      | ✓                     | ✓                   |
| share            | regular | x       | ✓      | ✓                     | ✓                   |

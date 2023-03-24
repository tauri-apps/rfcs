- Feature Name: mobile_plugin
- Start Date: 2023-03-23
- RFC PR: [tauri-apps/rfcs#0004](https://github.com/tauri-apps/rfcs/pull/9)

# Summary

Tauri needs access to native mobile APIs such as camera, geolocation and notifications. These APIs can be called by Rust code using `jni-rs` and `objc`, but writing Kotlin or Java code for Android and Swift code for iOS simplifies development and is a much more mature way of accessing these APIs.

# Motivation

With access to native Android projects and Swift packages for iOS, Tauri and its users can use any mobile API a native application might need. This proposal includes a way to call the native mobile code from both the webview and the Rust layers, giving more control to both application and library writers.

# Guide-level explanation

For the sake of simplicity, we can simply extend the existing Tauri plugin API to allow registering a native mobile project and linking it to the application.

## Build Script

The plugin's build script will be responsible for injecting the plugin's Android project to the application's project inside the `gen/android/.tauri/plugins` folder, and compiling and linking the Swift library for iOS using `swift-rs`:

```rust
use std::process::exit;

fn main() {
  if let Err(error) = tauri_build::mobile::PluginBuilder::new()
    .android_path("android")
    .ios_path("ios")
    .run()
  {
    println!("{error:#}");
    exit(1);
  }
}

```

## Example Plugin

Let's see an example plugin code for both Android and iOS:

```kotlin
package com.plugin.example

import android.app.Activity
import app.tauri.annotation.Command
import app.tauri.annotation.TauriPlugin
import app.tauri.plugin.JSObject
import app.tauri.plugin.Plugin
import app.tauri.plugin.Invoke

@TauriPlugin
class ExamplePlugin(private val activity: Activity): Plugin(activity) {
  private val implementation = new Implementation() // TODO: write implementation

  @Command
  fun getCurrentLocation(invoke: Invoke) {
    val precision = invoke.getInt("precision")
    val location = implementation.getCurrentLocation(precision)
    val response = JSObject()
    response.put("location", location)
    invoke.resolve(response)
  }
}
```

```swift
import UIKit
import WebKit
import Tauri
import SwiftRs

class ExamplePlugin: Plugin {
	@objc public func getCurrentLocation(_ invoke: Invoke) throws {
		let precision = invoke.getString("precision")
    // TODO: write implementation
    let location = getCurrentLocation(precision)
		invoke.resolve(["location": location as Any])
	}
}

@_cdecl("init_plugin_example")
func initPlugin(name: SRString, webview: WKWebView?) {
	Tauri.registerPlugin(webview: webview, name: name.toString(), plugin: ExamplePlugin())
}
```

### Runtime Registration

At runtime, we will provide an API the plugin can call to register the Android and iOS interfaces:

```rust
// generates the function that initializes the iOS plugin
#[cfg(target_os = "ios")]
tauri::ios_plugin_binding!(init_plugin_example);

tauri::plugin::Builder::new("example")
  .setup(|app, api| {
    // Register the Android plugin class ExamplePlugin in the com.plugin.example package
    #[cfg(target_os = "android")]
    let handle = api.register_android_plugin("com.plugin.example", "ExamplePlugin");
    // Register the iOS plugin
    #[cfg(target_os = "ios")]
    let handle = api.register_ios_plugin(init_plugin_example);
  });
```

## Calling a Command

The `register` functions return a `PluginHandle` that can be used to directly call a plugin command:

```rust
let handle = api.register_...();

#[derive(serde::Serialize)]
struct GetCurrentLocationPayload {
  precision: usize,
}

#[derive(serde::Deserialize)]
struct CurrentLocation {
  location: String,
}

let payload = GetCurrentLocationPayload {
  precision: 1,
};
// Run the plugin command `getCurrentLocation` with the given payload (any Serialize type)
let response: CurrentLocation = handle.run_mobile_plugin("getCurrentLocation", payload).unwrap();
```

The default plugin template will include this boilerplate, storing the plugin handle in the Tauri state using the newtype idiom so each plugin has its own handle.

Additionally, the mobile plugin command can be reached directly by the webview side:

```javascript
import { invoke } from '@tauri-apps/api/invoke'
const { location } = await invoke('plugin:example|getCurrentLocation', { precision: 0 })
```
  
# Reference-level explanation

To achieve this functionality, we need to change the existing codebase:

## Plugin Template

- Adapted to support both mobile and desktop
- iOS and Android projects

## CLI

- New commands to add Android and iOS projects to an existing plugin
- On dev and build, inject the `Tauri` dependency on each plugin so they can be compiled
  - This dependency exposes the native plugin API for Android and iOS and is injected dynamically so it's always in sync

## tauri

- New plugin APIs for registering and calling mobile plugin commands `PluginApi` and `PluginHandle`
- The IPC handler now sends a message to the native Kotlin and iOS IPC handlers if Rust did not respond to the command
  - With this change, the JavaScript layer can directly reach the native mobile APIs
- New macro to generate the function that initializes the iOS plugin `ios_plugin_binding!(fn_name)`
- Plugin payloads are converted to `serde_json::Value` and a Rust code converts that to a Swift dictionary using Objective-C or a Java map using JNI.
- Plugin responses are serialized by the native code and deserialized on Rust :(

## tauri-build

- New `mobile` module with a builder to run the Android and iOS build pipelines for a plugin
  - For Android, injects the Android Studio project to the application folder
  - On iOS, build and link the Swift package

## Platform Considerations

Plugin and user code must be careful with the target platforms when writing code for mobile. `#[cfg(target_os = "ios")]` and `#[cfg(target_os = "android")]` will be widely used, and the default plugin template takes care of the initial boilerplate for this, also preparing the plugin code to work on desktop by introducing a separate module to write the same functionality in Rust so macOS, Linux and Windows applications can call the same APIs. So to support all platforms, users need to write both Java, Swift and Rust code, taking care of using each platform's APIs to reach the plugins goals. This introduces a lot of work for plugin maintainers, but has a lot of benefits for application developers that can use all the functionality with an unified API.

## Security Considerations

Note that the IPC messages still go through the Rust IPC handler so our security guarantees are untouched and the isolation pattern is still used.

## Performance Considerations

This should be more performant than existing solutions since we directly use native Java and iOS code instead of relying on several JNI and Objective-C calls.

Unfortunately, we still need to serialize payloads and responses.

## Community Considerations

There is a small breaking change in the plugin setup hook (from `fn setup(app)` to `fn setup(app, api)`). Existing plugins will continue working after this change.

# Drawbacks

Currently the biggest drawback is still serialization of payloads and responses.

# Rationale and alternatives

This design does not introduce a lot of new concepts, mainly reusing the existing plugin and IPC functionalities.
The alternative of using JNI-rs and objc code is unsustainable for larger codebases.

# Prior art

The native plugin API was inspired by the Capacitor plugin interface, JS bridge types were forked from their codebase and the `Invoke` API is the same as the `PluginCall` Capacitor API. We could also offer some type of compatibility package that allows running Capacitor plugins on Tauri applications in the future. Capacitor also offers several hooks and helpers we do not have yet.

# Unresolved questions

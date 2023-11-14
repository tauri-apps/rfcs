- Feature Name: wry-raw-window-handle
- Start Date: 16-10-2023
- RFC PR: [tauri-apps/rfcs#12](https://github.com/tauri-apps/rfcs/pull/12)
- Implementation PR: https://github.com/tauri-apps/wry/pull/1041
- Tracking Issue: [tauri-apps/wry#677](https://github.com/tauri-apps/wry/issues/677)

# Summary

Make WRY agnostic of the windowing library used and accept a raw-window-handle or gtk-window-handle where applicable.

# Motivation

This has been requested in [tauri-apps/wry#677](https://github.com/tauri-apps/wry/issues/677) a long time ago and at work at CrabNebula,
we have been contracted to research and develop an integration of wry and wgpu/bevy. We had a successful results and it aligned with [tauri-apps/wry#677](https://github.com/tauri-apps/wry/issues/677)
so we did an initial implemention of this RFC for Windows which can be found here https://github.com/crabnebula-dev/wry/commits/feat/embedded-webview and we think
it is a good oprtunity to upstream this work.

# Guide-level explanation
There is two groups of window handles we will need to take in order to support the current platforms we support:

#### Raw window handle:
This group could leverage the `raw-window-handle` crate; this includes macOS, Windows and Linux (x11) platforms.
  - Using winit, tao, egui or any other windowing library that
    implements [`raw_window_handle::HasWindowHandle`](https://docs.rs/raw-window-handle/latest/raw_window_handle/trait.HasWindowHandle.html) trait
    (or previous versions of `raw-window-handle` crate):
    ```rs
    let window = Window::new(&evl); // Window implements HasWindowHandle
    let webview = WebView::new(&window);  
    ```
    
  - Using raw win32:
    ```rs
    let hwnd = CreateWindowExW(/* args */);
    let webview = WebView::new_hwnd(hwnd);  
    ```
    
  - Using raw AppKit NSWindow:
    ```rs
    let ns_window = msg_send![/* args */];
    let webview = WebView::new_ns_view(ns_window.contentView());  
    ```
    
  - Using raw X11 (not recommended):
    ```rs
    // must initialize gtk first
    gtk::init().unwrap(); 
    let window = XCreateWindow(/* args */);
    let webview = WebView::new_x11(window);

    // and in event loop, you need to advance the gtk loop as well
    loop {
      // 1. handle x11 event loop
      // 1. handle gtk event loop through `gtk::main_iteration_do`
    }  
    ```

#### Gtk handle:
This group leverages the gtk types, includes Linux platform only (X11 and Wayland). 
  - Using raw gtk:
    ```rs
    let gtk_window = gtk::ApplicationWindow::new(/* args */);
    let webview = WebView::new_gtk(&gtk_window);  
    ```
    
  - Using tao:
    ```rs
    let window = Window::new(/* args */);
    let gtk_window = window.gtk_window();
    let webview = WebView::new_gtk(&gtk_window);  
    ```


# Reference-level explanation

There is 2 APIs this RFC is proposing:
- `WebView::new(raw_window_handle)/Webview::new_gtk(gtk_wdiget_handle)` (current behavior)
  initializes the webview directly in the passed handle.
  - Supported platforms: Windows, macOS and Linux (X11 & Wyaland)
  - Does NOT support `webview.set_position()` as managing the size and position of the webview should be done by sizing or changing the position of
    the handle passed in.
- `WebView::new_as_child(parent_raw_window_handle)` initializes the Webview as a child in the passed window handle, this allows for multiple webviews in
  the same window handle passed. 
  - Supported platforms: Windows, macOS and Linux (X11 Only)
  - Supports `webview.set_position()/webview.set_size()` to change the position and size of the webview child inside the parent.

There will also be equivalent methods that is specific to each platform, in case someone don't want to
go through `raw-window-handle` crate:
- `WebView::new_hwnd(hwnd)` and `WebView::new_as_child_hwnd`: Windows Only
- `WebView::new_x11(hwnd)` and `WebView::new_as_child_x11`: Linux(X11) Only
- `WebView::new_ns_view(hwnd)` and `WebView::new_as_child_ns_view`: macOS Only

## Platform Considerations

- While Linux (X11) appears in both groups, it is recommended to be used through the gtk window handle instead of the raw X11 handle because
  to use the raw x11 handle, you'd still need to initialize gtk and advance its event loop alongside the x11 loop.
- Linux (Wayland) doesn't allow creating a raw `GdkWaylandWindow` from a raw wayland window, so it can only be part of the "Gtk handle" group.
- On Linux, which ever API you choose to use, the developer has to ensure that they initialize gtk through `gtk::init` and
  have a gtk event loop running on the thread, either alone or with another loop like X11 event loop using `gtk::main_iteration_do` 

## Community Considerations

Implementing this RFC will open up WRY to the broader Rust ecosystems (and potentioally other languages and runtimes).

# Drawbacks

_None that I can think of at the moment_

# Rationale and alternatives

Another alternative is to add this RFC behind a feature flag and make it opt-in for those who need it.

# Unresolved questions

### Android
While Android have a form of multi windows, it is not the same as multi windows on Desktop and the current Android implementation
doesn't require any access to a "Window" type of sorts and operates on the assumption that it is One-window, instead it requires an 
invokation of a macro to setup some JNI callbacks that is used later to add a webview to the current Android activity. There is no PoCs yet
for this but in theory, the relevant parts from TAO could be extracted out into WRY repo while keeping the current behavior which doesn't conflict with
this proposal.

### iOS 
_unexplored, needs someone with iOS experience_

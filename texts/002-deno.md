# RFC #002 Deno Enhancements

| Status        | Proposed       |
:-------------- |:---------------------------------------------------- |
| **RFC #0002**     | [0002](https://github.com/tauri-apps/rfcs/pull/NNN) |
| **Authors** | Daniel Thompson-Yvetot (denjell[at]tauri.studio), David  Lemarier (david[at]lemarier.ca)  |
| **External Sponsor** | Bartlomieju (biwanczuk[at]gmail.com)
| **Updated**   | 2020-06-29                |


## Objective
The main objective is to extrapolate the Rust source from tauri by providing a binary with built-in javascript support as a new entrypoint to develop cross-platform applications. 

## Motivation
Deno provides a complete API that is comparable to node. If we can have a light version of deno ~ 10 mb, and a tauri webview / api ~ 2 mb, we can have an electron style binary that is smaller than 12mb without compression -- (comparable to 130mb for the MacOS electron framework)

## User Benefit
The user may build a tauri app with only javascript and without the need to install anything else. They can build their app by downloading the tauri binary and writing some javascript code (using deno v8) following an Electron-like development pipeline. 

The user gains full access to the Deno Namespace/API and we don't have to maintain a complete API as a result. 

The binary is built with deno permissions are disabled by default. The user may enable the appropriate permissions giving them access to the file system, network settings and the environment. Access to security sensitive areas or functions require the use of permissions which need to be explicitly granted to the process. (https://deno.land/manual/getting_started/permissions#permissions)

The user will only need one central configuration file via `tauri.conf.json`.  From this file, they can configure the bundler, icons, app name, version and all of the other settings.  As such, no Cargo configuration will be required.

A user can create a simple application by getting the binary from our CDN. They then create a config file, write some javascript and the app is ready for deployment. An app can be comprised of only two files and a single binary; thats it.



## Design Proposal

Proposal is a seperate project outside of Tauri that runs along with the current implementation. The various pieces needed are as follows:

- deno-cli-light (v8, deno api, js compiler)
- tauri-core (runner etc)
- tauri-api (rpc, events etc)
- tauri-bundler (create .msi, .app, .pkg, .appimage etc...)

If they want to create a self contained binary with source (js) they need to use the "tauri-packer" which will require Rust. It will compile the binary inside a Vec<u8> (This will be defined in detail later)

Electron requires the developer to use the API on the renderer process which is an issue. The tauri rpc / events system can be used to allow the user to easily communicate between the renderer process and the main process.


https://www.figma.com/file/4xkR35cKnKTpALQYibAfwi/Untitled?node-id=0%3A1

### Alternatives Considered
Doing this manually is possible, but won't be easy for people to dive into.

### Performance Implications
Tauri.js will create a larger and slightly slower app than a tauri-core app. Onboarding and development is easier however.   

### Dependencies
Deno

### Engineering Impact
It may require a new interface for tauri which shouldn't require any large changes to the current system.

### Platforms and Environments
Windows, Linux, OSX, Android, iOS -- deno can be compiled on these platforms so it shouldn't break anything to bring the tauri.js to the same platforms as tauri (rust)

### Security Impact
Tauri.js will be as secure as tauri (rust) as it run in the deno sandbox with full deno permissions for the bound V8.

If we ship "bare js" - like the ASAR in electron, we have actually reduced security when compared to "traditional" tauri apps.

### Tutorials and Examples
We will need to make a new "recipe" page and potentially an entirely new doc page. This is not just a mere change of the `tauri.conf.json`

### Compatibility
This should be done before beta.

### User Impact
Existing users have the understanding that there will be changes to Tauri before we make a beta release, so now is the right time to do this. 

## Questions and Discussion Topics

How will dev mode work in this model (ie no rust, no cargo)?

How can we treeshake out the functionality if we are shipping a monolith library?

How does conditional compilation work between platforms?  Are there multiple binaries or just a single one?


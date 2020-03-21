# RFC #0000 Webview

| Status        | Proposed       |
:-------------- | ---------------------------------------------------- |
| **RFC #** | [0000](https://github.com/tauri-apps/governance-and-guidance/pull/1) |
| **Authors** | Daniel Thompson-Yvetot (denjell@tauri.studio), Tensor Programming (tensor@tauri.studio) |
| **External Sponsor** | Erlend (e.soghe@gmail.com) |
| **Updated** | 2020-03-10 |


## Objective
We would like to have more direct involvement in the critical upstream libraries upon which Tauri is built.

## Motivation
Tauri is a project that is heavily reliant on upstream bindings (currently boscop/web-view and zserge/webview). Over the past year there have been spurts of activity in both of these upstream resources, but communication with the lead developers has been challenging.

Furthermore, these two libraries have diverged in their source and approach and it is difficult to know where to contribute, let alone know when PRs will be merged.

Not only are there outstanding features (like [transparent and borderless windows](https://github.com/Boscop/web-view/commit/55f619190e6aa8c54fde8cf72d71a5126238a5e3), [additional menu items](https://github.com/Boscop/web-view/pull/125) etc.), but also [severe memory leaks](https://github.com/Boscop/web-view/issues/79).

Since we plan to release our Beta at some point in the next 4 months, we absolutely cannot carry this kind of architectural debt with us and need to come to a decision within the next 10 days.

## User Benefit
- Consumers of Tauri will be able to have assurance that we are able to protect and maintain essential code.
- The open source community at large becomes less fractured and our work will directly support other communities using webviews.
- Consuming from the "pure" webview source will enable Tauri projects to more easily use other bindings at some point in the future.

## Design Proposal

### In any Case
We will create generic mappings to the rust bindings, this will allow developers to more easily switch to other forks of webview and potentially other rendering engines entirely.

### Solution
We would like to pool our efforts with the OSS community. Our preferance was to take part in the creation of an organization along side @boscop. This is looking to be less likely at this point in time, so we will play it safe and create our own abstraction layer over the various webview backends. This will allow us and our users to pick and chose which is most ideal per platform, as well as make such changes non-breaking and open us up to patching in backends.

### Alternatives Considered
- Writing our own bindings.
- Doing nothing and waiting.

### Performance Implications
We expect general improvements across the board.

### Dependencies
The way this plays out will necessarily impact the dependency tree, but it will be transparent for consumers of Tauri.  Also, some of the C++ dependencies will be easier to consume on certain platforms.

### Engineering Impact
We expect binary sizes to remain the same and hope to have improved memory management for Windows platforms.

### Platforms and Environments
This will affect all platforms, but again, it will not affect the way in which Tauri works on a fundamental level.

### Security Impact
Maintaining some kind of influence over the bindings and the headers needed for properly and safely using webviews will absolutely enhance our security posture.

### Tutorials and Examples
Since this will mostly concern developers of the library itself, we should anticipate enhancing our own documentation in any case.

### Compatibility
There may be API changes, but it will fall in the scope of ALPHA.  For information regarding the API changes, refer to the webview repository's [readme](https://github.com/zserge/webview).  

### User Impact
We anticipate only forward-changes with additionally exposed APIs.

## Questions and Discussion Topics
What do you think? Is it going to be better for Tauri to hard fork and go its own way - or be part of the community and work together?

Would you be more willing to contribute to the webview organization or the Tauri organization? (Or doesn't it matter?)

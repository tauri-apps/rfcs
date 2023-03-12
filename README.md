# Tauri Request For Comment

This repo is dedicated to the Tauri RFC (Request for Comment) process. When there are significant changes to Tauri this allows the project to undergo a transparent conversation and confirm changes by accepting and merging them into the repo.

## When To Create An RFC

Not everything necessarily requires an RFC. Sometimes you may wish to have a discussion before writing a full RFC or to simply submit an idea. Here's when you could use a [Feature Request](https://github.com/tauri-apps/tauri/issues/new?assignees=&labels=type%3A+feature+request&template=feature_request.yml&title=%5Bfeat%5D+) to start off the conversation.

Be sure to check through the [existing feature requests in the Tauri repo](https://github.com/tauri-apps/tauri/labels/type%3A%20feature%20request) as your idea may already be there and would be a good place to add your input into!

## RFC Process

1) Create an RFC Pull Request by following the steps in the [Creating an RFC](#creating-an-rfc) section below.
2) Once a PR is opened in this repo then the review process begins. This will be left open for a minimum of 2 weeks to allow comment.
3) After this time period a member of the Tauri Working Group will handle closing the RFC. If accepted, the RFC will be assigned a number.
4) If an implementation is already underway then the respective PR will be added to the RFC to track. If no implementation is started then a tracking issue in the respective repo will be created and added to the RFC to track.

An accepted RFC does not necessairly mean that Tauri will be actively working on implimentation, but instead signals that the the details in the RFC align with the overall goals of Tauri. The tracking issue or implimentation PR are where the status of an RFC's implementation (in implimentation, open to someone contributing, etc.) can be found.

## Creating an RFC

Once you have enough information to begin an RFC you can follow this process to get started:

1) **Fork this repo**
2) **Copy the template** Move your copy of `template.md` into the `texts` folder, naming it in the scheme of `0000-feature.md`. Note: the number is literal and will be adjusted just before merging.
3) **Fill the template out** Replace all relevant sections with explanations. Put care into the details, as it will serve as a reference through the development process.
4) **Open a PR** After opening a PR, the RFC is open for comment. Discussion should happen in the comments of the PR. RFCs that are "invalid" (don't follow the format/proceedure, violate CoC, or are otherwise unable to be used) may be closed immediately.
5) **Merged into the repo** The final stage of an RFC that signals it is accepted is it being merged into this repo. If a PR for an implementation is already submitted then it will be used to track the status from there. Otherwise a tracking issue will be created in the respective repo to capture the implementation status.

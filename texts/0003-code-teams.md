# RFC #0003 Org Code Teams

| Status        | Proposed       |
:-------------- | ---------------------------------------------------- |
| **RFC #** | 0003 |
| **Authors** | Jacob Bolda ( jacob@tauri.studio / @JacobBolda#4211 ) |
| **Sponsor** | core |
| **Updated** | 2020-09-12 |

## Objective
Create teams for the various areas of code and docs to help expand our team, create sense of ownership, guide code improvements (through defined tasks), and facilitate decisions.

## Motivation
We are sorely in need of help now and in the future when core members get busy or burned out. We need a pipe to bring people in and a way for those on a team to step back when they need the headspace.

## Design Proposal

### Solution
Create teams (and a workflow to create more in the future) to cover the main parts of the code. Each team's primary function is to drive discussion and facilitate decisions around their area of the code. The team will make a final decision, but it cannot be finalized until the recommendation is put forward to the community to discuss. (The intent to prevent dictator or "from on high" type decisions.)

As decisions are made, the team should create tasks to take the decision to completion. The tasks should be of a size that is achievable in a PR. This will help scope work to something that anyone could reasonably hop into the code with minimal familiarity and help out.

Secondary responsibilities would be review the PRs and enable the docs to make it easy for someone to come in and contribute to that portion on the code. ("Docs" doesn't necessarily need to be something on tauri.studio.)

To address the inflow and outflow of members, we will list all team members and "open positions". Each team should have 3+ members, 3 of which being the primary members. The primary members should have the most expectation of responsibility. The primary members are not lifetime, static positions. There is an expectation that they change with some frequency and not be a "reward" to be earned.

Members may step back as an "alumnus" member. All roles operate in the open so this is doesn't reduce access in any manner. Rather, the intent is to signal that the time is not available for them to give real attention to the project. A way that one could temporarily or permanently step back without any guilt.

### Breakdown
#### Teams
- Tauri Core
- Tauri CLIs
  - usage
  - onboarding of new users
- Tauri APIs and Commands
  - importing Tauri and using it in your app
- Tauri Bundler
- Webview (interfacing with and work to make certain it has the APIs we need)

### Engineering Impact
#### Will this increase or decrease build times, developer workload etc.?
More hands should help spread the load and improve the velocity of the docs and code.

### Security Impact
#### What changes will this bring, if any, to the security model of CI, developers and users?
This should allow us to bring in more people to help out. More eyes covering every corner will result in more bugs being caught, and relieving some of the managerial duties from core should allow more attention to security.

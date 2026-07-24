---
type: Feature
title: "Add a read-only view exposing the current routing state"
labels: type:feature, area:routing, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose routing through a read view

### Description
There is no O(1) read view for the routing state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the routing state without mutating storage.
- Return a sensible default (not a panic) when routing is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/routing-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(routing): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the routing logic"
labels: type:test, area:routing, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover routing boundaries

### Description
The routing logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of routing, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/routing-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(routing): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated routing check into a shared helper"
labels: type:refactor, area:routing, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate routing checks

### Description
Multiple entrypoints repeat the same routing precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated routing check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/routing-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(routing): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the routing model and its invariants"
labels: type:docs, area:routing, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the routing model

### Description
The routing model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/routing.md` describing the routing data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/routing-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(routing): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when routing state changes"
labels: type:feature, area:routing, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a routing event

### Description
State changes to routing emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on routing state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/routing-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(routing): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current fees state"
labels: type:feature, area:fees, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose fees through a read view

### Description
There is no O(1) read view for the fees state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the fees state without mutating storage.
- Return a sensible default (not a panic) when fees is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/fees-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(fees): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the fees logic"
labels: type:test, area:fees, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover fees boundaries

### Description
The fees logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of fees, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/fees-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(fees): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated fees check into a shared helper"
labels: type:refactor, area:fees, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate fees checks

### Description
Multiple entrypoints repeat the same fees precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated fees check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/fees-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(fees): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the fees model and its invariants"
labels: type:docs, area:fees, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the fees model

### Description
The fees model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/fees.md` describing the fees data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/fees-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(fees): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when fees state changes"
labels: type:feature, area:fees, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a fees event

### Description
State changes to fees emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on fees state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/fees-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(fees): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current pairs state"
labels: type:feature, area:pairs, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose pairs through a read view

### Description
There is no O(1) read view for the pairs state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the pairs state without mutating storage.
- Return a sensible default (not a panic) when pairs is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pairs-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(pairs): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the pairs logic"
labels: type:test, area:pairs, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover pairs boundaries

### Description
The pairs logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of pairs, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/pairs-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(pairs): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated pairs check into a shared helper"
labels: type:refactor, area:pairs, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate pairs checks

### Description
Multiple entrypoints repeat the same pairs precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated pairs check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/pairs-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(pairs): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the pairs model and its invariants"
labels: type:docs, area:pairs, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the pairs model

### Description
The pairs model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/pairs.md` describing the pairs data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/pairs-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(pairs): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when pairs state changes"
labels: type:feature, area:pairs, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a pairs event

### Description
State changes to pairs emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on pairs state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pairs-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(pairs): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current quote state"
labels: type:feature, area:quote, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose quote through a read view

### Description
There is no O(1) read view for the quote state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the quote state without mutating storage.
- Return a sensible default (not a panic) when quote is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/quote-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(quote): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the quote logic"
labels: type:test, area:quote, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover quote boundaries

### Description
The quote logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of quote, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/quote-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(quote): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated quote check into a shared helper"
labels: type:refactor, area:quote, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate quote checks

### Description
Multiple entrypoints repeat the same quote precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated quote check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/quote-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(quote): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the quote model and its invariants"
labels: type:docs, area:quote, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the quote model

### Description
The quote model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/quote.md` describing the quote data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/quote-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(quote): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when quote state changes"
labels: type:feature, area:quote, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a quote event

### Description
State changes to quote emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on quote state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/quote-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(quote): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current admin state"
labels: type:feature, area:admin, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose admin through a read view

### Description
There is no O(1) read view for the admin state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the admin state without mutating storage.
- Return a sensible default (not a panic) when admin is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/admin-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(admin): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the admin logic"
labels: type:test, area:admin, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover admin boundaries

### Description
The admin logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of admin, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/admin-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(admin): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated admin check into a shared helper"
labels: type:refactor, area:admin, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate admin checks

### Description
Multiple entrypoints repeat the same admin precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated admin check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/admin-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(admin): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the admin model and its invariants"
labels: type:docs, area:admin, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the admin model

### Description
The admin model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/admin.md` describing the admin data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/admin-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(admin): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when admin state changes"
labels: type:feature, area:admin, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a admin event

### Description
State changes to admin emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on admin state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/admin-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(admin): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current storage state"
labels: type:feature, area:storage, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose storage through a read view

### Description
There is no O(1) read view for the storage state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the storage state without mutating storage.
- Return a sensible default (not a panic) when storage is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/storage-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(storage): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the storage logic"
labels: type:test, area:storage, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover storage boundaries

### Description
The storage logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of storage, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/storage-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(storage): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated storage check into a shared helper"
labels: type:refactor, area:storage, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate storage checks

### Description
Multiple entrypoints repeat the same storage precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated storage check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/storage-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(storage): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the storage model and its invariants"
labels: type:docs, area:storage, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the storage model

### Description
The storage model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/storage.md` describing the storage data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/storage-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(storage): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when storage state changes"
labels: type:feature, area:storage, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a storage event

### Description
State changes to storage emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on storage state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/storage-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(storage): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current events state"
labels: type:feature, area:events, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose events through a read view

### Description
There is no O(1) read view for the events state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the events state without mutating storage.
- Return a sensible default (not a panic) when events is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/events-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(events): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the events logic"
labels: type:test, area:events, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover events boundaries

### Description
The events logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of events, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/events-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(events): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated events check into a shared helper"
labels: type:refactor, area:events, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate events checks

### Description
Multiple entrypoints repeat the same events precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated events check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/events-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(events): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the events model and its invariants"
labels: type:docs, area:events, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the events model

### Description
The events model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/events.md` describing the events data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/events-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(events): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when events state changes"
labels: type:feature, area:events, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a events event

### Description
State changes to events emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on events state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/events-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(events): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current limits state"
labels: type:feature, area:limits, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose limits through a read view

### Description
There is no O(1) read view for the limits state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the limits state without mutating storage.
- Return a sensible default (not a panic) when limits is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/limits-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(limits): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the limits logic"
labels: type:test, area:limits, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover limits boundaries

### Description
The limits logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of limits, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/limits-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(limits): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated limits check into a shared helper"
labels: type:refactor, area:limits, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate limits checks

### Description
Multiple entrypoints repeat the same limits precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated limits check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/limits-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(limits): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the limits model and its invariants"
labels: type:docs, area:limits, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the limits model

### Description
The limits model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/limits.md` describing the limits data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/limits-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(limits): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when limits state changes"
labels: type:feature, area:limits, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a limits event

### Description
State changes to limits emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on limits state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/limits-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(limits): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current authorization state"
labels: type:feature, area:authorization, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose authorization through a read view

### Description
There is no O(1) read view for the authorization state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the authorization state without mutating storage.
- Return a sensible default (not a panic) when authorization is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/authorization-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(authorization): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the authorization logic"
labels: type:test, area:authorization, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover authorization boundaries

### Description
The authorization logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of authorization, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/authorization-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(authorization): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated authorization check into a shared helper"
labels: type:refactor, area:authorization, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate authorization checks

### Description
Multiple entrypoints repeat the same authorization precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated authorization check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/authorization-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(authorization): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the authorization model and its invariants"
labels: type:docs, area:authorization, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the authorization model

### Description
The authorization model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/authorization.md` describing the authorization data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/authorization-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(authorization): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when authorization state changes"
labels: type:feature, area:authorization, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a authorization event

### Description
State changes to authorization emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on authorization state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/authorization-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(authorization): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a read-only view exposing the current settlement state"
labels: type:feature, area:settlement, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose settlement through a read view

### Description
There is no O(1) read view for the settlement state, forcing callers to reconstruct it. This issue adds a bounded, read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only entrypoint returning the settlement state without mutating storage.
- Return a sensible default (not a panic) when settlement is unset.
- Reuse stored values rather than recomputing where possible.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/settlement-01-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: unset state, boundary values.
- Include the full test output in the PR description.

### Example commit message
`feat(settlement): add read view`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for the settlement logic"
labels: type:test, area:settlement, stack:rust, stack:soroban, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Cover settlement boundaries

### Description
The settlement logic is thinly tested at its boundaries. This issue adds focused boundary and rejection tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for the accept/reject boundaries of settlement, asserting exact typed error codes.
- Use the test-utils helpers; assert events where the flow emits them.
- Do not change contract logic unless a defect is found (note it).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/settlement-01-boundaries`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: exactly-at boundary, one over, unauthorized caller.
- Include the full test output in the PR description.

### Example commit message
`test(settlement): cover boundaries and rejections`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the repeated settlement check into a shared helper"
labels: type:refactor, area:settlement, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Deduplicate settlement checks

### Description
Multiple entrypoints repeat the same settlement precondition inline. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract the repeated settlement check into a private helper and route entrypoints through it.
- Behaviour unchanged; same rejections and typed codes.
- No ABI change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/settlement-01-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: each existing rejection still fires identically.
- Include the full test output in the PR description.

### Example commit message
`refactor(settlement): extract shared check helper`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the settlement model and its invariants"
labels: type:docs, area:settlement, stack:rust, stack:soroban, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document the settlement model

### Description
The settlement model and its invariants are undocumented, making audits harder. This issue documents them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/settlement.md` describing the settlement data model, its invariants, and the entrypoints that touch it.
- Cross-reference the code with a worked example; keep it accurate.
- Read the relevant module first.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/settlement-01-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify invariants against source.
- Include the full test output in the PR description.

### Example commit message
`docs(settlement): document the model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a dedicated event when settlement state changes"
labels: type:feature, area:settlement, stack:rust, stack:soroban, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Emit a settlement event

### Description
State changes to settlement emit no dedicated event, forcing indexers to infer them. This issue adds one.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `symbol_short!` event (<= 9 chars topic) on settlement state change carrying the relevant ids/amounts.
- Do not change fund movement; ensure no topic collision.
- Capture events in tests immediately after the mutating call.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/settlement-02-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: no topic collision, event payload correctness.
- Include the full test output in the PR description.

### Example commit message
`feat(settlement): emit state-change event`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.

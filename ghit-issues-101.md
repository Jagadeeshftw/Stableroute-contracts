---
type: Feature
title: "Add a read-only view exposing the current rewards state"
labels: type:feature, area:rewards, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose rewards

### Description
Callers can't read rewards state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current rewards state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/rewards-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(rewards): add read-only view`

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
title: "Emit a dedicated event when rewards state changes"
labels: type:feature, area:rewards, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on rewards

### Description
rewards state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever rewards state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/rewards-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(rewards): emit state-change event`

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
title: "Add boundary tests for the rewards logic"
labels: type:test, area:rewards, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test rewards

### Description
rewards's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for rewards at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/rewards-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(rewards): add boundary tests`

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
title: "Extract the repeated rewards check into a helper"
labels: type:refactor, area:rewards, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for rewards

### Description
rewards repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract rewards's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/rewards-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(rewards): extract shared check`

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
title: "Document the rewards model and its invariants"
labels: type:docs, area:rewards, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document rewards

### Description
rewards's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/rewards.md` describing the rewards model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/rewards-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(rewards): document model and invariants`

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
title: "Add a read-only view exposing the current staking state"
labels: type:feature, area:staking, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose staking

### Description
Callers can't read staking state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current staking state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/staking-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(staking): add read-only view`

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
title: "Emit a dedicated event when staking state changes"
labels: type:feature, area:staking, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on staking

### Description
staking state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever staking state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/staking-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(staking): emit state-change event`

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
title: "Add boundary tests for the staking logic"
labels: type:test, area:staking, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test staking

### Description
staking's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for staking at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/staking-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(staking): add boundary tests`

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
title: "Extract the repeated staking check into a helper"
labels: type:refactor, area:staking, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for staking

### Description
staking repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract staking's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/staking-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(staking): extract shared check`

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
title: "Document the staking model and its invariants"
labels: type:docs, area:staking, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document staking

### Description
staking's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/staking.md` describing the staking model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/staking-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(staking): document model and invariants`

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
title: "Add a read-only view exposing the current claim state"
labels: type:feature, area:claim, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose claim

### Description
Callers can't read claim state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current claim state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/claim-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(claim): add read-only view`

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
title: "Emit a dedicated event when claim state changes"
labels: type:feature, area:claim, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on claim

### Description
claim state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever claim state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/claim-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(claim): emit state-change event`

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
title: "Add boundary tests for the claim logic"
labels: type:test, area:claim, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test claim

### Description
claim's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for claim at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/claim-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(claim): add boundary tests`

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
title: "Extract the repeated claim check into a helper"
labels: type:refactor, area:claim, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for claim

### Description
claim repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract claim's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/claim-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(claim): extract shared check`

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
title: "Document the claim model and its invariants"
labels: type:docs, area:claim, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document claim

### Description
claim's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/claim.md` describing the claim model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/claim-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(claim): document model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.

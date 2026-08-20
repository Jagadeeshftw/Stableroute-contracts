---
type: Feature
title: "Add a read-only view exposing the current router state"
labels: type:feature, area:router, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose router

### Description
Callers can't read router state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current router state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/router-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(router): add read-only view`

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
title: "Emit a dedicated event when router state changes"
labels: type:feature, area:router, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on router

### Description
router state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever router state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/router-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(router): emit state-change event`

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
title: "Add boundary tests for the router logic"
labels: type:test, area:router, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test router

### Description
router's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for router at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/router-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(router): add boundary tests`

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
title: "Extract the repeated router check into a helper"
labels: type:refactor, area:router, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for router

### Description
router repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract router's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/router-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(router): extract shared check`

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
title: "Document the router model and its invariants"
labels: type:docs, area:router, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document router

### Description
router's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/router.md` describing the router model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/router-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(router): document model and invariants`

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
title: "Add a read-only view exposing the current pool state"
labels: type:feature, area:pool, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose pool

### Description
Callers can't read pool state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current pool state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pool-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(pool): add read-only view`

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
title: "Emit a dedicated event when pool state changes"
labels: type:feature, area:pool, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on pool

### Description
pool state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever pool state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pool-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(pool): emit state-change event`

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
title: "Add boundary tests for the pool logic"
labels: type:test, area:pool, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test pool

### Description
pool's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for pool at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/pool-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(pool): add boundary tests`

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
title: "Extract the repeated pool check into a helper"
labels: type:refactor, area:pool, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for pool

### Description
pool repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract pool's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/pool-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(pool): extract shared check`

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
title: "Document the pool model and its invariants"
labels: type:docs, area:pool, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document pool

### Description
pool's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/pool.md` describing the pool model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/pool-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(pool): document model and invariants`

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
title: "Add a read-only view exposing the current swap state"
labels: type:feature, area:swap, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose swap

### Description
Callers can't read swap state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current swap state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/swap-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(swap): add read-only view`

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
title: "Emit a dedicated event when swap state changes"
labels: type:feature, area:swap, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on swap

### Description
swap state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever swap state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/swap-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(swap): emit state-change event`

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
title: "Add boundary tests for the swap logic"
labels: type:test, area:swap, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test swap

### Description
swap's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for swap at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/swap-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(swap): add boundary tests`

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
title: "Extract the repeated swap check into a helper"
labels: type:refactor, area:swap, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for swap

### Description
swap repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract swap's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/swap-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(swap): extract shared check`

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
title: "Document the swap model and its invariants"
labels: type:docs, area:swap, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document swap

### Description
swap's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/swap.md` describing the swap model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/swap-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(swap): document model and invariants`

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
labels: type:feature, area:quote, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose quote

### Description
Callers can't read quote state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current quote state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/quote-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(quote): add read-only view`

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
labels: type:feature, area:quote, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on quote

### Description
quote state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever quote state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/quote-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
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
title: "Add boundary tests for the quote logic"
labels: type:test, area:quote, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test quote

### Description
quote's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for quote at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/quote-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(quote): add boundary tests`

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
title: "Extract the repeated quote check into a helper"
labels: type:refactor, area:quote, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for quote

### Description
quote repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract quote's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/quote-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(quote): extract shared check`

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
labels: type:docs, area:quote, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document quote

### Description
quote's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/quote.md` describing the quote model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/quote-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(quote): document model and invariants`

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
title: "Add a read-only view exposing the current liquidity state"
labels: type:feature, area:liquidity, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose liquidity

### Description
Callers can't read liquidity state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current liquidity state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/liquidity-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(liquidity): add read-only view`

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
title: "Emit a dedicated event when liquidity state changes"
labels: type:feature, area:liquidity, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on liquidity

### Description
liquidity state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever liquidity state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/liquidity-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(liquidity): emit state-change event`

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
title: "Add boundary tests for the liquidity logic"
labels: type:test, area:liquidity, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test liquidity

### Description
liquidity's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for liquidity at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/liquidity-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(liquidity): add boundary tests`

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
title: "Extract the repeated liquidity check into a helper"
labels: type:refactor, area:liquidity, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for liquidity

### Description
liquidity repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract liquidity's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/liquidity-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(liquidity): extract shared check`

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
title: "Document the liquidity model and its invariants"
labels: type:docs, area:liquidity, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document liquidity

### Description
liquidity's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/liquidity.md` describing the liquidity model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/liquidity-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(liquidity): document model and invariants`

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
title: "Add a read-only view exposing the current fee state"
labels: type:feature, area:fee, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose fee

### Description
Callers can't read fee state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current fee state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/fee-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(fee): add read-only view`

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
title: "Emit a dedicated event when fee state changes"
labels: type:feature, area:fee, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on fee

### Description
fee state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever fee state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/fee-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(fee): emit state-change event`

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
title: "Add boundary tests for the fee logic"
labels: type:test, area:fee, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test fee

### Description
fee's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for fee at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/fee-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(fee): add boundary tests`

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
title: "Extract the repeated fee check into a helper"
labels: type:refactor, area:fee, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for fee

### Description
fee repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract fee's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/fee-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(fee): extract shared check`

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
title: "Document the fee model and its invariants"
labels: type:docs, area:fee, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document fee

### Description
fee's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/fee.md` describing the fee model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/fee-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(fee): document model and invariants`

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
title: "Add a read-only view exposing the current oracle state"
labels: type:feature, area:oracle, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose oracle

### Description
Callers can't read oracle state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current oracle state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/oracle-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(oracle): add read-only view`

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
title: "Emit a dedicated event when oracle state changes"
labels: type:feature, area:oracle, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on oracle

### Description
oracle state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever oracle state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/oracle-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(oracle): emit state-change event`

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
title: "Add boundary tests for the oracle logic"
labels: type:test, area:oracle, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test oracle

### Description
oracle's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for oracle at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/oracle-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(oracle): add boundary tests`

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
title: "Extract the repeated oracle check into a helper"
labels: type:refactor, area:oracle, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for oracle

### Description
oracle repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract oracle's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/oracle-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(oracle): extract shared check`

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
title: "Document the oracle model and its invariants"
labels: type:docs, area:oracle, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document oracle

### Description
oracle's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/oracle.md` describing the oracle model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/oracle-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(oracle): document model and invariants`

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
labels: type:feature, area:admin, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose admin

### Description
Callers can't read admin state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current admin state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/admin-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(admin): add read-only view`

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
labels: type:feature, area:admin, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on admin

### Description
admin state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever admin state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/admin-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
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
title: "Add boundary tests for the admin logic"
labels: type:test, area:admin, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test admin

### Description
admin's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for admin at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/admin-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(admin): add boundary tests`

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
title: "Extract the repeated admin check into a helper"
labels: type:refactor, area:admin, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for admin

### Description
admin repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract admin's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/admin-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(admin): extract shared check`

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
labels: type:docs, area:admin, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document admin

### Description
admin's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/admin.md` describing the admin model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/admin-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(admin): document model and invariants`

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
title: "Add a read-only view exposing the current treasury state"
labels: type:feature, area:treasury, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose treasury

### Description
Callers can't read treasury state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current treasury state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/treasury-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(treasury): add read-only view`

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
title: "Emit a dedicated event when treasury state changes"
labels: type:feature, area:treasury, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on treasury

### Description
treasury state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever treasury state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/treasury-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(treasury): emit state-change event`

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
title: "Add boundary tests for the treasury logic"
labels: type:test, area:treasury, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test treasury

### Description
treasury's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for treasury at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/treasury-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(treasury): add boundary tests`

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
title: "Extract the repeated treasury check into a helper"
labels: type:refactor, area:treasury, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for treasury

### Description
treasury repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract treasury's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/treasury-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(treasury): extract shared check`

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
title: "Document the treasury model and its invariants"
labels: type:docs, area:treasury, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document treasury

### Description
treasury's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/treasury.md` describing the treasury model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/treasury-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(treasury): document model and invariants`

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
title: "Add a read-only view exposing the current allowlist state"
labels: type:feature, area:allowlist, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose allowlist

### Description
Callers can't read allowlist state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current allowlist state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/allowlist-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(allowlist): add read-only view`

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
title: "Emit a dedicated event when allowlist state changes"
labels: type:feature, area:allowlist, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on allowlist

### Description
allowlist state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever allowlist state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/allowlist-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(allowlist): emit state-change event`

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
title: "Add boundary tests for the allowlist logic"
labels: type:test, area:allowlist, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test allowlist

### Description
allowlist's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for allowlist at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/allowlist-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(allowlist): add boundary tests`

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
title: "Extract the repeated allowlist check into a helper"
labels: type:refactor, area:allowlist, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for allowlist

### Description
allowlist repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract allowlist's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/allowlist-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(allowlist): extract shared check`

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
title: "Document the allowlist model and its invariants"
labels: type:docs, area:allowlist, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document allowlist

### Description
allowlist's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/allowlist.md` describing the allowlist model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/allowlist-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(allowlist): document model and invariants`

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
title: "Add a read-only view exposing the current pair state"
labels: type:feature, area:pair, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose pair

### Description
Callers can't read pair state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current pair state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pair-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(pair): add read-only view`

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
title: "Emit a dedicated event when pair state changes"
labels: type:feature, area:pair, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on pair

### Description
pair state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever pair state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pair-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(pair): emit state-change event`

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
title: "Add boundary tests for the pair logic"
labels: type:test, area:pair, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test pair

### Description
pair's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for pair at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/pair-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(pair): add boundary tests`

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
title: "Extract the repeated pair check into a helper"
labels: type:refactor, area:pair, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for pair

### Description
pair repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract pair's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/pair-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(pair): extract shared check`

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
title: "Document the pair model and its invariants"
labels: type:docs, area:pair, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document pair

### Description
pair's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/pair.md` describing the pair model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/pair-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(pair): document model and invariants`

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
title: "Add a read-only view exposing the current slippage state"
labels: type:feature, area:slippage, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose slippage

### Description
Callers can't read slippage state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current slippage state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/slippage-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(slippage): add read-only view`

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
title: "Emit a dedicated event when slippage state changes"
labels: type:feature, area:slippage, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on slippage

### Description
slippage state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever slippage state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/slippage-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(slippage): emit state-change event`

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
title: "Add boundary tests for the slippage logic"
labels: type:test, area:slippage, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test slippage

### Description
slippage's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for slippage at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/slippage-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(slippage): add boundary tests`

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
title: "Extract the repeated slippage check into a helper"
labels: type:refactor, area:slippage, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for slippage

### Description
slippage repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract slippage's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/slippage-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(slippage): extract shared check`

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
title: "Document the slippage model and its invariants"
labels: type:docs, area:slippage, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document slippage

### Description
slippage's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/slippage.md` describing the slippage model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/slippage-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(slippage): document model and invariants`

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
labels: type:feature, area:settlement, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose settlement

### Description
Callers can't read settlement state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current settlement state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/settlement-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(settlement): add read-only view`

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
labels: type:feature, area:settlement, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on settlement

### Description
settlement state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever settlement state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/settlement-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
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
++++++
---
type: Feature
title: "Add boundary tests for the settlement logic"
labels: type:test, area:settlement, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test settlement

### Description
settlement's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for settlement at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/settlement-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(settlement): add boundary tests`

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
title: "Extract the repeated settlement check into a helper"
labels: type:refactor, area:settlement, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for settlement

### Description
settlement repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract settlement's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/settlement-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(settlement): extract shared check`

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
labels: type:docs, area:settlement, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document settlement

### Description
settlement's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/settlement.md` describing the settlement model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/settlement-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(settlement): document model and invariants`

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
title: "Add a read-only view exposing the current bridge state"
labels: type:feature, area:bridge, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose bridge

### Description
Callers can't read bridge state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current bridge state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/bridge-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(bridge): add read-only view`

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
title: "Emit a dedicated event when bridge state changes"
labels: type:feature, area:bridge, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on bridge

### Description
bridge state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever bridge state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/bridge-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(bridge): emit state-change event`

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
title: "Add boundary tests for the bridge logic"
labels: type:test, area:bridge, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test bridge

### Description
bridge's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for bridge at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/bridge-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(bridge): add boundary tests`

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
title: "Extract the repeated bridge check into a helper"
labels: type:refactor, area:bridge, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for bridge

### Description
bridge repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract bridge's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/bridge-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(bridge): extract shared check`

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
title: "Document the bridge model and its invariants"
labels: type:docs, area:bridge, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document bridge

### Description
bridge's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/bridge.md` describing the bridge model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/bridge-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(bridge): document model and invariants`

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
title: "Add a read-only view exposing the current vault state"
labels: type:feature, area:vault, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose vault

### Description
Callers can't read vault state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current vault state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/vault-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(vault): add read-only view`

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
title: "Emit a dedicated event when vault state changes"
labels: type:feature, area:vault, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on vault

### Description
vault state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever vault state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/vault-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(vault): emit state-change event`

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
title: "Add boundary tests for the vault logic"
labels: type:test, area:vault, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test vault

### Description
vault's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for vault at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/vault-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(vault): add boundary tests`

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
title: "Extract the repeated vault check into a helper"
labels: type:refactor, area:vault, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for vault

### Description
vault repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract vault's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/vault-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(vault): extract shared check`

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
title: "Document the vault model and its invariants"
labels: type:docs, area:vault, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document vault

### Description
vault's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/vault.md` describing the vault model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/vault-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(vault): document model and invariants`

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
title: "Add a read-only view exposing the current registry state"
labels: type:feature, area:registry, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose registry

### Description
Callers can't read registry state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current registry state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/registry-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(registry): add read-only view`

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
title: "Emit a dedicated event when registry state changes"
labels: type:feature, area:registry, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on registry

### Description
registry state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever registry state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/registry-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(registry): emit state-change event`

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
title: "Add boundary tests for the registry logic"
labels: type:test, area:registry, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test registry

### Description
registry's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for registry at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/registry-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(registry): add boundary tests`

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
title: "Extract the repeated registry check into a helper"
labels: type:refactor, area:registry, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for registry

### Description
registry repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract registry's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/registry-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(registry): extract shared check`

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
title: "Document the registry model and its invariants"
labels: type:docs, area:registry, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document registry

### Description
registry's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/registry.md` describing the registry model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/registry-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(registry): document model and invariants`

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
title: "Add a read-only view exposing the current pauser state"
labels: type:feature, area:pauser, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose pauser

### Description
Callers can't read pauser state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current pauser state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pauser-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(pauser): add read-only view`

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
title: "Emit a dedicated event when pauser state changes"
labels: type:feature, area:pauser, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on pauser

### Description
pauser state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever pauser state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/pauser-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(pauser): emit state-change event`

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
title: "Add boundary tests for the pauser logic"
labels: type:test, area:pauser, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test pauser

### Description
pauser's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for pauser at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/pauser-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(pauser): add boundary tests`

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
title: "Extract the repeated pauser check into a helper"
labels: type:refactor, area:pauser, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for pauser

### Description
pauser repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract pauser's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/pauser-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(pauser): extract shared check`

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
title: "Document the pauser model and its invariants"
labels: type:docs, area:pauser, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document pauser

### Description
pauser's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/pauser.md` describing the pauser model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/pauser-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(pauser): document model and invariants`

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
title: "Add a read-only view exposing the current upgrade state"
labels: type:feature, area:upgrade, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose upgrade

### Description
Callers can't read upgrade state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current upgrade state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/upgrade-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(upgrade): add read-only view`

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
title: "Emit a dedicated event when upgrade state changes"
labels: type:feature, area:upgrade, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on upgrade

### Description
upgrade state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever upgrade state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/upgrade-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(upgrade): emit state-change event`

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
title: "Add boundary tests for the upgrade logic"
labels: type:test, area:upgrade, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test upgrade

### Description
upgrade's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for upgrade at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/upgrade-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(upgrade): add boundary tests`

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
title: "Extract the repeated upgrade check into a helper"
labels: type:refactor, area:upgrade, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for upgrade

### Description
upgrade repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract upgrade's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/upgrade-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(upgrade): extract shared check`

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
title: "Document the upgrade model and its invariants"
labels: type:docs, area:upgrade, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document upgrade

### Description
upgrade's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/upgrade.md` describing the upgrade model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/upgrade-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(upgrade): document model and invariants`

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
title: "Add a read-only view exposing the current config state"
labels: type:feature, area:config, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose config

### Description
Callers can't read config state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current config state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/config-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(config): add read-only view`

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
title: "Emit a dedicated event when config state changes"
labels: type:feature, area:config, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on config

### Description
config state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever config state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/config-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(config): emit state-change event`

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
title: "Add boundary tests for the config logic"
labels: type:test, area:config, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test config

### Description
config's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for config at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/config-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(config): add boundary tests`

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
title: "Extract the repeated config check into a helper"
labels: type:refactor, area:config, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for config

### Description
config repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract config's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/config-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(config): extract shared check`

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
title: "Document the config model and its invariants"
labels: type:docs, area:config, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document config

### Description
config's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/config.md` describing the config model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/config-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(config): document model and invariants`

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
title: "Add a read-only view exposing the current dispute state"
labels: type:feature, area:dispute, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Expose dispute

### Description
Callers can't read dispute state. This issue adds a read-only view.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a read-only view returning the current dispute state without mutating storage.
- Return a sane default before init.
- Cover values and pre-init default in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/dispute-91-view`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: values after set, default before init.
- Include the full test output in the PR description.

### Example commit message
`feat(dispute): add read-only view`

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
title: "Emit a dedicated event when dispute state changes"
labels: type:feature, area:dispute, stack:rust, stack:soroban, priority:medium, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Event on dispute

### Description
dispute state changes are silent on-chain. This issue emits an event.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a documented event whenever dispute state changes, with the relevant fields.
- No duplicate emissions.
- Cover topic and payload in tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/dispute-92-event`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: emitted once, payload correct.
- Include the full test output in the PR description.

### Example commit message
`feat(dispute): emit state-change event`

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
title: "Add boundary tests for the dispute logic"
labels: type:test, area:dispute, stack:rust, stack:soroban, priority:high, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Boundary-test dispute

### Description
dispute's boundaries aren't fully tested. This issue adds cases.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add tests for dispute at min, max, zero, and over-limit inputs asserting typed errors where expected.
- Keep runs bounded.
- Note any unguarded boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/dispute-91-boundary`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: min, max, zero, over-limit.
- Include the full test output in the PR description.

### Example commit message
`test(dispute): add boundary tests`

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
title: "Extract the repeated dispute check into a helper"
labels: type:refactor, area:dispute, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Helper for dispute

### Description
dispute repeats an inline check. This issue extracts a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract dispute's repeated check into a helper returning a typed error; reuse at each call site.
- Behaviour identical.
- Tests pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/dispute-91-helper`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: same rejections, tests pass.
- Include the full test output in the PR description.

### Example commit message
`refactor(dispute): extract shared check`

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
title: "Document the dispute model and its invariants"
labels: type:docs, area:dispute, stack:rust, stack:soroban, priority:low, Stellar Wave, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---

## Document dispute

### Description
dispute's model/invariants aren't documented. This issue records them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `docs/dispute.md` describing the dispute model, entrypoints, and invariants.
- Cross-reference the code enforcing them.
- Keep accurate.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/dispute-91-model`
- Implement changes
  - **Write code in:** the relevant module.
  - **Write comprehensive tests in:** cover the new behaviour and edge cases.
- Test and commit

### Test and commit
- Run `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`.
- Cover edge cases: n/a — verify against source.
- Include the full test output in the PR description.

### Example commit message
`docs(dispute): document model and invariants`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.

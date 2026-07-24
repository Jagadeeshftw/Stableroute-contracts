---
type: Feature
title: "Release the reentrancy lock on every panic path in compute_route_fee, not just the success return"
labels: type:security, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Guarantee the reentrancy lock is always released, including on panic

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) calls `Self::enter_nonreentrant(&env)` early, but the matching `Self::exit_nonreentrant(&env)` is never actually called on the return path, and the validation guards (`PairNotRegistered`, `AmountBelowMin`, `AmountAboveMax`, `InsufficientLiquidity`, `RouteCooldownActive`) all `panic_with_error!` while the lock is held. Because Soroban rolls storage back on panic, the lock self-heals for a panicking transaction, but if the lock is ever set and the contract returns *normally* without releasing it (the current success path simply falls through without an `exit_nonreentrant`), the pair becomes permanently un-routable. This issue makes the lock lifecycle explicit and correct on every exit.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `Self::exit_nonreentrant(&env)` call immediately before the final `Self::apply_fee_cap(&env, fee)` return so the success path clears the lock.
- Confirm and document the panic-rollback assumption: every guard that panics under the lock relies on Soroban rolling back the `ReentrancyLock = true` write; add a test that routes twice in a row through the same pair to prove the lock does not stick.
- Do not change any error codes or events; this is a correctness fix for the existing guard.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-46-reentrancy-lock-release`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the missing `exit_nonreentrant` on the `compute_route_fee` success path.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert two back-to-back routes through one pair both succeed and the lock reads `false` after each.
  - **Add documentation:** clarify the lock lifecycle and rollback assumption in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the success-path release.
  - Validate security assumptions: the lock is never left set after a normal return; panics roll back the lock.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: two sequential routes, route after a guard-panic, lock state inspected via `env.as_contract`.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: release reentrancy lock on the compute_route_fee success path`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Resolve the duplicate error code 14 shared by NotAuthorized and TimelockNotElapsed"
labels: type:security, area:errors, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Fix the colliding RouterError discriminants at code 14

### Description
The `RouterError` enum in [`src/lib.rs`](src/lib.rs) declares `TimelockNotElapsed = 14`, yet `set_pair_liquidity` panics with `RouterError::NotAuthorized` and the oracle test `test_random_caller_cannot_update_liquidity` asserts `Error(Contract, #14)` for the unauthorized path — meaning two distinct semantic errors are mapped to the same on-the-wire code 14. A client that maps `#14` to "timelock not elapsed" will mis-report an authorization failure (and vice versa), silently breaking the append-only stability promise the file documents. This issue assigns each error a unique, append-only discriminant and aligns the tests.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Audit every `RouterError` variant referenced in the source (`NotAuthorized`, `ReentrantCall`, `RouteCooldownActive`, `TimelockNotElapsed`) and assign each a distinct discriminant that does not reuse any shipped code.
- Update every `#[should_panic(expected = "Error(Contract, #N)")]` test annotation to the corrected codes.
- Document the corrected code table and reaffirm the append-only rule (new variants take the next free number; never re-map a shipped one).
- Do not change any error *semantics*, only the discriminants and the asserting tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-47-error-code-collision`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — disambiguate the colliding discriminants in `RouterError`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the unauthorized-liquidity path and the timelock path panic with their own distinct codes.
  - **Add documentation:** add the corrected `RouterError` code table to [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on each disambiguated variant.
  - Validate security assumptions: no two variants share a discriminant; client error mapping is unambiguous.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: unauthorized liquidity caller, early timelock accept, reentrant call, cooldown active — each its own code.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: assign unique RouterError discriminants and align test annotations`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Replace the constructor-only init path documentation drift in the DataKey and Paused comments"
labels: type:docs, area:storage, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Align stale DataKey and Paused doc comments with the current implementation

### Description
Several `///` doc comments in [`src/lib.rs`](src/lib.rs) no longer match the code. `DataKey::Admin` is documented as "set once at `init`," but `init` now unconditionally panics and the admin is set by `__constructor`. The `DataKey::Paused` comment claims "No write entrypoint accepts calls until an unpause," yet only `register_pair`, `set_pair_fee_bps`, and `compute_route_fee` actually check the flag — `set_pair_liquidity`, `set_pair_min_amount`, `set_pair_max_amount`, `set_fee_recipient`, `set_oracle`, `set_timelock`, and `unregister_pair` execute freely while paused. The enum doc block also says instance storage has "none yet" despite hot singletons existing. This issue corrects the documentation to reflect reality (without changing behavior).

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Update `DataKey::Admin` to say it is set atomically by `__constructor`, not `init`.
- Correct the `DataKey::Paused` comment to state precisely which entrypoints are gated today, or explicitly mark the broader claim as aspirational and link the hardening issue.
- Refresh the enum-level storage rationale comment to describe the variants that have shipped since (cooldown, metrics, oracle, fee cap, timelock, reentrancy lock).
- Documentation/comment-only; no behavior, event, or error change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-48-doc-comment-drift`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — corrected `///` comments only.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** note the corrected invariants in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) consistency across all `DataKey` variants.
  - Validate security assumptions: documented behavior matches the actual guards in the code.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify each documented claim maps to a real guard or is explicitly marked aspirational.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: align DataKey, Admin, and Paused comments with current code`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Emit a config event from set_pair_min_amount, set_pair_max_amount, and set_fee_recipient"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add the still-missing configuration events to three setters

### Description
While `set_pair_fee_bps` (`fee_set`), `set_pair_liquidity` (`liq_set`), `set_pair_cooldown` (`cd_set`), `set_max_fee_absolute` (`maxfee`), and `set_oracle` (`orac_set`) all publish events in [`src/lib.rs`](src/lib.rs), three sibling admin setters still mutate persistent state silently: `set_pair_min_amount`, `set_pair_max_amount`, and `set_fee_recipient` emit nothing. Off-chain indexers therefore cannot reconstruct the min/max bounds history or fee-recipient changes from the event stream. This issue adds the three missing events to bring every state-changing setter to parity.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `min_set (source, destination, min_amount)`, `max_set (source, destination, max_amount)`, and `recip_set (recipient)` events using the established `symbol_short!` convention.
- Keep every topic symbol ≤ 9 characters (the `symbol_short!` limit).
- Do not change validation, return values, or error codes — events only.
- Match the payload tuple shape of the existing `fee_set` / `liq_set` events so indexers need no special-casing.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-49-missing-config-events`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `env.events().publish(...)` in the three setters.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each setter emits exactly the expected topic and data via `env.events().all()`.
  - **Add documentation:** extend the events reference in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) noting the emitted event on each setter.
  - Validate security assumptions: events leak no secrets; payload shapes match siblings.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: exact topic, data tuple, and event count per setter call.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: emit min_set, max_set, and recip_set configuration events`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Gate set_oracle, set_timelock, set_max_fee_absolute, and set_pair_cooldown behind the pause switch"
labels: type:enhancement, area:pause, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Extend the pause gate to the newer admin setters

### Description
The pause check in [`src/lib.rs`](src/lib.rs) is enforced in `register_pair`, `set_pair_fee_bps`, and `compute_route_fee`, but the admin setters added in later batches — `set_oracle`, `set_timelock`, `set_max_fee_absolute`, `set_pair_cooldown` — were never wired into it and execute freely while the router is paused. During an emergency stop an operator (or a compromised admin key) can still rotate the oracle, change the timelock, alter the fee ceiling, or adjust cooldowns, undermining the "freeze everything" guarantee. This issue brings the newer setters under the pause gate via a shared helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a private `require_not_paused(env)` helper (if not already present from earlier work) and call it at the top of `set_oracle`, `set_timelock`, `set_max_fee_absolute`, and `set_pair_cooldown`.
- Decide and document the exemptions (`pause`, `unpause`, admin-transfer accept) so the policy is explicit.
- Reuse `RouterError::ContractPaused` (#9); do not renumber errors.
- Keep `set_pair_liquidity` policy decision explicit (oracle feed may need to run while paused — document the chosen behavior).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-50-pause-newer-setters`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — pause guards on the four newer setters via the shared helper.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each panics with `ContractPaused` (#9) while paused and works after `unpause`.
  - **Add documentation:** update the "Pause semantics" table in [`README.md`](README.md) to include the newer setters.
  - Include NatSpec-style doc comments (`///`) noting the pause gate.
  - Validate security assumptions: no governance-relevant setter bypasses the pause during an emergency.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: each setter while paused, after unpause, exempt entrypoints still work.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: gate oracle/timelock/fee-cap/cooldown setters behind the pause switch`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a get_pending_admin_with_eta aggregate read returning the pending admin and its timelock eta"
labels: type:feature, area:admin, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Provide a one-call aggregate read of the pending admin handover

### Description
A dashboard watching a queued admin handover in [`src/lib.rs`](src/lib.rs) must call both `get_pending_admin()` and `get_pending_admin_eta()` as two separate ledger reads, and there is no atomic snapshot — between the two calls the handover could be accepted or cancelled, yielding an inconsistent view (pending address present but eta gone, or vice versa). This issue adds a single aggregate read mirroring the `get_pair_info` pattern, returning the pending admin and eta together.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `#[contracttype]` struct (e.g. `PendingAdminInfo { pending: Option<Address>, eta: Option<u64> }`) and a `get_pending_admin_info(env) -> PendingAdminInfo` read entrypoint.
- Read both `DataKey::PendingAdmin` and `DataKey::PendingAdminEta` in one invocation so the snapshot is internally consistent.
- Keep the two existing single-field getters for backward compatibility.
- No write path change; this is a read-only convenience.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-51-pending-admin-aggregate-read`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `PendingAdminInfo` struct and `get_pending_admin_info`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the aggregate matches the individual getters before propose, after propose (with eta), and after cancel/accept.
  - **Add documentation:** document the aggregate read in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) on the new struct and entrypoint.
  - Validate security assumptions: the snapshot is atomic; defaults are `(None, None)` when no transfer is queued.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: no pending, pending with eta, post-accept, post-cancel.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add get_pending_admin_info aggregate read for the handover queue`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Emit an adm_set event when accept_admin_transfer completes and a cancelled event on cancel"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add admin-handover lifecycle events for indexer auditability

### Description
The admin handover flow in [`src/lib.rs`](src/lib.rs) emits a `queued` event on `propose_admin_transfer` and an `executed` event on `accept_admin_transfer`, but `cancel_admin_transfer` emits **nothing** — an off-chain watcher that saw a `queued` event has no on-chain signal that the handover was aborted and will wait indefinitely for an `executed` that never comes. The lifecycle is therefore unobservable end to end. This issue adds a cancellation event and reconciles the handover event naming so the full propose → (cancel | execute) lifecycle is reconstructable from logs.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `cancelled` event (topic ≤ 9 chars, e.g. `adm_canc`) emitted by `cancel_admin_transfer`, carrying the cleared pending admin when one existed.
- Make the cancel event a no-op-safe emission: when nothing was pending, document whether an event is still emitted (recommend emitting only when a pending transfer was actually cleared).
- Do not rename the shipped `queued` / `executed` topics (append-only event stability); only add the cancel event.
- Keep validation, return values, and error codes unchanged.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-52-handover-cancel-event`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — emit the `cancelled` event in `cancel_admin_transfer`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the cancel event fires after a real cancel and is absent on a no-op cancel.
  - **Add documentation:** add the handover events to the events reference in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) noting the emitted event on cancel.
  - Validate security assumptions: the full propose/cancel/execute lifecycle is observable from events.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: cancel after propose, no-op cancel, propose→cancel→re-propose.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: emit a cancelled event when an admin handover is aborted`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Extend PairInfo to surface fee cooldown, route count, and cumulative volume in one read"
labels: type:feature, area:metrics, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add an extended aggregate read for the newer per-pair slots

### Description
`PairInfo` and `get_pair_info` in [`src/lib.rs`](src/lib.rs) predate the per-pair slots added in later batches — `PairCooldown`, `PairRouteCount`, and `PairVolume` — so a dashboard must issue three extra reads (`get_pair_cooldown`, `get_pair_route_count`, `get_pair_volume`) on top of `get_pair_info` to assemble a full pair view. The existing `PairInfo` struct shape is part of the ABI and should stay stable, so this issue adds a *new* extended aggregate read rather than mutating the existing struct.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a new `#[contracttype]` struct (e.g. `PairInfoExt`) carrying the existing `PairInfo` fields plus `cooldown_secs: u64`, `route_count: u64`, `volume: i128`.
- Add `get_pair_info_ext(env, source, destination) -> PairInfoExt` reading every per-pair slot in one shot with the documented defaults (cooldown 0, count 0, volume 0).
- Keep `PairInfo` and `get_pair_info` byte-for-byte unchanged for ABI stability.
- Reuse the default-sentinel conventions already established (min 0, max i128::MAX, liquidity 0, last_route_at 0).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-53-pair-info-ext`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `PairInfoExt` and `get_pair_info_ext`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the extended read matches the individual getters for an unconfigured, a configured, and a routed pair.
  - **Add documentation:** document the extended read in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) on the new struct and entrypoint.
  - Validate security assumptions: defaults match the individual getters; existing `PairInfo` is untouched.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: unconfigured pair defaults, fully configured pair, pair after several routes.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add get_pair_info_ext aggregate read with cooldown, count, and volume`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage for the per-pair cooldown rate limit using ledger timestamp control"
labels: type:test, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the per-pair route cooldown gate end to end

### Description
The cooldown subsystem in [`src/lib.rs`](src/lib.rs) — `set_pair_cooldown`, `get_pair_cooldown`, the `cd_set` event, the `RouteCooldownActive` error, and the `last + cooldown` comparison in `compute_route_fee` — has **no dedicated tests**. The existing suite never sets a non-zero cooldown, so the rate-limit branch (including the "first route always allowed" and "disabled when 0" behaviors) is entirely unverified and a regression would pass CI silently. This issue adds a focused cooldown suite driven by `env.ledger().set_timestamp`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: cooldown defaults to 0 (disabled) and a route succeeds repeatedly; `set_pair_cooldown` round-trips via `get_pair_cooldown` and emits `cd_set`.
- Cover: first route is always allowed; a second route before `last + cooldown` panics with the cooldown error; a route exactly at the boundary and one second after succeed.
- Cover: cooldown is per-pair (one pair's cooldown does not throttle another).
- Use `#[should_panic(expected = "Error(Contract, #N)")]` with the corrected cooldown code and `env.ledger().set_timestamp`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-54-cooldown-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the cooldown suite described above.
  - **Add documentation:** note the cooldown test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no `u64` underflow; disabled cooldown preserves prior behavior; per-pair isolation holds.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: disabled, first route, too-soon, at-boundary, one-second-after, two pairs independent.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover the per-pair route cooldown rate limit`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage for per-pair route count and cumulative volume accumulation"
labels: type:test, area:metrics, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the per-pair metrics counters in compute_route_fee

### Description
`get_pair_route_count` and `get_pair_volume` in [`src/lib.rs`](src/lib.rs), and their `saturating_add` accumulation inside `compute_route_fee` (`PairRouteCount`, `PairVolume`), have **no test coverage**. The suite asserts the global `TotalRoutesAllTime` counter but never checks that per-pair count increments once per route, that volume accumulates the routed `amount`, that both default to 0, or that two pairs accumulate independently. This issue adds focused coverage of the per-pair metrics surface.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: both getters default to 0 for an untouched pair.
- Cover: a single route increments `route_count` by 1 and adds `amount` to `volume`; three routes give count 3 and volume = sum of amounts.
- Cover: two distinct pairs accumulate independently and do not cross-contaminate.
- Cover: `quote_route` does NOT touch either counter (parity with its non-mutating contract).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-55-per-pair-metrics-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the per-pair metrics suite described above.
  - **Add documentation:** note the metrics test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: counters are monotonic; `quote_route` is non-mutating; pairs are isolated.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: zero routes, multiple routes, two pairs, quote non-mutation.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover per-pair route count and cumulative volume`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a reentrancy test using a malicious token mock that attempts to re-enter compute_route_fee"
labels: type:test, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the reentrancy guard against an actual re-entering caller

### Description
The reentrancy guard helpers `enter_nonreentrant` / `exit_nonreentrant` and the `ReentrantCall` error in [`src/lib.rs`](src/lib.rs) are documented as protecting the route path against a "future malicious token callback," but there is **no test** that simulates a re-entrant invocation. The lock is currently exercised only implicitly. This issue adds a test contract that re-enters the router during a guarded call and asserts the second entry is rejected, plus a test proving the lock is released so back-to-back legitimate calls succeed.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a small `#[cfg(test)]` mock contract that, when invoked, calls back into `compute_route_fee` on the router (simulating a token callback re-entry).
- Assert the re-entrant call panics with the `ReentrantCall` error code.
- Assert that after a normal (non-re-entrant) route the lock is `false` so the next route works (guards against a stuck lock).
- Read the `ReentrancyLock` slot via `env.as_contract(&id, ...)` to assert its state directly.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-56-reentrancy-mock-test`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if the guard is found incomplete).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the re-entering mock and the lock-release assertion.
  - **Add documentation:** note the reentrancy test approach in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on the mock and the test.
  - Validate security assumptions: re-entry is rejected; the lock self-clears for sequential calls.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: re-entrant call rejected, lock released on normal exit, two sequential routes.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: prove reentrancy guard rejects a re-entering caller and self-clears`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage for the constructor admin event and __constructor versus init divergence"
labels: type:test, area:lifecycle, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the constructor initialization event and legacy init rejection

### Description
`__constructor` in [`src/lib.rs`](src/lib.rs) sets the admin and publishes an `init` event, while the legacy `init` now unconditionally panics with `AlreadyInitialized` (#1). The existing tests assert the admin is set and that `init` rejects, but never assert that the constructor actually emitted the `init` event with the admin payload — the only on-chain signal that a router was deployed. This issue adds event-level coverage of the constructor and pins the constructor/init divergence.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: registering the contract with `(admin,)` emits exactly one `init` event carrying the admin address (assert via `env.events().all()`).
- Cover: `get_admin()` returns the constructor's admin immediately after deploy with no `init` call.
- Cover: any post-deploy `init(addr)` panics with #1 regardless of `addr` (including the original admin).
- Use a fresh `env.register(StableRouteRouter, (admin,))` rather than a shared helper for the event assertion.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-57-constructor-event-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the constructor event + init-rejection suite described above.
  - **Add documentation:** note the constructor test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: deploy is observable via the `init` event; `init` can never re-seize admin.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: constructor event payload, admin readable post-deploy, init rejects with #1.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover constructor init event and legacy init rejection`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage for set_max_fee_absolute and set_pair_cooldown authorization rejection"
labels: type:test, area:admin, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Extend negative-auth coverage to the newer admin setters

### Description
The negative-authorization suite in [`src/lib.rs`](src/lib.rs) (`mod test_i19_authorization`) asserts that non-admin callers are rejected on `register_pair`, `set_pair_fee_bps`, `pause`, and friends — but it predates the setters added later and never covers `set_max_fee_absolute`, `set_pair_cooldown`, `set_oracle`, or `set_timelock`. These newer admin-gated entrypoints have no proof that a wrong signer is rejected, so a missing `require_admin` on any of them would pass CI. This issue extends the auth matrix to the newer setters.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Using the existing `setup_scoped` pattern (authorize only `init`), add a negative-auth `#[should_panic]` case for `set_max_fee_absolute`, `set_pair_cooldown`, `set_oracle`, and `set_timelock`.
- Add a positive control proving each works with the admin's auth.
- Reuse `MockAuth` / `MockAuthInvoke` exactly as the existing module does.
- No production change expected unless a missing guard is discovered (then open a follow-up).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-58-newer-setter-auth-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — negative-auth cases for the four newer setters plus positive controls.
  - **Add documentation:** extend the auth-testing note in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no newer admin setter succeeds without the admin's auth.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: wrong signer per setter, correct signer succeeds.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: assert non-admin rejection on fee-cap, cooldown, oracle, timelock setters`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a force_admin_transfer escape hatch guarded by the timelock for a single-step recovery"
labels: type:feature, area:admin, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Provide a timelocked single-step admin recovery path

### Description
The two-step handover in [`src/lib.rs`](src/lib.rs) requires the *new* admin to call `accept_admin_transfer` from their own key. If the new admin key is generated but never able to sign (lost during the window, or the operator wants to hand control to a multisig that cannot easily call accept), the handover stalls and the only recourse is `cancel_admin_transfer` and a retry. This issue adds an optional `force_admin_transfer` that lets the current admin complete a handover in one step, but only after the governance timelock has elapsed, preserving the watcher warning window.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `force_admin_transfer(env, new_admin)` (admin-gated) that requires the timelock delay to have elapsed since a prior `propose_admin_transfer` for the same `new_admin`, then writes `Admin = new_admin` directly.
- Reuse `PendingAdminEta` / `TimelockNotElapsed`; reject if no matching proposal is queued or the eta has not passed.
- Emit the existing `executed` event so the handover is indistinguishable to indexers.
- Document the trade-off: this removes the new admin's accept step but keeps the timelock as the safety mechanism.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-59-force-admin-transfer`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `force_admin_transfer` reusing the timelock machinery.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert force before eta is rejected, force after eta succeeds, and it requires admin auth.
  - **Add documentation:** document the recovery path and its timelock dependency in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: force still honors the timelock; no instant single-step takeover.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: no proposal queued, force before eta, force after eta, unauthorized caller.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add timelocked force_admin_transfer recovery path with tests`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a remove_oracle entrypoint to revoke the scoped liquidity oracle role"
labels: type:feature, area:access-control, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Allow the admin to revoke the liquidity oracle

### Description
`set_oracle` in [`src/lib.rs`](src/lib.rs) can set or rotate the scoped oracle, but there is **no way to remove it** — once an oracle is configured, `set_pair_liquidity` permanently accepts that address in addition to the admin. If the oracle key is compromised, the admin can only rotate it to a new address (which still leaves *an* oracle authorized); they cannot return to an admin-only liquidity feed. This issue adds `remove_oracle` to clear the `DataKey::Oracle` slot.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `remove_oracle(env)` (admin-gated) that removes `DataKey::Oracle` and emits an `orac_rm` event (≤ 9 chars).
- After removal, `set_pair_liquidity` must accept only the admin again (the dual-auth `Some(caller) != oracle` check naturally degrades to admin-only when the slot is absent — verify and document this).
- Make removal idempotent (removing when none is set is a clean no-op).
- Keep `RouterError` append-only; reuse `NotInitialized` for the missing-admin path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-60-remove-oracle`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `remove_oracle` and the `orac_rm` event.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert oracle can update before removal, cannot after removal, admin still can, and removal is idempotent.
  - **Add documentation:** document oracle revocation in the "Roles & least privilege" section of [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: after removal only admin updates liquidity; compromised oracle can be fully revoked.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: remove when set, remove when unset, oracle blocked after removal, admin override.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add remove_oracle to revoke the scoped liquidity oracle role`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a clear_max_fee_absolute entrypoint so the absolute fee cap can be removed, not just lowered"
labels: type:feature, area:fees, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Allow removing the absolute fee ceiling

### Description
`set_max_fee_absolute` in [`src/lib.rs`](src/lib.rs) can set the absolute per-route fee ceiling but cannot *unset* it: once a cap is written to `DataKey::MaxFeeAbsolute`, the only way to "remove" it is to set an impractically large value, because passing `0` makes every route free and there is no sentinel for "no cap." Yet `apply_fee_cap` treats an *absent* slot as "no cap" — so the capability exists in the read path but is unreachable from the write path. This issue adds an entrypoint to clear the cap and restore the unbounded (bps-only) behavior.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `clear_max_fee_absolute(env)` (admin-gated) that removes `DataKey::MaxFeeAbsolute` and emits a `maxfee_rm` event (≤ 9 chars).
- After clearing, `get_max_fee_absolute` returns `None` and `apply_fee_cap` applies only the `MAX_FEE_BPS` relative bound.
- Make clearing idempotent (no-op when no cap is set).
- Keep `set_max_fee_absolute` and its `0`-means-free semantics unchanged.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-61-clear-fee-cap`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `clear_max_fee_absolute` and the `maxfee_rm` event.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert cap applies, then clear restores unbounded behavior, getter returns `None`, and clearing is idempotent.
  - **Add documentation:** document cap removal versus a zero cap in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: distinction between "cap = 0" (free) and "no cap" (bps-only) is preserved.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: set then clear, clear when unset, zero cap versus cleared cap.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add clear_max_fee_absolute to remove the absolute fee ceiling`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Reset PairRouteCount, PairVolume, and PairLastRouteAt when a pair is unregistered then re-registered"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Decide and enforce metric lifecycle on pair re-registration

### Description
`unregister_pair` in [`src/lib.rs`](src/lib.rs) removes only the `Pair` boolean slot and leaves `PairRouteCount`, `PairVolume`, `PairLastRouteAt`, `PairFeeBps`, `PairCooldown`, and the bounds slots intact. When the same corridor is later re-registered, it silently inherits the old metrics and config — a re-listed pair shows a non-zero lifetime route count and stale volume from its previous life, which is confusing for dashboards and accounting. This issue defines a clear metric lifecycle and offers an explicit reset path on unregister.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `purge_pair_metrics(env, source, destination)` (admin-gated) that removes `PairRouteCount`, `PairVolume`, and `PairLastRouteAt` and emits a `pair_mrst` event (≤ 9 chars).
- Document the intended semantics: by default `unregister_pair` preserves metrics (current behavior); the new entrypoint is the explicit reset.
- Keep `unregister_pair` backward compatible; do not auto-purge unless documented and tested.
- Reuse existing errors; bump no new error code unless strictly required.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-62-pair-metric-reset`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `purge_pair_metrics` and the reset event.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — route a pair, unregister, re-register, assert metrics survive by default, then purge and assert they reset to 0/None.
  - **Add documentation:** document the metric lifecycle in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: only admin can reset; no stale metric leaks into a re-listed pair after purge.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: metrics survive unregister, purge clears them, re-register starts clean after purge.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add purge_pair_metrics to reset per-pair counters on re-listing`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Validate that set_pair_cooldown rejects an absurdly large cooldown that would brick a pair"
labels: type:enhancement, area:routing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Bound the configurable per-pair cooldown to a sane maximum

### Description
`set_pair_cooldown` in [`src/lib.rs`](src/lib.rs) accepts any `u64` seconds value with no upper bound. Because `compute_route_fee` rejects a route until `last + cooldown` is reached, an admin (or a fat-fingered config script) can set a cooldown of, say, `u64::MAX` seconds and permanently brick the corridor after its first route, with no obvious indication of why routing stopped. This issue caps the cooldown at a documented maximum so a clearly-erroneous value is rejected at write time.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a named `MAX_COOLDOWN_SECS` constant (e.g. 30 days) and reject larger values in `set_pair_cooldown` with an append-only `CooldownTooLarge` error.
- Confirm `last + cooldown` cannot overflow `u64` under the new bound (document the headroom).
- Keep `0` (disabled) and all valid values within the cap working exactly as before; keep the `cd_set` event.
- Do not renumber existing errors.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-64-cooldown-upper-bound`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `MAX_COOLDOWN_SECS`, the cap check, and `CooldownTooLarge`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert at-cap accepted, above-cap rejected, 0 still disables, no overflow on the boundary comparison.
  - **Add documentation:** document the cooldown bound in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the constant and setter.
  - Validate security assumptions: no corridor can be silently bricked by an out-of-range cooldown; no `u64` overflow.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: at-cap, above-cap, zero, boundary timestamp comparison.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: bound set_pair_cooldown with MAX_COOLDOWN_SECS and a typed error`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Refactor the duplicated paused-flag check in register_pair and set_pair_fee_bps into a helper"
labels: type:refactor, area:pause, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Extract the inline pause check into a shared require_not_paused helper

### Description
The exact five-line block that reads `DataKey::Paused`, `unwrap_or(false)`, and `panic_with_error!(ContractPaused)` is copy-pasted inline at the top of `register_pair`, `set_pair_fee_bps`, and `compute_route_fee` in [`src/lib.rs`](src/lib.rs). This mirrors the `require_admin` duplication that was already refactored: a missed or subtly-different paste is a real correctness risk, and adding the gate to more entrypoints (see the pause-coverage work) would multiply the duplication. This issue centralizes the check into one private helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a private `fn require_not_paused(env: &Env)` that reads the flag and panics with `ContractPaused` (#9) when set.
- Replace the three inline blocks with a single call each; keep the call ordering identical (e.g. pause check before `require_admin` where it currently is).
- Behavior-preserving refactor: no change to events, errors, return values; all existing tests pass unchanged.
- Keep the helper private so it does not appear in the generated client ABI.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-65-require-not-paused-helper`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the `require_not_paused` helper and the three call-site replacements.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — confirm existing pause tests pass; add a case asserting the gate still fires post-refactor.
  - **Add documentation:** note the helper convention in [`CONTRIBUTING.md`](CONTRIBUTING.md).
  - Include NatSpec-style doc comments (`///`) on the helper.
  - Validate security assumptions: every previously-gated entrypoint is still gated identically.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: each gated entrypoint while paused, after unpause, ordering vs admin check.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`refactor: extract require_not_paused helper to dedupe the pause check`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Extract a pair_key set of private helpers to build per-pair DataKeys without repetitive cloning"
labels: type:refactor, area:storage, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Reduce the per-pair DataKey/clone boilerplate in compute_route_fee

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) constructs the same `(source, destination)` `DataKey` variants repeatedly, each requiring `source.clone()` / `destination.clone()` — the function clones the two symbols roughly a dozen times to build `Pair`, `PairMinAmount`, `PairMaxAmount`, `PairLiquidity`, `PairCooldown`, `PairLastRouteAt`, `PairRouteCount`, `PairVolume`, and `PairFeeBps` keys. This is noisy, error-prone (a wrong clone pairing would query the wrong slot), and obscures the logic. This issue introduces small private helpers to read each per-pair slot with its default in one expression.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add private getter helpers (e.g. `pair_liquidity(env, &s, &d) -> i128`, `pair_min(...)`, `pair_max(...)`, `pair_fee_bps(...)`, `pair_cooldown(...)`) that encapsulate the storage read and the documented default sentinel.
- Refactor `compute_route_fee`, `quote_route`, and the public getters to use them, reducing duplicated clones and centralizing the default sentinels in one place.
- Behavior-preserving: identical defaults, identical results; all existing tests pass unchanged.
- Keep helpers private so they do not bloat the client ABI.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-66-pair-slot-helpers`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the private per-pair read helpers and refactored call sites.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — confirm existing tests pass; add a default-sentinel parity check between the helpers and the public getters.
  - **Add documentation:** note the helper pattern in [`CONTRIBUTING.md`](CONTRIBUTING.md).
  - Include NatSpec-style doc comments (`///`) documenting each helper's default.
  - Validate security assumptions: the default sentinels are now defined exactly once and match the prior `unwrap_or` values.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: each slot's default via the helper equals the prior inline default; routing results unchanged.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`refactor: add private per-pair slot read helpers to dedupe key construction`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Consolidate the duplicated test setup helpers scattered across the test modules"
labels: type:refactor, area:testing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Unify the redundant per-module test setup functions

### Description
The test code in [`src/lib.rs`](src/lib.rs) defines the same setup logic many times: `setup_initialized`, `setup_initialized_with_id`, `setup_pair_with_fee`, `setup_routable_pair`, plus a near-identical private `setup` / `setup_pair` in each of `test_i14_pause_gating`, `test_i15_bounds_liquidity`, `test_i16_fee_arithmetic`, `test_i17_migration`, `test_i18_read_surface`, and `test_i41_fee_cap` — most just `mock_all_auths` + `register` + `init`. This duplication is hard to maintain (the constructor/init divergence already left some of them calling the now-panicking `init`). This issue consolidates the helpers into one shared test-support module.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Create a single `#[cfg(test)]` helper (e.g. a `test_support` submodule) exposing `deploy()`, `deploy_with_pair()`, and `deploy_routable_pair()` and have every test module import from it.
- Fix any helper still calling the legacy `init` (which now panics) to use the constructor path `env.register(StableRouteRouter, (admin,))`.
- Test-only refactor: no production code change; the full suite must pass unchanged.
- Keep test intent identical — only the setup plumbing is unified.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-67-unify-test-helpers`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change; only test-support consolidation.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — migrate existing modules to the shared helpers; ensure every test still passes.
  - **Add documentation:** note the shared test-support convention in [`CONTRIBUTING.md`](CONTRIBUTING.md).
  - Include NatSpec-style doc comments (`///`) on the shared helpers.
  - Validate security assumptions: no test weakens its assertions; auth/constructor setup is consistent everywhere.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: every migrated module compiles and passes; no helper calls the panicking `init`.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`refactor: consolidate duplicated test setup helpers into a shared module`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Migrate deprecated env.events().publish calls to the contractevent macro"
labels: type:refactor, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Replace the deprecated event API flagged by the crate-level allow

### Description
The very first line of [`src/lib.rs`](src/lib.rs) is `#![allow(deprecated)] // TODO: migrate Soroban events to #[contractevent].` — the contract suppresses deprecation warnings precisely because every `env.events().publish((symbol_short!(...),), data)` call uses the older, now-deprecated event API. This crate-wide allow also masks any *other* deprecation, which is a maintenance hazard. This issue performs the TODO: migrate to the typed `#[contractevent]` macro and remove the blanket allow.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Define typed event structs with `#[contractevent]` for each emitted topic (`init`, `pair_reg`, `unreg`, `fee_set`, `liq_set`, `cd_set`, `maxfee`, `orac_set`, `queued`, `executed`, `paused`, `route`) preserving the exact topic symbols and payload shapes for indexer compatibility.
- Replace every `env.events().publish(...)` with the macro-generated emit.
- Remove the crate-level `#![allow(deprecated)]` once no deprecated API remains.
- Keep event topics and data byte-compatible so existing off-chain consumers and the `route_event_payloads` test helper keep working.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-69-contractevent-migration`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `#[contractevent]` structs, migrated emits, and removal of the blanket allow.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each migrated event still decodes to the same topic and payload as before.
  - **Add documentation:** note the event API in [`docs/abi.md`](docs/abi.md) and [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on each event struct.
  - Validate security assumptions: topics/payloads are byte-compatible; no deprecated API remains; build is warning-clean.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`, and `cargo clippy --all-targets -- -D warnings`.
- Cover edge cases and failure paths: each event topic + payload preserved; no deprecation warnings.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`refactor: migrate events to #[contractevent] and drop the deprecated allow`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a get_router_stats aggregate read returning global route count and schema version"
labels: type:feature, area:metrics, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Provide a single global-stats read for dashboards

### Description
A monitoring dashboard for the router in [`src/lib.rs`](src/lib.rs) must currently issue several separate reads to learn the protocol-wide state: `get_total_routes_all_time`, `get_schema_version`, `is_paused`, `get_fee_recipient`, `get_oracle`, and `get_timelock`. There is no single-call snapshot of the global (non-pair) state, so the dashboard view can be internally inconsistent across calls. This issue adds one aggregate read mirroring the `get_pair_info` pattern at the protocol level.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `#[contracttype]` struct (e.g. `RouterStats { total_routes: u64, schema_version: u32, paused: bool, timelock: u64, has_oracle: bool, has_fee_recipient: bool }`) and a `get_router_stats(env) -> RouterStats` read entrypoint.
- Read all global slots in one invocation so the snapshot is internally consistent.
- Avoid returning the actual oracle/recipient addresses if privacy is a concern — booleans suffice; document the choice (or include the addresses if preferred).
- Keep all existing single-field getters for backward compatibility.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-70-router-stats`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `RouterStats` and `get_router_stats`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the aggregate matches the individual getters on a fresh, a configured, and a routed/paused contract.
  - **Add documentation:** document the stats read in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) on the struct and entrypoint.
  - Validate security assumptions: the snapshot is atomic; defaults match the individual getters.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: fresh contract defaults, after routes, while paused, with oracle/recipient set.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add get_router_stats aggregate read for global protocol state`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage for the absolute fee cap composed with the per-pair cooldown and bounds"
labels: type:test, area:fees, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test interactions between the fee cap, cooldown, and amount guards

### Description
The fee-cap suite `test_i41_fee_cap` in [`src/lib.rs`](src/lib.rs) tests `apply_fee_cap` in isolation, and the bounds suite tests min/max/liquidity in isolation, but no test exercises their **composition** in a single `compute_route_fee` call: e.g. a route that passes min/max/liquidity, is throttled by a cooldown on the second call, and whose fee is clamped by the absolute cap. Order-dependent bugs (cap applied before vs after a guard short-circuits, cooldown checked before effects) are exactly what isolated tests miss. This issue adds integration tests across these features.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: a pair configured with a fee, an absolute cap, min/max bounds, liquidity, and a cooldown — assert one route returns the clamped fee and the second within the cooldown is rejected.
- Cover: a below-min / above-max / over-liquidity amount is rejected *before* any counter, volume, or timestamp is written (assert metrics unchanged after a rejected route).
- Cover: `quote_route` reports the capped fee identically while ignoring cooldown/counter side effects.
- Use `env.ledger().set_timestamp` to drive the cooldown boundary.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-71-feature-composition-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a bug is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the composition suite described above.
  - **Add documentation:** note the integration test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: guards short-circuit before effects; cap and cooldown compose correctly.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: capped fee + cooldown reject, rejected route leaves metrics untouched, quote parity under cap.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover fee cap, cooldown, and bounds composition in compute_route_fee`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Assert that a rejected compute_route_fee leaves the route counter, volume, and timestamp unchanged"
labels: type:test, area:routing, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Prove the checks-effects ordering: failed routes mutate no state

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) is documented as following checks-effects-interactions: it validates (pair registered, min, max, liquidity, cooldown) before writing the counter, volume, and timestamp. Because Soroban rolls back state on panic, a rejected route must leave `TotalRoutesAllTime`, `PairRouteCount`, `PairVolume`, and `PairLastRouteAt` exactly as they were. No test currently asserts this rollback property — a future refactor that moved an effect *before* a guard would silently corrupt the metrics. This issue pins the property.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: record the metrics, attempt a route that panics on each guard (#5 unregistered, #6 zero, #10 below min, #11 above max, #12 over liquidity, cooldown), and assert all four metrics are unchanged afterward.
- Use `std::panic::catch_unwind` or per-case `#[should_panic]` tests paired with a separate read of the metrics on a fresh pair to confirm no increment happened.
- Cover the global counter and the per-pair count/volume/timestamp together.
- No production change expected unless the property fails (then open a follow-up bug).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-72-failed-route-no-mutation`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a bug is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the rollback-property suite described above.
  - **Add documentation:** note the checks-effects test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no failed route advances any counter or timestamp.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: each guard's rejection leaves global and per-pair metrics untouched.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: assert rejected routes leave counters, volume, and timestamp unchanged`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a quote_route_checked variant that also reports whether a route would be cooldown-blocked"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Surface cooldown state in the planner-only quote path

### Description
`quote_route` in [`src/lib.rs`](src/lib.rs) returns `(fee, net)` and is documented as a planner-only hook, but it ignores the per-pair cooldown entirely — a client can get a perfectly valid quote and then have the real `compute_route_fee` rejected with `RouteCooldownActive` because the cooldown window has not elapsed. The planner therefore cannot predict a guaranteed failure. This issue adds a quote variant that also reports whether the route would currently clear the cooldown, so off-chain planners can avoid doomed submissions.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `quote_route_checked(env, source, destination, amount) -> (i128, i128, bool)` returning `(fee, net, routable_now)` where `routable_now` is `false` when a non-zero cooldown has not yet elapsed since `PairLastRouteAt`.
- Reuse the exact cooldown comparison logic from `compute_route_fee` (factor it into a shared private helper to avoid drift).
- Keep `quote_route` unchanged and non-mutating; the checked variant is purely additive and also non-mutating.
- `routable_now` reflects only the cooldown, not pause; document that distinction.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-73-quote-route-checked`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `quote_route_checked` plus a shared cooldown-eligibility helper.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert `routable_now` is true with no cooldown, false within the window, and true after it elapses, with the fee/net matching `quote_route`.
  - **Add documentation:** document the planner cooldown signal in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: the checked variant is non-mutating; cooldown logic is shared with `compute_route_fee`.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: no cooldown, within cooldown, at boundary, after cooldown; fee parity with quote_route.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add quote_route_checked reporting cooldown routability`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Reject set_pair_fee_bps, bounds, and liquidity writes for a pair that equals source == destination"
labels: type:enhancement, area:pairs, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Apply the identity-pair rejection consistently across config setters

### Description
`register_pair` in [`src/lib.rs`](src/lib.rs) rejects `source == destination` with `SourceEqualsDestination` (#3), but the config setters — `set_pair_fee_bps`, `set_pair_min_amount`, `set_pair_max_amount`, `set_pair_liquidity`, `set_pair_cooldown` — accept an identity pair and happily write orphan slots for a corridor that can never be registered. `quote_route` and `compute_route_fee` also never explicitly reject an identity pair (they fail later on `PairNotRegistered`), so the identity invariant is enforced in exactly one place. This issue applies the rejection consistently.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add the `source == destination` check (reusing `SourceEqualsDestination` #3) to each per-pair config setter so an identity pair cannot accumulate config slots.
- Decide and document whether `quote_route` / `compute_route_fee` should reject identity early with #3 or keep failing with #5 (`PairNotRegistered`) — pick one and make it consistent.
- Preserve all existing sign/cap validation and events.
- Do not renumber errors.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-74-identity-pair-consistency`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — identity-pair guard in the config setters (and optionally the route path).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each setter rejects an identity pair with #3 and accepts a distinct pair.
  - **Add documentation:** document the identity-pair invariant across setters in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the affected setters.
  - Validate security assumptions: no identity-pair config can be created; the invariant is enforced uniformly.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: identity set attempt per setter, distinct-pair set succeeds.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: reject identity pairs consistently across config setters`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a downgrade guard so set_max_fee_absolute cannot silently widen an active absolute cap mid-flight"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Emit an explicit signal when the absolute fee cap is loosened

### Description
`set_max_fee_absolute` in [`src/lib.rs`](src/lib.rs) lets the admin raise or lower the absolute fee ceiling with no distinction between *tightening* (lowering, user-protective) and *loosening* (raising, extraction-enabling) the cap. A compromised or careless admin can quietly widen the cap and immediately charge much larger fees, with the `maxfee` event being the only signal — and an indexer cannot tell from the event alone whether the change increased risk to users. This issue makes a cap *increase* explicit and auditable.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- In `set_max_fee_absolute`, read the prior cap and emit a distinct event (e.g. `maxfee_up` vs `maxfee`) — or include a direction/old-value field in the payload — when the new cap is higher than the previous one.
- Optionally require the new cap increase to respect the governance timelock (queue-and-execute) — document the chosen policy; at minimum make the increase observable.
- Keep lowering (tightening) instant and backward compatible.
- Do not renumber errors; topics must stay ≤ 9 chars.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-75-fee-cap-loosen-signal`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — old-value read and the loosening signal in `set_max_fee_absolute`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert a cap raise emits the loosening signal, a cap lower does not, and the cap value round-trips.
  - **Add documentation:** document the loosening signal and its rationale in [`README.md`](README.md) / [`SECURITY.md`](SECURITY.md).
  - Include NatSpec-style doc comments (`///`) on the setter.
  - Validate security assumptions: a cap increase is always observable on-chain; tightening stays frictionless.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: raise from set, lower from set, set from unset, equal value.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: emit an explicit signal when the absolute fee cap is loosened`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Document the constructor-based deployment and the legacy init no-op in the README quickstart"
labels: type:docs, area:readme, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document deploying with the constructor and why init now panics

### Description
The router in [`src/lib.rs`](src/lib.rs) was hardened to set the admin via `__constructor` (`register(StableRouteRouter, (admin,))`), and the legacy `init` now unconditionally panics with `AlreadyInitialized`. Anyone following an older deployment guide that calls `init` after deploy will hit a confusing #1 panic. The README's quickstart predates this change and does not explain the constructor argument or the init deprecation. This issue documents the current, correct deployment flow.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a "Deploying" section to [`README.md`](README.md) showing the Soroban CLI `--admin` / constructor-arg invocation that sets the admin atomically.
- Explain that `init` is retained only for ABI compatibility and always panics #1 — never call it post-deploy.
- Cross-reference the front-running rationale (no deployed-but-uninitialized window).
- No production code change; optionally tighten the `///` on `init` / `__constructor` to match.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-76-constructor-deploy-guide`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; optionally align `init` / `__constructor` doc comments.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** the "Deploying" section in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) consistency on the lifecycle entrypoints.
  - Validate security assumptions: the documented flow matches the constructor-only initialization.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify the documented constructor signature matches `__constructor` and that `init` is described as always-panicking.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: document constructor deployment and the legacy init no-op`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Document the governance timelock model and recommended delay in a GOVERNANCE.md"
labels: type:docs, area:admin, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Write a governance reference covering the timelock and admin lifecycle

### Description
[`src/lib.rs`](src/lib.rs) implements a non-trivial governance surface — two-step admin handover with `propose`/`accept`/`cancel`, a configurable `Timelock` that stamps `PendingAdminEta`, `queued`/`executed` events, the scoped `Oracle` role, and the `pause`/`unpause` emergency stop — but there is no single document explaining how these compose for an operator. A new admin has no guidance on setting a sane timelock, what a watcher should monitor, or the exact accept-after-eta semantics. This issue produces a GOVERNANCE.md.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Create `GOVERNANCE.md` (linked from [`README.md`](README.md)) covering: the admin handover state machine, the timelock semantics (eta = propose time + delay, accept blocked until then), the oracle role's least-privilege scope, and the pause emergency stop.
- Include a recommended timelock range and the events a watcher should subscribe to (`queued`, `executed`, `paused`, `orac_set`).
- Reference the concrete entrypoints, `DataKey`s, and the `TimelockNotElapsed` error.
- No production code change; optionally align `///` comments.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-77-governance-md`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; optionally cross-link doc comments to GOVERNANCE.md.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** create `GOVERNANCE.md` and link it from [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) consistency on governance entrypoints.
  - Validate security assumptions: the documented model exactly matches the timelock and handover code.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: every documented state transition maps to a real entrypoint and event.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add GOVERNANCE.md covering timelock and admin lifecycle`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Update docs/abi.md and docs/storage.md to cover the cooldown, metrics, oracle, fee-cap, and timelock additions"
labels: type:docs, area:storage, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Bring the ABI and storage references up to date with the newer slots

### Description
The repository already ships `docs/abi.md` and `docs/storage.md`, but [`src/lib.rs`](src/lib.rs) has since grown several `DataKey` variants and entrypoints that those references almost certainly predate: `PairCooldown`, `PairRouteCount`, `PairVolume`, `Timelock`, `PendingAdminEta`, plus the `Oracle`, `MaxFeeAbsolute`, and `ReentrancyLock` slots, and entrypoints like `set_pair_cooldown`, `get_pair_route_count`, `set_oracle`, `set_max_fee_absolute`, and `set_timelock`. A reference that omits half the storage and ABI is worse than none. This issue reconciles both docs with the current source.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Audit every `DataKey` variant and public entrypoint in `src/lib.rs` against `docs/storage.md` and `docs/abi.md` and add any missing rows.
- For each new `DataKey`: key shape, value type, storage tier, default-when-absent, reading/writing entrypoints, TTL class.
- For each new entrypoint: signature, auth requirement, errors raised, event emitted.
- Add the newer events (`cd_set`, `maxfee`, `orac_set`, `queued`, `executed`) to the event catalog.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-78-abi-storage-refresh`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; optionally align stale `///` comments.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** update [`docs/abi.md`](docs/abi.md) and [`docs/storage.md`](docs/storage.md).
  - Include NatSpec-style doc comments (`///`) consistency across the newer variants.
  - Validate security assumptions: documented defaults match the `unwrap_or` values; no entrypoint or DataKey is omitted.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: every `DataKey` and public entrypoint appears exactly once in the refreshed docs.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: refresh abi and storage references for cooldown/metrics/oracle/timelock`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a migrate_v2_to_v3 scaffold and generalize the migration guard beyond a hardcoded v1 check"
labels: type:feature, area:migration, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Generalize the schema migration path for future versions

### Description
`migrate_v1_to_v2` in [`src/lib.rs`](src/lib.rs) hardcodes `if current != 1 { panic MigrationVersionMismatch }` and stamps `2`. There is no path to v3, and the next migration would either copy-paste the same pattern (duplication, easy to mis-number) or have to retrofit a generic stepper. Since the storage surface is actively growing (new `DataKey`s per batch), a clean, version-checked migration framework is needed before the next schema bump. This issue adds a v2→v3 scaffold and a reusable migration guard.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a private `require_schema(env, expected: u32)` helper that loads `SchemaVersion` (default 1) and panics with `MigrationVersionMismatch` unless it equals `expected`, then refactor `migrate_v1_to_v2` to use it.
- Add `migrate_v2_to_v3(env)` (admin-gated) that requires schema 2, performs any needed backfill (none yet — document that v3 readers default sensibly), and stamps 3.
- Keep `migrate_v1_to_v2` behavior identical; the new helper must not change existing test outcomes.
- Document the step-by-step migration policy (no skipping versions).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-81-migrate-v2-v3`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `require_schema` helper, refactored v1→v2, and `migrate_v2_to_v3`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert v1→v2→v3 stepping works, skipping v2→v3 from v1 is rejected, double-migrate is rejected, and admin auth is required.
  - **Add documentation:** document the migration framework in [`README.md`](README.md) / `CHANGELOG.md`.
  - Include NatSpec-style doc comments (`///`) on the helper and new entrypoint.
  - Validate security assumptions: only admin migrates; versions cannot be skipped or repeated.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: step-by-step migrate, skip rejected, double migrate rejected, pre-init migrate.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add migrate_v2_to_v3 scaffold and a reusable schema guard`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage proving the timelock applies to the eta stamped at propose time, not accept time"
labels: type:test, area:admin, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the timelock eta-stamping and mid-flight delay-change semantics

### Description
`propose_admin_transfer` in [`src/lib.rs`](src/lib.rs) stamps `PendingAdminEta = now + Timelock` at propose time, and the `set_timelock` doc comment explicitly states the change "applies to the next `propose_admin_transfer`; already-queued actions keep the eta they were stamped with." The existing timelock tests cover early/after-delay accept and cancel, but none proves the eta is frozen at propose time — i.e. that *changing the timelock after a proposal* does not move the already-queued eta. This subtle invariant is the crux of the warning-window guarantee and is untested.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: set timelock 100, propose at t=1000 (eta 1100), then `set_timelock(10)` — assert `get_pending_admin_eta` is still 1100 and accept at t=1050 is still rejected with `TimelockNotElapsed`.
- Cover: raising the timelock after a proposal likewise does not extend the queued eta.
- Cover: a fresh proposal after the timelock change uses the new delay.
- Use `env.ledger().set_timestamp` and the corrected `TimelockNotElapsed` code.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-82-timelock-eta-frozen`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if the invariant fails).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the eta-frozen suite described above.
  - **Add documentation:** note the timelock eta semantics in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: a queued handover's eta cannot be shortened by lowering the timelock mid-flight.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: lower delay after propose, raise delay after propose, fresh propose uses new delay.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: prove the timelock eta is frozen at propose time`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Allow propose_admin_transfer to reject re-proposing while a handover is already queued"
labels: type:enhancement, area:admin, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Guard against silently overwriting a queued admin handover

### Description
`propose_admin_transfer` in [`src/lib.rs`](src/lib.rs) overwrites `PendingAdmin` and re-stamps `PendingAdminEta` every time it is called, with no check for an already-queued transfer. This means an admin can silently replace a pending handover — and, more importantly, re-proposing the *same* new admin resets the eta, effectively letting the admin extend (or, by lowering the timelock first, the next proposal shortens) the window, weakening the watcher guarantee. This issue makes re-proposing explicit: either reject while one is queued, or require an explicit cancel first.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- In `propose_admin_transfer`, if a `PendingAdmin` is already set, reject with an append-only `TransferAlreadyQueued` error (or document a deliberate overwrite policy and emit a distinct `requeued` event).
- The admin must `cancel_admin_transfer` before proposing a different/refreshed handover, making the re-stamp explicit and auditable.
- Keep the `queued` event and the eta-stamping behavior for the first proposal unchanged.
- Do not renumber existing errors.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-83-reject-double-propose`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the already-queued guard and `TransferAlreadyQueued`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert a second propose while queued is rejected, and propose works again after cancel.
  - **Add documentation:** document the no-silent-overwrite policy in [`README.md`](README.md) / GOVERNANCE docs.
  - Include NatSpec-style doc comments (`///`) on the changed entrypoint.
  - Validate security assumptions: a queued handover's eta cannot be silently reset by a repeat propose.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: propose while queued, propose after cancel, propose after accept.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: reject re-proposing an admin transfer while one is already queued`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a self-transfer guard so an admin cannot propose itself as the new admin"
labels: type:enhancement, area:admin, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Reject a no-op admin handover that proposes the current admin

### Description
`propose_admin_transfer` in [`src/lib.rs`](src/lib.rs) accepts any `new_admin`, including the *current* admin's own address. Proposing yourself queues a meaningless handover, emits a misleading `queued` event, stamps a `PendingAdminEta`, and lets the admin "accept" their own role back — wasting storage, polluting the event stream for watchers, and obscuring genuine handovers. This issue rejects a self-targeted proposal as a clear configuration error.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- In `propose_admin_transfer`, compare `new_admin` against the current `DataKey::Admin` and reject equality with an append-only `SelfTransferNotAllowed` error (or reuse a suitable existing semantic if one fits).
- Keep all other propose behavior (eta stamping, `queued` event) unchanged for a genuine new admin.
- Document the rationale: a self-proposal is always a no-op and is rejected to keep the event stream meaningful.
- Do not renumber existing errors.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-84-reject-self-transfer`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the self-transfer guard and the new error.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert proposing the current admin is rejected and proposing a distinct address still works.
  - **Add documentation:** document the self-transfer rejection in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the changed entrypoint.
  - Validate security assumptions: no meaningless handover can be queued; genuine transfers unaffected.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: propose self rejected, propose distinct succeeds, propose self after rotation.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: reject proposing the current admin as a self-transfer no-op`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a batch get_pair_infos read that returns aggregate info for a Vec of pairs in one call"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Provide a batched pair-info read for dashboards

### Description
`get_pair_info` in [`src/lib.rs`](src/lib.rs) returns the aggregate slot view for a single pair, so a dashboard rendering N corridors must make N separate contract reads — each a round trip — and the resulting view can be inconsistent across calls if config changes mid-render. This issue adds a batched read that takes a `Vec` of `(source, destination)` pairs and returns a `Vec<PairInfo>` in one invocation, reducing round trips and giving an internally consistent snapshot.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `get_pair_infos(env, pairs: Vec<(Symbol, Symbol)>) -> Vec<PairInfo>` reusing the existing single-pair `get_pair_info` logic per element.
- Cap the input length with a named constant to bound gas; reject over-limit input with an append-only `BatchTooLarge` error (or reuse one if a batch error already exists).
- Reuse the existing `PairInfo` struct and its default sentinels; no struct change.
- Read-only; no events, no state change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-85-batch-pair-info-read`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `get_pair_infos`, the batch-size constant, and the over-limit guard.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert the batch result matches per-pair `get_pair_info`, empty input returns empty, and over-limit is rejected.
  - **Add documentation:** document the batched read in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: bounded length; each element matches the single-pair read exactly.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: empty batch, mixed configured/unconfigured pairs, max batch, over-limit.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add batched get_pair_infos read with a bounded length`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a property test asserting the absolute fee cap is never exceeded across the full input space"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Extend the proptest harness to cover the absolute fee cap invariant

### Description
The existing `proptest!` block in [`src/lib.rs`](src/lib.rs) asserts `fee <= amount`, `fee == 0` when `fee_bps == 0`, and quote/compute parity — but it never sets an absolute cap, so the `apply_fee_cap` clamp is outside the generated input space. A bug in the clamp (e.g. applying `min` against a stale/negative value, or clamping before the bps fee) would survive every property test. This issue adds cap-aware invariants so the absolute ceiling is provably honored across the full amount/fee/cap space.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add property tests generating `amount`, `fee_bps in 0..=MAX_FEE_BPS`, and a non-negative `cap`, asserting: the charged fee `<= cap` whenever a cap is set; the fee equals `min(amount*fee_bps/10_000, cap)`; and quote/compute agree under the cap.
- Reuse the existing fixed-seed `ProptestConfig` so CI stays deterministic.
- Keep the existing property tests; these are additive.
- Use wide max-amount and liquidity so boundary guards never pre-empt the arithmetic path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-87-fee-cap-proptest`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if an invariant fails).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the cap-aware proptest invariants described above.
  - **Add documentation:** note the cap invariant in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on the invariant tests.
  - Validate security assumptions: the absolute cap is never exceeded for any valid input; quote/compute parity holds under the cap.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: cap below/above the proportional fee, cap of 0, full amount/bps range, quote parity.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: add cap-aware property invariants for the absolute fee ceiling`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add a route_tag direction-collision property test now that the implementation echoes its inputs"
labels: type:test, area:routing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Strengthen route_tag tests and document its current echo behavior

### Description
`route_tag` in [`src/lib.rs`](src/lib.rs) returns its `(source, destination)` arguments unchanged, yet `test_route_tag` carries comments claiming it tests "the same inputs hash to the same tag" and "collision resistance" — comments that describe a hashing implementation that does not exist. The test currently only proves trivial equality/inequality of the echoed tuple. Until/unless a real hashing `route_tag` lands (tracked separately), the tests and comments should accurately reflect the echo behavior and pin its direction-sensitivity properties. This issue corrects the test narrative and adds property coverage for the echo.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Rewrite the `test_route_tag` comments to state plainly that the tag currently echoes `(source, destination)` (no hashing), removing the misleading "hash"/"collision-resistant" language.
- Add a property test asserting: `route_tag(a, b)` round-trips to `(a, b)`, `route_tag(a, b) != route_tag(b, a)` for `a != b`, and distinct pairs map to distinct tuples — across a generated symbol space.
- If a hashing implementation is introduced elsewhere, this issue's tests should be the natural place to upgrade — note that cross-reference.
- No production change in this issue (test + comment accuracy only).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-88-route-tag-accuracy`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — corrected `test_route_tag` comments only (no production behavior change).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the echo property test described above.
  - **Add documentation:** note `route_tag`'s current echo semantics in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) aligning `route_tag`'s comment with its actual behavior.
  - Validate security assumptions: the tests assert exactly what the implementation does, with no misleading claims.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: round-trip echo, direction flip, distinct pairs, across a generated symbol set.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: correct route_tag comments and add echo direction property test`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add test coverage asserting the oracle cannot set fees, pause, rotate admin, or upgrade"
labels: type:security, area:access-control, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Prove the oracle role's least-privilege scope across all governance entrypoints

### Description
The scoped oracle in [`src/lib.rs`](src/lib.rs) is documented to update pair liquidity "and nothing else." The existing tests prove the oracle *can* update liquidity and *cannot* call `set_oracle`/`pause` (one representative case), but they do not exhaustively prove the oracle is rejected on the *full* set of admin-only entrypoints — `set_pair_fee_bps`, `set_pair_min_amount`, `set_pair_max_amount`, `set_fee_recipient`, `set_timelock`, `set_max_fee_absolute`, `set_pair_cooldown`, `propose_admin_transfer`, `migrate_v1_to_v2`, `register_pair`, `unregister_pair`. A regression that accidentally widened the oracle's `set_pair_liquidity` dual-auth check to another entrypoint would pass CI. This issue makes the least-privilege boundary exhaustively tested.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- For each admin-only entrypoint, authorize only the configured oracle (via `mock_auths`) and assert the call panics (the entrypoint's `require_admin` must reject the oracle).
- Include a positive control proving the oracle still succeeds on `set_pair_liquidity`.
- Reuse the `MockAuth` / `MockAuthInvoke` pattern from the existing oracle and authorization tests.
- No production change expected unless a leak is found (then open a follow-up security fix).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-89-oracle-scope-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a scope leak is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — an oracle-rejection case per admin entrypoint plus the liquidity positive control.
  - **Add documentation:** extend the "Roles & least privilege" coverage note in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: the oracle is rejected on every governance entrypoint; only `set_pair_liquidity` accepts it.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: oracle rejected per admin entrypoint, oracle accepted on liquidity.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: exhaustively assert the oracle's least-privilege boundary`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.
++++++
---
type: Feature
title: "Add an is_routable read that reports whether a pair would pass every compute_route_fee guard for an amount"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Provide a single read that predicts route eligibility for an amount

### Description
`is_pair_active` in [`src/lib.rs`](src/lib.rs) reports only "registered AND liquidity > 0" and ignores the amount, the min/max bounds, the cooldown, and the pause flag. A client wanting to know whether a *specific* `amount` would route right now has to either replicate every guard in `compute_route_fee` off-chain (error-prone, drifts with the contract) or submit and risk a panic. This issue adds a non-mutating predicate that runs the full guard set for a given amount and returns a clear yes/no plus a reason code.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `route_eligibility(env, source, destination, amount) -> u32` (or a small enum/struct) returning 0 for "routable" or the `RouterError` code that *would* be raised (`PairNotRegistered`, `AmountBelowMin`, `AmountAboveMax`, `InsufficientLiquidity`, `RouteCooldownActive`, `ContractPaused`).
- Run the exact same checks as `compute_route_fee`, in the same order, but **never** mutate state or panic — factor the validation into a shared private helper that both call.
- Non-mutating: no counter, volume, timestamp, or event side effects.
- Document that this is a best-effort prediction (state can change before the real route).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-90-route-eligibility`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `route_eligibility` plus a shared validation helper reused by `compute_route_fee`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert it returns 0 for a clean route and the matching code for each guard failure, while leaving all metrics untouched.
  - **Add documentation:** document the eligibility predicate in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: the predicate matches `compute_route_fee`'s guards exactly and never mutates state.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: routable, unregistered, below min, above max, over liquidity, cooldown active, paused; metrics unchanged.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add non-mutating route_eligibility predicate mirroring route guards`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.

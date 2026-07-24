---
type: Feature
title: "Clear orphaned PairFeeBps, PairMinAmount, PairMaxAmount, and PairLiquidity slots on unregister_pair"
labels: type:feature, area:pair-lifecycle, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Clear orphaned config slots when a pair is unregistered

### Description
`unregister_pair` in [`src/lib.rs`](src/lib.rs) only removes the `DataKey::Pair` registration flag. Its own doc comment admits the gap: `PairFeeBps`, `PairMinAmount`, `PairMaxAmount`, and `PairLiquidity` are left **orphaned in storage**, so re-registering the same corridor later silently revives the old fee, bounds, and liquidity instead of starting clean. An operator who unregisters a corridor to retire a bad fee, then re-registers it, gets the stale fee back without any explicit write — a real correctness and safety hazard. This issue makes `unregister_pair` clear every per-pair config slot so re-registration always starts from defaults.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- In `unregister_pair`, after removing `DataKey::Pair`, also `remove` the `PairFeeBps`, `PairMinAmount`, `PairMaxAmount`, and `PairLiquidity` slots for the same `(source, destination)`.
- Note that `PairRouteCount`, `PairVolume`, and `PairLastRouteAt` reset is tracked separately (issue-4); scope this issue strictly to the **config** slots (fee, min, max, liquidity) to avoid overlap.
- Keep `unregister_pair` admin-gated and idempotent; removing absent slots must remain a no-op.
- Extend the existing `unreg` event payload, or emit a companion `cfg_clr` event, so indexers can see the config was wiped.
- Preserve the append-only error policy and the registration-first invariant.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-clear-orphaned-pair-config`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — extend `unregister_pair` to remove the four per-pair config keys; reuse a small private `clear_pair_config` helper to avoid repetition.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — assert that after unregister + re-register, `get_pair_fee_bps`, `get_pair_min_amount`, `get_pair_max_amount`, and `get_pair_liquidity` all return their documented defaults.
  - **Add documentation:** update [`README.md`](README.md) and [`docs/storage.md`](docs/storage.md) to state that unregister clears config; remove the "out of scope" caveat from the doc comment.
  - Include NatSpec-style doc comments (`///`) on the new helper, matching the existing style in `lib.rs`.
  - Validate security assumptions: no orphan revival, admin-only, idempotent on already-clean pairs.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: unregister a never-configured pair, unregister then re-register and confirm clean defaults, and unauthorized caller rejection.
- Include the full `cargo test` output and a short security notes section in the PR description.

### Example commit message
`feat: clear orphaned pair config slots on unregister_pair with tests and docs`

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
title: "Include the deterministic route_tag in the route event payload for off-chain correlation"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Emit the route_tag inside the route event so indexers can correlate without recomputing

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) emits a `route` event carrying `(source, destination, amount)`, while the separate `route_tag` entrypoint computes a deterministic `keccak256` identifier for the same pair. Today an off-chain indexer that wants the canonical route id has to either store a `(source, destination) → tag` mapping or re-run the XDR-encode-and-hash itself for every event. Emitting the `route_tag` directly in the `route` event payload removes that work and guarantees the on-chain and off-chain identifiers can never drift.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Compute the same `keccak256(xdr(source) || xdr(destination))` digest already produced by `route_tag` inside `compute_route_fee`'s EFFECTS block and add it to the `route` event payload (e.g. `(source, destination, amount, tag)`).
- Factor the hashing into a private `compute_route_tag(env, source, destination) -> BytesN<32>` helper so `route_tag` and `compute_route_fee` share one implementation and can never diverge.
- This is distinct from the existing "deterministic keyed route identifier" issue (issues-2), which replaced the placeholder tag; this issue is purely about surfacing that tag in the emitted event.
- Keep the event topic `route` unchanged; extend only the data tuple. Document the payload-shape change as a breaking event-ABI note.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-route-tag-in-event`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the private `compute_route_tag` helper, call it from both `route_tag` and the `route` event publish.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — use `env.events().all()` to assert the emitted tag equals `client.route_tag(source, destination)` and is direction-sensitive.
  - **Add documentation:** update the events reference in [`README.md`](README.md) and [`docs/abi.md`](docs/abi.md) to show the new payload shape.
  - Include NatSpec-style doc comments (`///`) on the new helper, matching the existing style in `lib.rs`.
  - Validate security assumptions: no change to fee math, no extra storage, deterministic output.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: forward vs reverse pair tags differ, repeated routes emit a stable tag, and the tag matches the standalone entrypoint.
- Include the full `cargo test` output and a short security notes section in the PR description.

### Example commit message
`feat: emit route_tag in the route event and share one hashing helper with tests and docs`

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
title: "Remove the duplicate setup_uninitialized test helper defined twice in the test module"
labels: type:refactor, area:test-harness, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Delete the duplicated setup_uninitialized helper in the inline test module

### Description
The inline `#[cfg(test)] mod test` block in [`src/lib.rs`](src/lib.rs) defines the helper `fn setup_uninitialized(env: &Env)` **twice** back to back — once around the "legacy pre-init tests" comment and again under the "Register the contract WITHOUT a constructor call" comment, with identical bodies. Two functions with the same name and signature in the same module is a real maintenance hazard: it is confusing to readers, invites edits to the wrong copy, and any future signature change must be duplicated. This refactor removes the redundant definition and keeps a single, well-documented helper.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Remove one of the two identical `setup_uninitialized` definitions, keeping the one with the clearer doc comment.
- Confirm every existing call site (the `NotInitialized`/pre-init assertions) still resolves to the surviving helper and that the full suite passes unchanged.
- This is a test-only cleanup — no entrypoint, storage, or error-code change. It must not alter coverage of any production path.
- Leave a one-line doc comment explaining when to use `setup_uninitialized` vs `setup_initialized`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-dedupe-setup-uninitialized`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — delete the redundant `setup_uninitialized` definition inside the `#[cfg(test)] mod test` block.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — no new tests required, but confirm the existing uninitialized-path tests still compile and pass against the single helper.
  - **Add documentation:** note the canonical test helpers in [`CONTRIBUTING.md`](CONTRIBUTING.md) so contributors do not re-add a duplicate.
  - Include NatSpec-style doc comments (`///`) on the surviving helper, matching the existing style in `lib.rs`.
  - Validate security assumptions: pure test refactor, no production behaviour change.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Confirm the suite count is unchanged except for the removed duplicate definition and all uninitialized-path tests still pass.
- Include the full `cargo test` output in the PR description.

### Example commit message
`refactor: remove duplicate setup_uninitialized test helper`

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
title: "Reject empty batches in register_pairs and set_pair_fees_bps with a typed EmptyBatch error"
labels: type:enhancement, area:batch, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Reject empty input vectors in the batch entrypoints

### Description
`register_pairs` and `set_pair_fees_bps` in [`src/lib.rs`](src/lib.rs) cap their input vectors at `MAX_BATCH_SIZE` and panic with `BatchTooLarge` when exceeded, but an **empty** `Vec` silently passes admin auth, the pause gate, and the size check, then loops zero times and returns — burning a full admin-signed transaction that does nothing. That is a wasted fee and a confusing no-op that hides caller bugs. This issue adds a typed `EmptyBatch` rejection so a malformed batch fails loudly.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a new append-only `RouterError::EmptyBatch = 19` (next free code after `BatchTooLarge = 18`).
- In both `register_pairs` and `set_pair_fees_bps`, panic with `EmptyBatch` when `entries.len() == 0`, after the pause gate and admin auth but before the size check.
- Keep error codes append-only — do not renumber any existing variant.
- Update the batch entrypoint doc comments to state the non-empty precondition.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-reject-empty-batch`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the `EmptyBatch` variant and the empty-length guards in both batch entrypoints.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, alongside `mod test_batch`) — assert both entrypoints panic with `Error(Contract, #19)` on an empty `Vec`.
  - **Add documentation:** add the new error to the error-code table in [`README.md`](README.md) and [`docs/abi.md`](docs/abi.md).
  - Include NatSpec-style doc comments (`///`) on the updated entrypoints, matching the existing style in `lib.rs`.
  - Validate security assumptions: still admin-gated, still pause-gated, no path that bypasses the new guard.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: empty batch, single-element batch (still succeeds), and oversized batch (still `BatchTooLarge`).
- Include the full `cargo test` output in the PR description.

### Example commit message
`feat: reject empty batches in register_pairs and set_pair_fees_bps with EmptyBatch error`

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
title: "Add a centralized extend_ttl helper for persistent pair slots and apply it on every write"
labels: type:enhancement, area:storage-ttl, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add a reusable persistent-TTL extension helper and call it on writes

### Description
The router stores admin, pair registration, fees, bounds, liquidity, counters, and timestamps in **persistent** storage, but no entrypoint calls `extend_ttl` on those entries. Persistent Soroban entries expire if their TTL is not bumped, after which reads revert to defaults — a registered pair could silently appear unregistered, or a configured fee could read as zero. This issue introduces one private `extend_pair_ttl` (and `extend_instance_ttl`) helper with documented threshold/extend constants and threads it through the hot write paths so long-lived state survives.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `pub const` TTL bump and threshold constants (e.g. `PAIR_TTL_THRESHOLD`, `PAIR_TTL_EXTEND`) with rationale in their doc comments.
- Add a private `extend_pair_ttl(env, key)` helper wrapping `env.storage().persistent().extend_ttl(...)` and call it from `register_pair`, `set_pair_fee_bps`, `set_pair_min_amount`, `set_pair_max_amount`, `set_pair_liquidity`, and the `compute_route_fee` write block.
- This is distinct from the broad issues-1 "bump TTL on every read and write" item: scope this issue to a **single shared helper plus constants** applied to the per-pair write paths, so it can land independently and cleanly.
- Do not change any fee, bounds, or auth semantics.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-extend-ttl-helper`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the constants, the `extend_pair_ttl` helper, and the calls on each write path.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — use `env.ledger()` controls and `env.as_contract` to advance the ledger and assert a written pair still reads correctly past the original TTL window.
  - **Add documentation:** describe the TTL strategy and constants in [`docs/storage.md`](docs/storage.md).
  - Include NatSpec-style doc comments (`///`) on the helper and constants, matching the existing style in `lib.rs`.
  - Validate security assumptions: no state expiry for configured pairs, no auth bypass, no extra cost on read-only paths.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: write then advance ledger past the threshold, repeated writes, and an unconfigured pair (no bump expected).
- Include the full `cargo test` output and a short security notes section in the PR description.

### Example commit message
`feat: add extend_pair_ttl helper and bump persistent TTL on every pair write`

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
title: "Add a set_timelock event and a get_governance_config aggregate read for the timelock state"
labels: type:enhancement, area:governance, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Emit an event on set_timelock and expose a single governance-state read

### Description
`set_timelock` in [`src/lib.rs`](src/lib.rs) silently writes `DataKey::Timelock` with no event, so a governance watcher cannot detect that the handover delay changed without polling `get_timelock`. Every other state-changing governance action (`pause`, `propose_admin_transfer`, `accept_admin_transfer`) emits an event; the timelock setter is the odd one out. This issue adds a `tl_set` event and a `get_governance_config` aggregate read that returns the current timelock, pending admin, and pending eta in one round-trip — distinct from the existing pending-admin-eta aggregate (issues-4) by also surfacing the configured delay.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Emit a `tl_set` event from `set_timelock` carrying the new `delay_seconds`.
- Add a `GovernanceConfig` `#[contracttype]` struct `{ timelock: u64, pending_admin: Option<Address>, pending_eta: Option<u64> }` and a `get_governance_config(env) -> GovernanceConfig` read.
- Reuse existing reads (`get_timelock`, `get_pending_admin`, `get_pending_admin_eta`) internally so there is one source of truth.
- Keep `set_timelock` admin-gated; do not change its effect on already-queued handovers.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-timelock-event-and-governance-read`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the `tl_set` publish, the `GovernanceConfig` struct, and the `get_governance_config` entrypoint.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — assert the event fires with the right value and that `get_governance_config` reflects timelock, pending admin, and eta after a `propose_admin_transfer`.
  - **Add documentation:** add both to the events and read references in [`README.md`](README.md) and [`docs/abi.md`](docs/abi.md).
  - Include NatSpec-style doc comments (`///`) on the new struct and entrypoint, matching the existing style in `lib.rs`.
  - Validate security assumptions: read-only aggregate, no auth change, event payload is non-sensitive.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: timelock unset (0), timelock set with no pending admin, and a full propose-with-delay flow.
- Include the full `cargo test` output in the PR description.

### Example commit message
`feat: emit tl_set event and add get_governance_config aggregate read with tests and docs`

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
title: "Add a paused-state read variant of quote_route that reports whether compute_route_fee would currently succeed"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add a quote_route_status read that surfaces the pause gate and every guard outcome

### Description
`quote_route` in [`src/lib.rs`](src/lib.rs) returns `(fee, net)` but intentionally ignores the pause switch, the min/max bounds, the liquidity floor, and the cooldown — so an integrator can get a clean quote for a route that `compute_route_fee` would immediately reject. The existing `is_routable` (issues-4) returns a single bool. This issue adds a richer `quote_route_status` read that returns the quote **plus a typed reason code** describing exactly why the live route would fail (paused, below min, above max, insufficient liquidity, cooldown, or OK), so callers can plan and explain rejections without a state-changing dry run.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `RouteStatus` `#[contracttype]` enum (or a `u32` reason field) covering: `Ok`, `Paused`, `BelowMin`, `AboveMax`, `InsufficientLiquidity`, `CooldownActive`, `NotRegistered`.
- Add `quote_route_status(env, source, destination, amount) -> (i128, i128, RouteStatus)` that mirrors every `compute_route_fee` precondition **read-only** — no counter, timestamp, liquidity decrement, or event writes.
- This is distinct from `is_routable` (single bool) and `quote_route_checked` (cooldown-only) by reporting the **specific** blocking reason across all guards including the pause gate.
- Reuse the same threshold reads as `compute_route_fee` so the two can never disagree.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-quote-route-status`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the `RouteStatus` type and the `quote_route_status` read.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — drive each branch: paused, below min, above max, low liquidity, active cooldown (with `env.ledger()` control), unregistered, and the OK path; assert no state mutation occurs.
  - **Add documentation:** document the read and the status enum in [`README.md`](README.md) and [`docs/abi.md`](docs/abi.md).
  - Include NatSpec-style doc comments (`///`) on the new type and entrypoint, matching the existing style in `lib.rs`.
  - Validate security assumptions: strictly read-only, no event emission, no TTL bump, no liquidity debit.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: each blocking reason in isolation and a fully routable pair returning `Ok`.
- Include the full `cargo test` output in the PR description.

### Example commit message
`feat: add quote_route_status read reporting the live route gate outcome with tests and docs`

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
title: "Move the cooldown and bounds checks ahead of the liquidity debit in compute_route_fee to avoid spending liquidity on a rejected route"
labels: type:security, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Reorder compute_route_fee so the cooldown check runs before liquidity is debited

### Description
In `compute_route_fee` ([`src/lib.rs`](src/lib.rs)) the CHECKS/EFFECTS ordering is subtly wrong: the **liquidity debit** (and its `liq_used` event) happens *before* the per-pair **cooldown** check. If a route passes the min/max/liquidity gates but is then rejected by `RouteCooldownActive`, the whole transaction reverts — which is correct — but the code has already written `PairLiquidity` and emitted `liq_used` earlier in the same body. While Soroban's atomic rollback saves correctness today, the ordering violates the CHECKS-then-EFFECTS discipline the contract documents elsewhere and is fragile: any future early-return-without-revert (or partial-effect refactor) would leak a liquidity decrement for a route that never happened. This issue reorders all read-only CHECKS (including cooldown) ahead of every EFFECT (liquidity debit, counters, timestamp, events).

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Move the cooldown check so it runs in the CHECKS phase, before the liquidity decrement and the `liq_used` emission.
- Keep the existing semantics: registered, positive amount, min, max, liquidity sufficiency, and cooldown must all pass before any storage write or event.
- This is distinct from the issues-4 "release the reentrancy lock on every panic path" item — that is about the lock; this is about the CHECKS/EFFECTS ordering of liquidity vs cooldown.
- Add an inline comment documenting the strict CHECKS-then-EFFECTS contract so the ordering is not accidentally regressed.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-checks-before-effects-ordering`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — reorder `compute_route_fee` so the cooldown gate precedes the liquidity debit and all other effects.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — assert that a cooldown-blocked route leaves `PairLiquidity`, the route counter, the volume, and the timestamp unchanged, and emits no `liq_used` or `route` event.
  - **Add documentation:** note the CHECKS-then-EFFECTS guarantee in [`SECURITY.md`](SECURITY.md).
  - Include NatSpec-style doc comments (`///`) updating `compute_route_fee`'s ordering description, matching the existing style in `lib.rs`.
  - Validate security assumptions: no effect is applied before all guards pass; no event leaks on a rejected route.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: cooldown-blocked route, first-ever route, and a route that passes all gates.
- Include the full `cargo test` output and a short security notes section in the PR description.

### Example commit message
`security: order all CHECKS before liquidity debit in compute_route_fee with tests`

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
title: "Reject a zero MaxFeeAbsolute cap that silently makes every route free"
labels: type:security, area:fee-policy, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Guard set_max_fee_absolute against a zero cap that zeroes all protocol fees

### Description
`set_max_fee_absolute` in [`src/lib.rs`](src/lib.rs) rejects negative caps but explicitly allows `0`, and its own doc comment admits "a cap of `0` makes every route effectively free." Because `apply_fee_cap` computes `min(computed_fee, max_fee_absolute)`, a single admin call with `0` silently drives **every** pair's fee to zero regardless of its `fee_bps` — a catastrophic, easy-to-make fat-finger that drains all protocol revenue with no warning. This issue rejects a zero cap with a typed error (admins who truly want free routing can set per-pair `fee_bps` to 0 deliberately) and emits a clearer signal when the cap is changed.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Reject `max_fee == 0` in `set_max_fee_absolute` with a new append-only `RouterError::FeeCapTooLow = 19` (or reuse `AmountMustBePositive` if preferred — document the choice).
- Keep the existing negative-cap rejection and the `maxfee` event.
- This is distinct from the issues-4 "downgrade guard so the cap cannot silently widen" item — that prevents loosening a cap; this prevents a footgun zero cap.
- Document that removing the cap entirely is the separate `clear_max_fee_absolute` path (issues-4), not setting it to 0.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-reject-zero-fee-cap`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add the zero-cap guard and the error variant.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, alongside `mod test_i41_fee_cap`) — assert a zero cap panics, a positive cap still clamps fees, and `compute_route_fee` keeps charging the bps fee under a sane cap.
  - **Add documentation:** update the fee-cap section in [`README.md`](README.md) and the error table in [`docs/abi.md`](docs/abi.md).
  - Include NatSpec-style doc comments (`///`) on the updated entrypoint, matching the existing style in `lib.rs`.
  - Validate security assumptions: no path can zero all fees with a single accidental write; revenue invariant preserved.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: zero cap rejected, negative cap rejected, smallest valid positive cap, and cap above the bps fee (no clamp).
- Include the full `cargo test` output and a short security notes section in the PR description.

### Example commit message
`security: reject a zero MaxFeeAbsolute cap that would zero all protocol fees`

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
title: "Add a get_pair_infos batch read that returns PairInfo for a Vec of pairs in one call"
labels: type:feature, area:read-surface, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add a batch PairInfo read covering registration, fee, bounds, liquidity, and last-route

### Description
`get_pair_info` in [`src/lib.rs`](src/lib.rs) returns the full `PairInfo` struct for a **single** corridor, so a dashboard that tracks N pairs must make N separate contract calls. The issues-4 `get_pair_infos` proposal is the right shape — but note the existing `PairInfo` struct (registered, fee_bps, min, max, liquidity, last_route_at) is the canonical return type here, whereas that issue references an extended info struct. This issue specifically batches the **existing** `PairInfo` shape so it can ship without depending on the PairInfo extension work, giving dashboards a one-call read of every corridor's core state.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `get_pair_infos(env, pairs: Vec<(Symbol, Symbol)>) -> Vec<PairInfo>` returning one `PairInfo` per input pair, in order, reusing the existing `get_pair_info` logic.
- Cap the input at `MAX_BATCH_SIZE` and panic with `BatchTooLarge` when exceeded, matching the write-batch entrypoints.
- Be explicit in the issue and PR that this batches the **current** `PairInfo` struct; if the separate PairInfo-extension issue lands first, the new field flows through automatically.
- Read-only — no auth, no writes, no events.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-get-pair-infos-batch-read`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — add `get_pair_infos`, looping over the input and delegating to the same per-pair reads as `get_pair_info`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, alongside `mod test_i18_read_surface`) — assert the returned Vec length and per-pair fields, including an unregistered pair returning defaults, and an oversized batch panicking.
  - **Add documentation:** document the batch read in [`README.md`](README.md) and [`docs/abi.md`](docs/abi.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint, matching the existing style in `lib.rs`.
  - Validate security assumptions: read-only, bounded batch size, no state change.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: empty-ish single pair, mixed registered/unregistered, and an oversized batch.
- Include the full `cargo test` output in the PR description.

### Example commit message
`feat: add get_pair_infos batch read returning PairInfo for a Vec of pairs with tests and docs`

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
title: "Add test coverage proving compute_route_fee debits PairLiquidity and emits liq_used only when liquidity is bounded"
labels: type:test, area:liquidity, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the liquidity-consumption branch and the i128::MAX sentinel skip

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) debits `amount` from `PairLiquidity` via `saturating_sub` and emits a `liq_used` event **only** when the stored liquidity is not the unbounded `i128::MAX` sentinel — when no oracle has set liquidity, the decrement and event are skipped entirely. This dual behaviour (debit-and-emit vs skip) is core to the routing model but is not directly asserted in the existing test modules. This issue adds focused coverage for both branches and the saturating-sub edge at exact-balance and over-balance.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Test that, with liquidity set, a route debits exactly `amount` and `get_pair_liquidity` reflects the remainder.
- Test that a `liq_used` event is emitted carrying `(source, destination, remaining)` only in the bounded case, and **not** emitted when liquidity is the unset sentinel.
- Test the exact-balance route (remaining becomes 0) and confirm a subsequent route fails with `InsufficientLiquidity`.
- This is distinct from the issues-1 "decrement reported pair liquidity" feature issue — that adds the behaviour; this issue verifies it with tests.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-liquidity-consumption-coverage`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected; if a gap is found, note it for a follow-up.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, near `mod test_i15_bounds_liquidity`) — cover the bounded debit, the sentinel skip, the exact-balance drain, and event presence/absence via `env.events().all()`.
  - **Add documentation:** none required beyond test doc comments.
  - Include NatSpec-style doc comments (`///`) on each new test explaining the branch it pins, matching the existing style in `lib.rs`.
  - Validate security assumptions: liquidity can never go negative; the sentinel is never decremented.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: bounded debit, sentinel skip, exact drain to zero, and over-balance rejection.
- Include the full `cargo test` output in the PR description.

### Example commit message
`test: cover compute_route_fee liquidity debit, sentinel skip, and liq_used event`

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
title: "Add test coverage that the upgrade entrypoint is admin-gated and emits the upgraded event even while paused"
labels: type:test, area:upgrade, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the upgrade auth gate and its deliberate paused-state exception

### Description
`upgrade` in [`src/lib.rs`](src/lib.rs) is admin-gated, emits an `upgraded` event with the new WASM hash, and **deliberately skips the pause gate** so a paused router can still be patched. The doc comment reasons about this trade-off, but there is no test asserting (a) a non-admin caller is rejected and (b) an upgrade still proceeds and emits its event while the router is paused. This issue adds that coverage, including a real round-trip upgrade to a second WASM that preserves pair state.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Assert a non-admin caller cannot call `upgrade`.
- Assert the `upgraded` event is emitted carrying the new hash.
- Assert an upgrade succeeds while `is_paused()` is true (the documented exception).
- Use the Soroban test util pattern for registering a second contract WASM / hash so the upgrade actually swaps code and pair registrations survive.
- Distinct from the issues-1 "add a contract upgrade entrypoint" feature item — that builds `upgrade`; this issue verifies its auth and paused-exception behaviour.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-upgrade-auth-and-paused-coverage`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — cover non-admin rejection, the `upgraded` event, the paused-state exception, and state survival across the upgrade.
  - **Add documentation:** none required beyond test doc comments.
  - Include NatSpec-style doc comments (`///`) on each new test, matching the existing style in `lib.rs`.
  - Validate security assumptions: only admin upgrades; paused exception does not become an escalation path.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: non-admin rejection, event emission, upgrade while paused, and pair state preserved post-upgrade.
- Include the full `cargo test` output in the PR description.

### Example commit message
`test: cover upgrade admin gating, upgraded event, and paused-state exception`

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
title: "Add test coverage that cancel_admin_transfer clears both the pending admin and the pending eta"
labels: type:test, area:governance, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test that cancel_admin_transfer wipes the pending admin and its queued eta together

### Description
`cancel_admin_transfer` in [`src/lib.rs`](src/lib.rs) removes both `DataKey::PendingAdmin` and `DataKey::PendingAdminEta`, and is documented as a no-op when nothing is pending. A botched cancel that cleared only one of the two slots would leave a dangling eta or a pending admin with no eta, breaking the timelock invariant. The existing suites cover propose/accept and the eta read, but there is no direct assertion that cancel clears **both** slots and is safely idempotent. This issue adds that coverage.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Test the full flow: set a timelock, propose a transfer (stamping pending admin + eta), then cancel and assert `get_pending_admin()` and `get_pending_admin_eta()` both return `None`.
- Test that calling `cancel_admin_transfer` with nothing pending is a no-op and does not panic.
- Test that a non-admin caller is rejected.
- Distinct from issues-4 "emit a cancelled event on cancel" (a feature) — this issue is pure cancel-state coverage.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-cancel-admin-transfer-coverage`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`) — cover the propose-then-cancel clearing of both slots, the no-pending no-op, and non-admin rejection.
  - **Add documentation:** none required beyond test doc comments.
  - Include NatSpec-style doc comments (`///`) on each new test, matching the existing style in `lib.rs`.
  - Validate security assumptions: no dangling eta, no orphan pending admin, admin-only cancel.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: cancel after propose, cancel with nothing pending, and unauthorized cancel.
- Include the full `cargo test` output in the PR description.

### Example commit message
`test: cover cancel_admin_transfer clearing both pending admin and eta`

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
title: "Add test coverage that is_pair_active returns false for unregistered, zero-liquidity, and unset-liquidity pairs"
labels: type:test, area:read-surface, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the three falsey branches and the true branch of is_pair_active

### Description
`is_pair_active` in [`src/lib.rs`](src/lib.rs) returns true only when a pair is both registered **and** has strictly positive reported liquidity. It short-circuits to false on an unregistered pair, and otherwise returns `liquidity > 0`, defaulting absent liquidity to 0. There is a subtlety here: a pair whose liquidity was never set by an oracle reads as `0` from this helper (not the `i128::MAX` sentinel that `compute_route_fee` uses), so `is_pair_active` is **false** for a freshly registered, never-funded pair even though `compute_route_fee` would treat it as unbounded. That divergence is worth pinning with tests so it is intentional and documented.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Test `is_pair_active` returns false for an unregistered pair.
- Test it returns false for a registered pair with liquidity explicitly set to 0 and for a registered pair with liquidity never set.
- Test it returns true for a registered pair with positive liquidity.
- Document (in the test doc comments and optionally a `lib.rs` note) the deliberate divergence between `is_pair_active`'s 0-default and `compute_route_fee`'s `i128::MAX` sentinel.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-is-pair-active-coverage`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — optional one-line doc clarification of the sentinel divergence; otherwise no production change.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, near `mod test_i18_read_surface`) — cover all four branches.
  - **Add documentation:** note the divergence in [`docs/abi.md`](docs/abi.md) under `is_pair_active`.
  - Include NatSpec-style doc comments (`///`) on each new test, matching the existing style in `lib.rs`.
  - Validate security assumptions: read-only, no state change.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: unregistered, zero liquidity, unset liquidity, and positive liquidity.
- Include the full `cargo test` output in the PR description.

### Example commit message
`test: cover is_pair_active across unregistered, zero, unset, and positive liquidity`

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
title: "Add a property test asserting compute_route_fee and quote_route agree even with an absolute fee cap configured"
labels: type:test, area:fee-policy, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Property-test quote/compute fee parity under a random absolute fee cap

### Description
The existing `prop_quote_matches_compute` property test in [`src/lib.rs`](src/lib.rs) asserts `quote_route` and `compute_route_fee` return the same fee — but only with **no** `MaxFeeAbsolute` configured. Both `quote_route` and `compute_route_fee` independently call `apply_fee_cap`, so a divergence in how the cap is applied on one path but not the other would not be caught today. This issue adds a property test that sets a random absolute cap and asserts the two paths still agree across the full `(amount, fee_bps, cap)` space, and that the charged fee never exceeds the cap.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `proptest!` case that registers a pair with a random `fee_bps`, sets a random positive `MaxFeeAbsolute`, and asserts `quote_route` fee == `compute_route_fee` fee for a random valid `amount`.
- Assert the resulting fee is `min(amount * fee_bps / 10_000, cap)` and never exceeds the cap.
- Keep the fixed `ProptestConfig { cases: 96 }` convention used by the existing property tests for deterministic CI.
- Distinct from issues-1's general fee-math property harness and issues-4's "cap never exceeded across the input space" — this issue specifically pins **quote/compute parity under a cap**.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-quote-compute-parity-under-cap`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) (inline `#[cfg(test)] mod test`, in the existing `proptest!` block) — add the parity-under-cap property using `setup_pair_with_fee` plus `set_max_fee_absolute`.
  - **Add documentation:** none required beyond test doc comments.
  - Include NatSpec-style doc comments (`///`) on the new property explaining the invariant, matching the existing style in `lib.rs`.
  - Validate security assumptions: the cap is applied identically on both read and compute paths.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases: cap below the bps fee (clamp active), cap above the bps fee (no clamp), and a free pair.
- Include the full `cargo test` output in the PR description.

### Example commit message
`test: property-test quote/compute fee parity under a random absolute fee cap`

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
title: "Document the complete DataKey storage map including Oracle, Timelock, ReentrancyLock, and PairCooldown in storage.md"
labels: type:docs, area:storage, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Bring docs/storage.md up to date with every current DataKey variant

### Description
The `DataKey` enum in [`src/lib.rs`](src/lib.rs) has grown to cover `Oracle`, `Timelock`, `PendingAdminEta`, `ReentrancyLock`, `PairCooldown`, `MaxFeeAbsolute`, `PairRouteCount`, `PairVolume`, and `SchemaVersion`, but [`docs/storage.md`](docs/storage.md) predates several of these additions. A storage reference that omits live keys misleads integrators about what state exists, its persistence class, and its TTL handling. This issue audits the enum against the doc and brings `storage.md` to full parity — every variant, its storage class (persistent vs instance), its default-on-absent value, and which entrypoints read/write it.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Enumerate every `DataKey` variant currently in `lib.rs` and confirm each appears in `storage.md` with: storage class, value type, default-on-absent, and the reading/writing entrypoints.
- Add the missing entries (at minimum `Oracle`, `Timelock`, `PendingAdminEta`, `ReentrancyLock`, `PairCooldown`, `MaxFeeAbsolute`).
- Note that this complements, not duplicates, the issues-2 STORAGE.md reference: this issue is a parity/freshness audit of the existing `docs/storage.md` against the current enum.
- Keep the doc's existing structure and formatting conventions.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-storage-map-parity`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no code change; if a `DataKey` doc comment is wrong, fix it in passing.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) — no tests required for a docs-only change; ensure `cargo test` still passes.
  - **Add documentation:** rewrite/extend [`docs/storage.md`](docs/storage.md) to cover every variant, cross-linked from [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) only where a `DataKey` comment is corrected, matching the existing style in `lib.rs`.
  - Validate security assumptions: documentation accuracy reduces operational misconfiguration risk.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Confirm every enum variant is represented and no stale key descriptions remain.
- Include a short diff summary of added/updated storage entries in the PR description.

### Example commit message
`docs: bring docs/storage.md to full parity with the current DataKey enum`

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
title: "Replace the placeholder router doc string and module header that still call StableRouteRouter a placeholder"
labels: type:docs, area:contract-docs, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Update the StableRouteRouter struct doc and migrate-events TODO header to reflect shipped behaviour

### Description
The `#[contract] pub struct StableRouteRouter` in [`src/lib.rs`](src/lib.rs) still carries the doc string "StableRoute router contract — placeholder for routing logic. In production this would integrate with path payments and liquidity data." The contract is no longer a placeholder: it has registration, fee/bounds/liquidity config, cooldowns, an oracle role, a governance timelock, a reentrancy guard, liquidity consumption, and metrics. The stale "placeholder" framing misleads new contributors and auditors about the contract's maturity. This issue rewrites the struct doc to describe the real router and updates the top-of-file `#![allow(deprecated)]` events-migration note to point at the tracked migration work rather than reading as an open TODO with no owner.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Rewrite the `StableRouteRouter` struct doc comment to accurately summarize the implemented capabilities and the intended path-payment integration as future work, not as "this is only a placeholder."
- Refresh the file-header comment so the deprecated-events migration note references the existing tracked migration issue rather than implying nothing is planned.
- Docs/comments only — no behaviour, entrypoint, storage, or error change.
- Do not duplicate the issues-1 "replace the placeholder README description" item; this issue targets the **in-code** struct/module doc strings specifically.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-router-docstring-refresh`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — update the struct doc comment and the file-header note.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) — no tests required; confirm `cargo build`/`cargo test` still pass.
  - **Add documentation:** ensure the struct doc and [`README.md`](README.md) describe the contract consistently.
  - Include NatSpec-style doc comments (`///`) in the rewritten struct doc, matching the existing style in `lib.rs`.
  - Validate security assumptions: accurate docs reduce auditor and integrator misunderstanding.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Confirm the word "placeholder" no longer describes the shipped router in the struct doc.
- Include a short before/after of the doc string in the PR description.

### Example commit message
`docs: replace placeholder router doc string with an accurate capability summary`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.

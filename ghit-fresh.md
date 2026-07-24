---
type: Feature
title: "Add a per-pair route rate limit with a configurable cooldown using PairLastRouteAt"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement a per-pair route cooldown rate limit

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) already stamps `DataKey::PairLastRouteAt(source, destination)` with `env.ledger().timestamp()` on every route, but nothing ever *reads* it to throttle activity — a pair can be routed an unbounded number of times in a single ledger. For a stablecoin router this is a real abuse surface (spam routing, oracle-front-running, griefing the route counter). This issue uses the already-stored `PairLastRouteAt` to enforce an optional minimum interval between routes per pair.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::PairCooldown(Symbol, Symbol)` (seconds) plus admin-gated `set_pair_cooldown` / `get_pair_cooldown`; default 0 (disabled, fully backward compatible).
- In `compute_route_fee`, after the liquidity check, compare `env.ledger().timestamp()` against `PairLastRouteAt + cooldown` and reject early routes with an append-only `RouteCooldownActive` error.
- Keep the existing timestamp stamp and `route` event; do not alter any current error codes.
- Bump the persistent-entry TTL on the new cooldown slot when written.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-31-route-cooldown`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::PairCooldown`, `set_pair_cooldown`, `get_pair_cooldown`, and the cooldown check in `compute_route_fee`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — use `env.ledger().set_timestamp` to assert a too-soon route is rejected and a route after the cooldown succeeds.
  - **Add documentation:** document the rate-limit model in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: cooldown cannot underflow; disabled (0) preserves current behavior; first route is always allowed.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: cooldown disabled, first route, route exactly at boundary, route one second early, cooldown changed mid-flight.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add per-pair route cooldown rate limit with tests and docs`

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
title: "Add per-pair lifetime route counters and cumulative volume tracking"
labels: type:feature, area:metrics, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement per-pair route counters and cumulative volume

### Description
The router keeps only a single global `DataKey::TotalRoutesAllTime` counter in [`src/lib.rs`](src/lib.rs); there is no per-pair breakdown of how many routes a corridor has served or how much volume has flowed through it. Operators and dashboards cannot rank corridors, detect a hot pair, or reconcile per-corridor fee accrual without scraping every `route` event off-chain. This issue adds on-chain per-pair route counts and cumulative routed volume.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::PairRouteCount(Symbol, Symbol)` (u64) and `DataKey::PairVolume(Symbol, Symbol)` (i128); increment both inside `compute_route_fee` using `saturating_add`.
- Add read entrypoints `get_pair_route_count` and `get_pair_volume` (both default 0 when absent).
- Extend `PairInfo` is out of scope — keep these as standalone getters so the existing struct shape stays stable.
- Do not change any existing error codes; bump TTL on the new slots when written.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-32-per-pair-metrics`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the two new `DataKey` variants, the increments in `compute_route_fee`, and the getters.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert count and volume increment per route, stay isolated per pair, and default to 0.
  - **Add documentation:** document the per-pair metrics in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: saturating arithmetic cannot panic; counters are monotonic; pairs are independent.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: zero routes, multiple routes one pair, two pairs independent, volume near i128::MAX.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add per-pair route counters and cumulative volume tracking`

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
title: "Add a minimum-output (slippage) guard so compute_route_fee rejects routes whose net falls below a caller floor"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement a caller-supplied minimum-output guard

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) returns the fee and the caller infers `net = amount - fee`, but there is no way for a caller to assert a *minimum acceptable net* in the same call. If the per-pair fee is raised by the admin between a client's quote and its route, the caller silently pays the higher fee with no protection — the classic slippage/MEV gap. This issue adds an optional `min_out` parameter (or a sibling `route_with_min_out`) that rejects the route when the computed net falls below the caller's floor.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `compute_route_fee_checked(env, source, destination, amount, min_out: i128) -> i128` that runs the existing validation, computes `net = amount - fee`, and rejects with an append-only `SlippageExceeded` error when `net < min_out`.
- Keep the original `compute_route_fee` unchanged for backward compatibility; the checked variant builds on the same internal logic (factor the shared body into a private helper).
- `min_out <= 0` means "no floor" and must behave exactly like the unchecked path.
- Emit the existing `route` event; do not renumber any error.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-33-min-out-guard`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — shared private compute helper plus `compute_route_fee_checked` and `SlippageExceeded`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert net-below-floor is rejected, net-at-floor passes, and `min_out <= 0` matches the unchecked result.
  - **Add documentation:** document the slippage-protection flow in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: parity between checked and unchecked fee math; no off-by-one at the floor; counter/timestamp/event semantics identical.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: net below floor, net exactly at floor, zero/negative floor, fee raised between quote and route.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add minimum-output slippage guard to route computation`

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
title: "Promote the hot Paused flag and Admin address to instance storage to cut per-call read cost"
labels: type:enhancement, area:storage, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Move hot global state into instance storage

### Description
The `DataKey` doc comment in [`src/lib.rs`](src/lib.rs) explicitly says instance storage "is reserved for hot configuration that we expect every invocation to touch — none yet," yet `DataKey::Admin` (read by every admin entrypoint) and `DataKey::Paused` (read by every gated write) live in `persistent()` storage and are loaded as separate persistent reads on every call. These are exactly the global singletons that belong in instance storage, which is bundled with the contract instance and cheaper/safer to access per-invocation. This issue migrates them.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Move `Admin`, `PendingAdmin`, and `Paused` to `env.storage().instance()`; keep per-pair keyed data in `persistent()`.
- Bump instance TTL with `env.storage().instance().extend_ttl(...)` on writes so the instance never archives.
- This is behavior-preserving from the client's perspective: same getters, same errors, same events; only the storage tier changes.
- Update the `DataKey` doc comment to reflect the new "hot global" classification.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-34-instance-storage-hot-state`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — switch the three singletons to instance storage and add instance TTL bumps.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — confirm all existing admin/pause/transfer behaviors are unchanged after the migration.
  - **Add documentation:** document the instance-vs-persistent split in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) updating the storage rationale.
  - Validate security assumptions: no semantics change; instance TTL kept alive; no key collision between tiers.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: init, pause/unpause, admin transfer, gated writes — all behave as before.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: move admin and paused singletons to instance storage`

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
title: "Replace the placeholder route_tag with a deterministic keyed route identifier"
labels: type:feature, area:routing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement a deterministic route identifier

### Description
`route_tag` in [`src/lib.rs`](src/lib.rs) is documented as a placeholder "used by the backend to verify route integrity," but it simply echoes its `(source, destination)` arguments back unchanged — it ignores the env, performs no hashing, and offers no integrity value whatsoever. Its only test asserts the echo. This issue replaces it with a real deterministic route identifier the backend can use to correlate on-chain routes with off-chain records.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Implement `route_tag(env, source, destination) -> BytesN<32>` as a deterministic hash over `(source, destination)` using `env.crypto()` (e.g. keccak256/sha256 of the encoded symbols).
- Make the tag direction-sensitive so `USDC→EURC` and `EURC→USDC` differ (matching the directional pair model).
- Optionally include the tag in the `route` event payload so indexers can key on it (note this is additive and document the payload change).
- Reject identical source/destination consistent with `register_pair` semantics, or document why echo-of-identity is acceptable.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-35-deterministic-route-tag`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — hashed `route_tag` implementation.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert determinism (same input → same tag), direction sensitivity, and distinct tags for distinct pairs.
  - **Add documentation:** document the route-identifier scheme in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the changed entrypoint.
  - Validate security assumptions: tag is collision-resistant for the symbol domain; no preimage leak of secrets.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: determinism, direction flip, two different pairs, repeated calls.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: replace placeholder route_tag with deterministic keyed identifier`

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
title: "Reject compute_route_fee and quote_route on a registered pair with zero reported liquidity"
labels: type:enhancement, area:liquidity, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Treat zero-liquidity pairs as inactive in routing

### Description
There is an inconsistency in [`src/lib.rs`](src/lib.rs): `is_pair_active` reports a pair as active only when liquidity is `> 0`, but `compute_route_fee` treats absent/unset liquidity as the `i128::MAX` "unbounded" sentinel — so a pair that an operator has explicitly set to `0` liquidity is reported inactive yet still routes any positive amount (because `unwrap_or(i128::MAX)` only fires when the slot is *absent*, while an explicit `0` makes every positive amount fail `InsufficientLiquidity` only by accident). The semantics of "unset vs explicitly zero" are muddled. This issue makes routing consistent with `is_pair_active`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Define and document the liquidity contract: *absent* slot = unbounded (sentinel), *explicit 0* = inactive corridor.
- In `compute_route_fee` and `quote_route`, when liquidity is explicitly set to `0`, reject with `InsufficientLiquidity` (#12) rather than relying on the amount comparison, so a 0-liquidity pair is uniformly non-routable.
- Keep absent-liquidity behavior (unbounded) unchanged for backward compatibility.
- Do not renumber errors; align the doc comments on `PairLiquidity`, `is_pair_active`, and `compute_route_fee`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-36-zero-liquidity-inactive`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — explicit-zero handling in the routing path.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert explicit-0 liquidity rejects all routes, absent liquidity stays unbounded, and `is_pair_active` agrees with routability.
  - **Add documentation:** document the absent-vs-zero liquidity semantics in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the affected functions.
  - Validate security assumptions: no pair is simultaneously "inactive" and "routable"; sentinel handling is explicit.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: absent liquidity, explicit-zero liquidity, positive liquidity at/over bound.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: treat explicit zero-liquidity pairs as non-routable in compute_route_fee`

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
title: "Require a pair to be registered before its fee, bounds, or liquidity can be configured"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Enforce registration-before-configuration for pair settings

### Description
In [`src/lib.rs`](src/lib.rs) the admin can call `set_pair_fee_bps`, `set_pair_min_amount`, `set_pair_max_amount`, and `set_pair_liquidity` for a pair that was **never registered** via `register_pair`. The config writes succeed and orphan storage slots for a non-existent corridor, which wastes rent, pollutes any future enumeration index, and can silently mis-configure a pair an operator never intended to enable. This issue requires the pair to exist before its parameters can be set.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- In each of the four config setters, check `DataKey::Pair(source, destination)` is `true` and reject otherwise with an append-only `PairNotRegistered` (#5) (reuse the existing code).
- Keep `register_pair` / `unregister_pair` unchanged; document that config must follow registration.
- Preserve all existing sign/cap validation and events.
- Consider whether `unregister_pair` should be blocked while config slots still exist (cross-reference the cleanup work) — out of scope here, note it.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-37-register-before-configure`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — registration guard in the four config setters.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each setter panics with #5 for an unregistered pair and succeeds after `register_pair`.
  - **Add documentation:** document the registration-first invariant in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the affected setters.
  - Validate security assumptions: no orphan config can be created; existing valid flows unaffected.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: unregistered set attempt per setter, set after register, set after unregister.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: require pair registration before fee/bounds/liquidity configuration`

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
title: "Add test coverage asserting idempotent register/unregister and event payloads for pair lifecycle"
labels: type:test, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test pair lifecycle events and idempotency edges

### Description
The existing suite in [`src/lib.rs`](src/lib.rs) covers basic register/unregister round-trips, but it never asserts the **emitted events** for the pair lifecycle (`pair_reg`, `unreg`, `fee_set`, `liq_set`, `init`) and never verifies edge behaviors like unregistering a never-registered pair, re-registering after unregister, or that `unregister_pair` leaves the configured fee untouched (the behavior its own doc comment claims). This issue adds event-assertion and lifecycle-edge coverage.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Use `env.events().all()` to assert the topic and data tuple emitted by `init`, `register_pair`, `unregister_pair`, `set_pair_fee_bps`, and `set_pair_liquidity`.
- Cover: unregister of a never-registered pair is a clean no-op, re-register after unregister works, and `unregister_pair` does NOT clear `PairFeeBps` (documenting the current behavior precisely).
- Use the existing `setup_initialized` helper; no production change expected unless a discrepancy is found.
- Assert exact event counts so a future extra/missing publish is caught.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-38-pair-lifecycle-event-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the event-assertion and lifecycle-edge suite described above.
  - **Add documentation:** note the pair-lifecycle test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: events are emitted exactly once per state change with the documented payload.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: event topic/data per entrypoint, no-op unregister, re-register, fee survives unregister.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover pair lifecycle events and idempotency edges`

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
title: "Add test coverage for compute_route_fee state mutations: counter, timestamp, and route event"
labels: type:test, area:routing, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the routing side effects of compute_route_fee

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) has four side effects on a successful call: it increments `TotalRoutesAllTime`, stamps `PairLastRouteAt`, and publishes the `route` event with `(source, destination, amount)`. The existing tests check the returned fee and the counter once, but never assert the `route` event payload, never verify the timestamp is stamped from a non-zero `env.ledger().set_timestamp`, and never confirm the counter is shared across multiple distinct pairs. This issue adds focused coverage of the mutation surface.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: `route` event topic `route` and data `(source, destination, amount)` via `env.events().all()`.
- Cover: `PairLastRouteAt` reflects a custom `env.ledger().set_timestamp` value after a route, and is `None` before.
- Cover: `TotalRoutesAllTime` increments once per route and is global across two different pairs.
- Cover: `quote_route` does NOT emit a `route` event nor change the counter (parity guard) — assert event count unchanged.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-39-route-side-effect-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the side-effect suite described above.
  - **Add documentation:** note the routing side-effect test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: every documented side effect is asserted; `quote_route` proven non-mutating.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: event payload, timestamp before/after, global counter across pairs, quote non-mutation.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover compute_route_fee counter, timestamp, and route event`

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
title: "Add test coverage for the version and get_schema_version distinction and NotInitialized paths"
labels: type:test, area:lifecycle, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the version surface and uninitialized-call failures

### Description
[`src/lib.rs`](src/lib.rs) exposes two distinct version concepts — `version()` returning the symbol `ROUTER_V2` and `get_schema_version()` returning the storage schema number — and a class of `NotInitialized` (#2) failures when admin-gated entrypoints are called before `init`. The existing suite checks `version()` once and the happy-path migration, but never asserts that admin entrypoints (`pause`, `set_pair_fee_bps`, `propose_admin_transfer`, `migrate_v1_to_v2`, etc.) panic with #2 on a fresh, uninitialized contract. This issue adds that coverage.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: `version()` is stable (`ROUTER_V2`) and independent of `get_schema_version()`.
- Cover: on a contract registered but NOT `init`-ed, each admin-gated entrypoint panics with `NotInitialized` (#2) — use `#[should_panic(expected = "Error(Contract, #2)")]`.
- Cover: `get_schema_version` returns 1 before any init/migration (default fallback).
- Register the contract WITHOUT calling `init` for the negative cases (do not reuse `setup_initialized`).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-40-version-uninitialized-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the version + NotInitialized suite described above.
  - **Add documentation:** note the lifecycle test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no admin entrypoint succeeds before initialization; the two version concepts are clearly separated.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: uninitialized pause/fee/transfer/migrate, schema-version default, version constant.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover version surface and uninitialized-call NotInitialized paths`

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
title: "Add test coverage for get_pair_info and quote_route defaults at i128 sentinel boundaries"
labels: type:test, area:pairs, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test default sentinels and quote_route net computation

### Description
`get_pair_info` in [`src/lib.rs`](src/lib.rs) returns `max_amount: i128::MAX` and `liquidity: 0` for an unconfigured pair, and `quote_route` returns `(fee, amount - fee)` — these specific default-sentinel values and the net subtraction are unverified. The suite asserts `get_pair_info` only on a fully-configured pair, never on a bare-registered or fully-unconfigured one, and never asserts that `quote_route`'s second tuple element equals `amount - fee` exactly across fee settings. This issue pins those defaults and the net math.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Cover: `get_pair_info` for (a) a never-touched pair returns `registered:false, fee_bps:0, min_amount:0, max_amount:i128::MAX, liquidity:0, last_route_at:0`, and (b) a bare-registered pair flips only `registered`.
- Cover: `quote_route` returns `net == amount - fee` for zero fee, a typical fee, and the `MAX_FEE_BPS` cap.
- Cover: `quote_route` and `compute_route_fee` agree on the fee value for identical configuration (parity).
- Use `setup_initialized`; no production change expected.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-41-defaults-quote-net-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the defaults + net-math suite described above.
  - **Add documentation:** note the defaults test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: default sentinels match doc comments; net is never negative for valid inputs.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: untouched pair, bare-registered pair, zero/typical/max fee net, quote-vs-compute parity.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover pair-info defaults and quote_route net computation`

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
title: "Add an explicit reentrancy and external-call ordering guard for the future fund-moving route path"
labels: type:security, area:routing, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden the fund-moving path against reentrancy and unsafe call ordering

### Description
The router in [`src/lib.rs`](src/lib.rs) is on a roadmap to move real SAC value (see the planned on-chain `route` work). Today `compute_route_fee` performs its storage writes (counter, timestamp) *before* doing any work, which is fine while no external calls exist — but once a `token::Client::transfer` is introduced, the checks-effects-interactions ordering and reentrancy posture become security-critical. This issue establishes an explicit reentrancy guard and a documented effects-before-interactions discipline so the fund-moving path is safe by construction before it ships.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `DataKey::ReentrancyLock` (bool) plus a private `nonreentrant` guard helper that sets the lock on entry and clears it on exit of any entrypoint making external token calls.
- Document and enforce checks-effects-interactions: validate, write effects (counter/timestamp/liquidity), then perform external transfers last.
- Add an append-only `ReentrantCall` error returned when the lock is already held.
- Provide a test that simulates a malicious token callback attempting to re-enter and asserts it is rejected (use a mock token contract that calls back).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-42-reentrancy-guard`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::ReentrancyLock`, the `nonreentrant` helper, `ReentrantCall`, and the ordering discipline.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — a mock re-entering token contract; assert the second entry panics with `ReentrantCall` and the lock is released on normal exit.
  - **Add documentation:** add a "Reentrancy & call ordering" section to [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the guard helper.
  - Validate security assumptions: lock always released on success and on the guarded panic paths; no entrypoint can be re-entered mid-transfer.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: normal call releases lock, re-entrant call rejected, nested distinct entrypoints, lock state after panic.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add reentrancy guard and checks-effects-interactions discipline`

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
title: "Enforce a global cumulative-fee cap so a single route cannot exceed a protocol-wide maximum charge"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden fee accrual with an absolute per-route fee ceiling

### Description
Fees in [`src/lib.rs`](src/lib.rs) are purely proportional: `amount * fee_bps / 10_000`, capped only relatively by `MAX_FEE_BPS` (10%). There is no *absolute* ceiling on what a single route can be charged, so a large-amount route at the maximum bps yields an unbounded fee — and a misconfigured or compromised admin setting a high bps directly translates to extraction proportional to amount with no safety stop. This issue adds an optional absolute per-route fee cap that bounds the worst case independent of `fee_bps`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::MaxFeeAbsolute` (i128) plus admin-gated `set_max_fee_absolute` / `get_max_fee_absolute`; default = unset = no absolute cap (backward compatible).
- In `compute_route_fee` and `quote_route`, after computing the proportional fee, clamp it to `min(fee, max_fee_absolute)` when the cap is set.
- Emit a `maxfee` configuration event; keep `RouterError` append-only (a setter reusing existing sign validation is sufficient — reject negative caps).
- Document the interaction with `MAX_FEE_BPS` (both bounds apply; the tighter wins).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-43-absolute-fee-cap`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::MaxFeeAbsolute`, setter/getter, and the clamp in the fee computation.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert fee is clamped when proportional fee exceeds the cap, unaffected below the cap, and unbounded when the cap is unset.
  - **Add documentation:** document the dual fee-bound model in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: cap cannot be negative; clamp applies identically in quote and compute; no overflow.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: cap unset, fee below cap, fee above cap, cap at zero, quote-vs-compute parity under the cap.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: enforce absolute per-route fee ceiling with tests and docs`

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
title: "Document the storage model and DataKey TTL classification in a STORAGE.md reference"
labels: type:docs, area:storage, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document the router storage and TTL model

### Description
[`src/lib.rs`](src/lib.rs) defines twelve `DataKey` variants with subtle semantics — keyed per-pair slots vs global singletons, default sentinels (`i128::MAX` for max-amount/liquidity, `0` for min-amount, `false` for pair/paused), and an instance-vs-persistent split the doc comment only gestures at. There is no single reference an integrator or auditor can read to understand what is stored, where, with what default, and how it ages out. This issue produces an authoritative storage reference document.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Create `docs/storage.md` (and link from [`README.md`](README.md)) with a table per `DataKey`: key shape, value type, storage tier, default-when-absent, the entrypoints that read/write it, and TTL class.
- Document the sentinel conventions explicitly (`i128::MAX` = unbounded, absent bool = false, etc.).
- Note the archival/TTL risk and reference any TTL-bumping work as the mitigation.
- No production code change beyond optionally tightening `///` comments on `DataKey` variants.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-44-storage-reference`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — optionally align `///` comments on `DataKey` variants (no behavior change).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** `docs/storage.md` plus a README link.
  - Include NatSpec-style doc comments (`///`) consistency across all `DataKey` variants.
  - Validate security assumptions: documented defaults exactly match the `unwrap_or` values in the source.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify each `DataKey` variant appears once with the correct documented default.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add authoritative storage model and DataKey TTL reference`

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
title: "Add a CONTRIBUTING.md with the contract conventions, append-only error policy, and PR checklist"
labels: type:docs, area:contributing, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document contributor conventions and the PR checklist

### Description
The repository has no `CONTRIBUTING.md`, yet [`src/lib.rs`](src/lib.rs) encodes several non-obvious conventions a contributor must follow to get a PR merged: the append-only `RouterError` numbering rule ("never reuse or renumber a variant once it has shipped"), the `symbol_short!` ≤ 9-character event-topic constraint, the admin-auth + pause-gate patterns, and the fmt/build/test commands. New contributors rediscover these by trial and error. This issue captures them in a single contributor guide.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Create `CONTRIBUTING.md` covering: the append-only error policy, the event-topic naming/length rule, the admin-auth and pause-gate patterns, the storage-tier/TTL conventions, and the local `cargo fmt --all -- --check` / `cargo build` / `cargo test` workflow.
- Include a PR checklist (tests added, NatSpec on new entrypoints, no error renumbering, events asserted, docs updated).
- Link the StableRoute Discord: https://discord.gg/37aCpusvx
- No production code change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-45-contributing-guide`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; optionally cross-link conventions in a top-of-file comment.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** create `CONTRIBUTING.md` and link it from [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) consistency note in the guide.
  - Validate security assumptions: the documented conventions match the actual enforced patterns in the source.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify every documented convention maps to a real pattern in `src/lib.rs`.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add CONTRIBUTING guide with conventions and PR checklist`

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

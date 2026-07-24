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

---
type: Feature
title: "Execute on-chain routed swaps via Stellar Asset Contract (SAC) path payments"
labels: type:feature, area:settlement, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement on-chain routed swaps via Stellar Asset Contract (SAC) path payments

### Description
Today [`compute_route_fee`](src/lib.rs) in [`src/lib.rs`](src/lib.rs) only *computes* a fee, bumps a counter, stamps `PairLastRouteAt`, and emits a `route` event — it **moves no value**. The router "never custodies funds" (see the `set_fee_recipient` doc comment), so the on-chain route record and the real transfer can diverge and there is no trustless guarantee a route ever executed. This issue adds a real `route(from, source, destination, amount)` entrypoint that transfers a configurable **SAC token** and pays the protocol fee to `FeeRecipient` atomically with the counter/timestamp updates.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add a `DataKey::RouteToken(Symbol)` mapping each routable `Symbol` (e.g. `USDC`) to its SAC `Address`, set by admin via a new `set_route_token` entrypoint.
- Add `route(from: Address, source: Symbol, destination: Symbol, amount: i128)` that calls `from.require_auth()`, reuses the existing validation in `compute_route_fee` (registered, min/max, liquidity, positive amount), then moves `amount` via `soroban_sdk::token::Client` and credits `fee` to the configured `FeeRecipient`.
- Reuse `BPS_DENOMINATOR` / `MAX_FEE_BPS` math; keep `RouterError` codes append-only (add e.g. `RouteTokenNotSet`, `FeeRecipientNotSet`).
- Preserve all existing invariants: pause gate, saturating counter increment, the `route` event (extend its payload with the fee transferred), and overflow-safe arithmetic.
- Bump the persistent-entry TTL for any long-lived token-mapping slot introduced.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-01-onchain-route-payments`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::RouteToken`, `set_route_token`, and the `route()` transfer logic via `soroban_sdk::token`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — use `env.register_stellar_asset_contract` / a mock token, asserting balance deltas, fee crediting, paused/unregistered failure, and event payloads.
  - **Add documentation:** update [`README.md`](README.md) and create `docs/router/routing.md` describing the validate → transfer → fee → record lifecycle with a sequence diagram.
  - Include NatSpec-style doc comments (`///`) on every new entrypoint, matching the existing style in `lib.rs`.
  - Validate security assumptions: correct `require_auth` on `from`, no double-spend on repeated `route`, fee never exceeds `amount`, overflow safety on balance math.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: token unset, fee recipient unset, paused contract, unregistered/over-max/under-min amount, insufficient liquidity, unauthorized caller.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: execute on-chain routed swaps via SAC path payments with tests and docs`

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
title: "Bump persistent-entry TTL on every router storage read and write to prevent state expiry"
labels: type:enhancement, area:storage, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement persistent-entry TTL bumping across all router storage access

### Description
Every storage slot in [`src/lib.rs`](src/lib.rs) — `DataKey::Admin`, `Pair`, `PairFeeBps`, `PairLiquidity`, etc. — is written to `env.storage().persistent()` but **no entry's TTL is ever extended**. On Soroban, persistent entries that are not bumped will eventually expire and become unreadable (state archival), which would silently brick a deployed router (admin lost, pairs unregistered). The `DataKey` doc comment even claims these values "need to survive the contract's instance TTL window," yet nothing enforces it.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `env.storage().persistent().extend_ttl(&key, threshold, extend_to)` calls after every write to a long-lived slot (`Admin`, `PendingAdmin`, `Pair`, `PairFeeBps`, `PairMin/MaxAmount`, `PairLiquidity`, `FeeRecipient`, `TotalRoutesAllTime`, `SchemaVersion`).
- Define named TTL constants (e.g. `INSTANCE_BUMP_THRESHOLD`, `INSTANCE_BUMP_AMOUNT`) so the policy is centralized and documented.
- Bump TTL on hot read paths too where appropriate (`compute_route_fee`, `get_pair_info`) so frequently-touched pairs stay alive.
- Do not change any existing return values, events, or error codes.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-02-storage-ttl-bumping`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — TTL constants plus `extend_ttl` calls on every persistent slot.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — use `env.ledger().set_*` / TTL test utilities to assert entries survive past the default expiry after a bump.
  - **Add documentation:** add a "Storage & TTL policy" section to [`README.md`](README.md) explaining the archival risk and the chosen thresholds.
  - Include NatSpec-style doc comments (`///`) documenting the TTL constants.
  - Validate security assumptions: no entry can expire under normal operation; TTL math cannot overflow.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: entry survives near-expiry, repeated bumps are idempotent, cold pairs vs hot pairs.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: bump persistent-entry TTL on all router storage access with tests`

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
title: "Add a contract upgrade entrypoint using update_current_contract_wasm with admin gating"
labels: type:feature, area:upgradeability, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement an admin-gated contract upgrade entrypoint

### Description
[`src/lib.rs`](src/lib.rs) ships a `migrate_v1_to_v2` schema migration and a `version()` returning `ROUTER_V2`, but there is **no way to swap the deployed WASM** — the only path to fix a bug is a full redeploy that loses all pair state. Soroban supports in-place code upgrades via `env.deployer().update_current_contract_wasm(new_wasm_hash)`. This issue adds an admin-gated `upgrade(new_wasm_hash: BytesN<32>)` entrypoint so the router can be patched without losing storage.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `upgrade(env, new_wasm_hash: BytesN<32>)` that loads `DataKey::Admin`, calls `admin.require_auth()`, then `env.deployer().update_current_contract_wasm(new_wasm_hash)`.
- Emit an `upgraded` event carrying the new hash for auditability.
- Reject the call when paused is NOT required — but document the trade-off (upgrades should arguably be allowed while paused).
- Keep `RouterError` append-only; reuse `NotInitialized` for the missing-admin path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-03-wasm-upgrade`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `upgrade` entrypoint plus the `upgraded` event.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — register a second WASM in the test env, upgrade, and assert state (admin, a registered pair) survives and the new code path is live.
  - **Add documentation:** add an "Upgrades" section to [`README.md`](README.md) describing the install-WASM → upgrade flow with the Soroban CLI.
  - Include NatSpec-style doc comments (`///`) on the new entrypoint.
  - Validate security assumptions: only admin can upgrade; event records the hash; no storage wiped.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: unauthorized caller, uninitialized contract, upgrade preserves all `DataKey` slots.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add admin-gated upgrade entrypoint via update_current_contract_wasm`

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
title: "Add batch pair registration and batch fee configuration entrypoints"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement batch pair registration and batch fee configuration

### Description
An operator bootstrapping the router today must call [`register_pair`](src/lib.rs) and [`set_pair_fee_bps`](src/lib.rs) once per pair, each a separate signed transaction. There is no bulk path in [`src/lib.rs`](src/lib.rs), so onboarding dozens of corridors is slow and expensive. This issue adds `register_pairs` and `set_pair_fees_bps` that accept a `Vec` of tuples and apply them atomically under a single admin auth.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `register_pairs(env, pairs: Vec<(Symbol, Symbol)>)` reusing the identity-rejection and pause checks of `register_pair`.
- Add `set_pair_fees_bps(env, entries: Vec<(Symbol, Symbol, u32)>)` reusing the `MAX_FEE_BPS` cap of `set_pair_fee_bps`.
- Each batch must be all-or-nothing: a single invalid entry panics the whole call (Soroban transactions are atomic) — document this.
- Emit per-item events identical to the single-item entrypoints so off-chain indexers need no special-casing.
- Cap batch length with a named constant to bound gas; add an append-only `BatchTooLarge` error.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-04-batch-pair-ops`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `register_pairs`, `set_pair_fees_bps`, batch-size constant, and `BatchTooLarge`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert all pairs registered, atomic rollback on a bad entry, and over-limit rejection.
  - **Add documentation:** document the batch entrypoints and their atomicity in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: single admin auth covers the whole batch; identical validation per item; bounded length.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: empty batch, max batch, one invalid entry rolls back all, paused.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add batch pair registration and batch fee configuration entrypoints`

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
title: "Decrement reported pair liquidity on each route and emit a liquidity-consumed event"
labels: type:feature, area:liquidity, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement liquidity decrement on routing

### Description
[`compute_route_fee`](src/lib.rs) reads `DataKey::PairLiquidity` and rejects amounts above it (`InsufficientLiquidity`), but it **never decrements the liquidity** after a successful route. The slot is only ever overwritten wholesale by the admin oracle in `set_pair_liquidity`, so the on-chain liquidity figure does not reflect consumption between oracle updates, allowing repeated routes to exceed real available liquidity. This issue makes `compute_route_fee` (and the on-chain `route` if present) debit `amount` from `PairLiquidity`.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- After the liquidity check in `compute_route_fee`, subtract `amount` from `PairLiquidity` using checked/saturating arithmetic and persist it.
- Skip the decrement when liquidity is the implicit `i128::MAX` "unset" sentinel (treat unset as unbounded) — document this clearly.
- Emit a `liq_used` event with `(source, destination, remaining_liquidity)`.
- Preserve the existing `InsufficientLiquidity` guard and all other validations; bump the slot TTL on write.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-05-liquidity-decrement`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — liquidity debit logic and the `liq_used` event in `compute_route_fee`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert liquidity drops by `amount`, repeated routes eventually hit `InsufficientLiquidity`, and unset liquidity stays unbounded.
  - **Add documentation:** document the consumption model and oracle top-up flow in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the changed function.
  - Validate security assumptions: no underflow; unset sentinel handled; oracle can still top up.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: exact-liquidity route, over-liquidity route, unset liquidity, zero remaining.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: decrement reported pair liquidity on each route with event and tests`

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
title: "Emit events for liquidity, min/max amount, and fee-recipient configuration changes"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Add missing configuration-change events to the router

### Description
Several admin write entrypoints in [`src/lib.rs`](src/lib.rs) mutate persistent state silently: [`set_pair_min_amount`](src/lib.rs), [`set_pair_max_amount`](src/lib.rs), and [`set_fee_recipient`](src/lib.rs) emit **no events**, while their siblings (`set_pair_fee_bps`, `set_pair_liquidity`, `register_pair`) all publish. Off-chain indexers cannot reconstruct the full config history, breaking auditability. This issue adds consistent events to every state-changing admin entrypoint.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add events: `min_set (source, destination, min_amount)`, `max_set (source, destination, max_amount)`, `recip_set (recipient)`.
- Follow the existing `symbol_short!` topic naming convention used by `fee_set` / `liq_set`.
- Do not change any return values, validation, or error codes — events only.
- Ensure topic symbols are ≤ 9 chars (`symbol_short!` constraint).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-06-config-events`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `env.events().publish(...)` calls in the three entrypoints.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert via `env.events().all()` that each entrypoint emits the expected topic and payload.
  - **Add documentation:** add an "Events" reference table to [`README.md`](README.md) listing every topic and payload.
  - Include NatSpec-style doc comments (`///`) noting the emitted event on each function.
  - Validate security assumptions: events carry no secrets; payloads match other entrypoints' shape.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: assert exact event count, topic, and data tuple per call.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: emit events for liquidity, min/max, and fee-recipient changes with tests`

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
title: "Add a remove_pair_fee entrypoint and clear pair config on unregister_pair"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement fee removal and full config cleanup on unregister

### Description
The doc comment on [`unregister_pair`](src/lib.rs) explicitly notes it "does not touch the configured fee — that is removed only when the admin overwrites it back to 0 (or calls a future remove_fee)." That `remove_fee` does not exist, so unregistering a pair leaves orphaned `PairFeeBps`, `PairMinAmount`, `PairMaxAmount`, `PairLiquidity`, and `PairLastRouteAt` entries that consume rent and can resurface stale config if the pair is re-registered. This issue adds `remove_pair_fee` and makes `unregister_pair` optionally purge all per-pair slots.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `remove_pair_fee(env, source, destination)` (admin-gated) that removes `DataKey::PairFeeBps` and emits a `fee_rm` event.
- Add `purge_pair(env, source, destination)` (or extend `unregister_pair` with a flag) that removes all per-pair slots in one call and emits `pair_purge`.
- Keep `unregister_pair` backward compatible (its current behavior stays the default).
- Keep `RouterError` append-only; reuse `NotInitialized` for the missing-admin path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-07-pair-config-cleanup`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `remove_pair_fee`, `purge_pair`, and the new events.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert fee resets to 0, purge clears every slot, and re-registration starts clean.
  - **Add documentation:** document the cleanup semantics and rent implications in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: only admin can purge; no slot leaks after purge.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: purge of never-registered pair, idempotent purge, fee removal then re-set.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add remove_pair_fee and full per-pair config purge on unregister`

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
title: "Add a paginated pair enumeration index so all registered corridors are discoverable on-chain"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement an on-chain pair enumeration index

### Description
The router stores each pair under a keyed slot `DataKey::Pair(Symbol, Symbol)` in [`src/lib.rs`](src/lib.rs), but there is **no way to list all registered pairs** — a client must already know the `(source, destination)` symbols to call `is_pair_registered` or `get_pair_info`. There is no registry/index. This issue adds a maintained `Vec` index plus a paginated read so dashboards can enumerate every corridor without off-chain log scraping.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `DataKey::PairIndex` storing a `Vec<(Symbol, Symbol)>`; push on `register_pair`, remove on `unregister_pair`.
- Add `list_pairs(env, start: u32, limit: u32) -> Vec<(Symbol, Symbol)>` returning a bounded page.
- Add `pair_count(env) -> u32`.
- Guard against unbounded growth: cap `limit` with a constant; keep the index consistent on idempotent register (no duplicates).
- Bump the index slot TTL on write.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-08-pair-enumeration`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::PairIndex`, index maintenance in register/unregister, `list_pairs`, `pair_count`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert pagination boundaries, dedup on idempotent register, and removal on unregister.
  - **Add documentation:** document enumeration and pagination in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: bounded reads, no duplicate entries, index stays in sync with `Pair` slots.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: empty index, start beyond length, limit over cap, register/unregister churn.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add paginated on-chain pair enumeration index with tests`

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
title: "Add a global default fee fallback used when a registered pair has no explicit fee"
labels: type:feature, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement a configurable global default fee

### Description
[`compute_route_fee`](src/lib.rs) and [`get_pair_fee_bps`](src/lib.rs) treat a pair with no explicit `PairFeeBps` as **free** (`unwrap_or(0)`). There is no protocol-wide default, so every new corridor must have its fee set individually or it routes at zero cost. This issue adds an admin-set `DefaultFeeBps` that `compute_route_fee` falls back to when a pair has no per-pair fee.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `DataKey::DefaultFeeBps`; add `set_default_fee_bps` (admin-gated, capped at `MAX_FEE_BPS`) and `get_default_fee_bps`.
- In `compute_route_fee` and `quote_route`, resolve fee as: per-pair fee if set, else default fee, else 0.
- Distinguish "explicitly set to 0" from "unset" so an operator can make a specific pair free even when a non-zero default exists (e.g. store `Option<u32>` or a separate "has fee" flag).
- Emit a `def_fee` event; keep `RouterError` append-only.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-09-default-fee`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::DefaultFeeBps`, setters/getters, and fallback resolution in `compute_route_fee`/`quote_route`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert default applies when unset, per-pair overrides default, and explicit-zero overrides a non-zero default.
  - **Add documentation:** document the fee-resolution precedence in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: default capped at `MAX_FEE_BPS`; precedence is unambiguous.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: default unset, default at cap, per-pair override, explicit free pair.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add configurable global default fee fallback with tests`

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
title: "Support bidirectional pair registration with an explicit canonical ordering option"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement bidirectional pair registration

### Description
`register_pair` in [`src/lib.rs`](src/lib.rs) treats direction as significant — its test `test_register_pair_round_trip` confirms that registering `USDC→EURC` leaves `EURC→USDC` unregistered. Most routing corridors are economically symmetric, forcing operators to register, fee, and fund both directions separately and risking config drift between the two. This issue adds an opt-in `register_pair_bidirectional` that registers both directions atomically and a helper to keep fees mirrored.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `register_pair_bidirectional(env, a, b)` that registers `(a,b)` and `(b,a)` under one admin auth, rejecting `a == b`.
- Add `set_pair_fee_bps_bidirectional(env, a, b, fee_bps)` mirroring fees both ways.
- Do not change the existing directional entrypoints; the bidirectional ones are pure additions built on them.
- Emit the existing `pair_reg` / `fee_set` events for both directions.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-10-bidirectional-pairs`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `register_pair_bidirectional`, `set_pair_fee_bps_bidirectional`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert both directions registered/feed, identity rejection, and events for both.
  - **Add documentation:** document directional vs bidirectional semantics in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: identity rejection preserved; single auth covers both writes.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: identity pair, idempotent both-direction register, fee mirroring.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: support bidirectional pair registration and mirrored fees with tests`

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
title: "Gate set_pair_liquidity and set_pair_min/max_amount behind the pause switch"
labels: type:enhancement, area:pause, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden the pause gate to cover all state-changing entrypoints

### Description
The `Paused` doc comment in [`src/lib.rs`](src/lib.rs) states "No write entrypoint accepts calls until an unpause," but the gate is only actually enforced in `register_pair` and `set_pair_fee_bps`. The admin config entrypoints [`set_pair_liquidity`](src/lib.rs), [`set_pair_min_amount`](src/lib.rs), [`set_pair_max_amount`](src/lib.rs), [`set_fee_recipient`](src/lib.rs), and [`unregister_pair`](src/lib.rs) all execute freely while paused, contradicting the documented invariant. This issue adds the pause check uniformly via a shared helper.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Extract a private `require_not_paused(env)` helper and call it at the top of every state-changing entrypoint.
- Decide and document which entrypoints are intentionally exempt (e.g. `pause`, `unpause`, `upgrade`, admin transfer accept) so the policy is explicit.
- Reuse `RouterError::ContractPaused`; do not renumber any error.
- Keep `compute_route_fee` behavior consistent with the documented intent (clarify whether routing is gated by pause).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-11-pause-gate-coverage`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `require_not_paused` helper and its call sites.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert each newly-gated entrypoint panics with `ContractPaused` while paused and succeeds after `unpause`.
  - **Add documentation:** add a "Pause semantics" table to [`README.md`](README.md) listing gated vs exempt entrypoints.
  - Include NatSpec-style doc comments (`///`) noting the pause gate on each function.
  - Validate security assumptions: no write path bypasses the gate; exemptions are deliberate and safe.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: each entrypoint while paused, idempotent pause/unpause, exempt entrypoints still work while paused.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: enforce pause gate on all state-changing entrypoints with tests`

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
title: "Validate that min_amount does not exceed max_amount when configuring pair bounds"
labels: type:feature, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Implement cross-validation of pair min/max amount bounds

### Description
[`set_pair_min_amount`](src/lib.rs) and [`set_pair_max_amount`](src/lib.rs) in [`src/lib.rs`](src/lib.rs) validate only their own sign, never each other. An admin can set `min > max`, which makes `compute_route_fee` reject **every** amount (it must be both `>= min` and `<= max`) and silently bricks the corridor. This issue cross-validates the two bounds so an inconsistent configuration is rejected at write time.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- In `set_pair_min_amount`, read the current `PairMaxAmount` (default `i128::MAX`) and reject if the new `min` exceeds it.
- In `set_pair_max_amount`, read the current `PairMinAmount` (default `0`) and reject if the new `max` is below it.
- Add an append-only error e.g. `InvalidAmountBounds`.
- Preserve existing sign checks and events.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-12-bounds-cross-validation`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — cross-validation in both setters plus `InvalidAmountBounds`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert `min > max` is rejected from either entrypoint, and valid `min == max` is accepted.
  - **Add documentation:** document the bounds invariant in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the changed functions.
  - Validate security assumptions: no corridor can be silently bricked by inconsistent bounds.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: min above max, max below min, equal bounds, unset opposite bound.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: cross-validate pair min/max amount bounds with tests`

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
title: "Add full test coverage for the two-step admin transfer lifecycle"
labels: type:test, area:admin, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the two-step admin transfer lifecycle

### Description
The two-step handover (`propose_admin_transfer`, `accept_admin_transfer`, `cancel_admin_transfer`, `get_pending_admin`) in [`src/lib.rs`](src/lib.rs) is entirely **untested** — the `#[cfg(test)] mod test` block has no case touching admin rotation. This is the most security-sensitive flow in the contract (a bug here can lock out or hijack the admin), yet errors `NoPendingAdminTransfer` (7) and `NotPendingAdmin` (8) have zero coverage. This issue adds a complete test suite for the lifecycle.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: successful propose→accept rotation, `adm_set` event emission, accept by wrong caller (`NotPendingAdmin`), accept with no pending (`NoPendingAdminTransfer`), cancel then accept fails, and old admin loses authority after rotation.
- Use the existing `setup_initialized` helper and `Address::generate`.
- Assert events with `env.events().all()` where the entrypoint publishes.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-13-admin-transfer-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a bug is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the full lifecycle suite described above with `#[should_panic(expected = "Error(Contract, #7)")]` / `#8` annotations.
  - **Add documentation:** note the admin-transfer test matrix in [`README.md`](README.md) or a `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: post-rotation, the old admin cannot call admin-gated entrypoints.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: wrong caller, no pending, cancel race, re-propose after cancel.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover the two-step admin transfer lifecycle end to end`

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
title: "Add test coverage for pause and unpause gating across state-changing entrypoints"
labels: type:test, area:pause, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test pause and unpause gating

### Description
The pause subsystem (`pause`, `unpause`, `is_paused`) and the `ContractPaused` (9) error in [`src/lib.rs`](src/lib.rs) have **no tests**. The existing suite never sets the contract paused, so the gates in `register_pair` and `set_pair_fee_bps` are unverified and any regression would pass CI silently. This issue adds dedicated coverage for the pause/unpause flow and the gated entrypoints.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: `is_paused` default false, `pause` flips it true and emits `paused`, `unpause` flips it false, gated entrypoints panic with `ContractPaused` (#9) while paused, and succeed after `unpause`.
- Assert event payloads via `env.events().all()`.
- Use `setup_initialized`; add `#[should_panic(expected = "Error(Contract, #9)")]` cases.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-14-pause-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the pause/unpause suite described above.
  - **Add documentation:** note the pause test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: paused contract rejects every gated write; unpause restores access.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: double pause, double unpause, gated entrypoint while paused, event assertions.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover pause and unpause gating across entrypoints`

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
title: "Add test coverage for min/max amount and liquidity guards in compute_route_fee"
labels: type:test, area:liquidity, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test min/max amount and liquidity guards

### Description
`compute_route_fee` in [`src/lib.rs`](src/lib.rs) enforces `AmountBelowMin` (10), `AmountAboveMax` (11), and `InsufficientLiquidity` (12), but the test module only exercises the basic-fee, unset-fee, unregistered, and zero-amount paths. The three boundary guards plus the setters `set_pair_min_amount` / `set_pair_max_amount` / `set_pair_liquidity` are **uncovered**. This issue adds boundary tests for all three guards.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: amount exactly at min (accepted) vs below min (#10), exactly at max (accepted) vs above max (#11), amount at vs above reported liquidity (#12), and unset bounds (min=0, max=i128::MAX, liquidity unset = unbounded).
- Assert the round-trip getters `get_pair_min_amount` / `get_pair_max_amount` / `get_pair_liquidity`.
- Verify `set_pair_liquidity` rejects negative input (`AmountMustBePositive`, #6).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-15-bounds-liquidity-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — boundary cases with `#[should_panic(expected = "Error(Contract, #10)")]` etc.
  - **Add documentation:** note the boundary test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no off-by-one at the bounds; unset sentinels behave as documented.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: at-bound, below/above bound, unset bound, negative liquidity.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover min/max and liquidity guards in compute_route_fee`

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
title: "Add overflow and saturating-arithmetic tests for fee computation near i128 bounds"
labels: type:test, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test fee-computation arithmetic at extreme amounts

### Description
The fee math in `compute_route_fee` / `quote_route` in [`src/lib.rs`](src/lib.rs) uses `checked_mul(fee_bps as i128).map(|n| n / BPS_DENOMINATOR).unwrap_or(0)` and the route counter uses `saturating_add(1)`. These overflow-defense paths are **never exercised** — no test routes an amount near `i128::MAX` or drives `TotalRoutesAllTime` toward saturation. This issue adds arithmetic edge-case tests so the safety code is provably correct.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: amount near `i128::MAX` with a non-zero fee returns 0 (checked_mul overflow path), truncating division (e.g. amount that does not divide evenly), and `quote_route` parity with `compute_route_fee` fee value.
- Cover the counter: assert `get_total_routes_all_time` increments per route and never panics.
- Use a high `max_amount` and matching liquidity so the boundary guards do not pre-empt the arithmetic path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-16-fee-arithmetic-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a bug is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — extreme-amount and truncation cases.
  - **Add documentation:** note the arithmetic invariants in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no panic on overflow; truncation matches documented integer-division semantics.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: i128::MAX amount, non-divisible amount, zero fee, counter increments.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover fee-computation overflow and truncation at i128 bounds`

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
title: "Add test coverage for the schema migration path and get_schema_version defaults"
labels: type:test, area:migration, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the schema migration path

### Description
`migrate_v1_to_v2`, `get_schema_version`, and the `MigrationVersionMismatch` (13) error in [`src/lib.rs`](src/lib.rs) are **untested**. There is no case asserting the v1→v2 transition, the admin-auth requirement, the default-of-1 fallback, or the double-migration rejection. A regression here could brick future upgrades. This issue adds a focused migration suite.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: `get_schema_version` returns 1 before migration, `migrate_v1_to_v2` stamps 2, a second migration panics with `MigrationVersionMismatch` (#13), and migration requires admin auth.
- Cover the `NotInitialized` (#2) path when migrate is called before `init`.
- Use `setup_initialized` and `Address::generate`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-17-migration-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a bug is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the migration suite described above.
  - **Add documentation:** document the migration test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: only admin migrates; idempotency is rejected, not silently re-run.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: default version, single migrate, double migrate, pre-init migrate.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover schema migration path and version defaults`

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
title: "Add test coverage for get_pair_info aggregate read and quote_route parity"
labels: type:test, area:pairs, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test the aggregate pair-info read and quote_route

### Description
`get_pair_info`, `is_pair_active`, `quote_route`, `get_pair_last_route_at`, and `get_total_routes_all_time` in [`src/lib.rs`](src/lib.rs) are read entrypoints with **no test coverage**. `get_pair_info`'s default behavior (e.g. `max_amount` defaulting to `i128::MAX`, `last_route_at` to 0) and `quote_route`'s non-mutating contract are unverified. This issue adds tests for the read surface.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Cover: `get_pair_info` returns correct defaults for an unconfigured pair and correct values after configuration.
- Cover: `is_pair_active` true only when registered AND liquidity > 0.
- Cover: `quote_route` returns the same fee as `compute_route_fee` but does **not** change `get_total_routes_all_time` or `get_pair_last_route_at`.
- Cover: `get_pair_last_route_at` is `None` before any route and `Some(timestamp)` after, using `env.ledger().set_timestamp`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-18-read-surface-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the read-surface suite described above.
  - **Add documentation:** note the read-surface test matrix in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: `quote_route` is provably non-mutating; defaults match doc comments.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: unconfigured pair defaults, active vs inactive, quote vs compute parity, timestamp before/after.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: cover get_pair_info aggregate read and quote_route parity`

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
title: "Add authorization tests asserting non-admin callers are rejected on every admin entrypoint"
labels: type:test, area:admin, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Test authorization on every admin-gated entrypoint

### Description
Every test in [`src/lib.rs`](src/lib.rs) calls `env.mock_all_auths()` in `setup_initialized`, so **no test ever asserts that an unauthorized caller is rejected**. The `require_auth()` guards on `register_pair`, `set_pair_fee_bps`, `set_pair_liquidity`, `pause`, `propose_admin_transfer`, etc., are entirely unverified against a wrong signer. This issue adds negative-auth tests using scoped/required auth mocking.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Use `env.mock_auths(&[...])` (or `set_auths` / `mock_all_auths_allowing_non_root_auth` patterns) to authorize only a non-admin address and assert the admin entrypoints panic on the unmet `require_auth`.
- Cover at least: `register_pair`, `unregister_pair`, `set_pair_fee_bps`, `set_pair_liquidity`, `set_pair_min_amount`, `set_pair_max_amount`, `set_fee_recipient`, `pause`, `unpause`, `propose_admin_transfer`, `migrate_v1_to_v2`.
- Document the auth-testing pattern so future entrypoints follow it.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-19-authorization-tests`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no production change expected (open a follow-up if a gap is found).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — a negative-auth case per admin entrypoint.
  - **Add documentation:** document the auth-testing approach in [`README.md`](README.md) or `docs/testing.md`.
  - Include NatSpec-style doc comments (`///`) on any new test helper.
  - Validate security assumptions: no admin entrypoint succeeds without the admin's auth.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: wrong signer per entrypoint, missing auth, correct signer still works.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: assert non-admin callers are rejected on all admin entrypoints`

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
title: "Harden init against unauthenticated front-running of the admin slot"
labels: type:security, area:admin, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden init against admin front-running

### Description
`init` in [`src/lib.rs`](src/lib.rs) takes an arbitrary `admin: Address` and calls `admin.require_auth()` — but because the caller chooses the address, anyone who observes a freshly deployed-but-uninitialized router can call `init` with **their own** address and seize the admin role before the legitimate operator. There is no deployer binding or constructor-time initialization. This issue closes the init front-running window.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Prefer a Soroban constructor (`#[contractimpl]` `__constructor` / `register(Contract, (admin,))`) so admin is set atomically at deploy time and `init` can be removed or made a no-op.
- If keeping `init`, bind it to the deployer or a deploy-time salt/auth so an attacker cannot pre-empt it.
- Preserve `AlreadyInitialized` (#1) semantics and the `init` event.
- Document the secure deployment procedure.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-20-init-frontrun-hardening`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — constructor-based or deployer-bound initialization.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert admin is set at deploy, a second init is rejected, and an attacker cannot seize the role.
  - **Add documentation:** add a "Secure deployment" section to [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the constructor/init.
  - Validate security assumptions: no window exists where an unrelated caller can claim admin.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: deploy sets admin, double init, attacker init attempt.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: harden router initialization against admin front-running`

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
title: "Add a timelock delay to admin handover and high-impact configuration changes"
labels: type:security, area:admin, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden governance with a timelock on sensitive changes

### Description
Admin handover (`propose_admin_transfer` / `accept_admin_transfer`) and config changes (`set_pair_fee_bps`, `set_fee_recipient`, `upgrade`) in [`src/lib.rs`](src/lib.rs) take effect **instantly** once authorized. A compromised admin key can rotate control or redirect fees in a single ledger with no warning window for users or watchers. This issue adds an optional timelock so high-impact actions are queued and only executable after a configurable delay.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add a `DataKey::Timelock` (delay in seconds) settable by admin, and `DataKey::PendingAction(...)` storing a queued action plus its earliest-execution `ledger().timestamp()`.
- Make `accept_admin_transfer` (and optionally fee-recipient / upgrade) honor the delay; reject early execution with an append-only `TimelockNotElapsed` error.
- Emit `queued` and `executed` events with the eta.
- Allow `cancel_admin_transfer` to also clear a queued action.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-21-governance-timelock`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — timelock storage, queue/execute logic, new error and events.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — advance `env.ledger().set_timestamp` to assert early-execute fails and post-delay execute succeeds.
  - **Add documentation:** document the timelock model and recommended delay in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: no sensitive action bypasses the delay; cancellation works.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: execute before eta, execute after eta, cancel queued, zero-delay config.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add governance timelock to admin handover and high-impact changes`

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
title: "Introduce role separation so liquidity oracle updates do not require the full admin key"
labels: type:security, area:access-control, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden access control with an oracle role

### Description
Every privileged write in [`src/lib.rs`](src/lib.rs) is gated on the single `DataKey::Admin` key. The `PairLiquidity` doc comment says liquidity is "Updated by an off-chain oracle via the admin entrypoint," which forces the hot, frequently-rotated oracle to hold the same key that can rotate admin, set fees, and upgrade the contract. This conflates a low-trust feed with full governance. This issue adds a dedicated oracle role limited to liquidity updates.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `DataKey::Oracle`; add `set_oracle` (admin-gated) and make `set_pair_liquidity` accept either the admin or the oracle.
- Keep all other entrypoints admin-only; the oracle must NOT be able to set fees, pause, rotate admin, or upgrade.
- Emit an `oracle_set` event; keep `RouterError` append-only.
- Document the trust model and least-privilege rationale.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-22-oracle-role`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — `DataKey::Oracle`, `set_oracle`, and the dual-auth check in `set_pair_liquidity`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert oracle can update liquidity, oracle cannot touch fees/pause/admin, and admin can still do everything.
  - **Add documentation:** add a "Roles & least privilege" section to [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the new entrypoints.
  - Validate security assumptions: oracle scope is strictly limited; admin remains the only governance key.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: oracle updates liquidity, oracle blocked elsewhere, oracle unset, admin override.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`feat: add scoped oracle role for liquidity updates with tests`

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
title: "Enforce that compute_route_fee and quote_route are blocked while the router is paused"
labels: type:security, area:pause, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden routing against execution during a pause

### Description
The `Paused` flag in [`src/lib.rs`](src/lib.rs) is described as an emergency stop, yet `compute_route_fee` — the entrypoint that mutates `TotalRoutesAllTime`, stamps `PairLastRouteAt`, and emits the `route` event — has **no pause check**. If a vulnerability is discovered and the admin pauses, routing accounting still proceeds, defeating the emergency stop's purpose. This issue gates the route path on the pause flag.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add the `ContractPaused` (#9) guard at the top of `compute_route_fee` (and any on-chain `route` entrypoint).
- Decide and document whether the read-only `quote_route` should also be blocked when paused (default: keep quotes available, block state-mutating routes).
- Reuse the existing pause-check pattern; do not renumber errors.
- Add tests proving routing is blocked while paused and resumes after unpause.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-23-pause-routing-gate`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — pause guard in `compute_route_fee`.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — assert `compute_route_fee` panics with `ContractPaused` while paused and succeeds after unpause.
  - **Add documentation:** clarify the emergency-stop guarantee in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) noting the pause gate.
  - Validate security assumptions: no route can be recorded while paused.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: route while paused, route after unpause, quote while paused (per chosen policy).
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`fix: block compute_route_fee while the router is paused with tests`

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
title: "Add a fuzz/property-test harness for fee math and bounds invariants"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Harden fee math with property-based testing

### Description
The fee and bounds logic in `compute_route_fee` / `quote_route` in [`src/lib.rs`](src/lib.rs) is only checked with a handful of hand-picked example tests. Subtle issues — fee exceeding amount, truncation asymmetry, off-by-one at min/max boundaries, overflow near `i128::MAX` — are exactly the class of bugs example tests miss. This issue adds a property-based harness asserting invariants hold across a wide input space.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add `proptest` (or `quickcheck`) as a dev-dependency in [`Cargo.toml`](Cargo.toml).
- Assert invariants: `fee <= amount` for any `fee_bps <= MAX_FEE_BPS`; `fee == 0` when `fee_bps == 0`; `quote_route` fee equals `compute_route_fee` fee for identical config; no panic across the valid amount range.
- Keep the existing example tests; the property tests are additive.
- Ensure the harness builds with `cargo test` and stays deterministic in CI (fixed seed if needed).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-24-fee-proptest`
- Implement changes
  - **Write code in:** [`Cargo.toml`](Cargo.toml) — add the property-testing dev-dependency.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — the proptest invariants described above.
  - **Add documentation:** document how to run and extend the property tests in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) on the invariant tests.
  - Validate security assumptions: invariants hold across the generated input space; CI is deterministic.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: full fee_bps range, full amount range, zero fee, boundary amounts.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`test: add property-based fuzz harness for fee math invariants`

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
title: "Document the full RouterError code table and stability guarantees"
labels: type:docs, area:errors, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document the RouterError code table

### Description
[`src/lib.rs`](src/lib.rs) defines thirteen `RouterError` variants (codes 1–13) with an append-only stability promise ("never reuse or renumber a variant once it has shipped"), but [`README.md`](README.md) documents none of them. Off-chain SDK and frontend developers must read the contract source to map a `Error(Contract, #N)` panic to a meaning. This issue adds a complete, authoritative error reference.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add a table to [`README.md`](README.md): code, variant name, the entrypoint(s) that raise it, and the operator-facing meaning/remedy.
- State the append-only guarantee and what it means for client integrations.
- Keep the table in sync with the source by referencing line/variant names; add a note to update it when new errors are appended.
- No production code change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-25-error-code-table`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — optionally tighten `///` comments on any under-documented variant (no behavior change).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones still pass.
  - **Add documentation:** the error reference table in [`README.md`](README.md) (and optionally `docs/errors.md`).
  - Include NatSpec-style doc comments (`///`) consistency check across all variants.
  - Validate security assumptions: documented remedies do not leak sensitive operational detail.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify every code 1–13 appears exactly once in the table.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add authoritative RouterError code reference table`

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
title: "Document the complete entrypoint and event reference for the StableRoute router"
labels: type:docs, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document the entrypoint and event reference

### Description
[`README.md`](README.md) describes the router only as a "placeholder" and lists no entrypoints or events, even though [`src/lib.rs`](src/lib.rs) exposes ~25 entrypoints and emits topics like `init`, `pair_reg`, `fee_set`, `liq_set`, `route`, `paused`, `adm_prop`, and `adm_set`. Integrators have no on-chain ABI reference. This issue produces a complete entrypoint and event catalog.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Document every public entrypoint: signature, auth requirement, parameters, return value, errors raised, and event emitted.
- Document every event: topic symbol, payload tuple shape, and emitting entrypoint.
- Group by subsystem (admin/governance, pairs, fees, liquidity, routing, lifecycle).
- Keep the README's existing setup/CI sections; add the reference as new sections (and optionally `docs/abi.md`).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-26-entrypoint-event-reference`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — optionally align any stale `///` comment with documented behavior (no behavior change).
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** the entrypoint + event catalog in [`README.md`](README.md) / `docs/abi.md`.
  - Include NatSpec-style doc comments (`///`) consistency across entrypoints.
  - Validate security assumptions: docs accurately reflect auth and pause behavior.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify each documented event topic matches a `symbol_short!` in the source.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add complete entrypoint and event reference for the router`

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
title: "Write a SECURITY.md threat model and responsible-disclosure policy for the router"
labels: type:docs, area:security, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document the router threat model and disclosure policy

### Description
The repository has no `SECURITY.md` and no documented threat model, despite the contract holding governance power (admin rotation, pause, fee control) and being on a roadmap to custody routed funds. Contributors and auditors have no guidance on trust assumptions or how to report a vulnerability. This issue creates a SECURITY.md grounded in the actual surfaces in [`src/lib.rs`](src/lib.rs).

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Create `SECURITY.md` covering: trust model (single admin, two-step transfer, pause), known limitations (no fund movement yet, no TTL bumping, instant config changes), and the responsible-disclosure process with a contact.
- Reference concrete entrypoints/`DataKey`s and the append-only `RouterError` policy.
- Link the StableRoute Discord for coordinated disclosure: https://discord.gg/37aCpusvx
- No production code change.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-27-security-md`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; optionally cross-link doc comments to SECURITY.md.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** create `SECURITY.md` and link it from [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) consistency check on security-relevant entrypoints.
  - Validate security assumptions: the documented model matches actual code behavior.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: ensure every documented trust assumption maps to real code.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: add SECURITY.md threat model and disclosure policy`

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
title: "Update README to replace the placeholder description with the real router architecture"
labels: type:docs, area:readme, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Document the real router architecture in the README

### Description
[`README.md`](README.md) still calls the contract a "Soroban contract placeholder for routing metadata" and points the project link at `your-org/stableroute`, but [`src/lib.rs`](src/lib.rs) is now a substantial router with pair registration, per-pair fees, min/max bounds, liquidity gating, two-step admin transfer, pause, and schema migration. The README understates the contract and misleads new contributors. This issue rewrites the architecture section to match reality.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Replace the placeholder "What this repo contains" with an accurate description of the router's subsystems and storage model (`DataKey` variants and their TTL class).
- Fix the broken/placeholder project link and the `git clone` directory name.
- Add a short architecture diagram or component list (storage, governance, routing).
- Keep the setup/commands/CI sections; do not remove working content.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-28-readme-architecture`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — no behavior change; ensure top-of-file `///` matches the README narrative.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — no new tests required; ensure existing ones pass.
  - **Add documentation:** the rewritten architecture section in [`README.md`](README.md).
  - Include NatSpec-style doc comments (`///`) alignment with documented architecture.
  - Validate security assumptions: README does not overstate guarantees the code does not provide.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: verify documented subsystems all exist in the source.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`docs: replace placeholder README with real router architecture`

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
title: "Extract a require_admin helper to remove duplicated admin-auth boilerplate"
labels: type:refactor, area:access-control, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Refactor duplicated admin-auth boilerplate into a helper

### Description
The exact six-line pattern that loads `DataKey::Admin`, falls back to `panic_with_error!(NotInitialized)`, and calls `admin.require_auth()` is copy-pasted into roughly a dozen entrypoints in [`src/lib.rs`](src/lib.rs) (`pause`, `unpause`, `register_pair`, `set_pair_fee_bps`, `set_pair_liquidity`, `set_pair_min_amount`, `set_pair_max_amount`, `set_fee_recipient`, `unregister_pair`, `propose_admin_transfer`, `cancel_admin_transfer`, `migrate_v1_to_v2`). This duplication is error-prone — a missed `require_auth` would be a critical bug. This issue centralizes it.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add a private `fn require_admin(env: &Env) -> Address` that loads the admin, panics with `NotInitialized` if absent, calls `require_auth`, and returns the address.
- Replace every duplicated block with a single call.
- This is a behavior-preserving refactor: no change to events, errors, or return values; all existing tests must pass unchanged.
- Keep the function private (`#[cfg]`-independent) so it does not appear in the generated client.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-29-require-admin-helper`
- Implement changes
  - **Write code in:** [`src/lib.rs`](src/lib.rs) — the `require_admin` helper and all call-site replacements.
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — confirm existing tests pass; add a case asserting auth is still required after the refactor.
  - **Add documentation:** note the helper convention in [`README.md`](README.md) or a `CONTRIBUTING.md`.
  - Include NatSpec-style doc comments (`///`) on the helper.
  - Validate security assumptions: every previously-gated entrypoint is still gated identically.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`.
- Cover edge cases and failure paths: uninitialized contract, unauthorized caller, authorized caller across entrypoints.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`refactor: extract require_admin helper to dedupe auth boilerplate`

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
title: "Add clippy, WASM target build, and coverage gate to the CI workflow"
labels: type:refactor, area:ci, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN
assignees: ''
---

## Refactor CI to add clippy, a WASM build, and a coverage gate

### Description
The CI workflow in [`.github/workflows/ci.yml`](.github/workflows/ci.yml) only runs `cargo fmt --all -- --check`, `cargo build`, and `cargo test` on the host target. It never runs `clippy`, never builds the actual `wasm32-unknown-unknown` artifact that gets deployed (so a WASM-only build break would pass CI), and has no coverage gate to back the "95 percent coverage" expectation in these bounties. This issue hardens the pipeline.

### Requirements and context
- **Repository scope:** `StableRoute-Org/Stableroute-contracts` only.
- Add a `cargo clippy --all-targets -- -D warnings` step.
- Add `rustup target add wasm32-unknown-unknown` and `cargo build --target wasm32-unknown-unknown --release` so the deployable artifact is verified (the crate is `cdylib`).
- Add a coverage step (e.g. `cargo llvm-cov`) and surface the percentage; optionally fail under a threshold.
- Keep the existing fmt/build/test steps and the `Swatinem/rust-cache` cache.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-30-ci-hardening`
- Implement changes
  - **Write code in:** [`.github/workflows/ci.yml`](.github/workflows/ci.yml) — clippy, WASM target build, and coverage steps. (Touch [`src/lib.rs`](src/lib.rs) only to fix any clippy lint surfaced.)
  - **Write comprehensive tests in:** [`src/lib.rs`](src/lib.rs) `#[cfg(test)] mod test` — ensure the suite still passes under coverage instrumentation.
  - **Add documentation:** update the "CI/CD" section of [`README.md`](README.md) to list the new gates.
  - Include NatSpec-style doc comments (`///`) on any code touched to satisfy clippy.
  - Validate security assumptions: the deployed WASM artifact is built and checked in CI.
- Test and commit

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, and `cargo test`; also run `cargo clippy` and the WASM build locally.
- Cover edge cases and failure paths: clippy clean, WASM build succeeds, coverage reported.
- Include the full `cargo test` output and a short **security notes** section in the PR description (threat model + mitigations).

### Example commit message
`ci: add clippy, wasm32 build, and coverage gate to the workflow`

### Guidelines
- **Minimum 95 percent test coverage** for impacted modules.
- Clear, reviewer-focused documentation.
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord for questions, reviews, and faster merges:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — if this issue and the maintainers helped you ship, we'd be grateful for a **5-star rating**. Clear questions in Discord and tidy, well-tested PRs are the fastest path to a merge and a reward.

---
type: Feature
title: "Add contractmeta! metadata so the deployed router WASM is self-describing"
labels: type:feature, area:metadata, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add contractmeta! metadata so the deployed router WASM is self-describing

### Description
`src/lib.rs` declares `#[contract] pub struct StableRouteRouter` but never emits a `contractmeta!` block, so the compiled WASM carries no on-chain name, version, or repository hint. Indexers and explorers therefore show the router as an anonymous contract next to `version()` returning only `ROUTER_V2`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `contractmeta!` block with `name`, `version`, `description`, and `repository` keys near the top of `src/lib.rs`.
- Keep the metadata `version` string in sync with `Cargo.toml` `package.version` and document the bump rule.
- Add a test asserting `version()` and `get_schema_version()` still return `ROUTER_V2` and the persisted schema version.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-contract-metadata`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(meta): declare contractmeta for the router WASM`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Include the computed fee and fee_bps in the route event payload"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Include the computed fee and fee_bps in the route event payload

### Description
`compute_route_fee` emits `(symbol_short!("route"),)` with only `(source, destination, amount)`, so an indexer cannot reconstruct what was actually charged — especially once `apply_fee_cap` clamps the raw bps result. The fee is computed after the event is published, which is why it is missing.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Move the `fee_bps` read and fee computation above the `route` event publish in `compute_route_fee`.
- Extend the payload to `(source, destination, amount, fee_bps, fee)` and note the ABI change in `docs/abi.md`.
- Assert the new payload shape in tests, including a case where `apply_fee_cap` clamps the fee.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-route-event-fee`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(events): include fee and fee_bps in the route event`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a per-pair pause switch so one corridor can be halted without pausing the router"
labels: type:feature, area:pause, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a per-pair pause switch so one corridor can be halted without pausing the router

### Description
The only kill switch today is the global `DataKey::Paused` flag checked at the top of `compute_route_fee`. Halting a single misbehaving corridor currently requires `unregister_pair`, which destroys configuration, or a global `pause()` that stops every route.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `DataKey::PairPaused(Symbol, Symbol)` slot with `set_pair_paused` / `is_pair_paused` entrypoints, admin-gated.
- Check the per-pair flag in `compute_route_fee` and `quote_route` alongside the existing global `Paused` check.
- Add a new append-only `RouterError::PairPaused` variant rather than reusing `ContractPaused` (#9).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-per-pair-pause`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(pause): add a per-pair pause switch`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a set_pair_config entrypoint that writes fee, bounds, and cooldown atomically"
labels: type:feature, area:pair-config, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a set_pair_config entrypoint that writes fee, bounds, and cooldown atomically

### Description
Configuring a corridor today needs four separate admin transactions: `set_pair_fee_bps`, `set_pair_min_amount`, `set_pair_max_amount`, and `set_pair_cooldown`. Between them the pair is live with a partially applied configuration.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `set_pair_config(source, destination, fee_bps, min_amount, max_amount, cooldown_secs)` reusing the existing per-setter validation.
- Validate the whole tuple before any write so a rejected call leaves storage untouched.
- Emit a single `pair_cfg` event with the full applied configuration.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-set-pair-config`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(config): add atomic set_pair_config entrypoint`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add an unregister_pairs batch entrypoint mirroring register_pairs"
labels: type:feature, area:batch, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add an unregister_pairs batch entrypoint mirroring register_pairs

### Description
`register_pairs` and `set_pair_fees_bps` accept `Vec` batches bounded by `MAX_BATCH_SIZE`, but `unregister_pair` remains single-shot. Decommissioning a corridor set costs one transaction per pair.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `unregister_pairs(pairs: Vec<(Symbol, Symbol)>)` enforcing `MAX_BATCH_SIZE` and rejecting empty batches with `EmptyBatch` (#19).
- Reuse the same orphan-slot cleanup that `unregister_pair` performs for fee, bounds, liquidity, and cooldown slots.
- Cover partial-failure semantics in tests: one invalid entry must revert the whole batch.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-batch-unregister`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(batch): add unregister_pairs batch entrypoint`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a get_limits read exposing MAX_FEE_BPS, MAX_BATCH_SIZE, and MAX_COOLDOWN_SECS"
labels: type:feature, area:read-api, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a get_limits read exposing MAX_FEE_BPS, MAX_BATCH_SIZE, and MAX_COOLDOWN_SECS

### Description
`MAX_FEE_BPS`, `BPS_DENOMINATOR`, `MAX_BATCH_SIZE`, and `MAX_COOLDOWN_SECS` are Rust `pub const`s only. An on-chain caller or a client that did not compile against the crate cannot discover the limits it must respect before submitting a transaction.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `RouterLimits` `#[contracttype]` struct and a `get_limits` read returning all four constants.
- Keep the struct field order stable and document it in `docs/abi.md`.
- Add a test asserting the returned values equal the compile-time constants.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-get-limits`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(read): expose protocol limits via get_limits`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a protocol-wide cumulative volume counter alongside TotalRoutesAllTime"
labels: type:feature, area:metrics, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a protocol-wide cumulative volume counter alongside TotalRoutesAllTime

### Description
`compute_route_fee` maintains `DataKey::TotalRoutesAllTime` and per-pair `PairVolume`, but there is no protocol-wide volume aggregate. Reconstructing it requires enumerating every pair and summing `get_pair_volume`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::TotalVolumeAllTime` accumulated with `saturating_add` in the effects phase of `compute_route_fee`.
- Add a `get_total_volume_all_time` read defaulting to 0 when absent.
- Assert in tests that a rejected route leaves the new counter unchanged.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-total-volume`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(metrics): track protocol-wide cumulative volume`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a get_router_config aggregate read for paused, timelock, oracle, recipient, and fee cap"
labels: type:feature, area:read-api, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a get_router_config aggregate read for paused, timelock, oracle, recipient, and fee cap

### Description
Reading global router state currently needs five calls: `is_paused`, `get_timelock`, `get_oracle`, `get_fee_recipient`, and `get_max_fee_absolute`. Clients polling the router pay five invocations and can observe a torn view across ledgers.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `RouterConfig` `#[contracttype]` struct and a `get_router_config` read returning a single consistent snapshot.
- Preserve `Option` semantics for oracle, fee recipient, and the absolute fee cap.
- Test the fully unset case returns defaults (`false`, `0`, three `None`s).

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-router-config-read`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(read): add get_router_config aggregate read`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a guardian role that can pause but cannot unpause or change configuration"
labels: type:feature, area:roles, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a guardian role that can pause but cannot unpause or change configuration

### Description
`pause()` is gated by `require_admin`, so incident response requires the full governance key. The scoped `Oracle` role already proves the least-privilege pattern works in this contract via the dual-auth check in `set_pair_liquidity`.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::Guardian` with `set_guardian` / `remove_guardian` / `get_guardian` admin-gated entrypoints.
- Allow the guardian to call `pause()` only; `unpause()` must stay admin-only so a compromised guardian cannot unpause.
- Add authorization tests asserting the guardian is rejected on fees, oracle, admin transfer, and upgrade.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-guardian-role`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(roles): add a pause-only guardian role`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a two-step oracle handover mirroring the admin transfer flow"
labels: type:feature, area:roles, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a two-step oracle handover mirroring the admin transfer flow

### Description
`set_oracle` writes `DataKey::Oracle` in one step, so a typo silently hands the liquidity feed to an address that may not exist. Admin rotation already uses the safer `propose_admin_transfer` / `accept_admin_transfer` pair.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `propose_oracle` / `accept_oracle` / `cancel_oracle_transfer` backed by a `DataKey::PendingOracle` slot.
- Require `require_auth()` from the proposed oracle in `accept_oracle`, mirroring `accept_admin_transfer`.
- Keep `remove_oracle` working as the immediate revocation path and clear any pending proposal.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-two-step-oracle`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(roles): add two-step oracle handover`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a top_up_pair_liquidity entrypoint so the oracle can increment instead of overwrite"
labels: type:feature, area:liquidity, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a top_up_pair_liquidity entrypoint so the oracle can increment instead of overwrite

### Description
`compute_route_fee` debits `PairLiquidity` with `saturating_sub`, but the only way to restore it is `set_pair_liquidity`, which overwrites absolutely. An oracle that reads a stale value can accidentally erase concurrent debits.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `top_up_pair_liquidity(caller, source, destination, delta)` using the same dual admin-or-oracle auth check.
- Reject non-positive deltas with `AmountMustBePositive` (#6) and use `saturating_add` for the accumulation.
- Leave the unbounded `i128::MAX` sentinel semantics documented and covered by a test.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-liquidity-topup`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(liquidity): add incremental top_up_pair_liquidity`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a minimum absolute fee floor so dust routes are not charged zero"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a minimum absolute fee floor so dust routes are not charged zero

### Description
The fee in `compute_route_fee` is `amount * fee_bps / BPS_DENOMINATOR` with integer truncation, so any amount below `10_000 / fee_bps` rounds to a zero fee. An attacker can split a large route into dust legs and pay nothing.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `DataKey::MinFeeAbsolute` with admin-gated `set_min_fee_absolute` / `get_min_fee_absolute` entrypoints.
- Apply the floor after `apply_fee_cap` and document the precedence when floor exceeds cap.
- Test the truncation boundary explicitly for `fee_bps = 1` and amounts around 10 000.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-min-fee-floor`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(fees): add an absolute minimum fee floor`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Cap set_timelock so an absurd delay cannot brick governance permanently"
labels: type:security, area:governance, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Cap set_timelock so an absurd delay cannot brick governance permanently

### Description
`set_timelock` accepts any `u64` delay with no upper bound, unlike `set_pair_cooldown`, which is capped by `MAX_COOLDOWN_SECS`. Setting `u64::MAX` makes every future `accept_admin_transfer` unreachable and permanently freezes admin rotation.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `MAX_TIMELOCK_SECS` constant (for example 30 days) and reject larger delays in `set_timelock`.
- Add an append-only `RouterError::TimelockTooLarge` variant rather than reusing `CooldownTooLarge` (#20).
- Test the boundary: exactly `MAX_TIMELOCK_SECS` is accepted, one second more is rejected.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-timelock-cap`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`fix(security): cap the governance timelock delay`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Require a timelock delay before an upgrade takes effect"
labels: type:security, area:upgrade, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Require a timelock delay before an upgrade takes effect

### Description
`upgrade` calls `require_admin` then immediately invokes `update_current_contract_wasm`, deliberately skipping the pause gate. A single compromised admin key can therefore swap the router logic in one transaction with no observation window for users.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `propose_upgrade(new_wasm_hash)` storing the hash and an eta derived from `DataKey::Timelock`.
- Make `upgrade` consume the queued proposal and reject with `TimelockNotElapsed` (#14) before the eta.
- Add `cancel_upgrade` and emit events for propose, cancel, and execute so watchers can monitor the window.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-upgrade-timelock`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(security): timelock the contract upgrade path`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Move the ReentrancyLock from persistent to temporary storage"
labels: type:security, area:reentrancy, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Move the ReentrancyLock from persistent to temporary storage

### Description
`enter_nonreentrant` and `exit_nonreentrant` write `DataKey::ReentrancyLock` to persistent storage. A lock is only meaningful for the duration of one invocation, so persisting it wastes rent, risks a stuck `true` value surviving across transactions, and pays a persistent-write cost on every route.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Move the lock to `env.storage().temporary()` so it cannot outlive the transaction.
- Add a one-time migration or cleanup path that clears any stale persistent `ReentrancyLock` entry.
- Test back-to-back `compute_route_fee` calls still succeed and the nested-call mock still trips the guard.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-reentrancy-temporary`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`fix(security): move the reentrancy lock to temporary storage`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Make pause and unpause emit an event only when the flag actually changes"
labels: type:enhancement, area:pause, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Make pause and unpause emit an event only when the flag actually changes

### Description
`pause()` and `unpause()` unconditionally write `DataKey::Paused` and publish a `paused` event, even when the flag already held that value. Indexers therefore see duplicate state transitions that never happened.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Read the current flag first and return early without a write or event when the value is unchanged.
- Keep both entrypoints idempotent from the caller's perspective — no new error on a redundant call.
- Test that a second consecutive `pause()` emits no additional event.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-pause-event-dedup`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(pause): emit paused events only on state change`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a nonce parameter to compute_route_fee so client retries are idempotent"
labels: type:feature, area:idempotency, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a nonce parameter to compute_route_fee so client retries are idempotent

### Description
`compute_route_fee` mutates `TotalRoutesAllTime`, `PairRouteCount`, `PairVolume`, and `PairLastRouteAt` on every call. A client that resubmits after a timeout double-counts volume and can trip the `RouteCooldownActive` (#17) gate on a route it believes never landed.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add an optional caller-supplied nonce recorded in a `DataKey::RouteNonce` temporary slot.
- Return the previously computed fee without re-applying effects when the same nonce is replayed.
- Document the retention window and its interaction with the per-pair cooldown.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-route-nonce`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(routing): add nonce-based idempotency to compute_route_fee`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add configurable rounding direction for the basis-point fee division"
labels: type:feature, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add configurable rounding direction for the basis-point fee division

### Description
The fee expression in `compute_route_fee` uses `n / BPS_DENOMINATOR`, which truncates toward zero and always rounds in the payer's favour. There is no way for governance to choose ceiling rounding for corridors where fee leakage matters.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `DataKey::FeeRoundUp` boolean with an admin-gated setter, defaulting to the current floor behaviour.
- Implement ceiling rounding as `(n + BPS_DENOMINATOR - 1) / BPS_DENOMINATOR` guarding against `checked_add` overflow.
- Ensure `quote_route` and `compute_route_fee` apply the identical rounding path.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b feature/contracts-fee-rounding`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(fees): support configurable fee rounding direction`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Reject a max_fee_absolute of zero that silently makes every route free"
labels: type:security, area:fees, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Reject a max_fee_absolute of zero that silently makes every route free

### Description
`set_max_fee_absolute` explicitly documents that "a cap of `0` makes every route effectively free" and only rejects negatives. Because `apply_fee_cap` takes `fee.min(cap)`, a zero cap zeroes protocol revenue across every corridor with no distinct signal.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Reject `max_fee == 0` with a dedicated append-only error, or require an explicit confirmation entrypoint.
- Provide `clear_max_fee_absolute` semantics as the supported way to remove the cap entirely.
- Test that a zero cap is rejected while a cap of 1 still clamps as expected.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b security/contracts-zero-fee-cap`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`fix(security): reject a zero absolute fee cap`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit a config event from set_timelock so governance changes are auditable"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Emit a config event from set_timelock so governance changes are auditable

### Description
`set_timelock` writes `DataKey::Timelock` after `require_admin` but publishes nothing, unlike `set_max_fee_absolute` (`maxfee`), `set_oracle` (`orac_set`), and `remove_oracle` (`orac_rm`). A change to the governance delay is invisible to watchers.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Publish a `tlock_set` event carrying the old and new delay values.
- Keep the topic within the nine-character `symbol_short!` limit, as noted for `orac_set`.
- Assert the event payload in tests including the zero-to-nonzero and nonzero-to-zero transitions.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-timelock-event`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(events): emit an event from set_timelock`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Emit an event from purge_pair_metrics so metric resets are auditable"
labels: type:enhancement, area:events, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Emit an event from purge_pair_metrics so metric resets are auditable

### Description
`purge_pair_metrics` clears per-pair counters but emits no event. Off-chain dashboards that track `get_pair_route_count` and `get_pair_volume` will see a silent jump to zero with nothing on-chain explaining it.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Publish a `metr_rm` event with the pair and the pre-purge route count and volume.
- Document the event in `docs/abi.md` alongside the existing pair lifecycle events.
- Test the payload for both a populated pair and a pair with no recorded metrics.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-purge-metrics-event`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`feat(events): emit an event from purge_pair_metrics`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract the fee computation into a pure fee_for helper shared by quote and compute"
labels: type:refactor, area:fee-math, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Extract the fee computation into a pure fee_for helper shared by quote and compute

### Description
The `amount.checked_mul(fee_bps as i128).map(|n| n / BPS_DENOMINATOR).unwrap_or(0)` expression plus `apply_fee_cap` is duplicated between `compute_route_fee` and `quote_route`. Any future change to rounding or capping has to be applied twice or the two paths silently diverge.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extract a single private `fee_for(env, amount, fee_bps) -> i128` helper and call it from both entrypoints.
- Keep the `checked_mul` overflow fallback and the `apply_fee_cap` clamp inside the helper.
- Add a property test asserting `quote_route` and `compute_route_fee` agree for every input.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-fee-for-helper`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(fees): extract a shared fee_for helper`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Extract per-pair storage read helpers to remove repeated unwrap_or defaults"
labels: type:refactor, area:storage, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Extract per-pair storage read helpers to remove repeated unwrap_or defaults

### Description
`compute_route_fee`, `quote_route`, `get_pair_info`, and `get_pair_info_ext` each repeat the same `storage().persistent().get(&DataKey::PairX(source.clone(), destination.clone())).unwrap_or(default)` pattern. The default sentinels (`0`, `i128::MAX`) are restated at every call site.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add private `read_pair_min`, `read_pair_max`, `read_pair_liquidity`, `read_pair_fee_bps`, and `read_pair_cooldown` helpers owning their defaults.
- Route every existing reader through the helpers so the sentinel values are defined once.
- Assert the sentinel semantics (`i128::MAX` meaning unbounded) in a dedicated test.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-pair-read-helpers`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(storage): centralize per-pair reads and defaults`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Reduce Symbol cloning in compute_route_fee by borrowing keys"
labels: type:refactor, area:performance, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Reduce Symbol cloning in compute_route_fee by borrowing keys

### Description
`compute_route_fee` clones `source` and `destination` more than a dozen times to build `DataKey` variants. Each clone is cheap individually but the accumulated host-object churn shows up in the CPU instruction budget of the hottest entrypoint.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Construct each `DataKey` once into locals where the same key is read then written (liquidity, route count, volume).
- Order the final moves so the last use consumes rather than clones `source` and `destination`.
- Record before/after CPU instruction counts from the test budget output in the PR description.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-reduce-clones`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(perf): reduce Symbol cloning in compute_route_fee`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Group the router impl into clearly delimited governance, config, read, and routing sections"
labels: type:refactor, area:code-organization, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Group the router impl into clearly delimited governance, config, read, and routing sections

### Description
`src/lib.rs` is over 4 000 lines with entrypoints interleaved — `set_fee_recipient` sits between metrics reads and the fee cap helpers, and `route_tag` follows `compute_route_fee`. Navigating the single `#[contractimpl]` block is slow for new contributors.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Reorder the impl into commented sections: lifecycle, governance, roles, pair config, reads, routing, upgrade.
- Do not change any signature, error code, or event so the ABI is byte-for-byte identical.
- Note the section convention in `CONTRIBUTING.md` so future entrypoints land in the right block.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-impl-sections`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(layout): group router entrypoints into labelled sections`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Consolidate the numbered test submodules into behaviour-named modules"
labels: type:refactor, area:tests, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Consolidate the numbered test submodules into behaviour-named modules

### Description
The test tree uses issue-numbered modules such as `test_i14_pause_gating`, `test_i16_fee_arithmetic`, `test_i19_authorization`, and `test_i153_version_uninitialized`. The numbers are meaningless once the issues close and make it hard to find where a behaviour is covered.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Rename the modules to behaviour-first names (`pause_gating`, `fee_arithmetic`, `authorization`, `version_reads`).
- Keep the issue reference as a module-level `//!` doc comment rather than in the identifier.
- Confirm the total test count is unchanged before and after the move.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b refactor/contracts-test-module-names`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`refactor(tests): rename numbered test modules by behaviour`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a test asserting every RouterError discriminant is unique"
labels: type:test, area:errors, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a test asserting every RouterError discriminant is unique

### Description
`RouterError` documents an append-only numbering policy across 20 variants, and a duplicate discriminant has already shipped once in this contract's history. Nothing in the test suite mechanically enforces uniqueness on new variants.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a test collecting every variant as `u32` and asserting no duplicates and no gaps below the maximum.
- Fail the test with a message naming the offending code so the fix is obvious.
- Reference the append-only policy from `CONTRIBUTING.md` in the test doc comment.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-error-code-uniqueness`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(errors): assert RouterError codes are unique`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a boundary test for the cooldown comparison at exactly last plus cooldown"
labels: type:test, area:cooldown, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a boundary test for the cooldown comparison at exactly last plus cooldown

### Description
`compute_route_fee` gates on `env.ledger().timestamp() < last + cooldown`, making the boundary timestamp inclusive — a route at exactly `last + cooldown` is allowed. That off-by-one is easy to invert during a refactor and no test pins it.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Drive `env.ledger().set_timestamp` to `last + cooldown - 1`, `last + cooldown`, and `last + cooldown + 1`.
- Assert the first rejects with `RouteCooldownActive` (#17) and the latter two succeed.
- Also cover `cooldown == 0` disabling the gate and the first-ever route with no recorded timestamp.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-cooldown-boundary`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(cooldown): pin the inclusive cooldown boundary`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for MAX_COOLDOWN_SECS acceptance and rejection"
labels: type:test, area:cooldown, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add boundary tests for MAX_COOLDOWN_SECS acceptance and rejection

### Description
`set_pair_cooldown` rejects values above `MAX_COOLDOWN_SECS` (2 592 000) with `CooldownTooLarge` (#20), and the constant's doc comment relies on that cap to prove the `last + cooldown` addition cannot overflow `u64`. The exact accept/reject edge is untested.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Assert `MAX_COOLDOWN_SECS` is accepted and `MAX_COOLDOWN_SECS + 1` panics with `CooldownTooLarge`.
- Assert `u64::MAX` is rejected and leaves `get_pair_cooldown` at its previous value.
- Add a routing test at the maximum cooldown proving no overflow panic occurs.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-cooldown-max-boundary`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(cooldown): cover MAX_COOLDOWN_SECS boundaries`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add boundary tests for MAX_BATCH_SIZE in register_pairs and set_pair_fees_bps"
labels: type:test, area:batch, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add boundary tests for MAX_BATCH_SIZE in register_pairs and set_pair_fees_bps

### Description
Both batch entrypoints bound input length by `MAX_BATCH_SIZE` (100) and reject with `BatchTooLarge` (#18), plus `EmptyBatch` (#19) for zero-length input. The exact accept/reject edge at 100 versus 101 entries is not pinned by a test.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Assert a batch of exactly `MAX_BATCH_SIZE` entries succeeds for both entrypoints.
- Assert `MAX_BATCH_SIZE + 1` panics with `BatchTooLarge` and writes nothing.
- Assert an empty `Vec` panics with `EmptyBatch` for both entrypoints.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-batch-size-boundary`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(batch): cover MAX_BATCH_SIZE accept and reject edges`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a route_tag regression test pinning known keccak256 digests"
labels: type:test, area:route-tag, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a route_tag regression test pinning known keccak256 digests

### Description
`route_tag` returns `keccak256(xdr(source) || xdr(destination))`. The documented determinism guarantee means an off-chain backend recomputes the digest independently, so any change to the pre-image construction is a silent breaking change for those consumers.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Pin the exact 32-byte digest for at least two fixed pairs as hard-coded expected values.
- Assert the direction-sensitivity property that `route_tag(a, b) != route_tag(b, a)`.
- Document in the test why the constants must never be regenerated to make a failing test pass.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-route-tag-vectors`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(route-tag): pin known keccak256 digest vectors`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add full test coverage for force_admin_transfer authorization and timelock behaviour"
labels: type:test, area:governance, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add full test coverage for force_admin_transfer authorization and timelock behaviour

### Description
`force_admin_transfer` is the single-step recovery escape hatch that bypasses the `propose` / `accept` handshake. Being the most dangerous governance entrypoint in `src/lib.rs`, it needs the strictest coverage of who may call it and when.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Assert a non-admin caller is rejected and the admin slot is unchanged.
- Assert any pending admin and `PendingAdminEta` are cleared by a successful force transfer.
- Assert the emitted event payload and behaviour while the router is paused.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-force-transfer-tests`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(governance): cover force_admin_transfer fully`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add test coverage for purge_pair_metrics clearing counters without touching configuration"
labels: type:test, area:metrics, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add test coverage for purge_pair_metrics clearing counters without touching configuration

### Description
`purge_pair_metrics` sits next to `unregister_pair` and clears per-pair metric slots. Nothing asserts it leaves `PairFeeBps`, `PairMinAmount`, `PairMaxAmount`, `PairLiquidity`, and `PairCooldown` intact, which is the whole point of a separate entrypoint.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Route several times, purge, then assert `get_pair_route_count` and `get_pair_volume` return 0.
- Assert fee, bounds, liquidity, and cooldown reads are byte-identical before and after the purge.
- Assert the entrypoint is admin-gated and idempotent on a never-routed pair.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-purge-metrics-tests`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(metrics): cover purge_pair_metrics isolation`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add test coverage asserting get_pair_info_ext agrees with every individual getter"
labels: type:test, area:read-api, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add test coverage asserting get_pair_info_ext agrees with every individual getter

### Description
`PairInfoExt` duplicates nine fields that are each independently readable via `get_pair_fee_bps`, `get_pair_min_amount`, `get_pair_max_amount`, `get_pair_liquidity`, `get_pair_cooldown`, `get_pair_route_count`, `get_pair_volume`, and `get_pair_last_route_at`. Drift between the aggregate and the singles would be invisible.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Configure a pair with distinct non-default values for every field and assert field-by-field agreement.
- Repeat for a fully unconfigured pair to pin the default sentinels.
- Assert `PairInfo` and `PairInfoExt` agree on their shared base fields.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-pair-info-ext-parity`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(read): assert get_pair_info_ext parity with singles`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add test coverage proving remove_oracle restores admin-only liquidity writes"
labels: type:test, area:roles, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add test coverage proving remove_oracle restores admin-only liquidity writes

### Description
`remove_oracle` documents that removing `DataKey::Oracle` degrades the dual-auth check in `set_pair_liquidity` to admin-only, because `Some(caller)` can never equal `None`. That reasoning is the compromised-key recovery path and deserves a direct test.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Set an oracle, prove it can write liquidity, call `remove_oracle`, then assert the same call panics with `NotAuthorized` (#16).
- Assert the admin can still write liquidity after revocation.
- Assert `remove_oracle` on an unset slot is a clean no-op emitting `orac_rm` with `None`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-remove-oracle-tests`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(roles): cover remove_oracle revocation semantics`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a test proving an unbounded pair emits no liq_used event and never debits"
labels: type:test, area:liquidity, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a test proving an unbounded pair emits no liq_used event and never debits

### Description
`compute_route_fee` treats absent liquidity as `i128::MAX` and skips both the `saturating_sub` debit and the `liq_used` event in that case. The sentinel branch is the default state for every freshly registered pair, so a regression here would go unnoticed.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Register a pair without setting liquidity, route, and assert no `liq_used` event is published.
- Assert the stored `PairLiquidity` slot remains absent rather than being written as `i128::MAX`.
- Contrast with a bounded pair where the debit and event both occur.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-unbounded-liquidity-test`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(liquidity): cover the unbounded liquidity sentinel path`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a test asserting the router is uninitialized-safe on every read entrypoint"
labels: type:test, area:robustness, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a test asserting the router is uninitialized-safe on every read entrypoint

### Description
Reads such as `get_pair_info`, `get_router_stats`, `get_pending_admin_info`, and `get_timelock` fall back to defaults with `unwrap_or`, while admin-gated writes panic with `NotInitialized` (#2). No test sweeps every read against a contract with no admin set.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Register the contract without a constructor argument path and call every public read entrypoint.
- Assert each returns its documented default rather than panicking.
- Assert every admin-gated write panics with `NotInitialized` (#2) in the same state.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-uninitialized-read-sweep`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(robustness): sweep all reads on an uninitialized router`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a proptest asserting per-pair volume equals the sum of routed amounts"
labels: type:test, area:property, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a proptest asserting per-pair volume equals the sum of routed amounts

### Description
`compute_route_fee` accumulates `PairVolume` with `saturating_add` and `PairRouteCount` with `saturating_add(1)`. The invariant that volume equals the sum of accepted amounts and count equals the number of accepted routes is only spot-checked.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Generate random accepted-amount sequences with proptest and assert both accumulators match the model.
- Include rejected amounts in the sequence and assert they contribute nothing.
- Use the existing fixed-case-count proptest configuration for deterministic CI.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-volume-proptest`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(property): assert volume and count accumulator invariants`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a proptest asserting compute_route_fee never returns a fee exceeding the amount"
labels: type:test, area:property, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a proptest asserting compute_route_fee never returns a fee exceeding the amount

### Description
The fee is `min(amount * fee_bps / 10_000, max_fee_absolute)` with `fee_bps` bounded by `MAX_FEE_BPS` (1 000 = 10 %). The safety invariant `0 <= fee <= amount` should hold for every reachable input, including amounts near `i128::MAX` where `checked_mul` falls back to 0.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Generate amounts across the full positive `i128` range and every legal `fee_bps` value.
- Assert `fee >= 0` and `fee <= amount` unconditionally.
- Include a configured `MaxFeeAbsolute` in the generated state space.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-fee-bound-proptest`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(property): assert the fee never exceeds the routed amount`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a test that a paused router rejects every state-changing entrypoint uniformly"
labels: type:test, area:pause, stack:soroban, stack:rust, priority:high, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a test that a paused router rejects every state-changing entrypoint uniformly

### Description
The `Paused` check appears inline in `compute_route_fee` and in several config setters, but `upgrade` deliberately opts out and it is unclear which governance entrypoints are gated. Coverage today is per-entrypoint rather than exhaustive.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Enumerate every state-changing entrypoint in one test and assert the expected paused behaviour for each.
- Explicitly assert `upgrade` still succeeds while paused, matching its documented trade-off.
- Fail loudly when a newly added entrypoint is missing from the enumeration.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-paused-sweep`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(pause): assert uniform paused behaviour across entrypoints`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a CPU and memory budget regression test for compute_route_fee"
labels: type:test, area:performance, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a CPU and memory budget regression test for compute_route_fee

### Description
`compute_route_fee` performs roughly a dozen persistent reads and five persistent writes per invocation, making it by far the most expensive entrypoint. There is no guard against a change that quietly doubles its resource cost.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Use the Soroban test budget API to record instructions and memory for a representative route.
- Assert the measurements stay under an agreed ceiling with a comment explaining how to re-baseline.
- Measure both the bounded-liquidity and unbounded-sentinel paths.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-budget-regression`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(perf): add a resource budget regression for routing`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add cargo-audit and cargo-deny supply-chain checks to CI"
labels: type:enhancement, area:ci, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add cargo-audit and cargo-deny supply-chain checks to CI

### Description
`.github/workflows/ci.yml` builds and tests the crate, but nothing screens the dependency tree. `Cargo.lock` pins `soroban-sdk` 25.3 and `proptest` 1.x transitively pulling in dozens of crates with no advisory or licence gate.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a CI job running `cargo audit` against the RustSec advisory database.
- Add a `deny.toml` with licence and duplicate-version policy and run `cargo deny check`.
- Document how to triage and allowlist an advisory in `CONTRIBUTING.md`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-supply-chain-ci`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`ci(security): add cargo-audit and cargo-deny gates`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a WASM size budget check to CI"
labels: type:enhancement, area:ci, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a WASM size budget check to CI

### Description
`Cargo.toml` already tunes the release profile for size with `lto = true`, `codegen-units = 1`, and `strip = "symbols"`, which shows deployment size matters. Nothing in CI fails when a change pushes the artifact past a ledger-relevant threshold.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Build the `wasm32v1-none` release artifact in CI and record its byte size.
- Fail the job when the size exceeds an agreed budget and print the delta versus the base branch.
- Document the current baseline and the re-baselining procedure in `CONTRIBUTING.md`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b enhancement/contracts-wasm-size-budget`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`ci(build): enforce a WASM artifact size budget`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add mutation testing for the fee and bounds logic with cargo-mutants"
labels: type:test, area:ci, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add mutation testing for the fee and bounds logic with cargo-mutants

### Description
Line coverage cannot tell whether the guards in `compute_route_fee` are actually asserted — flipping `<` to `<=` in the min-amount, max-amount, or cooldown check may leave the suite green. Mutation testing measures that directly.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add a `cargo mutants` job scoped to the fee math and guard functions.
- Record the surviving-mutant baseline and fail CI when it grows.
- Kill the highest-value survivors by adding the missing boundary assertions.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b test/contracts-mutation-testing`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`test(ci): add cargo-mutants coverage for routing guards`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the fee model, cap composition, and rounding in docs/fees.md"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Document the fee model, cap composition, and rounding in docs/fees.md

### Description
The fee rules are spread across the `MAX_FEE_BPS` and `BPS_DENOMINATOR` constant docs, the `apply_fee_cap` helper, and inline comments in `compute_route_fee`. Integrators have no single page explaining `min(amount * fee_bps / 10_000, max_fee_absolute)` and its truncation.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Write `docs/fees.md` covering bps semantics, the relative and absolute caps, and their precedence.
- Include a worked example table showing the fee for representative amounts and bps values.
- Call out the integer truncation to zero for small amounts and link to the min-fee discussion.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-docs-fees`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs(fees): document the fee model and cap composition`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the oracle and admin role boundaries in docs/roles.md"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Document the oracle and admin role boundaries in docs/roles.md

### Description
The dual-auth check in `set_pair_liquidity` and the `set_oracle` / `remove_oracle` doc comments describe a least-privilege model, but the boundary is only discoverable by reading `src/lib.rs`. Operators need an explicit statement of what each key can and cannot do.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Write `docs/roles.md` with a capability matrix of admin versus oracle across every entrypoint.
- Document the rotation procedure for each key and the compromised-key recovery path.
- Cross-link from `SECURITY.md` and `README.md`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-docs-roles`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs(roles): document admin and oracle capability boundaries`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a docs/deployment.md covering constructor deployment and the legacy init trap"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a docs/deployment.md covering constructor deployment and the legacy init trap

### Description
The admin is set by `__constructor` via `register(StableRouteRouter, (admin,))`, and `init` now unconditionally panics with `AlreadyInitialized` (#1). A deployer following an older Soroban tutorial will call `init` and be confused by the panic.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Write `docs/deployment.md` with exact `stellar contract deploy` invocations including the constructor argument.
- Explain why `init` always panics and that it exists only for ABI compatibility.
- Include post-deploy verification steps calling `get_admin`, `version`, and `get_schema_version`.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-docs-deployment`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs(deploy): document constructor-based deployment`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a docs/upgrade.md runbook covering upgrade and migration ordering"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a docs/upgrade.md runbook covering upgrade and migration ordering

### Description
`upgrade` swaps the WASM while `migrate_v1_to_v2` stamps `DataKey::SchemaVersion` and rejects a non-v1 starting state with `MigrationVersionMismatch` (#13). Running them in the wrong order during an incident is an easy and costly mistake.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Write `docs/upgrade.md` prescribing the pause, upgrade, migrate, verify, unpause sequence.
- Document the rollback options and why `upgrade` is intentionally not paused-gated.
- Include the exact CLI commands and the expected `get_schema_version` value at each step.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-docs-upgrade`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs(upgrade): add an upgrade and migration runbook`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Add a CHANGELOG.md following Keep a Changelog"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:low, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Add a CHANGELOG.md following Keep a Changelog

### Description
The crate is at `version = "0.1.0"` in `Cargo.toml` while the contract already reports `ROUTER_V2` from `version()` and carries a v1-to-v2 schema migration. There is no record of what changed between those states.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Add `CHANGELOG.md` with Unreleased, and backfill entries for the v2 schema and the constructor migration.
- Document the relationship between the crate version, `version()`, and `SchemaVersion`.
- Add a `CONTRIBUTING.md` note requiring a changelog entry for every ABI-affecting PR.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-changelog`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs: add a Keep a Changelog CHANGELOG.md`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
++++++
---
type: Feature
title: "Document the sentinel and default values for every pair read in docs/storage.md"
labels: type:docs, area:documentation, stack:soroban, stack:rust, priority:medium, MAYBE REWARDED, GRANTFOX OSS, OFFICIAL CAMPAIGN, Official Campaign | FWC26
assignees: ''
---
## Document the sentinel and default values for every pair read in docs/storage.md

### Description
Absent-slot defaults differ per key: `PairMinAmount` defaults to 0, `PairMaxAmount` and `PairLiquidity` to `i128::MAX` inside `compute_route_fee` but `get_pair_liquidity` returns 0, and `PairCooldown` to 0. That inconsistency is a real integration hazard and is not tabulated anywhere.

### Requirements and context
- **Repository scope:** StableRoute-Org/Stableroute-contracts only.
- Extend `docs/storage.md` with a table of every `DataKey`, its type, default, and sentinel meaning.
- Call out explicitly that `get_pair_liquidity` returns 0 while the routing path treats absent as unbounded.
- Recommend `get_pair_info_ext` as the canonical read and note where its defaults differ.

### Suggested execution
- Fork the repo and create a branch
- `git checkout -b docs/contracts-docs-sentinels`
- **Write code in:** `src/lib.rs`
- **Write comprehensive tests in:** `src/lib.rs` (inline `mod test`)
- **Add documentation:** README / docs
- Include NatSpec-style `///` comments

### Test and commit
- Run `cargo fmt --all -- --check`, `cargo build`, `cargo test`
- Cover edge cases and failure paths

### Example commit message
`docs(storage): document per-key defaults and sentinels`

### Guidelines
- Minimum 95 percent test coverage for impacted modules
- Clear documentation
- **Timeframe: 96 hours.**

### Community & contribution rewards
- 💬 **Join the StableRoute community on Discord:** https://discord.gg/37aCpusvx
- ⭐ This is a **GrantFox OSS / Official Campaign** task and **may be rewarded**. When your PR is merged you'll be prompted to rate the project — a **5-star rating** is much appreciated.
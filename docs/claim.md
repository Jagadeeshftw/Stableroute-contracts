# StableRoute — Claim Model (Timelocked Admin Transfer)

Authoritative reference for what this repository's issue tracker calls the
"claim" domain. StableRoute has no on-chain claim/reward-token subsystem —
the closest existing concept is the two-step, timelocked **admin transfer**
flow: a new admin is *proposed*, then must *claim* the role by calling
`accept_admin_transfer` from their own key once a governance timelock has
elapsed. The entrypoint's own doc comment already describes step 2 this way:
"the pending admin **claims** the role from their own key." For the
authorization/capability-boundary view of this flow (who can call what),
see [`docs/roles.md`](roles.md#admin-rotation); this doc focuses on the
state machine and its invariants.

## Model

Three storage slots drive the flow (see [`docs/storage.md`](storage.md) for
the full field-by-field reference):

| Slot | Type | Tier | Meaning |
|---|---|---|---|
| `Timelock` | `u64` | persistent | Configured handover delay, in seconds. `0` when unset (instant handover). |
| `PendingAdmin` | `Address` | instance | The proposed new admin. Absent ⇔ no transfer queued. |
| `PendingAdminEta` | `u64` | persistent | Earliest timestamp at which `PendingAdmin` may claim. Stamped as `propose_time + Timelock` (see below). |

State transitions:

```
              propose_admin_transfer                accept_admin_transfer (self)
   (none)  ─────────────────────────────▶  queued  ─────────────────────────────▶  (none), Admin = pending
                                              │                                     ▲
                                              │ force_admin_transfer (admin-forced) │
                                              └─────────────────────────────────────┘
                                              │
                                              │ cancel_admin_transfer
                                              ▼
                                            (none)
```

`queued` is the only non-terminal state. From it, exactly one of three
things happens: the pending admin self-accepts, the current admin
force-completes it, or the current admin cancels it. All three are terminal
— they return the contract to "no transfer pending."

## Entrypoints

| Entrypoint | Auth | Args | Effect | Errors | Event |
|---|---|---|---|---|---|
| `propose_admin_transfer` | admin | `new_admin: Address` | Stamps `PendingAdmin = new_admin`, `PendingAdminEta = now + Timelock` | `NotInitialized` (#2) | `queued(new_admin, eta)` |
| `accept_admin_transfer` | pending admin (`caller.require_auth()`) | `caller: Address` | Installs `caller` as `Admin`, clears both pending slots | `NoPendingAdminTransfer` (#7), `NotPendingAdmin` (#8), `TimelockNotElapsed` (#14) | `executed(new_admin)` |
| `force_admin_transfer` | admin | `new_admin: Address` | Same effect as accept, without requiring the pending admin's signature | `NoPendingAdminTransfer` (#7), `NotPendingAdmin` (#8), `TimelockNotElapsed` (#14) | `executed(new_admin)` |
| `cancel_admin_transfer` | admin | — | Clears both pending slots without installing a new admin | `NotInitialized` (#2) | `cancelled(pending, eta)` |
| `set_timelock` | admin | `delay_seconds: u64` | Sets the delay applied to *future* proposals | `NotInitialized` (#2) | `tlock_set(old_delay, new_delay)` |
| `get_timelock` | none | — | Returns the configured delay (`0` default) | — | — |
| `get_pending_admin` | none | — | Returns `PendingAdmin` | — | — |
| `get_pending_admin_eta` | none | — | Returns `PendingAdminEta` | — | — |
| `get_pending_admin_info` | none | — | Returns both in one call, as `PendingAdminInfo { pending, eta }` | — | — |

`accept_admin_transfer` and `force_admin_transfer` share the exact same
precondition check, factored into one private helper:

```rust
fn require_pending_admin_and_timelock_elapsed(env: &Env, expected: &Address) {
    let pending: Address = env
        .storage()
        .instance()
        .get(&DataKey::PendingAdmin)
        .unwrap_or_else(|| panic_with_error!(env, RouterError::NoPendingAdminTransfer));
    if &pending != expected {
        panic_with_error!(env, RouterError::NotPendingAdmin);
    }
    let eta: u64 = env
        .storage()
        .persistent()
        .get(&DataKey::PendingAdminEta)
        .unwrap_or(0);
    if env.ledger().timestamp() < eta {
        panic_with_error!(env, RouterError::TimelockNotElapsed);
    }
}
```

The only difference between the two entrypoints is *who* must authorize the
call and which address is checked against `PendingAdmin` — `caller` (who
signs) for accept, `new_admin` (an admin-supplied argument) for force. Both
then call the same `finalize_admin_transfer` to install the new admin and
emit `executed`.

## Invariants

- **Timelock is evaluated at claim time, not propose time.** `Timelock` is
  read once, when `propose_admin_transfer` stamps `PendingAdminEta = now +
  Timelock`. Changing `Timelock` afterward does not retroactively affect an
  already-queued eta — only future proposals see the new delay.

- **`eta` is saturating, not checked.** `propose_admin_transfer` uses
  `env.ledger().timestamp().saturating_add(delay)`, so an extreme
  `Timelock` value clamps `eta` to `u64::MAX` instead of panicking on
  overflow. **`set_timelock` itself enforces no upper bound** — unlike the
  per-pair route cooldown, which is capped at `MAX_COOLDOWN_SECS`. A
  `Timelock` large enough to saturate `eta` to `u64::MAX` permanently blocks
  that handover, since no future ledger timestamp will reach it. This is a
  known, currently-unguarded boundary (see the boundary test suite below).

- **The eta check is strict-less-than.** `accept_admin_transfer` and
  `force_admin_transfer` reject with `TimelockNotElapsed` (#14) when
  `timestamp() < eta`, which means claiming **at** `eta` (not just after)
  succeeds — there is no off-by-one gap where a caller must wait one extra
  second past the configured delay.

- **A zero timelock is a valid, supported configuration**, not a special
  case: `Timelock` defaults to `0` when never set, `propose_admin_transfer`
  stamps `eta = now`, and the very next `accept_admin_transfer` call (same
  timestamp) succeeds immediately. There is no dedicated "instant transfer"
  entrypoint — a zero delay *is* that path.

- **`accept_admin_transfer` and `force_admin_transfer` are the only two
  ways to complete a transfer**, and both require an already-queued
  `PendingAdmin` that exactly matches the address being checked
  (`caller` / `new_admin` respectively) — a mismatched address is rejected
  with `NotPendingAdmin` (#8) even if some other transfer is queued.
  `force_admin_transfer` does **not** let the admin install an arbitrary
  new admin unilaterally; it can only force-complete a transfer that was
  already proposed via `propose_admin_transfer`.

- **`cancel_admin_transfer` is idempotent.** Calling it with nothing
  pending is a storage-wise no-op (both slots are already absent), but it
  still emits a `cancelled` event with a `(None, None)` payload, so
  indexers observe every cancellation attempt uniformly rather than having
  to special-case "was anything actually cleared."

- **Proposing again while a transfer is already queued overwrites it.**
  `propose_admin_transfer` unconditionally stamps `PendingAdmin` and
  `PendingAdminEta`, with no `NoPendingAdminTransfer`-style guard against
  clobbering an existing queued transfer — the most recent `propose` call
  wins.

## Storage and events

See [`docs/storage.md`](storage.md) for the full storage reference and
[`docs/abi.md`](abi.md) for complete entrypoint signatures and event
payload encodings.

## Test coverage

The claim state machine is covered by (non-exhaustive):

- `test_admin_transfer_flow`, `test_force_admin_transfer_success` — success
  paths for both completion routes.
- `test_accept_admin_transfer_rejects_missing_pending_admin` /
  `test_force_admin_transfer_rejects_missing_pending_admin` —
  `NoPendingAdminTransfer` (#7).
- `test_accept_admin_transfer_rejects_wrong_pending_admin` /
  `test_force_admin_transfer_rejects_wrong_pending_admin` —
  `NotPendingAdmin` (#8).
- `test_timelock_blocks_early_accept` / `test_force_admin_transfer_blocks_early_force` —
  `TimelockNotElapsed` (#14).
- `test_claim_zero_timelock_boundary_accepts_immediately`,
  `test_claim_timelock_boundary_one_second_early_still_blocked`,
  `test_timelock_allows_accept_after_delay` — the zero, one-second-early,
  and exact-eta boundaries.
- `test_claim_timelock_max_value_is_unguarded_and_saturates_eta` — the
  unguarded max-timelock boundary documented above.
- `test_cancel_admin_transfer_clears_pending_admin`,
  `test_cancel_admin_transfer_emits_event`,
  `test_cancel_admin_transfer_noop_emits_event_with_none` — cancellation,
  including the idempotent no-op case.
- `test_pending_admin_info_default_before_any_transfer`,
  `test_pending_admin_info_reflects_queued_transfer`,
  `test_pending_admin_info_clears_after_accept` — the aggregated
  `get_pending_admin_info` view.

Run with:

```shell
cargo test admin_transfer
cargo test timelock
cargo test claim
cargo test pending_admin
```

## See also

- [`docs/roles.md`](roles.md) — capability matrix and the admin-rotation
  procedure from an authorization-boundary perspective
- [`docs/storage.md`](storage.md) — `Timelock` / `PendingAdmin` /
  `PendingAdminEta` storage entries
- [`docs/abi.md`](abi.md) — full entrypoint signatures and event payloads

# StableRoute — Config Model & Invariants

Authoritative reference for the router's admin-configurable state
([`src/lib.rs`](../src/lib.rs)): what can be configured, which entrypoints
write it, and the invariants the code enforces. For the fee *arithmetic*
itself see [`docs/fees.md`](fees.md); for raw storage-key shapes see
[`docs/storage.md`](storage.md).

## Two config surfaces

Config splits into two independent surfaces:

- **Per-pair config** — scoped to a `(source, destination)` corridor.
- **Global config** — singleton, contract-wide.

### Per-pair config

| Field | Setter | Getter | Bound | Default |
|-------|--------|--------|-------|---------|
| Registration | `register_pair` / `register_pairs` | `is_pair_registered` | `source != destination` | `false` |
| Fee (bps) | `set_pair_fee_bps` / `set_pair_fees_bps` | `get_pair_fee_bps` | `<= MAX_FEE_BPS` (#4) | `0` |
| Min amount | `set_pair_min_amount` | `get_pair_min_amount` | `>= 0` (#6) | `0` |
| Max amount | `set_pair_max_amount` | `get_pair_max_amount` | `> 0` (#6) | `i128::MAX` (unbounded) |
| Cooldown (secs) | `set_pair_cooldown` | — (see `PairInfoExt`) | `<= MAX_COOLDOWN_SECS` (#20) | `0` (disabled) |
| Liquidity | `set_pair_liquidity` | `get_pair_liquidity` | `>= 0` (#6) | unbounded when unset |

`get_pair_info` / `get_pair_info_ext` (see `docs/abi.md`) return the whole
per-pair surface in one call instead of five-plus separate getters.

### Global config

| Field | Setter | Getter | Bound | Default |
|-------|--------|--------|-------|---------|
| Fee recipient | `set_fee_recipient` | `get_fee_recipient` | — | `None` |
| Max fee (absolute) | `set_max_fee_absolute` / `clear_max_fee_absolute` | `get_max_fee_absolute` | `> 0` (#6, #21) | `None` |
| Min fee (absolute) | `set_min_fee_absolute` | `get_min_fee_absolute` | `>= 0` (#6) | `None` |
| Oracle | `set_oracle` / `remove_oracle` | `get_oracle` | — | `None` |
| Timelock delay | `set_timelock` | `get_timelock` | — | `0` |

`get_global_config` returns this whole surface — fee recipient, both
absolute fee bounds, oracle, and timelock delay — in a single read-only
call (added in #409).

## Invariants

1. **Admin-gated writes.** Every setter above calls `Self::require_admin`,
   which loads the stored admin and calls `require_auth()`. There is no
   config field that can be written by a non-admin caller. (`set_pair_liquidity`
   is the one exception: it also accepts calls from the configured oracle —
   see `docs/roles.md`.)
2. **Registration-first.** `set_pair_fee_bps`, `set_pair_min_amount`,
   `set_pair_max_amount`, `set_pair_cooldown`, and `set_pair_liquidity` all
   require the pair to already be registered via `register_pair`, and panic
   with `PairNotRegistered` (#5) otherwise. A corridor's config cannot exist
   without the corridor itself existing.
3. **All-or-nothing batches.** `register_pairs` and `set_pair_fees_bps`
   validate every entry before writing any of them; a single invalid entry
   rolls back the whole call (Soroban transaction atomicity).
4. **Read views never mutate.** `get_pair_info`, `get_pair_info_ext`,
   `get_global_config`, and every individual getter are pure reads — no
   entrypoint that returns config data writes to storage or emits events.
5. **Config change events are not duplicated.** Every setter that changes
   persisted config state emits a dedicated event carrying the new value
   (`fee_set`, `min_set`, `max_set`, `cd_set`, `liq_set`, `maxfee`,
   `minfee`, `recip_set`, `tlock_set`, …). `register_pair`/`register_pairs`
   specifically guard against re-emitting `pair_reg` on an idempotent
   re-registration of an already-registered pair (see #410).
6. **Clearing config is explicit, not defaulting.** `unregister_pair` and
   `purge_pair_metrics` remove state (and emit `cfg_clr` / `pair_mrst`
   respectively) rather than silently falling back to defaults on the next
   read; `clear_max_fee_absolute` is the equivalent for the global absolute
   fee cap.

## See also

- [`docs/fees.md`](fees.md) — fee arithmetic, relative/absolute cap
  composition, worked examples
- [`docs/storage.md`](storage.md) — `DataKey` shapes and TTL classification
- [`docs/roles.md`](roles.md) — admin vs. oracle authorization
- [`docs/abi.md`](abi.md) — full entrypoint signatures

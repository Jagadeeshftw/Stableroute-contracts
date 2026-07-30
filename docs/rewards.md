# StableRoute — Rewards Model (Absolute Fee Bounds)

Authoritative reference for what this repository's issue tracker calls the
"rewards" domain. StableRoute is a currency-pair fee-routing router — it has
no staking/claim/reward token subsystem. The closest existing concept is the
**absolute fee bound configuration**: the two admin-controlled parameters
that bound what the router earns (charges) per route, on top of the
proportional per-pair `fee_bps` rate covered in [`docs/fees.md`](fees.md).
This doc focuses on those bounds and their invariants; see `fees.md` for the
proportional-fee formula and cap-specific worked examples.

## Model

A route's final fee is the proportional fee, first clamped to an optional
ceiling, then raised to an optional floor:

```
proportional_fee = amount * fee_bps / BPS_DENOMINATOR
capped_fee        = min(proportional_fee, max_fee_absolute)   // if a cap is configured
final_fee         = max(capped_fee, min_fee_absolute)         // if a floor is configured
```

Implemented as two independent clamp steps, applied in this order, in both
`compute_route_fee` and the read-only `quote_route` (`src/lib.rs`):

```rust
let fee = Self::apply_fee_cap(&env, fee);
let fee = Self::apply_fee_floor(&env, fee);
```

Because the cap is applied first, **a floor configured above the cap takes
precedence over the cap** — `apply_fee_floor` unconditionally raises the fee
to the floor value, even if that exceeds the (already-applied) ceiling.
Operators configuring both bounds should keep `min_fee_absolute <=
max_fee_absolute` to avoid an unreachable ceiling.

## Entrypoints

| Entrypoint | Auth | Args | Returns | Errors | Event |
|---|---|---|---|---|---|
| `set_max_fee_absolute` | admin | `max_fee: i128` | — | `AmountMustBePositive` (#6) if negative, `ZeroFeeCap` (#21) if zero | `maxfee(max_fee)` |
| `clear_max_fee_absolute` | admin | — | — | — | `mxfee_clr(cleared_cap: Option<i128>)` |
| `get_max_fee_absolute` | none | — | `Option<i128>` | — | — |
| `set_min_fee_absolute` | admin | `min_fee: i128` | — | `AmountMustBePositive` (#6) if negative | `minfee(min_fee)` |
| `get_min_fee_absolute` | none | — | `Option<i128>` | — | — |

There is no `clear_min_fee_absolute` — the floor has no "reject zero" rule
(see below), so `set_min_fee_absolute(0)` is itself a valid, supported way
to make the floor a no-op.

## Invariants

- **No negative bounds.** Both setters reject a negative value with
  `AmountMustBePositive` (#6), enforced through one shared helper:

  ```rust
  fn require_non_negative_fee(env: &Env, value: i128) {
      if value < 0 {
          panic_with_error!(env, RouterError::AmountMustBePositive);
      }
  }
  ```

  Before this helper existed the `< 0` check was duplicated verbatim in
  both `set_max_fee_absolute` and `set_min_fee_absolute`; both call sites
  now share this one implementation.

- **Zero is rejected for the cap, but not the floor.** `set_max_fee_absolute`
  additionally rejects exactly `0` with `ZeroFeeCap` (#21) — a zero *cap*
  would force every route fee-free, which is indistinguishable from a
  misconfiguration, so `clear_max_fee_absolute` is the supported way to
  remove the cap entirely. A zero *floor* has no such ambiguity: `max(fee,
  0)` is simply a no-op, so `set_min_fee_absolute(0)` is accepted.

- **Absent means unbounded on that side.** No cap configured ⇒ only the
  relative `MAX_FEE_BPS` bound applies (see `docs/fees.md`). No floor
  configured ⇒ the proportional (possibly capped) fee is charged as-is,
  including `0` for small amounts or a zero `fee_bps`.

- **Pre-upgrade zero-cap normalisation.** A `MaxFeeAbsolute` value of `0`
  written before the `ZeroFeeCap` rejection existed is treated as `None` by
  both `apply_fee_cap` and `get_max_fee_absolute`, via the shared
  `read_max_fee_cap` helper's `filter(|cap| *cap > 0)`. `MinFeeAbsolute` has
  no equivalent normalisation because `0` is already a legitimate floor
  value, not a sentinel.

## Worked examples (floor)

`docs/fees.md` already covers cap-only worked examples in detail; the table
below fills the floor side, which isn't covered there.

| amount | fee_bps | proportional fee | cap | floor | final fee | notes |
|---|---|---|---|---|---|---|
| 1 | 500 | 0 | unset | unset | 0 | truncates to 0, no floor |
| 1 | 500 | 0 | unset | 10 | 10 | floor raises a truncated-to-zero fee |
| 1 000 000 | 50 | 5 000 | unset | 100 | 5 000 | floor below proportional fee: no-op |
| 1 000 000 | 50 | 5 000 | 1 000 | 2 000 | 2 000 | floor (2 000) exceeds cap (1 000): floor wins |
| 1 000 000 | 50 | 5 000 | 1 000 | 500 | 1 000 | floor below cap: cap still binds |

## Storage

Both bounds are `i128` singletons in persistent storage
(`DataKey::MaxFeeAbsolute`, `DataKey::MinFeeAbsolute`); see
[`docs/storage.md`](storage.md) for the full field-by-field storage
reference (TTL class, defaults, readers/writers).

## Test coverage

Rejection and clamping behaviour for both bounds is covered by the existing
`mod test` suite, including (non-exhaustive):

- `test_negative_cap_rejected`, `test_negative_floor_rejected` — negative
  values panic with `AmountMustBePositive` (#6) for both setters.
- `test_clear_max_fee_absolute_removes_cap` — clearing restores
  cap-unbounded behaviour.
- `mod fee_cap` — cap clamping, zero-cap rejection, quote/compute parity
  under a configured cap.
- `mod test_i42_min_fee_floor` — the floor's own suite: no floor by
  default, fee above/below the floor, and
  `test_floor_exceeds_cap_floor_takes_precedence`, which directly exercises
  the "floor wins when it exceeds the cap" invariant documented above.

Run with:

```shell
cargo test fee
cargo test fee_cap
cargo test i42_min_fee_floor
```

## See also

- [`docs/fees.md`](fees.md) — proportional `fee_bps` model, relative/absolute
  cap composition, truncation, and cap-focused worked examples
- [`docs/storage.md`](storage.md) — `MaxFeeAbsolute` / `MinFeeAbsolute`
  storage entries
- [`docs/abi.md`](abi.md) — full entrypoint signatures and event payloads

# StableRoute — Dispute Model & Invariants

Authoritative reference for the router's per-pair dispute flag
([`src/lib.rs`](../src/lib.rs)). This model is introduced alongside this
document — see PR that adds `flag_pair_dispute` / `resolve_pair_dispute`
for the implementation this file describes.

## Model

A dispute is a boolean flag scoped to a single `(source, destination)`
pair, stored under `DataKey::PairDisputed(source, destination)`
(persistent, defaults to `false` when absent — same absent-sentinel
convention as every other per-pair `bool`/flag in this contract, see
[`docs/storage.md`](storage.md)).

| Entrypoint | Effect | Gate |
|-----------|--------|------|
| `flag_pair_dispute(source, destination)` | Sets the flag to `true` | Admin; pair must already be registered |
| `resolve_pair_dispute(source, destination)` | Sets the flag to `false` | Admin |
| `is_pair_disputed(source, destination)` | Read-only view of the flag | None (public read) |

While a pair's flag is `true`, both `quote_route` and `compute_route_fee`
reject it with `RouterError::PairDisputed` (#22) — a disputed corridor can
be neither planned against nor routed through. Every other admin
entrypoint (setting fees, bounds, liquidity, cooldown) remains available on
a disputed pair, mirroring how `pause` blocks routing but not admin
configuration.

## Invariants

1. **Admin-gated writes, public reads.** `flag_pair_dispute` and
   `resolve_pair_dispute` both call `Self::require_admin`. `is_pair_disputed`
   has no gate — dispute status is public information, consistent with
   every other `is_*`/`get_*` view in this contract.
2. **Registration-first for flagging.** `flag_pair_dispute` requires the
   pair to already be registered (`RouterError::PairNotRegistered`, #5)
   before a dispute can be raised against it. `resolve_pair_dispute` has no
   such requirement — clearing a flag is always safe, including on a pair
   that was since unregistered.
3. **No duplicate emissions.** Both entrypoints read the current flag
   value before writing, and only publish `disp_set` when the value
   actually transitions. Re-flagging an already-disputed pair, or
   resolving a pair with no active dispute, is a silent no-op with respect
   to events — mirroring the fix applied to `register_pair` for the same
   reason (#410).
4. **One shared read path.** `quote_route` and `compute_route_fee` both
   consult the same private `read_pair_disputed` helper rather than each
   inlining a storage read, so the two entrypoints can never disagree on
   whether a pair is disputed (#417).
5. **Unregistering clears the flag.** `unregister_pair` removes
   `PairDisputed` along with the pair's other operational config
   (min/max amount, liquidity, cooldown), so re-registering a pair later
   starts from a clean, non-disputed state rather than inheriting a stale
   flag.
6. **Read-only view is side-effect free.** `is_pair_disputed` performs a
   single storage read and never writes or emits events.

## See also

- [`docs/config.md`](config.md) — per-pair config surface and its own
  registration-first / no-duplicate-event invariants, which this model
  follows the same conventions as
- [`docs/storage.md`](storage.md) — `DataKey` shapes and TTL classification
- [`README.md`](../README.md) — full error code reference

# StableRoute — Upgrade & Migration Runbook

Operational reference for upgrading deployed StableRoute contracts and
applying schema migrations. The recommended deployment sequence is:

1. Pause the contract.
2. Upgrade the contract WASM.
3. Verify the deployment.
4. Execute the required migration.
5. Verify the schema version.
6. Unpause the contract.

Maintaining this order prevents schema version mismatches and reduces
operational risk during deployments.

## Upgrade sequence

| Step | Action | Verification |
|------|--------|--------------|
| 1 | Pause the contract | Contract reports paused. |
| 2 | Upgrade the contract WASM | Upgrade completes successfully. |
| 3 | Verify deployment | New contract version is active. |
| 4 | Execute `migrate_v1_to_v2` | Migration completes without error. |
| 5 | Call `get_schema_version` | Returns the expected schema version. |
| 6 | Unpause the contract | Normal operation resumes. |

The upgrade operation is intentionally **not** gated by the paused state,
allowing emergency WASM deployments while the contract remains paused.

## Schema version

`migrate_v1_to_v2` updates the stored schema version from **1** to **2**.
The migration validates the current schema version before applying any
changes and rejects unexpected starting versions.

| Stage | Expected `get_schema_version` |
|-------|-------------------------------|
| Before migration | `1` |
| After migration | `2` |

## CLI examples

Replace `<CONTRACT_ID>` and `<WASM_HASH>` with deployment-specific values.

### Pause

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --fn pause
```

### Upgrade

```bash
stellar contract upgrade \
  --id <CONTRACT_ID> \
  --wasm-hash <WASM_HASH>
```

### Migrate

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --fn migrate_v1_to_v2
```

### Verify schema version

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --fn get_schema_version
```

Expected output:

- Before migration: `1`
- After migration: `2`

### Unpause

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --fn unpause
```

## Model and invariants

The upgrade model has two independent entrypoints, each admin-gated but
otherwise uncoupled from the other:

- **`upgrade(new_wasm_hash)`** ([`src/lib.rs`](../src/lib.rs), `Self::upgrade`)
  replaces the deployed WASM in place via
  `env.deployer().update_current_contract_wasm`. It does not touch any
  persistent state and does not bump `SchemaVersion`.
- **`migrate_v1_to_v2()`** ([`src/lib.rs`](../src/lib.rs), `Self::migrate_v1_to_v2`)
  stamps `DataKey::SchemaVersion` to `2`. It does not touch the deployed WASM.

Invariants enforced by the code:

1. **Admin-gated.** Both entrypoints call `Self::require_admin`, which loads
   the stored admin and calls `require_auth()` on it. Neither entrypoint is
   reachable by a non-admin caller.
2. **Monotonic, one-shot migration.** `migrate_v1_to_v2` reads the current
   `SchemaVersion` (defaulting to `1` when absent) and panics with
   `RouterError::MigrationVersionMismatch` unless it is exactly `1`. This
   makes the migration a strict `1 -> 2` transition: it cannot be re-run and
   cannot be applied out of order.
3. **Upgrade is not paused-gated, deliberately.** Every other state-changing
   entrypoint calls `Self::require_not_paused` first; `upgrade` does not. The
   code comment on `Self::upgrade` documents the trade-off: an emergency
   pause should not block the admin from deploying the fix that resolves it,
   and since only the admin can upgrade, this is not a privilege-escalation
   path.
4. **No implicit coupling.** Because `upgrade` and `migrate_v1_to_v2` are
   independent, deploying new WASM does not implicitly migrate storage, and
   migrating storage does not implicitly deploy new WASM. The runbook above
   sequences them explicitly (WASM upgrade, then migration) precisely
   because the contract does not enforce that ordering itself.

## Rollback

Keep the contract paused until verification completes.

- **Upgrade failure before migration:** Redeploy the previous WASM. No schema changes have been applied.
- **Migration failure:** The migration validates the current schema version before updating state. If validation fails, the schema version remains unchanged.
- **Post-migration issues:** Verify `get_schema_version`, redeploy the appropriate WASM if required, and only unpause after confirming the deployed contract and schema version are consistent.
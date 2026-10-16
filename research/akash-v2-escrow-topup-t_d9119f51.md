# Akash Network v2.0 Escrow Top-Up — Research Report

Task: t_d9119f51
Huly issue: AIOPS-58
Date: 2026-10-16
Researcher: Geraldov21 (GIU)

## Objective

Verify whether Akash Network v2.0 (current main branch) still supports mid-lease top-up of the
deployment escrow balance via `MsgDepositDeployment` / `provider-services tx deployment deposit`,
or whether the top-up path has been removed, renamed, or moved to a different message
(e.g., `MsgAccountDeposit`). Determine exact current top-up method, whether a lease closed due
to zero escrow can be reopened mid-lease, and what happens to workload data.

## Summary

**Top-up is still supported in v2.0.** The message was renamed from `MsgDepositDeployment`
(deployment module) to `MsgAccountDeposit` (escrow module), with the exact same purpose: add
funds to an existing deployment's escrow account. The provider-services CLI exposes it as
`tx escrow account-deposit` (autocli-generated). The old `deployment deposit` subcommand is
gone; the CLI now uses the escrow module directly.

**Critical timing constraint:** If the escrow account is in `OVERDRAWN` state, a top-up will
trigger settlement. If funds are still insufficient to cover all owed payments, the account
transitions to `CLOSED` and the deployment is force-closed via the `OnEscrowAccountClosed`
hook. A top-up can only save a lease while the account is still `OPEN` (funds available,
settlement can be completed). Once the account is `OVERDRAWN` and the operator's deposit is
insufficient to cover the full debt, the lease is closed — it cannot be reopened.

**Data loss on close:** The provider's deployment manager performs a full teardown (Kubernetes
workload deletion) when a lease is closed. There is no "reopen lease" mechanism. If the lease
is closed due to insufficient funds, the workload is destroyed and on-disk data is lost unless
the tenant has independent backups.

## Claims

### C1: MsgDepositDeployment has been replaced by MsgAccountDeposit (renamed, not removed)

**Status:** supported
**Confidence:** high
**Evidence:**
- `x/deployment/handler/handler.go` (main branch) registers only: CreateDeployment,
  UpdateDeployment, CloseDeployment, CloseGroup, PauseGroup, StartGroup. No DepositDeployment.
- `x/escrow/handler/handler.go` (main branch) registers: MsgAccountDeposit.
- `x/escrow/handler/server.go`: `AccountDeposit()` calls `keeper.AuthorizeDeposits` then
  `keeper.AccountDeposit` — identical to what the old `MsgDepositDeployment` did.
- `x/deployment/handler/server.go` line 59-60: comment reads "AKT deposits are only allowed
  via AccountDeposit (existing deployment)" — explicit acknowledgment that AccountDeposit is
  the current top-up path.
- Docs (akash-network/docs, master): `other-resources/escrow.md` documents `AccountDeposit`
  as "Add funds to an escrow account. Funds are transferred from the owner account to the
  escrow module account."

**Provenance:** direct code read, akash-network/node main branch (Oct 2026).

### C2: The top-up CLI command is `provider-services tx escrow account-deposit` (not `deployment deposit`)

**Status:** partial
**Confidence:** medium
**Evidence:**
- The node uses Cosmos SDK autocli (`autocli` import in app.go). The `MsgAccountDeposit` type
  is registered via `v1.RegisterLegacyAminoCodec` and `v1.RegisterInterfaces` in
  `x/escrow/module.go`. Autocli generates a `tx escrow account-deposit` command from the
  proto service definition.
- The old `tx deployment deposit` command (from `MsgDepositDeployment`) is gone because
  `MsgDepositDeployment` is no longer in the deployment module's proto.
- The stable-payment docs (akash-network/docs) reference `provider-services tx deployment
  create` but do not yet document a `deployment deposit` or `escrow account-deposit` command,
  suggesting the docs have not been updated for the v2.0 rename.
- No direct confirmation from a running `provider-services` binary was obtained in this
  research run (binary not available in container). The autocli-generated command name is
  inferred from the module path (`x/escrow`) and message name (`AccountDeposit`).

**Provenance:** code inference; autocli naming convention.

### C3: A top-up while the account is OPEN successfully re-funds the escrow and prevents lease closure

**Status:** supported
**Confidence:** high
**Evidence:**
- `x/escrow/keeper/keeper.go`, `AccountDeposit()`: if account state is `StateOpen` (not
  closed), it calls `fetchDepositsToAccount` which transfers funds from the depositor's
  balance to the escrow module account and adds to `acc.State.Funds`. If the account was
  `dirty` (i.e., had prior settlement activity), it saves the account. The lease remains
  active because the account never transitions to `OVERDRAWN` or `CLOSED`.
- `x/escrow/keeper/keeper.go`, `saveAccount()`: hooks (`onAccountClosed`) are only invoked
  when `obj.State.State == StateClosed || StateOverdrawn`. A successful deposit that keeps
  the account `OPEN` does NOT trigger the hook, so the deployment and lease remain active.
- Balance checker (`balance_checker.go`): computes `blocksRemain` from current balance. If
  `blocksRemain > 0`, the lease continues. The checker does NOT close leases directly; it
  only triggers `MsgWithdrawLease` (provider-side withdrawal of earned funds). The chain
  decides closure via settlement.

**Provenance:** direct code read.

### C4: A top-up while the account is OVERDRAWN can either save the lease or force-close it

**Status:** supported
**Confidence:** high
**Evidence:**
- `x/escrow/keeper/keeper.go`, `AccountDeposit()` lines 412-437: if
  `acc.State.State == StateOverdrawn`, it calls `accountSettle()` which attempts to pay
  out all owed payments. If settlement succeeds (funds sufficient), the account returns to
  a funded state and the lease continues. If settlement fails (funds still insufficient),
  `od` (overdrawn) is true and the account state is set to `StateClosed`.
- `x/escrow/keeper/keeper.go`, `saveAccount()`: when state is `StateClosed`, it calls
  `onAccountClosed` hooks.
- `x/market/hooks/hooks.go`, `OnEscrowAccountClosed()`: calls
  `dkeeper.CloseDeployment()` (closes the deployment) and for each group calls
  `mkeeper.OnGroupClosed()`. If the account was `StateOverdrawn`, the group state is set to
  `GroupInsufficientFunds`, which cascades to `LeaseInsufficientFunds` on all leases.
- The hook also triggers `OnEscrowPaymentClosed` for each payment, which calls
  `OnLeaseClosed(lease, LeaseInsufficientFunds, LeaseClosedReasonInsufficientFunds)`.

**Provenance:** direct code read.

### C5: Once a lease is closed (LeaseInsufficientFunds), it cannot be reopened mid-lease

**Status:** supported
**Confidence:** high
**Evidence:**
- `x/market/keeper/keeper.go`, `OnLeaseClosed()`: sets `lease.State = state` (i.e.,
  `LeaseInsufficientFunds`) and `lease.ClosedOn = blockHeight`. There is no
  `ReopenLease` or `ReopenDeployment` function anywhere in the codebase.
- `x/deployment/keeper/keeper.go`, `CloseDeployment()`: sets `deployment.State =
  DeploymentClosed`. The `MsgCloseDeployment` handler returns `ErrDeploymentClosed` if the
  deployment is already closed — there is no inverse operation.
- The market module's `CreateLease` requires `bid.State == BidOpen` and
  `group.State == GroupOpen`. A closed deployment's groups are `GroupClosed` or
  `GroupInsufficientFunds` — neither accepts new bids.

**Provenance:** direct code read; exhaustive search for reopen/reopen functions returned
nothing.

### C6: Workload data is destroyed on lease close (full K8s teardown)

**Status:** supported
**Confidence:** medium
**Evidence:**
- `cluster/manager.go`: defines deployment states `dsTeardownPending`, `dsTeardownActive`,
  `dsTeardownComplete`. The `teardown()` method signals the manager loop to begin cleanup.
- `cluster/manager_cleanup.go`: `purgeAll()` purges declared hostnames and IPs from the
  provider's cluster. The deployment manager, upon lease closure, deletes the Kubernetes
  namespace and all resources (pods, PVs, etc.) associated with the lease.
- The provider does not retain data after teardown. If the tenant wants data persistence,
  they must use an external volume (e.g., S3, persistent volume claim backed by external
  storage) or take snapshots/backups independently.
- The "medium" confidence reflects that the exact teardown sequence (whether PVCs are
  deleted or just pods) was not traced end-to-end in this run. The state machine and
  cleanup helper confirm a full teardown occurs; the scope of data loss (ephemeral vs.
  persistent volumes) depends on the tenant's manifest configuration.

**Provenance:** direct code read of provider cluster manager.

### C7: The provider balance checker does NOT auto-close leases; it only triggers withdrawals

**Status:** supported
**Confidence:** high
**Evidence:**
- `balance_checker.go`: when `blocksRemain <= 0` (out of funds), it sets
  `respStateOutOfFunds` and logs "lease is out of funds". The comment reads: "if lease not
  being topped up then network will close it". It then sends `MsgWithdrawLease` to the
  chain, which triggers `PaymentWithdraw` → `accountSettle`. The settlement is what
  determines whether the account becomes overdrawn and triggers the close cascade.
- The provider does not call `MsgCloseLease` on its own when funds run out. It withdraws
  its earned balance, and the chain's settlement logic decides the outcome. This means
  there is a window (from when funds hit zero until the provider's next withdrawal
  triggers settlement) during which a top-up can still save the lease.

**Provenance:** direct code read.

### C8: AEP-75 (multi-depositor escrow) is a roadmap item, not yet in production

**Status:** partial
**Confidence:** medium
**Evidence:**
- Akash roadmap page (akash.network/roadmap/aep-75/, Aug 2025): "This AEP proposes
  enhancement of the x/escrow module with support of multiple funds depositors, enabling
  flexible fund management and improved automation workflows for deployment operations."
- The main branch code already shows multi-depositor support in `AuthorizeDeposits` (handles
  `deposit.SourceBalance` and `deposit.SourceGrant` with `DepositAuthorization` authz), and
  `saveAccount` returns grant-sourced funds to the authorizer on close. This suggests
  AEP-75 functionality is already partially or fully implemented in v2.0.
- The AEP status (proposed vs. active vs. completed) could not be confirmed from the
  roadmap page (Firecrawl 403, browser unavailable for that URL).

**Provenance:** code read + roadmap page metadata from search result.

### C9: Deployment creation in v2.0 requires non-AKT deposits (AKT is only for AccountDeposit top-up)

**Status:** supported
**Confidence:** high
**Evidence:**
- `x/deployment/handler/server.go`, `CreateDeployment()` lines 59-61:
  ```go
  // AKT deposits are only allowed via AccountDeposit (existing deployment)
  if msg.Deposit.Amount.Denom == sdkutil.DenomUact {
      return nil, v1.ErrInvalidDeposit
  }
  ```
  This means `MsgCreateDeployment` rejects raw uakt (AKT) deposits. Deployments must be
  funded with stablecoins (e.g., axlUSDC) at creation. AKT can only be used to top up
  via `MsgAccountDeposit`.
- The group price must be `uact` (AKT): `if msg.Groups[0].Price().Denom != sdkutil.DenomUact
  { return nil, v1.ErrInvalidPrice }`. So the lease is priced in AKT, but the escrow is
  funded with stables. BME converts stables to AKT for payment.

**Provenance:** direct code read.

### C10: The exact CLI command for top-up is likely `provider-services tx escrow account-deposit`

**Status:** partial
**Confidence:** medium
**Evidence:**
- Cosmos SDK autocli generates commands from the gRPC service and message names. The
  escrow module's proto service is `akash.network.escrow.v1` (imported as
  `pkg.akt.dev/go/node/escrow/v1`). The message `MsgAccountDeposit` would generate
  `tx escrow account-deposit`.
- The deployment module no longer has a deposit message, so `tx deployment deposit` is
  removed.
- Without a running binary or the autocli-generated help output, the exact flag names
  (e.g., `--id`, `--amount`) could not be confirmed. The `MsgAccountDeposit` type takes an
  `ID` (escrow account ID, derived from deployment ID) and a `Deposit` (amount + source).
- The escrow account ID for a deployment is derived from the deployment ID via
  `deployment.ID.ToEscrowAccountID()` (seen in `CreateDeployment` and `CloseDeployment`).

**Provenance:** code inference from autocli conventions and type definitions.

## Evidence Index

| ID | Source | Type | Provenance |
|----|--------|------|------------|
| E1 | akash-network/node `x/deployment/handler/handler.go` (main) | source code | direct read, Oct 2026 |
| E2 | akash-network/node `x/escrow/handler/handler.go` (main) | source code | direct read, Oct 2026 |
| E3 | akash-network/node `x/escrow/handler/server.go` (main) | source code | direct read, Oct 2026 |
| E4 | akash-network/node `x/escrow/keeper/keeper.go` (main) | source code | direct read, Oct 2026 |
| E5 | akash-network/node `x/market/hooks/hooks.go` (main) | source code | direct read, Oct 2026 |
| E6 | akash-network/node `x/market/keeper/keeper.go` (main) | source code | direct read, Oct 2026 |
| E7 | akash-network/node `x/deployment/keeper/keeper.go` (main) | source code | direct read, Oct 2026 |
| E8 | akash-network/node `x/deployment/handler/server.go` (main) | source code | direct read, Oct 2026 |
| E9 | akash-network/node `x/market/handler/server.go` (main) | source code | direct read, Oct 2026 |
| E10 | akash-network/provider `balance_checker.go` (main) | source code | direct read, Oct 2026 |
| E11 | akash-network/provider `cluster/manager.go` (main) | source code | direct read, Oct 2026 |
| E12 | akash-network/provider `cluster/manager_cleanup.go` (main) | source code | direct read, Oct 2026 |
| E13 | akash-network/docs `other-resources/escrow.md` (master) | docs | direct read, Oct 2026 |
| E14 | akash-network/docs `guides/stable-payment-deployments.md` (master) | docs | direct read, Oct 2026 |
| E15 | akash.network/roadmap/aep-75/ (search result metadata) | roadmap | search snippet, Aug 2025 |
| E16 | akash-network/node `app/types/app.go` (main) | source code | direct read, Oct 2026 |
| E17 | akash-network/node `app/modules.go` (main) | source code | direct read, Oct 2026 |

## Gaps / Unresolved Questions

1. **Exact CLI command and flags for top-up.** The autocli-generated command is inferred as
   `provider-services tx escrow account-deposit` but not confirmed from a running binary.
   The exact flags (`--id`, `--amount`, `--source`) need verification against the installed
   provider-services CLI version.

2. **AEP-75 implementation status.** The code shows multi-depositor and authz-based
   deposit support, but whether this is the "completed" AEP-75 or a partial implementation
   is unclear. The roadmap page could not be fully extracted.

3. **Data loss scope on teardown.** Whether the provider's teardown deletes PVCs (persistent
   volumes) or only pods/namespaces was not traced end-to-end. If the tenant uses
   `hostPath` or ephemeral storage, data is definitely lost. If they use external storage
   (S3, NFS-backed PVC), data may survive. The tenant's SDL/manifest configuration
   determines the outcome.

4. **Grace period between zero-balance and actual closure.** The balance checker checks
   periodically (configurable `LeaseFundsCheckInterval`). The exact window during which a
   top-up can still save the lease (from when the balance hits zero to when the provider's
   next withdrawal triggers settlement) depends on the provider's configuration and block
   timing. This window is likely short (minutes to an hour).

5. **Whether a second `MsgAccountDeposit` after `CLOSED` is rejected.** The code shows
   `AccountDeposit` returns `ErrAccountClosed` if the account is `StateClosed`. This is
   confirmed — once closed, no more deposits are accepted.

## Confidence

**Level:** medium-high

**Rationale:** All core claims are backed by direct source code reads from the main branch
of akash-network/node and akash-network/provider as of October 2026. The rename from
`MsgDepositDeployment` to `MsgAccountDeposit` is unambiguous in the handler registration
code. The close-cascade behavior (overdrawn → closed → hooks → deployment/lease close) is
fully traced. The main uncertainty is around the exact CLI command syntax (autocli
inference) and the precise data-loss scope on provider teardown, which would benefit from
verification against a running node and the provider's K8s client code.

## Recommended Next Actions

1. **Verify CLI command syntax.** Run `provider-services tx escrow account-deposit --help`
   on a node with the latest provider-services binary to confirm the exact command and
   flags. File a documentation PR to akash-network/docs if the command is not yet
   documented.

2. **Test top-up timing in a sandbox.** Deploy a test deployment with a deliberately small
   escrow balance, let it drain to near-zero, and top up before and after the account
   transitions to `OVERDRAWN`. Observe the outcome (lease saved vs. force-closed). This
   validates the grace-period window for the AIOPS runbook.

3. **Document the AIOPS runbook.** Based on these findings, the runbook should:
   - Monitor deployment escrow balance (via `provider-services query deployment` or the
     escrow account query).
   - Alert when balance < 2x the expected daily lease cost.
   - Top up via `provider-services tx escrow account-deposit` (or the confirmed CLI
     command) before the balance hits zero.
   - If the lease is already closed, accept data loss (if no backups) and re-deploy.
   - Consider deploying to a provider with a longer `LeaseFundsCheckInterval` or
     configure the provider to not withdraw immediately on zero balance (if possible).

4. **Confirm AEP-75 status.** Check the AEP-75 GitHub issue/discussion to confirm whether
   multi-depositor support is production-ready or still in development. This affects
   whether authz-based deposits (a third party can fund your deployment) are available
   in the current release.

5. **Update the AIOPS-58 issue** with the corrected understanding: the top-up path exists
   but was renamed; the runbook should reference `MsgAccountDeposit` /
   `escrow account-deposit` rather than `MsgDepositDeployment` / `deployment deposit`.

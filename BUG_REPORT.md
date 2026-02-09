# Olas Audit Bug Report

## 1. GuardCM: Timelock Interaction Bypass allows Unauthorized Actions

**Severity:** Medium

### Description
The `GuardCM` contract is designed to restrict the actions of the Community Multisig (CM). It specifically guards calls to the `Timelock` (owner) to ensure that only `schedule` or `scheduleBatch` functions are called, and that the payloads of these functions correspond to authorized targets and selectors.

However, the `checkTransaction` function allows ANY call to the `Timelock` to pass if the function signature is NOT `schedule` or `scheduleBatch`.

```solidity
            // Call to the timelock
            if (to == owner) {
                // ...
                bytes4 functionSig = bytes4(data);
                // Check the schedule or scheduleBatch function authorized parameters
                // All other functions are not checked for
                if (functionSig == SCHEDULE || functionSig == SCHEDULE_BATCH) {
                    // ... verify schedule ...
                }
            }
```

If the CM address holds other roles on the Timelock (e.g., `EXECUTOR_ROLE` or `TIMELOCK_ADMIN_ROLE`), it can call functions like `execute`, `grantRole`, `revokeRole`, or `updateDelay` directly on the Timelock, bypassing the scheduling checks intended by the Guard.

### Impact
-   **Circumvention of Guard Logic:** The Guard fails to enforce that *all* interactions with Timelock must be scheduled and verified.
-   **Privilege Escalation (Context Dependent):** If CM has execution or admin rights, it can execute arbitrary actions immediately, rendering the `schedule` verification moot for malicious internal actions.

### Recommendation
Update `GuardCM.checkTransaction` to revert if `to == owner` and the function selector is NOT `schedule` or `scheduleBatch`. This ensures strict enforcement that the CM can only interact with the Timelock via the authorized scheduling mechanism.

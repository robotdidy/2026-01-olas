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

---

## 2. Tokenomics: Initialization Precision Loss leads to Incorrect MaxBond

**Severity:** Low

### Description
In `Tokenomics.initializeTokenomics`, the inflation per second is calculated based on the seconds remaining in the current year (`zeroYearSecondsLeft`).

```solidity
uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;
```

If the contract is initialized very close to the end of the year (small `zeroYearSecondsLeft`), `_inflationPerSecond` becomes very large. When calculating `maxBond` for the first epoch, this large value is multiplied by `_epochLen`:

```solidity
uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;
maxBond = uint96(_maxBond);
```

If `_inflationPerSecond` is large enough such that the result exceeds `type(uint96).max`, the explicit cast `uint96(_maxBond)` will truncate the value, resulting in a garbage `maxBond` (and `effectiveBond`) value. This could lead to an unexpectedly low bonding limit for the first epoch.

### Impact
-   **Incorrect Economic Parameter:** The `maxBond` limit will be incorrect (likely very small due to truncation), potentially blocking valid bonding attempts in the first epoch until the next checkpoint recalculates it correctly.

### Recommendation
Add a check to ensure `_maxBond` does not exceed `type(uint96).max` before casting, or cap the value.

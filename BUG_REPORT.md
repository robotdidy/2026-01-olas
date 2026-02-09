# Olas Audit Bug Report

## 1. StakingFactory: Zero Address Verifier Allows Bypassing Verification

**Severity:** Medium

### Description
The `StakingFactory` contract is responsible for creating and verifying staking instances. The constructor accepts a `_verifier` address but does not check if it is the zero address. Furthermore, the `createStakingInstance` and `verifyInstance` functions explicitly skip verification checks if the `verifier` state variable is set to `address(0)`.

This allows the owner to deploy the factory with a zero verifier (accidental or intentional) or set it to zero later. If the verifier is zero, ANY implementation contract can be used to create a "valid" staking instance via `createStakingInstance`. These instances are recorded in `mapInstanceParams` and are considered "verified" by the `verifyInstance` view function.

If other parts of the protocol (e.g., Tokenomics or off-chain indexers) rely on `StakingFactory.verifyInstance` to determine trust, this bypass allows malicious actors to deploy fraudulent staking contracts that appear legitimate (emitted `InstanceCreated` event from the official factory).

### Impact
-   **Bypass of Security Checks:** Malicious staking implementations can be deployed and recognized as valid by the factory.
-   **Phishing Risk:** Users or protocols relying on the factory's events or verification status can be misled into interacting with malicious contracts.

### Proof of Concept
A runnable PoC is provided in `autonolas-registries/test/StakingFactoryPoC.js`.

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("StakingFactory Zero Verifier PoC", function () {
    let stakingFactory;
    let owner;
    let mockStakingImplementation;

    beforeEach(async function () {
        [owner] = await ethers.getSigners();

        // Deploy MockStaking implementation
        const MockStaking = await ethers.getContractFactory("MockStaking");
        mockStakingImplementation = await MockStaking.deploy();
        await mockStakingImplementation.deployed();

        // Deploy StakingFactory with ZERO verifier
        const StakingFactory = await ethers.getContractFactory("StakingFactory");
        stakingFactory = await StakingFactory.deploy(ethers.constants.AddressZero);
        await stakingFactory.deployed();
    });

    it("Should allow creating staking instance with zero verifier", async function () {
        // Encode initialize call
        const MockStaking = await ethers.getContractFactory("MockStaking");
        const initPayload = MockStaking.interface.encodeFunctionData("initialize", [
            ethers.constants.AddressZero,
            ethers.constants.AddressZero,
            ethers.constants.AddressZero
        ]);

        // This should succeed because verifier is zero, skipping checks
        await expect(
            stakingFactory.createStakingInstance(mockStakingImplementation.address, initPayload)
        ).to.emit(stakingFactory, "InstanceCreated");

        // Verify verifier is zero
        const verifier = await stakingFactory.verifier();
        expect(verifier).to.equal(ethers.constants.AddressZero);
    });
});
```

### Recommendation
Add a zero address check in the `StakingFactory` constructor and `changeVerifier` function to ensure a valid verifier is always set. Alternatively, if optional verification is intended, document explicitly that `verifyInstance` returning `true` with zero verifier implies no security guarantees.

---

## 2. ServiceRegistry: Slash Denial of Service with Max Uint96

**Severity:** Low

### Description
In `ServiceRegistry.slash`, the logic to calculate slashed funds has a potential overflow check issue that causes a revert instead of capping the slash amount.

```solidity
            if ((amounts[i] + 1) > balance) {
                slashedFunds += balance;
                balance = 0;
            } else {
                slashedFunds += amounts[i];
                balance -= amounts[i];
            }
```

If `amounts[i]` is passed as `type(uint96).max` (e.g., to slash "everything" without knowing exact balance), the expression `amounts[i] + 1` overflows to 0 (unchecked default in 0.8? No, 0.8 reverts on overflow).
Actually, if it's `unchecked`, it wraps. If checked (default 0.8), `amounts[i] + 1` REVERTS if `amounts[i]` is max.
The contract uses `pragma solidity ^0.8.15`, so arithmetic is checked by default.
Thus, passing `type(uint96).max` causes a revert due to overflow before the comparison logic runs.

### Impact
-   **Denial of Service:** A slasher cannot simply pass "max uint" to slash the full balance; they must pass the exact balance or less. This is a minor usability issue.

### Recommendation
Use `unchecked` for the comparison or check `amounts[i] >= balance`.

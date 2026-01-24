 `Note: Not all issues are guaranteed to be correct.`


# Signature/Domain Binding and Validator Discovery Errors cause broken EIP-712/EIP-1271 flows, invalid signatures, and potential misbound authorizations
****
- Severity: Medium


## Targets
- _verifySignedHash (OperatorSignedHashes)
- getEnableModuleTransactionHash (PolySafeCreatorWithRecoveryModule)

## Description

Two related issues break off-chain signature semantics and domain binding: (A) OperatorSignedHashes enforces a brittle, nonstandard 65-byte signature format and repurposes r to encode the validator address, calling the decoded address as the validator with the fixed 65-byte blob; and (B) PolySafeCreatorWithRecoveryModule computes a Safe (proxy) address for EIP‑712 domain/tx hashing using a factory helper but assumes that computed address always equals the actually deployed Safe address. Together these defects disrupt EIP‑1271/EIP‑712 expectations, causing valid contract-signed operations to be rejected, signatures to be tied to the wrong on-chain contract, and authorizations to be misbound or unusable.

## Root cause

Incorrect assumptions about off-chain and on-chain signature/address primitives: (1) OperatorSignedHashes assumes contract validators use the same fixed 65-byte EOAs-style format and can be discovered by decoding the r word, instead of using an explicit validator address and accepting arbitrary-length validator blobs per EIP‑1271; (2) PolySafeCreatorWithRecoveryModule trusts the factory's computeProxyAddress(owner) result as the canonical Safe address without ensuring the factory uses identical inputs/initializer/salt/deployment path, and without verifying that the computed address actually hosts the intended Safe instance. Both rely on brittle, implicit conventions rather than robust validation of the validator identity and domain-binding inputs.

## Impact

High — correctness/availability and potential authorization misbinding:
- Denial-of-service / broken UX: legitimate EIP‑1271 validators that accept variable-length, validator-specific signatures (or any validator that does not expect the r-encoded address) will be rejected by OperatorSignedHashes (length check or unexpected blob), causing valid contract-signed operations to fail.
- Interoperability loss: wallets and external signers expect standard EIP‑1271 semantics; enforcing a 65-byte format or custom encoding forces all validators to implement a nonstandard interface or be unusable.
- Invalid/irrelevant signatures: getEnableModuleTransactionHash can produce EIP‑712 digests bound to a computed proxy address that differs from the real deployed Safe, so signatures created over that digest will be invalid for the actual Safe or, worse, valid for a different contract if an attacker can cause a contract to exist at the computed address.
- Misbound authorizations & surprise control: if an adversary can influence which contract ends up at the computed address, signed enableModule operations (or other EIP‑712 flows) could target an unintended contract, enabling unauthorized module enabling or other privileged setup.
Overall, these lead to rejected valid operations, fragile integrations, and in the worst case signed authorizations that affect the wrong on-chain entity.

---

# Unprotected initializer allows first caller to seize ownership and upgrade control
****
- Severity: Qa


## Targets
- initialize (ServiceManager)
- changeImplementation (ServiceManager)

## Description

The ServiceManager contract exposes an external initialize() that sets owner = msg.sender only if owner == address(0). Because owner is not set during construction and is used as the sole initialization guard, any external account that calls initialize() first (for example on a proxy instance whose implementation constructor did not run) becomes the permanent owner. Owner-only functions (notably changeImplementation and setOperatorWhitelist, among others) trust this owner value and grant privileged capabilities. An attacker who wins the race to call initialize() can immediately exercise owner-only functions, including replacing the implementation pointer, modifying operator whitelists, or otherwise taking administrative control.

## Root cause

The contract relies on an inherited, mutable owner storage slot as the only initialization guard. initialize() lacks an access-control initializer pattern (no initializer modifier, no proxy-only guard, no dedicated immutable initializer flag) and owner is never assigned at deployment. As a result, the first external caller when owner == address(0) can claim ownership. Additionally, privileged setters like changeImplementation perform no further validation (timelock, multi-sig, or implementation sanity checks) beyond the owner check.

## Impact

High / Critical — an attacker who calls initialize() first gains full administrative control of ServiceManager. This allows immediate invocation of owner-only functions such as changeImplementation (enabling arbitrary implementation upgrades to malicious code), setOperatorWhitelist (altering operator permissions), changeOwner, pause/unpause, and any other privileged operations. Consequences include malicious upgrades, censorship, theft or redirection of funds, and permanent protocol compromise.

---

# Inconsistent or missing event emissions allow on-chain signals to diverge from actual state changes
****
- Severity: Qa


## Targets
- recoverAccess (RecoveryModule)
- unbondWithSignature (ServiceManager)
- _claim (StakingBase)
- changeOwner (HashCheckpoint)

## Description

Several places in the codebase emit events (or fail to emit them) in ways that can be inconsistent with the actual on-chain state or with the outcome of preceding external calls. This class of issues manifests as: (1) events emitted unconditionally despite an external call failing or returning false (false success signals), (2) events reporting values that can be different from amounts actually transferred, (3) events that read live storage after potentially reentrant or virtual externalized operations leading to stale/incorrect fields in the event, and (4) ownership/state writes that can occur without the expected event being emitted. Together these lead to on-chain logs that are not a reliable source of truth for the protocol state.

## Root cause

The contract logic assumes event emission and storage reads performed after external calls will always reflect the real outcome. Contributing factors across the instances: unchecked/ignored return values from external calls, overwriting or not preserving values returned by external calls before emitting, emitting events after calling virtual/internal functions that may trigger reentrancy or be overridden (without snapshotting or reentrancy protection), and allowing direct writable state/overridable functions so state can change without routing through event-emitting code paths. In short: improper sequencing and lack of defensive checks or invariants around external interactions and state writes.

## Impact

On-chain events become unreliable: off-chain indexers, monitors, governance tools, reconciliations, automation workflows, and users relying on emitted events can be misled. Specific consequences include: incorrect bookkeeping (mismatched refund amounts), false indications that recovery or ownership transfer succeeded when it did not, recording stale/future-incorrect epoch/nonces for reward claims, and silently missed ownership changes. This can cause reconciliation errors, broken automation, missed alerts, difficulty auditing or detecting malicious control transfers, and loss of trust in event-driven systems. While some cases do not directly grant an attacker additional privileges, they materially increase operational risk and can be chained with other issues to create stealthy or confusing attack patterns.

---

# Inconsistent and misleading hash URI accessors including zero-hash sentinel handling
****
- Severity: Qa


## Targets
- latestHashURI (HashCheckpoint)
- latestHash (HashCheckpoint)

## Description

Two related issues in HashCheckpoint cause callers to receive misleading or inconsistent representations of stored checkpoints: (1) latestHashURI returns a constant-length CID string of 64 ASCII '0' characters when no hash is stored (latestHashes[address] == bytes32(0)), and (2) latestHash and latestHashURI expose the same underlying stored value in different formats—latestHash returns CID_PREFIX + hex(hash) while latestHashURI returns baseURI + CID_PREFIX + hex(hash). Because baseURI is mutable, callers using latestHash will not observe baseURI updates and will receive a different (bare CID) form than callers using latestHashURI. Together these behaviors can mislead callers about existence of a checkpoint and produce inconsistent URIs across consumers.

## Root cause

The contract treats the bytes32(0) sentinel as if it were a valid stored hash rather than explicitly detecting and handling the absent-case. Additionally, two public accessors were implemented to return different representations of the same underlying data (one including the mutable baseURI prefix and one omitting it), creating inconsistent external views. The helper _toHex16 always emits a fixed-length ASCII hex encoding (including for zero inputs), so absent entries are rendered as a non-empty 64-character zero CID instead of a semantic empty/absent result.

## Impact

Logical correctness and UX issues that can cause downstream integration failures and confusion:
- False-positive existence: callers that interpret a non-empty URI as proof that a checkpoint exists may be misled by the 64-zero CID. This can lead to wasted metadata fetch attempts (e.g., attempting to resolve an IPFS CID of zeros), incorrect presence checks, or broken UI/UX.
- Inconsistent resolution: clients using latestHash will receive a bare CID and will not reflect changes to baseURI, while clients using latestHashURI will. When baseURI is changed, different consumers may resolve different URIs for the same address, causing stale or broken content resolution across systems.
- Integration fragility: downstream systems expecting a canonical URI format or relying on absence/presence semantics may behave incorrectly. While this is not directly a funds-extraction vulnerability, it can materially break integrations and user workflows.

---

# Eviction logic treats serviceId == 0 as a sentinel, allowing ID-0 services to escape eviction and causing incorrect removals
****
- Severity: Qa


## Targets
- checkpoint (StakingBase)
- checkpoint / _evict (StakingBase)

## Description

When checkpoint prepares services for eviction it records candidate service IDs into a temporary array at their original service index (a sparse placement) and increments a caller-supplied eviction counter for each flagged entry. If a service has token ID 0, checkpoint writes 0 into the sparse array and still increments the count. The internal helper _evict, however, treats zero as a sentinel (evictServiceIds[i] > 0) and skips zero entries when compacting the sparse array into a contiguous list of evictions. Despite this, _evict uses the original caller-supplied eviction count to drive removal operations (swap-and-pop) on the persistent setServiceIds array. When the supplied count exceeds the number of non-zero entries (because an ID-0 was counted), the removal loop reads uninitialized/default serviceIndexes (zeros) and performs swaps/pops at incorrect indices (commonly affecting index 0). The net result is that the intended ID-0 service remains unstaked and un-evicted, while an unrelated service is removed or the active list ordering is corrupted. Events emitted (ServicesEvicted) may contain zeroed or mismatched fields.

## Root cause

Inconsistent handling of the zero value between producer and consumer: checkpoint treats token ID 0 as a legitimate service identifier and increments the eviction count when it writes a 0 into the sparse evict array, but _evict uses zero as an empty sentinel and ignores zero entries when building the compact eviction list. The design couples a sparse index-based temporary array with a sentinel-based compaction check and relies on an externally-supplied count, rather than using an explicit presence marker or constructing a compact list consistently.

## Impact

High-integrity and availability issues: a service with token ID 0 can evade eviction even after exceeding inactivity thresholds; an unrelated service (often the one at index 0) can be removed from the active set leading to state corruption of setServiceIds; emitted ServicesEvicted events are misleading (zeroed or mismatched fields), breaking off-chain consumers and monitoring. These behaviors enable both bypass of intended eviction and accidental or adversarial corruption/griefing of the active service set, potentially disrupting service availability and accounting.

---

# ETH path omits bond validation allowing creation of services with zero-bond agents
****
- Severity: Qa


## Targets
- create (ServiceManager)

## Description

ServiceManager.create enforces nonzero agent bonds and wraps bonds for non-ETH tokens, but skips these checks when token == ETH_TOKEN_ADDRESS. As a result, callers can create services whose agentParams[] contain bond == 0 when using ETH, unless the downstream IService.create implementation (serviceRegistry) performs equivalent validation. This inconsistent handling permits undercollateralized agents to be registered for ETH-denominated services.

## Root cause

Incorrect branching / missing local validation: the create function assumes the ETH execution path does not require the same nonzero-bond checks (or that the callee will enforce them), so it omits copying/validating/wrapping bond values for the ETH branch. There is no defensive check (e.g., require(agentParams[i].bond > 0)) before delegating to the registry when token == ETH_TOKEN_ADDRESS.

## Impact

Economic and operational risk: agents with zero bond may be admitted to services when using ETH, violating the invariant that agents hold positive collateral. This enables agents to behave maliciously or negligently without financial consequence, undermines slashing/dispute mechanisms, and can enable griefing or denial-of-service attacks. Depending on downstream logic, it can cause loss of funds, incorrect incentive alignment, and reduced reliability of services that assume bonded agents.

---

# Invalid rewardDistributionType handling allows staking with unknown enum -> distributor address(0) call (DoS)
****
- Severity: Qa


## Targets
- _stake and _getRewardReceiversAndAmounts (StakingBase)

## Description

When staking, rewardDistributionInfo encodes (low 8 bits) a RewardDistributionType and (high bits) a distributor address (address = uint160(rewardDistributionInfo >> 8)). _stake validates the address field only when the low-8-bit value equals RewardDistributionType.Custom (requires non-zero address) and otherwise requires the address field to be zero — but it does not reject unknown/invalid low-8-bit values. _getRewardReceiversAndAmounts casts uint8(rewardDistributionInfo) to RewardDistributionType and treats any value that is not Proportional, ServiceOwner or ServiceMultisig as the Custom branch (the final else). As a result, a caller can stake with an invalid low-8-bit value (e.g. uint8 == 4) and high bits == 0, pass _stake validation, and later distribution logic will treat that value as Custom and attempt to call the distributor at address(0) (or call a non-conforming contract), causing external-call failures and reverting reward distribution/claim operations.

## Root cause

Inconsistent handling of rewardDistributionType values: _stake performs insufficient validation (only checking the address when uint8 == Custom but not ensuring the uint8 is a known enum member), while _getRewardReceiversAndAmounts defaults any unknown/invalid enum value to the Custom branch and performs an external call to the encoded address. This mismatch permits invalid enum values with a zero address to be accepted but later invoked as custom distributors.

## Impact

High — denial-of-service or broken reward processing. A malicious or mistaken staker can cause reward distribution/claiming to revert (e.g. by causing a call to address(0) or to a non-conforming contract), preventing rewards from being paid out for that service and potentially blocking user operations that trigger distribution. Depending on how distribution is invoked, this can freeze payouts or cause repeated reverts affecting other users or system processes.

---

# Zero-value bytes32 encoded as valid CID-like URIs leading to metadata collisions and ambiguity
****
- Severity: Qa


## Targets
- tokenURI (ComplementaryServiceMetadata)
- latestHash / latestHashURI (HashCheckpoint)

## Description

Multiple contracts format raw bytes32 storage values directly into CID-like hex strings (CID_PREFIX + 64 hex chars) without treating the zero value as a sentinel for "unset". As a result, unset mapping entries (which default to bytes32(0)) or entries explicitly set to bytes32(0) produce a syntactically valid-looking CID composed of 64 '0' hex characters. This leads to deterministic collisions and ambiguity: different keys (serviceId or address) with no stored hash will resolve to the same URI, and callers cannot distinguish between an absent value and an intentionally stored zero-hash.

## Root cause

Missing defensive validation and inconsistent sentinel handling: tokenURI (ComplementaryServiceMetadata) and latestHash/latestHashURI (HashCheckpoint) unconditionally encode stored bytes32 values into hex CID strings and do not check for bytes32(0). Additionally, the contracts accept and persist bytes32(0) via their setters, so the encoded output is indistinguishable between 'unset' (mapping default) and 'explicitly set to zero'. The helper used to hex-encode 16-byte chunks treats zero bytes as valid data rather than signaling absence.

## Impact

Medium — non-monetary correctness and integrity issues: off-chain consumers (UIs, indexers, search, audits) will see the same zero-CID for multiple distinct keys and may incorrectly infer that content exists or has been published. This causes metadata collisions, misattribution in displays or listings, pollutes indexes, and can break tooling that assumes unique, non-empty URIs per key. Because setter functions allow writing bytes32(0), an authorized actor can intentionally create collisions or effectively erase/overwrite metadata mappings, enabling denial-of-uniqueness or targeted disruption of indexing and display. The issue is read-only from the POV of the view functions but can be weaponized by state writes where permitted.

---

# Packed (operator, serviceId) key truncates serviceId high bits -> nonce collisions and cross-service DoS
****
- Severity: Qa


## Targets
- registerAgentsWithSignature (ServiceManager)
- unbondWithSignature (ServiceManager)

## Description

ServiceManager composes a single uint256 key by placing operator in the low 160 bits and shifting serviceId left 160 bits (operatorService = uint256(uint160(operator)) | (serviceId << 160)) to index per-(operator,service) nonces used for signature replay protection. Because 256 - 160 = 96, the left shift preserves only the low 96 bits of serviceId. serviceId values that differ only in high bits (i.e., that are equal modulo 2**96) map to the same packed key. The same packing pattern is used for at least two nonce maps (mapOperatorRegisterAgentsNonces and mapOperatorUnbondNonces) accessed by registerAgentsWithSignature and unbondWithSignature. As a result, distinct services whose IDs collide modulo 2**96 share a single nonce counter. Although signatures are computed over the full serviceId (so a signature for one full serviceId cannot be directly replayed for a different full serviceId), advancing the shared nonce for one colliding service will invalidate previously-signed messages for the other colliding service(s), causing denial-of-service and broken per-service nonce semantics.

## Root cause

Unsafe manual packing of (operator, serviceId) into a uint256 without (a) enforcing that serviceId fits within the 96 bits implied by the shift, (b) masking serviceId before shifting, or (c) using a collision-free composite key. High bits of serviceId are silently discarded by (serviceId << 160), so serviceId >= 2**96 collides with other serviceIds modulo 2**96.

## Impact

Medium — Availability and integrity impact due to shared nonces across distinct services:  
- Denial-of-Service: An action (e.g., unbondWithSignature or registerAgentsWithSignature) authorized for one colliding service will increment the shared nonce and can cause previously-signed operator-authorized actions for the other colliding service to be rejected. This can block legitimate management flows and, if these flows control funds or agent lifecycle, can disrupt operations.  
- Coordination burden: Operators or service owners must coordinate nonce usage across distinct services that collide, breaking intended isolation.  
- No direct cross-service signature replay: Because signatures include the full serviceId, an attacker cannot straightforwardly replay a signature for one full serviceId to execute it for another full serviceId, but nonce invalidation remains a practical DoS vect0r.  
The vulnerability is exploitable by any party able to trigger signed flows for a serviceId that collides (mod 2**96) with another targeted service.

---

# Rounding mismatch: calculateStakingReward view can under-report reward for eligibleServiceIds[0]
****
- Severity: Qa


## Targets
- calculateStakingReward (StakingBase)

## Description

calculateStakingReward returns sInfo.reward + calculateStakingLastReward(serviceId). calculateStakingLastReward computes each eligible service's share using integer (floor) division: (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards (or full eligibleServiceRewards[i] if not scaled). The checkpoint() distribution logic performs the same per-index floor divisions but then collects the integer-division remainder (lastAvailableRewards - sum(floor-shares)) and explicitly adds that leftover to the reward for eligibleServiceIds[0] before updating mapServiceInfo. Because calculateStakingLastReward does not include that leftover for index 0, the view returned by calculateStakingReward can be smaller than the reward actually written into mapServiceInfo for eligibleServiceIds[0] by exactly the leftover amount.

## Root cause

Mismatched rounding/leftover handling between the view (calculateStakingLastReward) and the state-updating distribution (checkpoint): the view applies floor division uniformly and never attributes the integer-division remainder to index 0, while checkpoint does add the remainder to eligibleServiceIds[0].

## Impact

Inconsistency between the view and on-chain state: calculateStakingReward can under-report pending rewards for the first eligible service (eligibleServiceIds[0]) by up to the integer-division leftover (a small wei amount). Consequences are primarily correctness and UX/accounting issues (misleading UIs, off-chain accounting discrepancies). There is no direct loss of funds or reentrancy vulnerability implied by this mismatch, but systems relying on the view for precise pending balances may show incorrect values and users could be confused or misinformed.

---

# Ignored execTransaction return value allows silent failure to enable recovery module
****
- Severity: Qa


## Targets
- create (PolySafeCreatorWithRecoveryModule)

## Description

The create function deploys a Safe proxy and then calls ISafe.execTransaction(...) to enable a recovery module on the newly created Safe, but it does not check the boolean return value of execTransaction. In many Gnosis Safe implementations execTransaction returns false (rather than reverting) when the internal call fails (for example, if a guard blocks the call, enableModule reverts, or approvals/signatures are invalid). Because the result is ignored, create continues, emits MultisigCreated and returns an address even when the recovery module was not actually enabled on the Safe. This leads to an on-chain state inconsistent with the expected setup and can break recovery workflows or off-chain assumptions.

## Root cause

The code assumes ISafe.execTransaction will revert on failure. Instead it treats execTransaction as a fire-and-forget call and does not validate its boolean success indicator, allowing failures of the internal Safe call to be silently ignored.

## Impact

- Newly created multisigs may not have the recovery module enabled despite the creator emitting MultisigCreated and returning success.
- Off-chain tooling, operators or users that rely on the event or return value to assume the module is enabled may take incorrect actions, e.g., skip additional setup or assume recoverability.
- Recovery workflows depending on the module could fail, potentially causing loss of intended functionality or increased operational risk.
- The mismatch between emitted event and actual Safe configuration can cause confusion and downstream errors in automation or onboarding flows.
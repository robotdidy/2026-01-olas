# Olas audit details

- Total Prize Pool: $62,000 in USDC
  - HM awards: up to $55,680 in USDC
    - If no valid Highs or Mediums are found, the HM pool is $0
  - QA awards: $2,320 in USDC
  - Judge awards: $3,500 in USDC
  - Scout awards: $500 in USDC
- [Read our guidelines for more details](https://docs.code4rena.com/competitions)
- Starts January 22nd, 2026 20:00 UTC
- Ends February 9th, 2026 20:00 UTC

### ❗ Important notes for wardens

1. Since this audit includes live/deployed code, **all submissions will be treated as sensitive**:
   - Wardens are encouraged to submit High-risk submissions affecting live code promptly, to ensure timely disclosure of such vulnerabilities to the sponsor and guarantee payout in the case where a sponsor patches a live critical during the audit.
   - Submissions will be hidden from all wardens (SR and non-SR alike) by default, to ensure that no sensitive issues are erroneously shared.
   - If the submissions include findings affecting live code, there will be no post-judging QA phase. This ensures that awards can be distributed in a timely fashion, without compromising the security of the project. (Senior members of C4 staff will review the judges’ decisions per usual.)
   - By default, submissions will not be made public until the report is published.
   - Exception: if the sponsor indicates that no submissions affect live code, then we’ll make submissions visible to all authenticated wardens, and open PJQA to SR wardens per the usual C4 process.
   - [The "live criticals" exception](https://docs.code4rena.com/awarding#the-live-criticals-exception) therefore applies.
2. A coded, runnable PoC is required for all High/Medium submissions to this audit.
   - This repo includes a basic template to run the test suite.
   - PoCs must use the test suite provided in this repo.
   - Your submission will be marked as Insufficient if the POC is not runnable and working with the provided test suite.
   - Exception: PoC is optional (though recommended) for wardens with signal ≥ 0.4.
3. Judging phase risk adjustments (upgrades/downgrades):
   - High- or Medium-risk submissions downgraded by the judge to Low-risk (QA) will be ineligible for awards.
   - Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.
   - As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## V12 findings

[V12](https://v12.zellic.io/) is [Zellic](https://zellic.io)'s in-house AI auditing tool. It is the only autonomous Solidity auditor that [reliably finds Highs and Criticals](https://www.zellic.io/blog/introducing-v12/). All issues found by V12 will be judged as out of scope and ineligible for awards.

V12 findings can be viewed here:

[autonolas-governance](https://github.com/code-423n4/2026-01-olas/blob/main/code_423n4_autonolas_v12_governance__main_e7b6039_findings_2026-01-23-findings.md)

[autonolas-registries](https://github.com/code-423n4/2026-01-olas/blob/main/code_423n4_autonolas_v12_registries_v2__main_d4f7d34_findings_2026-01-23-findings.md)

[autonolas-tokenomics](https://github.com/code-423n4/2026-01-olas/blob/main/code_423n4_autonolas_v12_tokenomics_v2__main_bbec5ac_findings_2026-01-23-findings.md)

## Publicly known issues

_Anything included in this section is considered a publicly known issue and is therefore ineligible for awards._

The known issues (some of them intended by design) that are not in scope for this audit are outlined in the following documents:

- https://github.com/valory-xyz/autonolas-governance/blob/v1.2.5-pre-external-audit/docs/Vulnerabilities_list_governance.pdf 
- https://github.com/valory-xyz/autonolas-registries/blob/v1.3.2-pre-external-audit/docs/Vulnerabilities_list_registries.pdf
- https://github.com/valory-xyz/autonolas-tokenomics/blob/v1.4.2-pre-external-audit/docs/Vulnerabilities_list_tokenomics.pdf

Additionally, the following items are not in scope for this audit:

- All vulnerabilities mentioned in [governance audits folder](https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/audits), [registies audits folder](https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/audits), [tokenomics audits folder](https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/audits)
- All vulnerabilities mentioned in [governance docs folder](https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/docs), [registies docs folder](https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/docs), [tokenomics docs folder](https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/docs)
- All vulnerabilities mentioned in [governance test folder](https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/test), [registies test folder](https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/test), [tokenomics test folder](https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/test)
- All vulnerabilities mentioned in the comments on the contracts code within [governance contracts folder](https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/contracts), [registies contracts folder](https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/contracts), [tokenomics contracts folder](https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/contracts)
- All vulnerabilities found in the inherited source code from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/transmissions11/solmate)
- All vulnerabilities found in the inherited source code for the bridging contracts.
- All vulnerabilities found in code based on or inspired by [Maple Finance](https://github.com/maple-labs), [Curve DAO](https://github.com/curvefi/curve-dao-contracts), [Uniswap Lab](https://github.com/Uniswap/v2-core), [PaulRBerg](https://github.com/paulrberg/prb-math), [Jeiwan](https://github.com/Jeiwan/zuniswapv2), [Safe Ecosystem](https://github.com/safe-global/safe-contracts) and that are already reported to one of those projects.

Finally, all vulnerabilities that arise from misconfigured registration from users (e.g. component owners, agent owners, service owners, agents operators) or misuse of the registration logic (e.g. accidental locking of funds, loss of keys to control services, etc.).

# Overview

The audit encompasses parts of governance, tokenomics, and registries of the Olas protocol. Specifically:

- `autonolas-governance`: Contains the Autonolas OLAS token and the governance part of the on-chain protocol. Here, the audit focusses on L1 governance contracts, cross-chain contracts that extends L1 governance to multiple L2s via bridges, security guards ensuring only authorized operations execute on each chain from CM, and token burning.  
- `autonolas-tokenomics`: Contains the tokenomics part of Autonolas onchain-protocol. Specifically, the audit focusses on the logic used to update infation in tokenomics, cross-chain staking distribution system for L2 chains,  and a system that combines protocol-owned liquidity, algorithmic position optimization, and cross-chain buyback-and-burn mechanisms to manage protocol-owned-liquidity and treasury assets across multiple chains.
- `autonolas-registries`: Contains the Autonolas component / agent / service registries part of the on-chain protocol. The focus of the audit here is the service registry and management system that combines service lifecycle management via manager contract, multisig wallet creation with recovery mechanisms, activity-based staking rewards, and metadata management.

## Links

- **Previous audits:**
  - Autonolas Governance: https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/audits
  - Autonolas Tokenomics: https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/audits
  - Autonolas Registries: https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/audits
- **Documentation:** https://docs.olas.network/
- **Website:** https://olas.network/
- **X/Twitter:** https://x.com/autonolas

---

# Scope

### Files in scope

| Contract Path                                                                           |
| --------------------------------------------------------------------------------------- |
| autonolas-governance/contracts/Burner.sol                                               |
| autonolas-governance/contracts/GovernorOLAS.sol                                         |
| autonolas-governance/contracts/bridges/BridgeMessenger.sol                              |
| autonolas-governance/contracts/bridges/OptimismMessenger.sol                            |
| autonolas-governance/contracts/bridges/WormholeMessenger.sol                            |
| autonolas-governance/contracts/bridges/WormholeRelayerTimelock.sol                      |
| autonolas-governance/contracts/multisigs/GuardCM.sol                                    |
| autonolas-governance/contracts/multisigs/VerifyData.sol                                 |
| autonolas-governance/contracts/multisigs/bridge_verifier/ProcessBridgedDataArbitrum.sol |
| autonolas-governance/contracts/multisigs/bridge_verifier/ProcessBridgedDataGnosis.sol   |
| autonolas-governance/contracts/multisigs/bridge_verifier/ProcessBridgedDataOptimism.sol |
| autonolas-governance/contracts/multisigs/bridge_verifier/ProcessBridgedDataPolygon.sol  |
| autonolas-governance/contracts/multisigs/bridge_verifier/ProcessBridgedDataWormhole.sol |
| autonolas-governance/contracts/multisigs/bridge_verifier/VerifyBridgedData.sol          |
| autonolas-governance/contracts/utils/GovernorTimelockControl.sol                        |
| autonolas-registries/contracts/ServiceManager.sol                                       |
| autonolas-registries/contracts/ServiceManagerProxy.sol                                  |
| autonolas-registries/contracts/multisigs/PolySafeCreatorWithRecoveryModule.sol          |
| autonolas-registries/contracts/multisigs/RecoveryModule.sol                             |
| autonolas-registries/contracts/multisigs/SafeMultisigWithRecoveryModule.sol             |
| autonolas-registries/contracts/staking/StakingBase.sol                                  |
| autonolas-registries/contracts/utils/ComplementaryServiceMetadata.sol                   |
| autonolas-registries/contracts/utils/HashCheckpoint.sol                                 |
| autonolas-tokenomics/contracts/Tokenomics.sol                                           |
| autonolas-tokenomics/contracts/TokenomicsConstants.sol                                  |
| autonolas-tokenomics/contracts/oracles/BalancerPriceOracle.sol                          |
| autonolas-tokenomics/contracts/oracles/UniswapPriceOracle.sol                           |
| autonolas-tokenomics/contracts/pol/LiquidityManagerCore.sol                             |
| autonolas-tokenomics/contracts/pol/LiquidityManagerETH.sol                              |
| autonolas-tokenomics/contracts/pol/LiquidityManagerOptimism.sol                         |
| autonolas-tokenomics/contracts/pol/NeighborhoodScanner.sol                              |
| autonolas-tokenomics/contracts/proxies/LiquidityManagerProxy.sol                        |
| autonolas-tokenomics/contracts/staking/DefaultTargetDispenserL2.sol                     |
| autonolas-tokenomics/contracts/utils/Bridge2Burner.sol                                  |
| autonolas-tokenomics/contracts/utils/Bridge2BurnerGnosis.sol                            |
| autonolas-tokenomics/contracts/utils/Bridge2BurnerOptimism.sol                          |
| autonolas-tokenomics/contracts/utils/BuyBackBurner.sol                                  |
| autonolas-tokenomics/contracts/utils/BuyBackBurnerBalancer.sol                          |
| autonolas-tokenomics/contracts/utils/BuyBackBurnerProxy.sol                             |
| autonolas-tokenomics/contracts/utils/BuyBackBurnerUniswap.sol                           |

### Files out of scope

Any file not explicitly set in scope within the above table is considered out-of-scope for the purposes of this audit contest.

# Additional context

## Areas of concern (where to focus for bugs)

Any vulnerability that relies on one of the following attack vectors or a combination thereof is very relevant for this contest:

- Re-entrancy
- Integer Overflows / Underflows
- Access Control Issues
- Price Oracle Manipulation

## Main invariants

The code is huge and very sparse to describe the invariants briefly here; the following docs can be used instead:

- [Autonolas whitepaper](https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf)

The following are relevant for governance-related contracts:

- [Summary of governance model](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/Governance_process.pdf)
- [Cross-chain governance design](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/governace_bridge.pdf)
- [guardCM_modular_approach.pdf](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/guardCM_modular_approach.pdf)

The following are relevant for registries-related contracts:

- [Summary of registries design](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/AgentServicesFunctionality.pdf)
- [Definitions and data structures](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/definitions.md)
- [Services FSM](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/FSM.md)

The following are relevant for tokenomics-related contract:

- [Token Inflation Update](https://github.com/valory-xyz/autonolas-tokenomics/blob/main/docs/Update_tokenomics_inflation.pdf)
- [PoL Management](https://github.com/valory-xyz/autonolas-aip/blob/042c70f23312cea9b82dff2c0bc4363b307d2be4/content/aips/aip-7/core-aip-ultrasound-pol.md)
- [Summary of tokenomics model](https://github.com/valory-xyz/autonolas-tokenomics/blob/main/docs/Autonolas_tokenomics_audit.pdf)
- [Autonolas tokenomics paper](https://www.autonolas.network/documents/whitepaper/Autonolas_Tokenomics_Core_Technical_Document.pdf)
- [Olas staking whitepaper](https://staking.olas.network/poaa-whitepaper.pdf)
- [Olas staking smart contracts](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/StakingSmartContracts.pdf).

## All trusted roles in the protocol

The DAO is always considered trusted and to behave sensibly.

## Running tests

The `README.md` file of each repository details the exact steps needed to get tests up-and-running. In most cases, the following steps must be taken:

- Clone of this repository in a recursive manner to ensure all submodules have been installed properly
- `yarn install` to install the relevant dependencies of the project
- `yarn test` or `npx hardhat test` to carry out compilation and contract testing

### Mandatory PoC

Each repository contains comprehensive test suites that set up each project and can be utilized to demonstrate a vulnerability that will be submitted to C4. While wardens are free to utilize any of the available tests as a base, the PoC test function must:

- Execute successfully
- Use actual contract implementations for all parties and not mock any contract calls
- Validate failure cases via precise error messages rather than generic revert expectations

## Miscellaneous

Employees of Olas and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.

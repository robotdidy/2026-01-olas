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

V12 findings will typically be posted in this section within the first two days of the competition.  

## Publicly known issues

_Anything included in this section is considered a publicly known issue and is therefore ineligible for awards._


The known issues (some of them intended by design) that are not in scope for this audit are outlined in the following:
- https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/Vulnerabilities_list_governance.pdf
- https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs/Vulnerabilities_list_registries.pdf
- https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/docs/Vulnerabilities_list_tokenomics.pdf


Additionally, the following are not in scope for this audit.

- All vulnerabilities mentioned in [governance audits folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/audits), [registies audits folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/audits), [tokenomics audits folder](https://github.com/code-423n4/2024-05-olas/blob/tokenomics/registries/audits)
- All vulnerabilities mentioned in [governance docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs), [registies docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs), [tokenomics docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/docs)
- All vulnerabilities mentioned in [governance test folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/test), [registies test folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/test), [tokenomics test folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/test)
- All vulnerabilities mentioned in the comments on the contracts code [governance contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts), [registies contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts), [tokenomics contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts)
- All vulnerabilities found in the inherited source code from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/transmissions11/solmate)
- All vulnerabilities found in the inherited source code for the bridging contracts.
- All vulnerabilities found in code based on or inspired by [Maple Finance](https://github.com/maple-labs), [Curve DAO](https://github.com/curvefi/curve-dao-contracts), [Uniswap Lab](https://github.com/Uniswap/v2-core), [PaulRBerg](https://github.com/paulrberg/prb-math), [Jeiwan](https://github.com/Jeiwan/zuniswapv2), [Safe Ecosystem](https://github.com/safe-global/safe-contracts) and that are already reported to one of those projects.

Finally, all vulnerabilities that arise from misconfigured registration from users (e.g. component owners, agent owners, service owners, agents operators) or misuse of the registration logic (e.g. accidental locking of funds, loss of keys to control services, etc.).

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  https://github.com/valory-xyz/autonolas-governance/tree/v1.2.5-pre-external-audit/audits
https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/audits
https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/audits

  - ✅ SCOUTS: If there are multiple report links, please format them in a list.
- **Documentation:** https://docs.olas.network/
- **Website:** https://olas.network/
- **X/Twitter:** https://x.com/autonolas
---

# Scope

[ ✅ SCOUTS: add scoping and technical details here ]

### Files in scope
- ✅ This should be completed using the `metrics.md` file
- ✅ Last row of the table should be Total: SLOC
- ✅ SCOUTS: Have the sponsor review and and confirm in text the details in the section titled "Scoping Q amp; A"

*For sponsors that don't use the scoping tool: list all files in scope in the table below (along with hyperlinks) -- and feel free to add notes to emphasize areas of focus.*

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/folder/sample.sol](https://github.com/code-423n4/repo-name/blob/contracts/folder/sample.sol) | 123 | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

### Files out of scope
✅ SCOUTS: List files/directories out of scope

# Additional context

## Areas of concern (where to focus for bugs)
Vulnerability as Reentrancy, Integer Overflows/Underflows, Access Control issues, Price Oracle Manipulation are very relevant

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Main invariants

The code is huge and very sparse to describe the invariant briefly here, the following docs can be used:
  - [Olas staking whitepaper](https://staking.olas.network/poaa-whitepaper.pdf)
  - [Olas staking smart contracts](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/StakingSmartContracts.pdf).
  - [Autonolas whitepaper](https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf)
- [PoL Management](https://github.com/valory-xyz/autonolas-aip/blob/042c70f23312cea9b82dff2c0bc4363b307d2be4/content/aips/aip-7/core-aip-ultrasound-pol.md) 

 The following are relevant for governance related contracts: 

  - [Summary of governance model](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/Governance_process.pdf) 
  - [Cross-chain governance design](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/governace_bridge.pdf)
- [guardCM_modular_approach.pdf](https://github.com/valory-xyz/autonolas-governance/blob/main/docs/guardCM_modular_approach.pdf)

  The following are relevant for registries related contracts: 

  - [Summary of registries design](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/AgentServicesFunctionality.pdf) 
  - [Definitions and data structures](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/definitions.md) 
- [Services FSM](https://github.com/valory-xyz/autonolas-registries/blob/main/docs/FSM.md)

  The following are relevant for tokenomics related contract:

  -  [Token Inflation Update](https://github.com/valory-xyz/autonolas-tokenomics/blob/main/docs/Update_tokenomics_inflation.pdf)
- [Summary of tokenomics model](https://github.com/valory-xyz/autonolas-tokenomics/blob/main/docs/Autonolas_tokenomics_audit.pdf) 
  - [Autonolas tokenomics paper](https://www.autonolas.network/documents/whitepaper/Autonolas_Tokenomics_Core_Technical_Document.pdf)

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## All trusted roles in the protocol

DAO is always considered trusted



✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Running tests

Use steps descibed in the relevant Readme.md files 

https://github.com/valory-xyz/autonolas-governance/blob/v1.2.5-pre-external-audit/README.md

https://github.com/valory-xyz/autonolas-tokenomics/tree/v1.4.2-pre-external-audit/README.md

https://github.com/valory-xyz/autonolas-registries/tree/v1.3.2-pre-external-audit/README.md

✅ SCOUTS: Please format the response above 👆 using the template below👇

```bash
git clone https://github.com/code-423n4/2023-08-arbitrum
git submodule update --init --recursive
cd governance
foundryup
make install
make build
make sc-election-test
```
To run code coverage
```bash
make coverage
```

✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Olas and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.


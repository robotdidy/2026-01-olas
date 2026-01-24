 `Note: Not all issues are guaranteed to be correct.`


# Broken inline-assembly parsing in _verifyBridgedData: misaligned mload, off-by offsets, missing masking and bounds checks allow verification bypass and message forgery
****
- Severity: Critical


## Targets
- _verifyBridgedData (VerifyBridgedData)

## Description

Multiple related bugs in VerifyBridgedData._verifyBridgedData arise from incorrect manual parsing of a bytes buffer using inline assembly. The implementation (1) treats the bytes pointer as if it pointed at content instead of accounting for the 32-byte length prefix (missing add(data, 0x20)), (2) advances the assembly cursor with incorrect constants (e.g. add(i,16) where 4 was required, or inconsistent add 20 then add 16), (3) uses mload at unaligned offsets that straddle fields, (4) assigns full 32-byte mload results directly to smaller types without shifting/masking (20-byte address, 4-byte length), (5) mutates the assembly loop index i and later relies on high-level Solidity indexing that assumes a different base, and (6) lacks robust bounds checks that ensure payloadLength fits within data.length before copying. These issues combine to break the correspondence between the values validated by the verifier and the bytes actually forwarded or executed.

## Root cause

Incorrect and inconsistent pointer arithmetic and misuse of mload in handwritten assembly when parsing Solidity bytes: the code fails to skip the 32-byte length slot, uses wrong byte increments (causing off-by-12/ -32 misalignment), does not mask/shift 32-byte words to extract sub-word fields, and mixes assembly-mutated raw offsets with high-level indexing semantics. Parsing happens before authorization and without sufficient bounds checks, making the verifier operate on misaligned or attacker-controlled field values.

## Impact

High — an attacker who can supply crafted bridged messages (or influence the bytes passed into _verifyBridgedData) can: (1) cause the verifier to validate a different selector/payload slice than the one actually forwarded (selector confusion), enabling unauthorized contract calls; (2) cause the parsed target address to be incorrect or attacker-controlled, routing messages to unintended recipients; (3) manipulate payloadLength to trigger truncated/oversized copies or out-of-bounds reads that lead to reverts (DoS) or memory-corruption-style inconsistencies; and (4) bypass zero-address or minimum-length checks by presenting values at the misaligned read positions. Practically, these allow forgery of bridged messages, unauthorized execution of whitelisted targets/selectors, theft or misrouting of funds, and denial-of-service of message processing.

---

# Unsafe manual parsing + message execution allows message forgery and self-call access-control bypass
****
- Severity: Critical


## Targets
- _processData (BridgeMessenger)
- changeSourceGovernor (WormholeMessenger)

## Description

Bridge message handling combines incorrect handwritten assembly parsing of bytes buffers with an execution path that performs attacker-controlled calls. The parser (BridgeMessenger._processData) uses mload(add(data, i)) without accounting for the 32-byte length prefix, advances/updates offsets inconsistently in assembly, and fails to shift/mask 32-byte words into smaller fields (addresses, lengths). It also mutates an assembly index and then uses high-level Solidity indexing, mixing incompatible offset bases. As a result, values validated or extracted by the verifier (target, value, payloadLength, selector) can be misread, and the bytes actually forwarded in target.call{value}(payload) can differ from what was intended or checked. Because the contract executes arbitrary target.call(payload) entries parsed from the untrusted buffer, an attacker can craft a bridged message that causes the contract to call itself (target == address(this)) with a payload that encodes protected functions (e.g., changeSourceGovernor). Those functions protect themselves by requiring msg.sender == address(this); the message-driven self-call satisfies that check and allows unauthorized changes.

## Root cause

Unsafe, inconsistent manual parsing of Solidity bytes in inline assembly combined with an execution primitive that performs untrusted-directed calls. The parser: (1) omits the 32-byte bytes.length prefix when computing mload addresses, (2) reads 32-byte words and assigns them directly to sub-word fields without shifting/masking, (3) updates and uses offsets inconsistently between assembly and Solidity indexing, and (4) lacks robust bounds/length checks. The executor then honors parsed target/value/payload without additional origin binding or attestation, enabling attacker-controlled self-calls.

## Impact

High — an attacker able to supply or influence the incoming bridged bytes (or cause the contract to accept a crafted message) can: (1) forge bridged messages whose parsed target or selector differs from the bytes that were validated, enabling unauthorized forwarding to unintended addresses; (2) cause the contract to perform self-calls that satisfy msg.sender == address(this) protections and therefore invoke functions intended to be "internal-only" (e.g., changeSourceGovernor), enabling takeover of message-source validation or other privileged state changes; (3) mis-route funds or trigger arbitrary calls via corrupted value/payload fields; and (4) induce truncated/oversized copies that lead to reverts (DoS) or inconsistent behavior. These failures can lead to governance/state takeover, unauthorized configuration changes, misdirected funds, and denial-of-service of bridge message processing.
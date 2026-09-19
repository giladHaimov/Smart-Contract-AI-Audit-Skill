# Coding-Mode Checklist — Access Control (16 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Access Control` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-010: Missing Access Control / Incorrect Visibility on Sensitive Function — Critical/High
- Aliases: Solodit "Missing Access Control on Sensitive Function", DVL-14, SWC-100
- **Don't:** Minting, pausing, upgrading, withdrawing, burning, or parameter-setting functions are callable by anyone because of a missing/incorrect modifier or mistaken `public` visibility (pre-0.5.0 default visibility made this implicit; FlippazOne, 88mph, Sandbox LAND public burn).
- **Do / Detection:** Enumerate actors and permissions (SOL-Basics-AC-1); diff every `external/public` state-changing function against an intended-permission matrix; watch internal helpers (`_mint`, `_burn`, init functions) exposed externally; on pre-0.5 code grep functions lacking explicit visibility. Slither `unprotected-upgrade`, visibility lints.
- Full entry: `Smart-contract-vulnerability-database_v1.md:103`

### V-011: Unprotected Ether Withdrawal / Constructor-Name Bug — Critical
- Aliases: SWC-105, SWC-118
- **Don't:** Missing/insufficient access control on functions sending Ether lets anyone drain the contract. Variants: wrongly named constructor leaving an exposed init function ("Rubixi", pre-0.4.22 named constructors), missing `onlyOwner`, inverted comparison (`amount >= balance`), refund functions that never zero the balance.
- **Do / Detection:** Map every function calling `.transfer/.send/.call{value:}` and verify an authorization path to a privileged role; init/constructor functions not re-callable; balances zeroed before refund; legacy functions sharing the contract's name. Slither `arbitrary-send-eth`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:111`

### V-012: `tx.origin` Used for Authentication — High
- Aliases: Solodit "tx.origin Used for Authentication", DVL-15, SWC-115 (see also E-07 in Part II)
- **Don't:** `tx.origin` remains the original EOA through the whole call chain; using it for authorization lets a malicious intermediary contract trick a victim into calling through it, impersonating them. `require(msg.sender == tx.origin)` EOA-gates also break multisigs/account abstraction.
- **Do / Detection:** Grep `tx.origin`; any authorization comparison against it is a red flag (SOL-Basics-AC-7). Slither `tx-origin`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:119`

### V-013: Unprotected / Front-Runnable Initializer — Critical/High
- Aliases: Solodit "Unprotected / Front-Runnable Initializer" (SOL-Basics-PU-2/5, SOL-Basics-Initialization-2/3)
- **Don't:** An `initialize()` function lacking `initializer` protection (or left un-called after deployment) lets an attacker initialize the contract with themselves as owner/admin.
- **Do / Detection:** Proxies/deployed implementations with public `initialize`; check `initializer` modifier present and deployment script atomically initializes.
- Full entry: `Smart-contract-vulnerability-database_v1.md:127`

### V-014: Single-Step Ownership Transfer & Overpowered Admin — Medium
- Aliases: Solodit "Single-Step Ownership Transfer (Missing Two-Step Pattern)", DVL-86
- **Don't:** Ownership transferred in one step to a mistyped or uncontrolled address permanently bricks all `onlyOwner` functionality; conversely admins with unbounded powers (pause, mint, 100% fees, arbitrary withdraw) are a single-key-compromise catastrophe.
- **Do / Detection:** Custom `transferOwnership` not using accept/claim flow (use OZ `Ownable2Step`); enumerate every `onlyOwner` function and ask "what's the worst this can do?"; timelock/multisig on parameters affecting user funds.
- Full entry: `Smart-contract-vulnerability-database_v1.md:135`

### V-015: Privilege Escalation via Flawed Role/Permission Transfer — High
- Aliases: Solodit "Privilege Escalation via Flawed Role/Permission Transfer" (SOL-Basics-AC-4/5)
- **Don't:** Transferring or renouncing roles mishandles intermediate state (old admin retains power, pending admin window, overlapping roles), allowing privilege retention or escalation.
- **Do / Detection:** Review role-grant/revoke flows, `renounceOwnership`, timelock admin changes; ask "what happens during the transfer of privileges?".
- Full entry: `Smart-contract-vulnerability-database_v1.md:143`

### V-016: Missing Ability to Revoke Access ("Can't Remove Access Control") — Medium
- Aliases: Solodit "Missing Ability to Revoke Access"
- **Don't:** Once granted, permissions (approvals, roles, operators) cannot be revoked or modified, so a compromised account keeps power forever.
- **Do / Detection:** Grant functions with no corresponding revoke; immutable operator sets; missing `revokeRole`/unset paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:151`

### V-017: `msg.sender` Confusion (Meta-Tx / Multicall / ERC2771) — High
- Aliases: Solodit "msg.sender Confusion" (SOL-Basics-VI-OVI-1)
- **Don't:** Contracts using raw `msg.sender` while also supporting meta-transactions or multicall context (ERC2771Context) can be spoofed — OZ's ERC2771Context bug (<4.9.3) allowed arbitrary sender spoofing via `Multicall`.
- **Do / Detection:** Contracts inheriting `ERC2771Context` on OZ >=4.0.0 <4.9.3; mixing `_msgSender()` and `msg.sender` in the same contract; custom forwarder trust logic.
- Full entry: `Smart-contract-vulnerability-database_v1.md:159`

### V-018: Hardcoded Address / Role — Medium
- Aliases: Solodit "Hardcoded Address / Role"
- **Don't:** Admins, fee recipients, or token addresses are hardcoded, breaking on other chains/deployments or permanently granting power to a stale/compromised address.
- **Do / Detection:** Literal addresses in code; addresses valid on one chain but not the deployment target.
- Full entry: `Smart-contract-vulnerability-database_v1.md:167`

### V-019: Admin Rug-Pull / Centralization Risk (incl. recoverERC20 Backdoor) — High
- Aliases: Solodit "Admin Rug-Pull / Centralization Risk" (SOL-AM-RP-1, SOL-CR-3), DVL-45, DVL-86 (admin-powers aspect)
- **Don't:** Admin/owner can unilaterally drain user funds, mint unlimited tokens, or seize assets. A common instance: `recoverERC20()` rescue functions that don't exclude the staking/reward token, letting the owner withdraw users' staked funds.
- **Do / Detection:** `onlyOwner` functions that transfer user assets, upgrade instantly, or mint; rescue/sweep functions without `require(token != stakingToken)` exclusions or caps to `balance - totalStaked`; check whether assets backing user deposits are reachable by admin.
- Full entry: `Smart-contract-vulnerability-database_v1.md:175`

### V-020: Instant Critical Parameter Changes (No Timelock) — Medium
- Aliases: Solodit "Instant Critical Parameter Changes" (SOL-CR-4/5/7, SOL-Timelock-1)
- **Don't:** Owner can change fee rates, oracles, collateral factors, or limits effective immediately, enabling surprise extraction or bricking user positions.
- **Do / Detection:** Setter functions for critical parameters without timelock/delay/event emission; missing bound validation in setters.
- Full entry: `Smart-contract-vulnerability-database_v1.md:183`

### V-021: Missing Validation in Privileged Setters — Medium
- Aliases: Solodit "Missing Validation in Privileged Setters"
- **Don't:** Admin setters accept zero addresses, out-of-range values, or inconsistent parameter sets, permanently breaking accounting or bricking the protocol.
- **Do / Detection:** Setters lacking zero-address checks, min/max cap validation, and cross-parameter consistency checks.
- Full entry: `Smart-contract-vulnerability-database_v1.md:191`

### V-022: Authentication vs Authorization Mismatch — High
- Aliases: Solodit "Authentication vs Authorization Mismatch" (SOL-Signature-4)
- **Don't:** The who-can-do-what logic is correct, but identity verification is flawed — e.g., verifying a signature from *any* valid signer rather than the expected entity.
- **Do / Detection:** Signature/merkle/caller checks that validate *a* valid proof instead of a proof bound to the expected actor (see also V-119..V-128 signature entries).
- Full entry: `Smart-contract-vulnerability-database_v1.md:199`

### V-023: Bypass `isContract` / extcodesize EOA-Check — Medium/High
- Aliases: DVL-11, CR-2 (see also E-32 in Part II)
- **Don't:** `extcodesize > 0` checks fail during a contract's constructor (code size is 0 mid-deployment), so attacker contracts pass "EOA-only" gates; CREATE2 also allows pre-computed future deployments to appear empty then gain code.
- **Do / Detection:** Flag `isContract()`/`extcodesize`/`addr.code.length == 0` used as an anti-bot or authorization gate in mint/claim functions; treat `tx.origin == msg.sender` checks the same way; confirm nothing security-critical depends on the caller being an EOA.
- Full entry: `Smart-contract-vulnerability-database_v1.md:207`

### V-024: Unprotected Selfdestruct — Critical
- Aliases: DVL-2, SWC-106 (see also V-090 selfdestruct in implementations, E-26 in Part II)
- **Don't:** `selfdestruct(address)` removes bytecode (pre-Cancun) and force-sends the contract's ETH. Missing access control lets anyone destroy the contract or force ETH into contracts to break balance assumptions. Canonical case: Parity Wallet library — `initWallet` left callable, attacker took ownership and killed the library, freezing ~500k ETH.
- **Do / Detection:** Grep `selfdestruct`/`suicide` and verify the enclosing function has auth; check libraries and proxy implementations for unprotected `init*` functions; flag logic relying on exact `address(this).balance == x`. Slither `suicidal`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:215`

### V-025: Missing Ownership Check in Custom NFT Transfer — Critical
- Aliases: DVL-43
- **Don't:** A custom `transferFrom` that omits the owner/approved check lets anyone move any NFT — stealing staked or escrowed NFTs held by protocols using the custom implementation.
- **Do / Detection:** In custom ERC721/ERC1155 code, verify `_isApprovedOrOwner` (or equivalent) on every transfer path including batch/operator variants; diff against OpenZeppelin reference; test transferring a token you don't own.
- Full entry: `Smart-contract-vulnerability-database_v1.md:223`

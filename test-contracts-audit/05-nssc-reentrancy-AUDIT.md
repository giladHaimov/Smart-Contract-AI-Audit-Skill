# Audit Report — 05-nssc-reentrancy.sol

**Scope:** `/Users/giladhaimov/dev/Smart-Contract-AI-Audit-Skill/test-contracts/05-nssc-reentrancy.sol`
(crytic/not-so-smart-contracts, `re_entrancy/reentrance.sol`, educational deliberately-vulnerable contract, `pragma solidity ^0.4.15`)

**Methodology:** Walked all 293 entries of `Smart-contract-vulnerability-database_v1.md` (Part I V-001..V-197, Part II E-01..E-37, Part III KB-01..KB-59) per `AUDIT_MODE.md`'s category order, cross-referencing `reference/INDEX.md` for triage and opening full entries for any candidate match. The file is a 42-line teaching contract containing one intentionally vulnerable function (`withdrawBalance`) and two intentionally corrected variants (`withdrawBalance_fixed`, `withdrawBalance_fixed_2`) for comparison.

---

## 1. Summary

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 0 |
| Medium | 2 |
| Low | 3 |
| **Total** | **6** |

Scope: single file, 42 lines, `contract Reentrance`. One state-changing mapping (`userBalance`), three withdrawal-path functions, no access control, no proxy/oracle/governance/token-standard surface. The headline finding is the intentional classic reentrancy bug in `withdrawBalance` (V-001); the remaining findings are compiler-era hygiene issues consistent with the file's 2016-vintage `pragma solidity ^0.4.15`.

---

## 2. Findings

### [V-001] Classic Reentrancy (State Change After External Call) — Critical

**Location:** `05-nssc-reentrancy.sol:14-21`, `withdrawBalance`

**Issue:** `withdrawBalance()` sends the caller's entire tracked balance via `msg.sender.call.value(userBalance[msg.sender])()` on line 17 — a raw low-level call that forwards all remaining gas — and only zeroes `userBalance[msg.sender]` afterward, on line 20. If `msg.sender` is a contract, its fallback function executes during the `call.value()` and can re-enter `withdrawBalance()` before line 20 runs, seeing the same non-zero `userBalance[msg.sender]` on every re-entrant call and draining far more ETH than the caller ever deposited. This is the textbook DAO-hack pattern (Checks-Effects-Interactions violated: external interaction happens before the effect). The file's own inline comments even flag the fallback-callback risk without fixing it in this function — the two functions below (`withdrawBalance_fixed`, `withdrawBalance_fixed_2`) demonstrate the fix but do not remove or protect the vulnerable original, so the exploitable path remains fully callable in the deployed contract.

**Reference:** V-001 — Smart-contract-vulnerability-database_v1.md:29
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-107 https://swcregistry.io/docs/SWC-107 ; DeFiVulnLabs (DVL-4) https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** Apply Checks-Effects-Interactions: zero the balance before the external call, exactly as `withdrawBalance_fixed` already does in this same file —
```solidity
function withdrawBalance(){
    uint amount = userBalance[msg.sender];
    userBalance[msg.sender] = 0;
    if( ! (msg.sender.call.value(amount)() ) ){
        throw;
    }
}
```
or use `msg.sender.transfer(amount)` after zeroing (as in `withdrawBalance_fixed_2`), or add a `nonReentrant` mutex. In production, remove the vulnerable variant entirely rather than leaving it reachable alongside the corrected ones.

---

### [V-187] Outdated Compiler / Floating Pragma — Medium

**Location:** `05-nssc-reentrancy.sol:1`, `pragma solidity ^0.4.15;`

**Issue:** The pragma floats (`^0.4.15` resolves to any `>=0.4.15 <0.5.0` compiler, i.e. up to `0.4.26`). This is not a library file (it's a standalone deployable `contract Reentrance`), so the entry's library carve-out does not apply. `0.4.x` predates Solidity's built-in arithmetic overflow checks (added in `0.8.0`), predates mandatory explicit function visibility (added in `0.5.0`), and predates `revert()`/custom errors (still uses `throw`). Several official known-compiler-bug windows (Part III) overlap the resolvable `0.4.15–0.4.26` range — e.g. KB-17 (`ABIDecodeTwoDimensionalArrayMemory`, 0.4.16→0.8.4), KB-32 (`SignedArrayStorageCopy`, 0.4.7→0.5.10), KB-33/KB-34 (ABIEncoderV2 storage/constructor-arg bugs, 0.4.16→0.5.9/0.5.10), KB-40 (`ExpExponentCleanup`, →0.4.25), KB-41 (`EventStructWrongData`, 0.4.17→0.4.25), KB-43 (`NestedArrayFunctionCallDecoder`, →0.4.22), KB-44 (`ZeroFunctionSelector`, →0.4.18) — see Coverage note below for why none of these are separately flagged as live findings.

**Reference:** V-187 — Smart-contract-vulnerability-database_v1.md:1549
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-102 https://swcregistry.io/docs/SWC-102 ; SWC-103 https://swcregistry.io/docs/SWC-103 ; docs.soliditylang.org https://docs.soliditylang.org

**Suggested fix:** Pin an exact, modern compiler version (`pragma solidity 0.8.24;` or later) and port the contract forward — this also removes the need for the `V-037` SafeMath concern below and replaces `throw` with `revert()`. If the file must stay `0.4.x` for historical/educational fidelity, pin the exact patch (`pragma solidity 0.4.26;`, the last and most-patched `0.4.x` release) instead of floating.

---

### [V-135] Fixed Gas Stipends (`transfer`) Break on Gas Repricing — Medium

**Location:** `05-nssc-reentrancy.sol:33-40`, `withdrawBalance_fixed_2`

**Issue:** `withdrawBalance_fixed_2` sends ETH via `msg.sender.transfer(userBalance[msg.sender])` (line 38), which hardcodes a 2300-gas stipend. This correctly blocks reentrancy (the intent of this "fixed" variant), but any recipient whose fallback needs more than 2300 gas — a Gnosis Safe / multisig wallet, or any contract whose fallback got more expensive after the EIP-1884 `SLOAD` repricing — will have every withdrawal permanently revert, since there is no alternate withdrawal path once `transfer` starts failing. Note also that the external call on line 38 still precedes the zeroing on line 39 (Interactions before Effects); this is safe only because `transfer`'s stipend is too small for the callee to re-enter and call back into `withdrawBalance_fixed_2`, not because the ordering itself is correct.

**Reference:** V-135 — Smart-contract-vulnerability-database_v1.md:1123 (also E-25 — Smart-contract-vulnerability-database_v1.md:1835, same root cause, not double-counted)
**External refs:** Solodit https://solodit.cyfrin.io ; DeFiVulnLabs https://github.com/SunWeb3Sec/DeFiVulnLabs ; SWC-134 https://swcregistry.io/docs/SWC-134 ; docs.soliditylang.org https://docs.soliditylang.org

**Suggested fix:** Reorder to Checks-Effects-Interactions (zero the balance before sending, as in `withdrawBalance_fixed`) and replace `transfer()` with `call{value: amount}("")` guarded by a reentrancy modifier, so smart-contract-wallet recipients are not permanently locked out of their own funds.

---

### [V-037] Integer Overflow / Underflow (pre-0.8, no SafeMath) — Low

**Location:** `05-nssc-reentrancy.sol:11`, `addToBalance`

**Issue:** `userBalance[msg.sender] += msg.value;` runs under `pragma solidity ^0.4.15`, which has no built-in overflow protection (added only in `0.8.0`) and this file uses no SafeMath library — matching the entry's detection line ("pragma <0.8 without SafeMath on user-influenced values") exactly. Rated Low rather than the database's default High because the accumulator is `msg.value`-driven ETH, and wrapping `userBalance[msg.sender]` past `2^256-1` wei would require depositing roughly `10^51` times the entire circulating ETH supply — not reachable in practice. Flagged for completeness/hygiene rather than as an exploitable path in this specific instance.

**Reference:** V-037 — Smart-contract-vulnerability-database_v1.md:323
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-101 https://swcregistry.io/docs/SWC-101 ; DeFiVulnLabs https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** Upgrade to Solidity `>=0.8.0` (also addresses V-187) to get compiler-enforced checked arithmetic; if staying on `0.4.x`, use OpenZeppelin's `SafeMath` for the `+=`.

---

### [V-193] Use of Deprecated Solidity Functions — Low

**Location:** `05-nssc-reentrancy.sol:6` (`constant`), `:18`, `:29` (`throw`)

**Issue:** `getBalance` is declared `constant` (line 6) — the pre-`0.4.17` spelling of `view`/`pure`, deprecated in favor of `view`. `withdrawBalance` and `withdrawBalance_fixed` both use `throw` (lines 18 and 29) to revert — `throw` was deprecated in Solidity `0.4.13` in favor of `revert()`/`require()`/`assert()` and was fully removed in `0.5.0`, meaning this file cannot compile as-is on any `>=0.5.0` compiler despite the floating pragma nominally allowing up to `0.4.26` (it stops there for this exact reason).

**Reference:** V-193 — Smart-contract-vulnerability-database_v1.md:1597
**External refs:** SWC-111 https://swcregistry.io/docs/SWC-111

**Suggested fix:** Replace `constant` with `view`, and replace each `throw` with `revert("withdraw failed")` (or `require(...)`), which also unblocks upgrading past `0.5.0` per the V-187 fix above.

---

### [V-145] State Variable Default Visibility — Low

**Location:** `05-nssc-reentrancy.sol:4`, `mapping (address => uint) userBalance;`

**Issue:** `userBalance` is declared with no explicit visibility keyword, defaulting to `internal`. In this file the omission is benign — a dedicated `getBalance(address)` getter (line 6) already exposes read access — but it is exactly the pattern SWC-108 warns about: a reviewer or integrator skimming the state-variable declarations could wrongly assume the mapping is `public` (auto-getter) or wrongly assume the data is inaccessible, when in practice `internal` storage is still fully readable off-chain via `eth_getStorageAt`.

**Reference:** V-145 — Smart-contract-vulnerability-database_v1.md:1205
**External refs:** SWC-108 https://swcregistry.io/docs/SWC-108

**Suggested fix:** Add explicit `internal` (or `public`, removing the now-redundant `getBalance`) to `mapping (address => uint) userBalance;` for clarity.

---

## 3. Coverage note

All 293 entries were walked category-by-category per the mandated order (Reentrancy → Access Control → Oracle → Math & Rounding → Accounting & Fees → Token Standards → DeFi Mechanics → Proxy & Upgradeability → DoS → MEV & Front-running → Signature & Replay → External Calls → Storage → Logic Error → Governance → Cross-Chain & Multichain → Other → Part II E-01..E-37 → Part III KB-01..KB-59).

Whole categories were not applicable to this file and are noted rather than force-fit:

- **Reentrancy beyond V-001 (V-002..V-009):** single shared-state variable (`userBalance`), no cross-function/cross-contract state coupling, no NFT/ERC777/flash-loan hooks, no modifier-embedded external calls — only the one classic-reentrancy instance applies.
- **Access Control (V-010..V-025):** no `owner`, no privileged roles, no `selfdestruct`, no initializer — every function only ever touches `msg.sender`'s own balance entry, so there is no privilege boundary to bypass.
- **Oracle (V-026..V-036):** no price feeds or external data dependencies.
- **Accounting & Fees, Token Standards, DeFi Mechanics (V-049..V-083):** no fee logic, no ERC20/721/777 token, no AMM/lending/staking mechanics — this is a plain ETH balance ledger.
- **Proxy & Upgradeability (V-084..V-093):** no `delegatecall`, no proxy pattern, no constructor at all.
- **DoS (V-094..V-105) beyond the gas-stipend note under V-135:** no loops, no arrays, no push-payment-to-untrusted-list pattern.
- **MEV & Front-running (V-106..V-118), Signature & Replay (V-119..V-128):** no swaps/auctions/deadlines, no signature verification anywhere in the file.
- **External Calls (V-129..V-138) beyond V-135:** the `.call.value()()` return value is explicitly checked with `throw` on failure in both `withdrawBalance` and `withdrawBalance_fixed` (V-129 does not apply — return value is checked, just not before the state write); no `delegatecall`, no unauthenticated callbacks.
- **Storage (V-139..V-150) beyond V-145:** no assembly, no manual slot manipulation, no uninitialized pointers, no struct/array deletion.
- **Logic Error, Governance, Cross-Chain & Multichain (V-151..V-186):** no business-logic surface beyond the one balance ledger; no voting, no bridging.
- **Part II (E-01..E-37):** E-02 (reentrancy, language-level view) is the same root cause as V-001 and is folded into that finding rather than double-reported; E-25 is folded into the V-135 finding for the same reason. No other Part II entry has a matching precondition in this 42-line file (no assembly, no storage packing, no delegatecall, no signature precompile use).
- **Part III (KB-01..KB-59):** version-gated against the floating pragma's resolvable range `0.4.15–0.4.26`. Several bug windows overlap that range (KB-17, KB-32, KB-33, KB-34, KB-40, KB-41, KB-43, KB-44, and the `0.4.x` variants of KB-35/36/37) — enumerated under the V-187 finding above rather than reported as eight-plus separate line items, since they share one root cause (floating into an old, unpinned compiler) and none of their triggering constructs (2D memory arrays, ABIEncoderV2, `exp`, event-emitting libraries, nested-array function-call decoding, raw function-selector arithmetic, constructors) are present anywhere in this file. KB-45 (`DelegateCallReturnValue`) was checked specifically since it's fixed exactly at `0.4.15` — the pragma's floor already has the fix, and the file uses no `delegatecall` regardless, so it is not flagged.

---

## 4. Non-findings worth noting

- **V-010 — Missing/incorrect function visibility:** all five functions (`getBalance`, `addToBalance`, `withdrawBalance`, `withdrawBalance_fixed`, `withdrawBalance_fixed_2`) omit explicit visibility, defaulting to `public` under the pre-0.5.0 rule. Unlike the classic SWC-100 failure mode (an internal helper like `_mint` accidentally left externally callable), every one of these functions is *intended* to be publicly callable by any user acting on their own balance — the implicit-public default matches the intended access level here, so no privilege escalation results. Not reported as a V-010 finding; captured instead under V-193 as a deprecated-syntax hygiene item.
- **V-129 — Unchecked low-level call return value:** `withdrawBalance` and `withdrawBalance_fixed` both wrap `msg.sender.call.value(...)()` in `if (!(...)) { throw; }` — the return value is checked correctly in both. Ruled out.
- **V-011 — Unprotected ether withdrawal draining the whole contract:** withdrawal amounts are gated by each caller's own `userBalance[msg.sender]` entry, not an unguarded whole-balance sweep; a caller can only ever claim what they themselves deposited (modulo the V-001 reentrancy bug, which is reported separately). Ruled out as a distinct finding.
- **V-059 — Force-fed ETH breaking accounting:** the contract never reads `address(this).balance` for any accounting decision (only the `userBalance` mapping), so ETH force-sent via `selfdestruct` cannot desynchronize accounting — it would simply become permanently stuck, which is a design limitation of the demo file rather than an exploitable accounting bug. Ruled out.
- **withdrawBalance_fixed (lines 23-31):** correctly applies Checks-Effects-Interactions — `userBalance[msg.sender]` is cached to `amount` and zeroed (lines 26-27) *before* the external call (line 28). Verified as the correct reference pattern the V-001 fix should follow; not itself a finding.

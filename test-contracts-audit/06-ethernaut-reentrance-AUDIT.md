# Audit Report — 06-ethernaut-reentrance.sol

**Scope:** `test-contracts/06-ethernaut-reentrance.sol` (30 lines, 1 contract: `Reentrance`) — OpenZeppelin/Ethernaut "Reentrance" CTF level, fetched verbatim.

**Not in scope / unverifiable:** the file imports `openzeppelin-contracts-06/math/SafeMath.sol`, which is not present in this repo checkout. `SafeMath`'s revert behavior is treated as unverifiable rather than assumed safe or unsafe.

**Compiler:** `pragma solidity ^0.6.12;` (floating within the 0.6.x line; since 0.6.12 was the final 0.6.x release, the effective compiled version is 0.6.12).

**Dependency:** `import "openzeppelin-contracts-06/math/SafeMath.sol";` — a non-canonical import path (not `@openzeppelin/contracts`), consistent with the community remapping convention Ethernaut/Foundry setups use to pin OpenZeppelin's 0.6.x branch. The `SafeMath.sol` source itself is **not present in this single-file scope** and cannot be resolved or verified — see Non-findings below.

**Method:** Walked all 293 entries of `Smart-contract-vulnerability-database_v1.md` via `reference/INDEX.md`, category order per `AUDIT_MODE.md` (Part I categories in doc order, then Part II E-01..E-37, then Part III KB-01..KB-59 version-gated against the pinned pragma).

---

## 1. Summary

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 1 |
| Medium | 2 |
| Low | 0 |
| **Total** | **4** |

---

## 2. Findings

### [V-001] Classic Reentrancy (State Change After External Call) — Critical

**Location:** `06-ethernaut-reentrance.sol:19-27`, `Reentrance.withdraw(uint256 _amount)`

**Issue:** `withdraw()` sends ETH to `msg.sender` via `msg.sender.call{value: _amount}("")` at line 21 — a raw external call that forwards all remaining gas and hands control to the callee — **before** the sender's ledger entry is decremented at line 25 (`balances[msg.sender] -= _amount;`). If `msg.sender` is a contract, its `receive()`/fallback executes during that call and can re-enter `withdraw()` while `balances[msg.sender]` still holds its pre-withdrawal value, so the `require`-equivalent guard at line 20 (`if (balances[msg.sender] >= _amount)`) passes again on every reentrant call. Recursing until the contract's ETH balance is exhausted lets an attacker who deposited a small amount via `donate()` drain the entire contract balance — this is the textbook Checks-Effects-Interactions violation the "Reentrance" level is built to teach. Same root cause as E-02 (Reentrancy, Language-Level View) — folded into this one finding per audit-mode guidance rather than double-reported.

**Reference:** V-001 — Smart-contract-vulnerability-database_v1.md:29 (see also E-02 — Smart-contract-vulnerability-database_v1.md:1651)

**External refs:** https://swcregistry.io/docs/SWC-107, https://solodit.cyfrin.io, https://github.com/SunWeb3Sec/DeFiVulnLabs, https://docs.soliditylang.org (security considerations — re-entrancy)

**Suggested fix:** Apply Checks-Effects-Interactions: decrement `balances[msg.sender]` *before* the external call, e.g.
```solidity
function withdraw(uint256 _amount) public {
    require(balances[msg.sender] >= _amount, "insufficient balance");
    balances[msg.sender] -= _amount;
    (bool result,) = msg.sender.call{value: _amount}("");
    require(result, "transfer failed");
}
```
and/or add a `nonReentrant` modifier (OpenZeppelin `ReentrancyGuard`) as defense in depth.

---

### [V-037] Integer Underflow on Unsafe Subtraction (pre-0.8, no SafeMath) — High

**Location:** `06-ethernaut-reentrance.sol:25`, `Reentrance.withdraw(uint256 _amount)`

**Issue:** The contract is pinned to Solidity `^0.6.12` (pre-0.8, no built-in overflow/underflow checks) and explicitly uses `SafeMath` for the credit path (`balances[_to] = balances[_to].add(msg.value);`, line 12), but the debit path at line 25 uses raw `balances[msg.sender] -= _amount;` instead of `.sub()`. Combined with the V-001 reentrancy above: during the recursive `withdraw()` chain, every nested call frame's `.call{value:}` eventually fails once the contract's ETH is exhausted (the low-level call returns `false` rather than reverting), and execution falls through to line 25 in each unwinding frame. The **innermost** frame decrements `balances[msg.sender]` to `0` first; the **next** frame up then executes `balances[msg.sender] -= _amount` against an already-zeroed balance, silently wrapping to a near-`type(uint256).max` value with no revert, since this subtraction is outside `SafeMath`'s protection. This corrupts the victim's own ledger entry independent of, and in addition to, the ETH drained by V-001.

**Reference:** V-037 — Smart-contract-vulnerability-database_v1.md:323

**External refs:** https://swcregistry.io/docs/SWC-101, https://solodit.cyfrin.io, https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** Use `balances[msg.sender] = balances[msg.sender].sub(_amount);` (consistent with the `.add()` used in `donate()`), or fix V-001 first (Checks-Effects-Interactions ordering) — with the state update moved ahead of the external call, the underflow precondition (reaching the debit line with a stale/zeroed balance from nested frames) no longer arises.

---

### [V-167] Code With No Effects — Ignored Success Branch — Medium

**Location:** `06-ethernaut-reentrance.sol:22-24`, `Reentrance.withdraw(uint256 _amount)`

**Issue:** The call's success is captured (`(bool result,) = msg.sender.call{value: _amount}("");`) and branched on (`if (result) { ... }`), but the body of that branch is the bare expression statement `_amount;` — a self-reference with no assignment, no call, no effect. This is the exact "statement with no side effects compiles silently" pattern the entry describes. Net effect: the success/failure of the ETH transfer is checked but never acted upon — a failed transfer (`result == false`) is silently swallowed and `balances[msg.sender] -= _amount` still executes unconditionally on the next line regardless of whether the ETH actually left the contract, and a successful transfer triggers a no-op instead of any real bookkeeping/event.

**Reference:** V-167 — Smart-contract-vulnerability-database_v1.md:1383

**External refs:** https://swcregistry.io/docs/SWC-135

**Suggested fix:** Either enforce the check (`require(result, "call failed");`) or remove the dead branch entirely — see the V-001 suggested fix above, which folds this into a single `require(result, ...)` after the (now-safe, post-effects) external call.

---

### [V-187] Outdated / Floating Compiler Pragma (0.6.x, EOL) — Medium

**Location:** `06-ethernaut-reentrance.sol:2`, `pragma solidity ^0.6.12;`

**Issue:** The pragma floats (`^0.6.12`) rather than pinning an exact version, and 0.6.x is an old, no-longer-maintained compiler line — deployable contracts should pin an exact, current version per the entry's guidance ("floating pragmas are acceptable only for libraries"). Cross-referencing Part III of the database against the effective pinned version (0.6.12, the final 0.6.x release): 12 of the 59 cataloged known-compiler-bug entries have an `introduced..fixed` window that nominally spans 0.6.12 — KB-04 (`LostStorageArrayWriteOnSlotOverflow`, 0.1.0→0.8.32), KB-06 (`FullInlinerNonExpressionSplitArgumentEvaluationOrder`, 0.6.7→0.8.21), KB-07 (`MissingSideEffectsOnSelectorAccess`, 0.6.2→0.8.21), KB-09 (`AbiReencodingHeadOverflowWithStaticArrayCleanup`, 0.5.8→0.8.16), KB-10 (`DirtyBytesArrayToStorage`, 0.0.1→0.8.15), KB-12 (`DataLocationChangeInInternalOverride`, 0.6.9→0.8.14), KB-13 (`NestedCalldataArrayAbiReencodingSizeValidation`, 0.5.8→0.8.14), KB-16 (`SignedImmutables`, 0.6.5→0.8.9), KB-17 (`ABIDecodeTwoDimensionalArrayMemory`, 0.4.16→0.8.4), KB-18 (`KeccakCaching`, <0.4.x→0.8.3), KB-19 (`EmptyByteArrayCopy`, <0.4.x→0.7.4), KB-20 (`DynamicArrayCleanup`, <0.4.x→0.7.3). None of these entries' actual trigger constructs (dynamic/2D arrays, `abi.decode` reencoding, signed immutables, function-inlining with multi-value arguments, internal-override data-location changes) are present anywhere in this 30-line contract — it has no arrays, structs, assembly, or overridden functions — so no individual KB finding is raised; the residual risk is generic (being on an EOL line without 0.7.x/0.8.x hardening such as native overflow checks) rather than a demonstrated trigger.

**Reference:** V-187 — Smart-contract-vulnerability-database_v1.md:1549

**External refs:** https://swcregistry.io/docs/SWC-102, https://swcregistry.io/docs/SWC-103, https://solodit.cyfrin.io, https://docs.soliditylang.org

**Suggested fix:** Pin an exact, current compiler version (e.g. `pragma solidity 0.8.25;`) and port the contract off `SafeMath`-based 0.6.x patterns to native checked arithmetic — this would also close V-037 for the credit path automatically and make the debit path's missing `.sub()`/checked-subtraction (V-037 above) revert instead of wrap.

---

## 3. Coverage note

All 293 entries were walked: Part I (V-001..V-197) in category order (Reentrancy → Access Control → Oracle → Math & Rounding → Accounting & Fees → Token Standards → DeFi Mechanics → Proxy & Upgradeability → DoS → MEV & Front-running → Signature & Replay → External Calls → Storage → Logic Error → Governance → Cross-Chain & Multichain → Other), Part II (E-01..E-37), and Part III (KB-01..KB-59, version-gated against the pinned `^0.6.12` pragma, effective version 0.6.12).

This is a 30-line, single-contract, no-inheritance, no-admin, no-token-interface ETH ledger — most Part I categories have no applicable surface and produced no findings after being read, not skipped: Access Control (no privileged roles at all — `donate`/`withdraw`/`balanceOf` are intentionally permissionless per-account operations), Oracle, DeFi Mechanics, Token Standards, Proxy & Upgradeability, MEV & Front-running, Signature & Replay, Storage (beyond the one mapping already covered), Governance, Cross-Chain & Multichain. Math & Rounding, Logic Error, Other, and the Reentrancy/External-Calls cluster produced the four findings above.

Part III (KB-01..KB-59): see the V-187 finding for the full version-gating pass — 12 entries' version windows nominally cover 0.6.12 but none has its trigger construct present in this contract, so they are folded into the V-187 finding rather than reported individually.

## 4. Non-findings worth noting

- **V-188 (Vulnerable Dependency Versions) / unresolved `SafeMath` import:** The import path `openzeppelin-contracts-06/math/SafeMath.sol` is not the canonical npm package path and its target file is not present in this single-file audit scope, so the actual `SafeMath` implementation (and thus whether it matches a known-good OpenZeppelin 0.6.x release vs. a modified/backdoored copy) **cannot be verified**. This is flagged as a supply-chain verification gap, not a confirmed vulnerability — the observable usage (`.add()` in `donate()`) behaves exactly as the standard OpenZeppelin 0.6.x `SafeMath.add` would, and nothing in the visible code depends on `SafeMath` internals in a way that would change if the import resolved differently. Recommend re-running this check with the actual dependency tree (`node_modules`/`lib`) in scope before sign-off.
- **V-059 (Force-Feeding ETH Breaks Accounting):** `receive() external payable {}` (line 29) and plain `selfdestruct`-forced ETH would inflate `address(this).balance` above `sum(balances)`, but no code path in this contract performs an equality/inequality check against `address(this).balance` — accounting is entirely via the `balances` mapping, so force-feeding has no exploitable effect here. Ruled out.
- **V-129 (Unchecked Low-Level Call Return Value):** The `call`'s return value *is* captured into `result` and branched on (line 22), so it is not silently ignored in the SWC-104 sense; the actual defect is that the branch does nothing and failure isn't enforced — reported as V-167 instead of double-reporting the same three lines under both entries.
- **V-012 (`tx.origin` for authentication) / E-07:** No `tx.origin` usage anywhere in the file. Ruled out.
- **V-101 (Permanently Locked Funds):** `withdraw()` provides a working (if reentrant) exit path for depositors; funds are not structurally lockable. Ruled out.
- **E-24 (63/64 Gas Forwarding Rule) / V-098:** `call{value: _amount}("")` forwards all remaining gas per EVM rules, which is precisely what *enables* full-depth reentrancy in V-001 above rather than a separate griefing concern — folded into V-001, not reported separately.
- **KB-01..KB-59 individually:** see Coverage note and the V-187 finding — version windows overlap the pinned 0.6.12 compiler for 12 entries, but no contract code matches any of their trigger patterns, so no individual KB finding is raised.

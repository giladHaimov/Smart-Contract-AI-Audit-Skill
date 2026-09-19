# Audit Report — 01-weth9.sol

**Scope:** `test-contracts/01-weth9.sol`
(WETH9 — canonical Wrapped Ether, fetched verbatim from https://github.com/gnosis/canonical-weth, GPL-3.0, `pragma solidity >=0.4.22 <0.6`)

**Methodology:** Walked all 293 entries of `Smart-contract-vulnerability-database_v1.md` (Part I V-001..V-197, Part II E-01..E-37, Part III KB-01..KB-59) per `AUDIT_MODE.md`'s category order, cross-referencing `reference/INDEX.md` for triage and opening full entries for any candidate match. WETH9 is a 77-line, 8+-year-battle-tested canonical contract securing tens of billions of dollars in production; the low finding count and absence of Critical/High findings reflects the actual state of the code, not a truncated pass.

---

## 1. Summary

| Severity | Count |
|---|---|
| Critical | 0 |
| High | 0 |
| Medium | 4 |
| Low | 3 |
| **Total** | **7** |

Scope: single file, 60 lines of logic (deposit/withdraw/ERC20), no oracle, no access control surface (no owner/admin functions exist at all), no proxy/upgrade pattern, no signatures, no loops. `withdraw()` correctly follows Checks-Effects-Interactions, so no reentrancy finding was raised despite the ETH-sending call. Findings center on old-Solidity/legacy-ERC20 patterns (approve race, unified fallback, fixed 2300-gas stipend, floating pragma) that are all well-known, accepted characteristics of this specific canonical, immutable, already-deployed contract rather than defects introduced by this file.

---

## 2. Findings

### [V-070] ERC20 Approval Race Condition — Medium

**Location:** `01-weth9.sol:49-53`, `approve`

**Issue:** `approve(guy, wad)` unconditionally overwrites `allowance[msg.sender][guy]` with the new `wad` — no zero-then-set requirement, and no `increaseAllowance`/`decreaseAllowance` helpers are exposed. A spender watching the mempool can front-run an allowance change from N to M: submit `transferFrom` for the old N right before the new `approve(M)` lands, then spend M afterward, extracting `N + M` total instead of the intended `M`. This is the textbook SWC-114 ERC-20 approve race, baked into the ERC-20 `approve` semantics itself.

**Reference:** V-070 — Smart-contract-vulnerability-database_v1.md:591
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-114 https://swcregistry.io/docs/SWC-114

**Suggested fix:** No code change recommended to this specific immutable, already-deployed contract. For new deployments, add `increaseAllowance`/`decreaseAllowance` or require `approve(spender, 0)` before setting a new non-zero allowance; downstream integrators of the existing WETH9 should be advised to zero-out allowances before changing them.

---

### [V-135] Fixed Gas Stipend (`.transfer`) Breaks on Gas Repricing — Medium

**Location:** `01-weth9.sol:38-43`, `withdraw`

**Issue:** `withdraw()` pays out with `msg.sender.transfer(wad)`, which hardcodes a 2300-gas stipend. If the caller is a smart-contract wallet whose `receive`/fallback needs more than 2300 gas (multisigs, Gnosis Safe fallback logic, or any contract wallet after a gas-repricing hard fork such as EIP-1884, which raised `SLOAD` cost enough to break prior 2300-gas assumptions), the withdrawal permanently reverts for that caller — their WETH is stuck unless they withdraw through a different, non-contract EOA path. Effects/state are updated before the transfer (correct CEI), so this is an availability issue, not a fund-loss/reentrancy issue.

**Reference:** V-135 — Smart-contract-vulnerability-database_v1.md:1123 (also E-04:1667, E-25:1835)
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-134 https://swcregistry.io/docs/SWC-134 ; DeFiVulnLabs https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** No change recommended to this specific immutable contract (changing it would require a new deployment/migration). For any new WETH-style contract, replace `msg.sender.transfer(wad)` with a reentrancy-guarded `(bool ok, ) = msg.sender.call{value: wad}(""); require(ok);`.

---

### [V-077] Phantom Function — Fake `permit` on WETH9 — Medium

**Location:** `01-weth9.sol:31-33`, `function() external payable` (fallback)

**Issue:** WETH9 has no `permit` function, but its unified fallback (`function() external payable { deposit(); }`) accepts calls to *any* undefined selector without reverting — it simply runs `deposit()` with whatever `msg.value` was sent (0 for a typical `permit(...)` call). A caller/integrator that does `IERC20Permit(weth).permit(...)` (e.g. via `SafeERC20.safePermit`, assuming WETH supports permit because many modern wrapped-ETH-style tokens do) gets a *silent success* — no revert, no allowance set, `Deposit(owner, 0)` emitted — and code that assumes the allowance was set then proceeds to call `transferFrom` for zero effective allowance. This database entry names WETH9 explicitly as the canonical example of this exact failure mode.

**Reference:** V-077 — Smart-contract-vulnerability-database_v1.md:647
**External refs:** DeFiVulnLabs https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** This is an integration-risk finding, not a defect fixable in WETH9 itself (the fallback-swallows-everything behavior is core to the contract's fallback-deposit design). Downstream integrators must not call `permit`/`safePermit` on arbitrary/user-supplied ERC20 addresses without verifying `token.code.length > 0` and confirming the allowance actually increased afterward, rather than trusting the `permit` call not reverting.

---

### [V-187] Floating Pragma Spans a 14-Month, Multi-Point-Release Compiler Window — Medium

**Location:** `01-weth9.sol:16`, `pragma solidity >=0.4.22 <0.6;`

**Issue:** The pragma floats across the entire `0.4.22`–`0.5.17` range (roughly May 2018–July 2019, dozens of point releases), meaning the actual bytecode deployed depends on whichever compiler a given deployer/toolchain selects within that window rather than one audited, pinned version. Per Part III of the database (KB-01..KB-59, version-gated against `introduced..fixed`), several known compiler bugs' vulnerable windows fall entirely or partly inside `[0.4.22, 0.6.0)`, e.g.: KB-18 `KeccakCaching` (before-0.4.x–0.8.3, Medium), KB-19 `EmptyByteArrayCopy` (before-0.4.x–0.7.4, Medium), KB-20 `DynamicArrayCleanup` (before-0.4.x–0.7.3, Medium), KB-28 `YulOptimizerRedundantAssignmentBreakContinue` backport (0.5.8–0.5.16, Medium), KB-32 `SignedArrayStorageCopy` (0.4.7–0.5.10, Medium), and KB-40 `ExpExponentCleanup` (before-0.4.x–0.4.25, High). None of these have a triggering code pattern in this specific file (no inline assembly, no signed arrays, no `**` exponentiation, no byte-array-to-storage copies), so no single KB entry is raised as a standalone live finding — but the floating pragma itself is the real, present-tense risk: any future recompilation of this exact source under an untested point release inherits whatever bugs exist in that specific version, not just the ones catalogued today.

**Reference:** V-187 — Smart-contract-vulnerability-database_v1.md:1549
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-102 https://swcregistry.io/docs/SWC-102 ; SWC-103 https://swcregistry.io/docs/SWC-103 ; docs.soliditylang.org https://docs.soliditylang.org

**Suggested fix:** No change recommended for the already-deployed, immutable mainnet WETH9 (changing its pragma would require a new deployment and would not affect the live contract's bytecode). For any new deployment or fork of this source, pin an exact version — either the last pre-0.6 release actually used at deployment (`pragma solidity 0.4.22;` verified against the real deployed bytecode) if bit-for-bit compatibility matters, or migrate the source to a modern pinned `0.8.x` pragma for new deployments.

---

### [V-076] No Zero-Address / Self-Address Check in `transferFrom` — Low

**Location:** `01-weth9.sol:59-76`, `transferFrom` (and `transfer`, which calls it)

**Issue:** `transferFrom(src, dst, wad)` never validates `dst != address(0)` or `dst != address(this)`. A `transfer`/`transferFrom` to `address(0)` permanently increments `balanceOf[address(0)]` with no way to recover it (no `zeroAddress` withdraw path). A transfer to `address(this)` (the WETH9 contract's own address) similarly credits `balanceOf[<WETH9 address>]`, which is unrecoverable in practice — nothing in the contract lets `msg.sender == address(this)` invoke `withdraw()`, so those tokens are effectively stuck forever. This only affects a user who mis-targets their own transfer (not attacker-exploitable against a third party), and is standard behavior for pre-"safe-ERC20" era tokens, but it does match the entry's failure mode exactly.

**Reference:** V-076 — Smart-contract-vulnerability-database_v1.md:639
**External refs:** ConsenSys best practices https://consensys.github.io/smart-contract-best-practices

**Suggested fix:** No change recommended to this immutable, already-deployed contract (this is accepted, unchangeable behavior of the live token millions of contracts already integrate against). For a new token, add `require(dst != address(0) && dst != address(this))` in the internal transfer path.

---

### [V-137] Fallback With Heavy Logic Under the 2300-Gas Stipend — Low

**Location:** `01-weth9.sol:31-37`, `function() external payable` (fallback) calling `deposit`

**Issue:** The unified fallback forwards straight into `deposit()`, which performs a storage write (`balanceOf[msg.sender] += msg.value`) and emits a `Deposit` event (a `LOG3`). Both together cost well over 2300 gas (a cold `SSTORE` alone is 20000 gas pre-Berlin / 2900+ warm-access cost post-Berlin, plus `LOG` costs). Any sender using `.send()`/`.transfer()` to push plain ETH into WETH9 (only 2300 gas forwarded) will have the fallback run out of gas and revert — the deposit silently fails rather than crediting the sender. Only `.call{value: x}("")` (forwarding all gas) or calling `deposit()` directly succeeds. This is a genuine, real-world WETH9 gotcha (several historical incidents of naive integrations pushing ETH via `.transfer` into WETH9 and having it revert). Separately, the fallback also accepts arbitrary non-empty calldata for any undefined selector without reverting, masking typos/mistaken calls as silent deposits (see the related V-077 finding above for the specific `permit` instance of this).

**Reference:** V-137 — Smart-contract-vulnerability-database_v1.md:1139
**External refs:** ConsenSys best practices https://consensys.github.io/smart-contract-best-practices

**Suggested fix:** No change recommended to this immutable, already-deployed contract. For a new deposit-only fallback/receive, either keep the logic within a 2300-gas budget or clearly document that only `.call{value:}` (or the explicit `deposit()` function) is a supported deposit path; add `require(msg.data.length == 0)` on a receive-only fallback so mistaken calls with data revert visibly instead of silently depositing.

---

### [V-037] Unchecked Arithmetic on User-Influenced Balances (pre-0.8 Solidity) — Low

**Location:** `01-weth9.sol:34-37`, `deposit`; `01-weth9.sol:59-76`, `transferFrom`

**Issue:** The pragma (`>=0.4.22 <0.6`) predates Solidity 0.8's built-in overflow/underflow checks, and no SafeMath-style library is used. `balanceOf[msg.sender] += msg.value` (deposit) and `balanceOf[dst] += wad` (transferFrom) are unchecked additions on user-influenced values, matching this entry's detection line ("Pragma <0.8 without SafeMath on user-influenced values") literally. In practice this is not exploitable: `balanceOf` values are ETH-backed 1:1, and total Ether supply (~1.2×10²⁶ wei) is many orders of magnitude below `2^256` (~1.15×10⁷⁷), so no realistic sequence of deposits/transfers can wrap a `uint256` balance. All subtractions (`balanceOf[msg.sender] -= wad` in `withdraw`, `balanceOf[src] -= wad` and `allowance[src][msg.sender] -= wad` in `transferFrom`) are correctly preceded by an adequacy `require`, so underflow is already guarded. Rated Low (not the database's default High) specifically because of ETH's bounded supply — this would be a High-severity gap in a similarly-unprotected token contract with a mintable or otherwise unbounded supply.

**Reference:** V-037 — Smart-contract-vulnerability-database_v1.md:323 (also E-08:1699, E-35:1915)
**External refs:** Solodit https://solodit.cyfrin.io ; SWC-101 https://swcregistry.io/docs/SWC-101 ; DeFiVulnLabs https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** No change recommended to this immutable, already-deployed contract (overflow is not practically reachable given ETH's supply cap). For any new token where supply is not inherently bounded the same way, use Solidity ≥0.8's checked arithmetic or an explicit SafeMath library.

---

## 3. Coverage note

All 293 entries were walked category-by-category per the mandated order (Reentrancy → Access Control → Oracle → Math & Rounding → Accounting & Fees → Token Standards → DeFi Mechanics → Proxy & Upgradeability → DoS → MEV & Front-running → Signature & Replay → External Calls → Storage → Logic Error → Governance → Cross-Chain & Multichain → Other → Part II E-01..E-37 → Part III KB-01..KB-59).

Whole categories were not applicable to this specific file and are noted rather than force-fit:

- **Reentrancy (V-001..V-009, E-02):** `withdraw()` is the only function moving value out and it follows strict Checks-Effects-Interactions (`require` → state decrement → `.transfer` → event); no state write follows an external call anywhere in the file. Ruled out (see Non-findings for detail).
- **Access Control (V-010..V-025):** the contract has zero privileged/admin functions — no owner, no minter role, no pausable, no selfdestruct, no initializer. The entire category is structurally not applicable.
- **Oracle (V-026..V-036):** no price feeds or external data dependencies.
- **Accounting & Fees (V-049..V-062)** beyond V-059 (see Non-findings): no share/reward/fee mechanics — `balanceOf` is a 1:1 ETH ledger.
- **DeFi Mechanics (V-078..V-083), Governance (V-175..V-182), Proxy & Upgradeability (V-084..V-093), Signature & Replay (V-119..V-128), Cross-Chain & Multichain (V-183..V-186):** no staking/lending/AMM logic, no voting, no proxy/`delegatecall` pattern, no signature verification, no bridging — this is a standalone base ETH-wrapper contract.
- **DoS (V-094..V-105)** beyond the fallback-gas point already folded into V-137: no loops, no unbounded arrays, no queue structures; `withdraw()` only pushes ETH to `msg.sender` (self-initiated, not a forced push to a third party), so V-095/V-101 do not apply.
- **MEV & Front-running (V-106..V-118)** beyond V-070: no swaps, auctions, liquidations, or deadline-sensitive logic.
- **External Calls (V-129..V-138)** beyond V-135/V-137: `.transfer()` is a high-level call that reverts on failure automatically, so V-129 (unchecked low-level call return value) does not apply; no `.call`/`.delegatecall`/`.staticcall` anywhere in the file.
- **Storage (V-139..V-150), most of Part II's storage-tagged entries (E-09, E-13..E-20, E-28, E-29, E-33):** no inline assembly, no manual storage-slot manipulation, no uninitialized pointers, no dynamic arrays or mappings-in-structs deletion.
- **Logic Error (V-151..V-174):** manually traced every function; no wrong-formula, off-by-one, or ordering bug found (see Non-findings for the specific self-transfer check).
- **Part II (E-01, E-03, E-05..E-07, E-10..E-24, E-26, E-30..E-34, E-36, E-37):** each checked against the file; either the language-level concern maps onto a Part I finding already reported above (E-02→CEI ruled out, E-04/E-25→V-135, E-08/E-35→V-037), or the triggering construct (assembly, delegatecall, signatures, randomness, transient storage, CREATE2, precompile calls) is simply absent from the file.
- **Part III (KB-01..KB-59):** version-gated against the floating pragma's effective range `[0.4.22, 0.6.0)`. Folded into the V-187 finding above rather than raised as 15+ separate near-duplicate low-value findings, since none of the in-range KB entries have a matching triggering code construct in this minimal file (no assembly, no signed arrays, no `**`, no ABIEncoderV2 struct/array edge cases, no byte-array-to-storage copies) — the live risk is the floating pragma itself, not a specific already-triggered bug.

---

## 4. Non-findings worth noting

- **V-001/V-002/E-02 — Reentrancy in `withdraw`:** `require(balanceOf[msg.sender] >= wad); balanceOf[msg.sender] -= wad; msg.sender.transfer(wad); emit Withdrawal(...)` — state is decremented *before* the external call, and `.transfer` additionally caps the callee to a 2300-gas stipend that cannot re-enter `withdraw`/`transfer`/`transferFrom` (each requires more than 2300 gas to execute meaningfully). Correct CEI. Ruled out.
- **V-054 — Self-transfer (`src == dst`) inflating balances:** in `transferFrom`, `balanceOf[src] -= wad;` followed by `balanceOf[dst] += wad;` with `src == dst` operates on the same storage slot sequentially (decrement then increment), netting to unchanged balance — not the double-count pattern seen in some flawed token implementations. Ruled out.
- **V-059/E-27 — Force-fed ETH via `selfdestruct` breaking accounting:** `totalSupply()` is defined as `address(this).balance` directly (not a separately-tracked counter), so force-fed ETH simply makes `totalSupply()` read slightly higher than `sum(balanceOf)` — it never causes the contract to be under-collateralized relative to `balanceOf` claims (deposits/withdrawals always move ETH 1:1 with `balanceOf`), so no fund-loss or insolvency path exists. Ruled out.
- **V-071 — Unlimited/misused approvals:** the `allowance[src][msg.sender] != uint(-1)` check in `transferFrom` is the standard, intentional "infinite approval" gas-saving pattern (skips decrementing a max-uint allowance) — correct, documented ERC20 convention, not an accidental unlimited-approval bug in the contract itself. The risk the entry describes (a user granting max approval to an untrusted spender) is caller-side, not something WETH9 can prevent. Ruled out as a finding against this file.
- **V-129/E-05/E-36 — Unchecked low-level call return value:** `.transfer()` is a Solidity high-level call that automatically reverts the whole transaction on failure (unlike `.call`/`.send`), so there is no unchecked-return-value pattern here despite superficially resembling one. Ruled out.
- **V-145 — State variable default visibility:** `name`, `symbol`, `decimals`, `balanceOf`, `allowance` are all explicitly declared `public`. Ruled out.
- **V-190/V-194 — Hidden backdoor / arbitrary jump via inline assembly:** `grep -n "assembly"` over the file returns no matches — zero assembly blocks anywhere. Ruled out.
- **V-191 — Right-to-left-override control characters:** scanned the full file for Unicode codepoints > U+2000 (covers U+202D/U+202E bidi controls); none found. Ruled out.
- **V-195 — Missing events for critical changes:** every state-changing function (`deposit`, `withdraw`, `approve`, `transferFrom`) emits a corresponding event (`Deposit`, `Withdrawal`, `Approval`, `Transfer`). Ruled out.
- **V-042/V-043/V-044 — Downcasting/upcasting/signed-conversion errors:** the only fixed-width type narrower than `uint256` is `decimals` (`uint8 = 18`, a hardcoded constant never computed from a wider type). No downcasting or signed/unsigned conversion occurs anywhere. Ruled out.

---

## Statistics

- Findings by severity: Critical 0, High 0, Medium 4, Low 3, Total 7.
- No finding required a code change to the file itself: this exact bytecode is already deployed and immutable at `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` on Ethereum mainnet holding billions of dollars — every "Suggested fix" above is framed for new deployments/forks, not a patch to this file.

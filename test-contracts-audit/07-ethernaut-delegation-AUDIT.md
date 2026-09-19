# Audit Report — 07-ethernaut-delegation.sol

**Scope:** `test-contracts/07-ethernaut-delegation.sol` (31 lines, 2 contracts: `Delegate`, `Delegation`) — OpenZeppelin/Ethernaut "Delegation" CTF level, fetched verbatim.

**Compiler:** `pragma solidity ^0.8.0;` (floating, no upper bound — resolves to any 0.8.x compiler ≥ 0.8.0 available to the build).

**Method:** Walked all 293 entries of `Smart-contract-vulnerability-database_v1.md` via `reference/INDEX.md`, category order per `AUDIT_MODE.md` (Part I categories in doc order, then Part II E-01..E-37, then Part III KB-01..KB-59 version-gated against the pinned pragma).

---

## 1. Summary

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 0 |
| Medium | 0 |
| Low | 2 |
| **Total** | **3** |

---

## 2. Findings

### [V-132] Unsafe `delegatecall` — Storage-Slot Collision Enables Full Ownership Takeover — Critical

**Location:** `07-ethernaut-delegation.sol:25-30`, `Delegation.fallback()` (root cause enabled by `Delegate.pwn()` at lines 11-13)

**Issue:** `Delegation.fallback()` forwards the *entire* `msg.data` of any call it doesn't otherwise match to `address(delegate).delegatecall(msg.data)` (line 26) with no selector whitelist and no check on what function is being invoked. `delegatecall` executes the target's code (`Delegate`'s bytecode) but in the **caller's (`Delegation`'s) storage, `address(this)`, and `msg.sender` context** — only the code comes from `Delegate`.

The two contracts' storage layouts collide exactly at slot 0: `Delegate.owner` (line 5) is `Delegate`'s slot 0, and `Delegation.owner` (line 17) is `Delegation`'s slot 0 (`Delegation.delegate` at line 18 is slot 1). Because Solidity storage layout is purely positional (see E-13/E-14 mechanism), any `SSTORE` to slot 0 that `Delegate`'s code performs, when run via `delegatecall` from `Delegation`, physically writes to `Delegation`'s slot 0 — i.e., `Delegation.owner`, not `Delegate.owner`.

`Delegate.pwn()` (lines 11-13) is a public, unauthenticated function that does exactly `owner = msg.sender;` — an `SSTORE` to slot 0. An attacker sends a raw transaction to `Delegation` with calldata `abi.encodeWithSignature("pwn()")`. `Delegation` has no matching function selector, so `fallback()` fires and `delegatecall`s that calldata into `Delegate`. `Delegate.pwn()` executes, writes `msg.sender` (the attacker, preserved through `delegatecall`) into slot 0 of the *executing contract's* storage — which is `Delegation`'s storage. `Delegation.owner` is now the attacker, granting full ownership of `Delegation` with a single unauthenticated call and zero ETH.

**Reference:** V-132 — Smart-contract-vulnerability-database_v1.md:1099 (see also E-22 — Smart-contract-vulnerability-database_v1.md:1811, and the missing-access-control precondition on `pwn()`, V-010 — Smart-contract-vulnerability-database_v1.md:103)

**External refs:** https://swcregistry.io/docs/SWC-112 (Unsafe delegatecall), https://solodit.cyfrin.io, https://github.com/SunWeb3Sec/DeFiVulnLabs, https://docs.soliditylang.org (delegatecall / storage-layout security considerations)

**Suggested fix:** Do not forward arbitrary `msg.data` to a delegatecall target from an untrusted/public-facing fallback. Concretely: (1) remove the generic forwarding fallback entirely and expose only the specific functions `Delegation` intends to proxy, each calling `delegate.someFunction()` via a normal (non-delegatecall) external call if state isolation is desired; or (2) if delegatecall-based proxying to shared logic is truly intended, adopt a structured-storage/EIP-1967 proxy pattern (e.g. OpenZeppelin `Proxy`/`UUPSUpgradeable`) where the implementation contract is written to never declare state variables that alias the proxy's own slots, and restrict which selectors may be delegated; and (3) independently, add `require(msg.sender == owner)` to `pwn()` (or delete it) so that even if reached, an unauthorized caller cannot mutate slot 0.

---

### [V-010] Missing Access Control on `Delegate.pwn()` — Low

**Location:** `07-ethernaut-delegation.sol:11-13`, `Delegate.pwn()`

**Issue:** Independent of the delegatecall vector above, `Delegate.pwn()` is `public` with no modifier and unconditionally sets `owner = msg.sender`. If `Delegate` is ever interacted with directly (its address is discoverable via the `delegate` variable's storage slot even though the field itself is not marked `public` in `Delegation`), any caller can become `Delegate`'s owner. Impact here is low in isolation — nothing in this file gates behavior on `Delegate.owner` — but it is a genuine, independently-exploitable missing-access-control instance on a state-changing function, and it is precisely the primitive the Critical finding above pivots through.

**Reference:** V-010 — Smart-contract-vulnerability-database_v1.md:103

**External refs:** https://swcregistry.io/docs/SWC-100, https://solodit.cyfrin.io, https://github.com/SunWeb3Sec/DeFiVulnLabs

**Suggested fix:** Add `require(msg.sender == owner, "not owner");` (or an `onlyOwner` modifier) to `pwn()`, or remove the function if it serves no legitimate purpose. This alone would also close the Critical finding's exploit path, since the `delegatecall` preserves `msg.sender`, and the check would evaluate against `Delegation.owner` (the real owner) when reached via the fallback.

---

### [V-187] Floating Pragma — Low

**Location:** `07-ethernaut-delegation.sol:2`, `pragma solidity ^0.8.0;`

**Issue:** The pragma floats across the entire `0.8.x` line with no upper bound, so the contract can be compiled with any compiler from 0.8.0 up to whatever the newest 0.8.x release is at build time. For a deployable (non-library) contract this means the actually-deployed bytecode depends on whichever toolchain compiles it, rather than a version the author verified.

**Reference:** V-187 — Smart-contract-vulnerability-database_v1.md:1549

**External refs:** https://swcregistry.io/docs/SWC-103, https://solodit.cyfrin.io, https://docs.soliditylang.org

**Suggested fix:** Pin an exact version, e.g. `pragma solidity 0.8.25;` (or whichever version the deployment toolchain targets), and cross-check that exact version against the Part III known-compiler-bugs list before shipping.

---

## 3. Coverage note

All 293 entries were walked: Part I (V-001..V-197) in category order (Reentrancy → Access Control → Oracle → Math & Rounding → Accounting & Fees → Token Standards → DeFi Mechanics → Proxy & Upgradeability → DoS → MEV & Front-running → Signature & Replay → External Calls → Storage → Logic Error → Governance → Cross-Chain & Multichain → Other), Part II (E-01..E-37), and Part III (KB-01..KB-59, version-gated against the floating `^0.8.0` pragma).

Most Part I categories are not applicable: this file has no ETH/token transfers, no oracle reads, no arithmetic beyond trivial assignment, no loops, no signatures, no governance, and is not itself an upgradeable proxy (it only uses `delegatecall` in a way that mimics one) — so Reentrancy, Oracle, Math & Rounding, Accounting & Fees, Token Standards, DeFi Mechanics, DoS, MEV & Front-running, Signature & Replay, Governance, and Cross-Chain & Multichain produced no findings; this was confirmed by reading, not skipped.

Part III (KB-01..KB-59): the floating pragma technically spans several known-bug windows (e.g. KB-08 `StorageWriteRemovalBeforeConditionalTermination` 0.8.13→0.8.17, KB-11 `InlineAssemblyMemorySideEffects` 0.8.13→0.8.15), but none of their trigger patterns (conditional-termination storage-write elision, inline assembly, ABIEncoderV2 struct/array edge cases, etc.) are present in this 31-line contract — no inline assembly, no structs, no dynamic arrays. No KB finding is raised beyond the general floating-pragma note (V-187) above.

## 4. Non-findings worth noting

- **V-130 / E-23 (Call to Address Without Code):** `Delegation`'s constructor casts `_delegateAddress` to `Delegate` without an `extcodesize`/code-existence check, and `fallback()` doesn't verify `delegate` has code before delegatecalling it. Ruled out as a reportable finding here because a delegatecall to an empty address returns `(true, "")` in this contract with no downstream logic gated on the return value beyond the no-op `if (result) { this; }` (line 27-29) — there is no fund-loss or accounting consequence in this file, unlike the token-transfer scenario V-130 targets.
- **V-137 (Fallback Function Pitfalls):** The fallback intentionally accepts arbitrary calldata to forward it — that's the (flawed) design intent, not an accidental "mistaken call routed to fallback" case the entry targets, and this behavior is already captured as the root cause of the Critical finding above rather than reported separately.
- **E-06 / V-131 (Authorized Proxy / Arbitrary Call From User Input):** Considered as a possible alternate framing (the contract does act as an "authorized proxy" forwarding arbitrary calldata) — folded into the V-132/E-22 finding rather than double-reported, since it's the same root cause (unrestricted delegatecall forwarding) viewed from a different angle, per the audit-mode guidance not to split one root cause into multiple finding blocks.
- **V-090 (`selfdestruct`/`delegatecall` in Implementation Contract):** `Delegate` contains no `selfdestruct`, and `Delegate` is not itself an upgradeable-proxy "implementation" in the OZ sense (no proxy admin, no EIP-1967 slots) — the applicable delegatecall risk is fully captured by V-132/E-22 instead.
- **V-145 (State Variable Default Visibility):** Both `owner` fields are explicitly `public`; `delegate` in `Delegation` (line 18) has no explicit visibility (defaults to `internal`), which is intentional/harmless here (no getter was needed) — not flagged as a finding, just noted.
- **KB-01..KB-59 individually:** see Coverage note above — version windows overlap the floating pragma but no contract code matches any bug's trigger pattern, so no individual KB entry is raised as a separate finding.

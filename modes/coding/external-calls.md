# Coding-Mode Checklist — External Calls (21 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `External Calls` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-129: Unchecked Low-Level Call Return Value — High
- Aliases: Solodit "Unchecked Low-Level Call Return Value", SWC-104 (see also E-05, E-36 in Part II)
- **Don't:** `call`/`delegatecall`/`send`/`staticcall` results ignored; failed transfers or calls are treated as successful, corrupting accounting (relayer "executed" flags, lost funds).
- **Do / Detection:** Grep low-level calls without checking the returned bool; `require(success)` after every call. Slither `unchecked-lowlevel`, `unchecked-send`, `unused-return`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1075`

### V-130: Call to Address Without Code — Medium/High
- Aliases: Solodit "Call to Address Without Code" (SOL-EC-12, SOL-LL-3) (see also E-23 in Part II)
- **Don't:** Low-level calls to EOAs or selfdestructed/zero addresses succeed silently (return true with empty returndata), so token transfers or payouts to non-existent contracts are recorded as done.
- **Do / Detection:** `extcodesize`/contract-existence checks before low-level calls; OpenZeppelin `Address.functionCall`; token address validation.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1083`

### V-131: Arbitrary Call From User Input (Call Injection) — Critical
- Aliases: Solodit "Arbitrary Call From User Input" (SOL-EC-5, SOL-Defi-AS-6), DVL-7 (see also E-06 in Part II)
- **Don't:** User-supplied target + calldata executed by the contract (routers, aggregators, multicalls, "authorized proxies") lets attackers call anything as the contract — draining approvals and balances (the classic approval-drainer pattern).
- **Do / Detection:** `(bool s,) = target.call(data)` with user-controlled target/data; missing target/calldata whitelisting; generic `callOther(addr, payload)`-style functions on contracts holding permissions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1091`

### V-132: Unsafe `delegatecall` — Critical
- Aliases: Solodit "Unsafe delegatecall" (SOL-EC-3/10/11), DVL-3, SWC-112 (see also E-22 in Part II)
- **Don't:** Delegatecalling untrusted or user-supplied addresses executes foreign code in the caller's storage context — total storage/ownership hijack and fund seizure.
- **Do / Detection:** Any `delegatecall` to non-library or non-whitelisted code; delegate-called code preserving the caller's storage layout. Slither `controlled-delegatecall`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1099`

### V-133: Unvalidated Return Data (Returndata Bomb / Truncated Data) — Medium
- Aliases: Solodit "Unvalidated Return Data" (SOL-EC-8/9, SOL-LL-4)
- **Don't:** Callee returns huge returndata (gas-grief via copy costs) or short data that decodes to misleading defaults; precompile calls also skip return-size checks.
- **Do / Detection:** Raw calls decoding returndata without size checks; calls to precompiles.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1107`

### V-134: Unauthenticated Callback / Flash-Loan & Swap Callback Spoofing — Critical
- Aliases: Solodit "Callback Caller Not Verified" (SOL-Defi-AS-12), DVL-46, DVL-51, DVL-80
- **Don't:** Flash-swap/flash-mint callbacks (`uniswapV3SwapCallback`, `onFlashLoan`, `executeOperation`, Sense-style `onSwap`) invoked by anyone let attackers fake the callback to mint/withdraw/settle without paying.
- **Do / Detection:** Callbacks must verify `msg.sender` is the canonical pool/lender — recompute pool address via `CREATE2(factory, salt(token0,token1,fee))` and compare; validate initiator/origin and that token/amounts match a loan actually requested; never trust calldata for these checks; test calling the callback directly as an EOA.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1115`

### V-135: Fixed Gas Stipends (`transfer`/`send`) Break on Gas Repricing — Medium
- Aliases: Solodit "Fixed Gas Stipends" (SOL-Basics-Payment-6), DVL-42, SWC-134 (see also E-25 in Part II)
- **Don't:** `transfer()`/`send()` hardcode 2300 gas; receivers needing more gas (multisigs, Gnosis Safe fallbacks, post-fork repricing like EIP-1884) cause permanent send failures. Fixed `.gas(x)` stipends have the same fragility.
- **Do / Detection:** Grep `.transfer(`/`.send(` for ETH and `{gas: N}`; verify receiver set can't include smart-contract wallets; use `.call{value:}("")` with reentrancy protection.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1123`

### V-136: try/catch Silent Failure & Gas Shortage — Medium
- Aliases: Solodit "try/catch Silent Failure & Gas Shortage" (SOL-Heuristics-5), DVL-88
- **Don't:** try/catch swallows external failures (including out-of-gas inside the try), and execution continues as if the operation succeeded — e.g., a failed repayment or oracle update treated as success.
- **Do / Detection:** Empty `catch {}` blocks or catch paths that proceed as if the call succeeded; returndata-size assumptions; prefer explicit success booleans with enforced handling.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1131`

### V-137: Fallback Function Pitfalls — Low
- Aliases: CR-6
- **Don't:** (1) Heavy logic in `fallback`/`receive` breaks when Ether arrives via `send`/`transfer` (2300 gas); (2) fallbacks that accept arbitrary calldata mask user errors when nonexistent functions are called.
- **Do / Detection:** `receive`/`fallback` gas-dependence; require `require(msg.data.length == 0)` in deposit-only fallbacks so mistaken calls revert visibly.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1139`

### V-138: Interface Types Instead of Raw Addresses — Low
- Aliases: CR-8
- **Don't:** Passing raw `address` parameters and casting internally loses compile-time type safety; wrong contract types are only caught at runtime.
- **Do / Detection:** Function parameters typed `address` that are immediately cast to a contract/interface; recommend typed parameters.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1147`

### E-02: Reentrancy (Language-Level View) — Critical
- Aliases: SC-2 (see also V-001 in Part I)
- **Don't:** Any interaction with another contract or Ether transfer hands over control, letting the callee call back before the interaction completes; applies to any external call and to cross-contract state dependencies.
- **Do / Detection:** Verify Checks-Effects-Interactions ordering; flag state writes after external calls, missing reentrancy guards on withdraw/claim functions, and read-only reentrancy of dependent contracts.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1651`

### E-05: Call Stack Depth — Low
- Aliases: SC-5 (see also V-105, V-129 in Part I)
- **Don't:** External calls fail beyond depth 1024; `.send`/low-level calls return `false` instead of throwing.
- **Do / Detection:** Always check return values of `.send`, `.call`, `.delegatecall`, `.staticcall`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1675`

### E-06: Authorized Proxies (Arbitrary-Call Identity Assumption) — High
- Aliases: SC-6 (see also V-131 in Part I)
- **Don't:** A contract that can call arbitrary addresses with user-supplied data lets users assume the proxy's identity and privileges.
- **Do / Detection:** Flag generic `callOther(addr, payload)`-style functions; ensure proxy contracts hold no permissions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1683`

### E-22: Delegatecall Context Semantics — Critical
- Aliases: EV-10 (see also V-132 in Part I)
- **Don't:** `delegatecall` executes target code in the caller's context: storage, `address(this)`, balance, `msg.sender` and `msg.value` are the caller's. A mismatched storage layout or untrusted target destroys the caller's state.
- **Do / Detection:** Flag delegatecall to user-supplied/untrusted addresses; verify exact storage-layout compatibility between proxy and implementation; check for selfdestruct reachable via delegatecall.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1811`

### E-23: Calls to Non-Existent Contracts Succeed — High
- Aliases: EV-11 (see also V-130 in Part I)
- **Don't:** The EVM treats a call to an address with no code as successful with empty return data; raw `call` in assembly and low-level patterns can bypass Solidity's `extcodesize` check.
- **Do / Detection:** Flag low-level `call` where target code existence isn't verified (e.g., token calls to self-destructed or wrong-chain addresses).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1819`

### E-30: ecrecover Failure and Malleability (Precompile Behavior) — High
- Aliases: EV-18 (see also V-121, V-122 in Part I)
- **Don't:** The `ecrecover` precompile returns `address(0)` on invalid input rather than reverting, and signatures are malleable; unguarded use enables auth bypass/replay.
- **Do / Detection:** Flag `ecrecover` without zero-address check, missing low-`s`/v-range validation, and missing nonce/domain separators (recommend OZ ECDSA/EIP-712).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1875`

### E-31: Precompiled Contracts Range and Chain Differences — Medium
- Aliases: EV-19 (see also V-133 in Part I)
- **Don't:** Addresses 1-0x0a (mainnet) are precompiles implemented by the execution environment; other EVM chains may use different/additional precompiles. Calls have chain-specific gas/behavior and can fail silently.
- **Do / Detection:** For cross-chain contracts, flag interactions with low addresses; always check precompile call success and validate outputs.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1883`

### E-36: Exception Bubbling and Low-Level Call Return Values — High
- Aliases: EV-24 (see also V-129 in Part I)
- **Don't:** High-level Solidity calls bubble up exceptions by default, but `.call`/`.delegatecall`/`.staticcall`/`.send` only return `false` on failure; unchecked return values silently ignore failed transfers/calls.
- **Do / Detection:** Flag every low-level call whose success boolean is unchecked; verify returndata length assumptions after calls.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1923`

### KB-45: DelegateCallReturnValue — introduced 0.3.0, fixed 0.4.15 — Low
- **Don't:** Low-level `.delegatecall()` returned the called function's return value coerced to bool instead of the execution outcome.
- **Do / Detection:** Check solc >= 0.4.15; always check `(bool success, bytes memory data)` semantics explicitly.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2165`

### KB-49: IdentityPrecompileReturnIgnored — introduced before 0.4.x, fixed 0.4.7 — Low
- **Don't:** Failure of the identity precompile (0x04) was ignored by generated code.
- **Do / Detection:** Check solc >= 0.4.7; always check precompile call success flags.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2185`

### KB-53: LibrariesNotCallableFromPayableFunctions — introduced 0.4.0, fixed 0.4.2 — Low
- **Don't:** Library functions threw when called from a call that received Ether.
- **Do / Detection:** Check solc >= 0.4.2; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2205`

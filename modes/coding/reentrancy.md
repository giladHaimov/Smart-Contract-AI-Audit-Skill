# Coding-Mode Checklist — Reentrancy (9 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Reentrancy` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-001: Classic Reentrancy (State Change After External Call) — Critical
- Aliases: Solodit "Classic Reentrancy", DVL-4, SWC-107, CP-1
- **Don't:** A contract sends ETH or calls an external contract before updating internal balances, letting the callee re-enter and repeat the withdrawal against stale state (The DAO). Includes modifier reentrancy (external call inside a modifier whose ordering matters).
- **Do / Detection:** Flag any external call (`.call{value:}`, token transfer, hook) that precedes state updates; verify Checks-Effects-Interactions ordering; value-moving functions without `nonReentrant`; external calls inside modifiers. Slither `reentrancy-eth` / `reentrancy-no-eth` / `reentrancy-benign` / `reentrancy-events`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:29`

### V-002: Cross-Function & Cross-Contract Reentrancy — Critical/High
- Aliases: Solodit "Cross-Function Reentrancy", DVL-49, CP-2
- **Don't:** Attacker re-enters a *different* function sharing the same state (e.g., re-enter `withdraw` during `deposit`'s callback, or `transfer` during `withdrawBalance`), or a sibling contract in the same system, bypassing per-function guards or exploiting inconsistent intermediate state.
- **Do / Detection:** Map all functions reading/writing shared state and check guards cover the whole set (one mutex, not per-function); trace cross-contract calls within a protocol cluster; test re-entering `deposit`/`transfer`/`claim` during a `withdraw` callback (checklist SOL-EC-1).
- Full entry: `Smart-contract-vulnerability-database_v1.md:37`

### V-003: Read-Only Reentrancy — High
- Aliases: Solodit "Read-Only Reentrancy", DVL-5
- **Don't:** A `view` function (e.g., `get_virtual_price`, share price, exchange rate) returns a stale/manipulated value when read by a dependent contract *during* a reentrant callback, since reentrancy guards don't block `view`s.
- **Do / Detection:** Map every protocol that consumes another protocol's view functions for pricing; check whether `view` pricing depends on balances updated after external calls; invariants: prices must not be computable while reserves/balances are mid-transition; missing locks around `remove_liquidity`-style flows.
- Full entry: `Smart-contract-vulnerability-database_v1.md:45`

### V-004: NFT Hook Reentrancy (`_safeMint` / `safeTransferFrom` Callbacks) — High
- Aliases: Solodit "NFT Hook Reentrancy", DVL-9
- **Don't:** ERC721/1155 safe-transfer and safe-mint invoke `onERC721Received`/`onERC1155Received` on the receiver, handing control to the attacker mid-function — e.g., re-entering `mint()` to exceed per-wallet limits because counters update after the mint.
- **Do / Detection:** State updates (supply caps, per-wallet limits) after `_safeMint`/safe transfers; missing `nonReentrant` on mint/claim/redeem flows; verify `require(totalSupply + n <= MAX)` can't be bypassed by re-entering mid-callback.
- Full entry: `Smart-contract-vulnerability-database_v1.md:53`

### V-005: ERC777 / Token-Hook Callback Reentrancy — Critical/High
- Aliases: Solodit "ERC777 Callback Reentrancy", DVL-6
- **Don't:** ERC777 `tokensReceived`/`tokensToSend` hooks (and ERC677 `onTokenTransfer`, ERC827/ERC1363 callbacks) execute recipient/sender code during transfers, enabling re-entry into protocols that assume ERC20-like transfer semantics (Cream, dForce, Hundred Finance).
- **Do / Detection:** Treat any `transfer`/`transferFrom` of tokens that may have hooks as an external call: state changes must precede the transfer or sit behind `nonReentrant`; check whether whitelisted token sets can include ERC777-compatible tokens (imBTC-style).
- Full entry: `Smart-contract-vulnerability-database_v1.md:61`

### V-006: Flash Loan / Flash Mint Callback Reentrancy — High
- Aliases: Solodit "Flash Loan / Flash Mint Callback Reentrancy" (checklist SOL-Defi-FlashLoan-1/2)
- **Don't:** During `flashLoan`/`flashMint` the borrower executes arbitrary code while pool state is mid-update, enabling re-entry, share-price manipulation, or double-use of the same liquidity.
- **Do / Detection:** Functions callable during flash-loan callbacks (deposit/withdraw/exchange-rate reads); missing same-block or reentrancy protection around flash-loanable pools.
- Full entry: `Smart-contract-vulnerability-database_v1.md:69`

### V-007: Pitfalls in Reentrancy Solutions (Indirect Calls & Broken Mutexes) — High
- Aliases: CP-3
- **Don't:** Two documented failure modes: (1) a function with no external call of its own is still reentrant because it calls a function that calls out — fixes must propagate up the call chain; (2) naive mutexes where the attacker claims the lock and never releases it, deadlocking the contract.
- **Do / Detection:** Trace full call graphs from any external call to all ancestors; check "claimed once" flags are set before any nested untrusted call; audit custom lock implementations for claim-without-release paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:77`

### V-008: External Calls in Modifiers (Modifier Reentrancy) — Medium
- Aliases: CR-5; modifier-reentrancy variant of SWC-107 (see V-001)
- **Don't:** Modifier code runs before the function body, so external calls or state changes inside a modifier violate Checks-Effects-Interactions and create hidden reentrancy paths easy to miss.
- **Do / Detection:** Audit modifiers for external calls, state writes, and ordering relative to the function body; prefer `require` inside the function for anything beyond pure checks.
- Full entry: `Smart-contract-vulnerability-database_v1.md:85`

### V-009: Gas-Stipend Reentrancy Fragility (Constantinople Lesson, historical) — Medium (historical)
- Aliases: CP-16
- **Don't:** Historical: EIP-1283's reduced SSTORE gas metering (withdrawn) would have made `send()`/`transfer()`'s 2300-gas stipend sufficient for reentrant state writes. Lesson: gas-stipend assumptions are fork-fragile.
- **Do / Detection:** Never rely on the 2300-gas stipend as a reentrancy defense; require CEI + reentrancy guards regardless of transfer method.
- Full entry: `Smart-contract-vulnerability-database_v1.md:93`


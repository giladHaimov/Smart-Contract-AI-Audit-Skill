# Coding Mode

Goal: while writing or editing Solidity, avoid reintroducing any of the 293 cataloged bugs — cheaply, at the keyboard, without re-reading the whole database every time. Consult this before considering a function/contract done, not after.

## How to use this

1. Look at what kind of code you're about to write or just wrote (a withdraw function, a new ERC20 integration, a proxy, a signature check, a loop over a dynamic array...).
2. Find the matching row(s) in the router table below.
3. Read that category file (`coding/<slug>.md`) — each entry there gives a **Don't** (the failure mode) and a **Do / Detection** (the concrete fix/check), reused verbatim from the full database, plus a line pointer if you need the full Aliases/Sources.
4. Apply the relevant Don'ts before moving on. Most functions only touch 1-3 categories — you don't need to read all 21 files for one function.

Skip straight to the always-on list below for the handful of mistakes severe and common enough to just default-avoid regardless of what you're writing.

## Always-on (Critical / Critical-High severity — 28 entries, know these by default)

- **V-001/V-002/E-02 Reentrancy** — never leave a state update after an external call/token transfer; Checks-Effects-Interactions or `nonReentrant`, and guard the whole set of functions sharing that state, not just one.
- **V-005 ERC777/hook-token reentrancy** — treat any token transfer as a potential external call (hooks can run arbitrary code).
- **V-010 Missing access control** — every state-changing function that should be privileged needs an explicit modifier/check; don't rely on default visibility.
- **V-011 Unprotected ether withdrawal** — withdrawal paths need an ownership/balance check, not just "anyone can call it."
- **V-013 Unprotected initializer** — `initialize()` on upgradeable contracts needs `initializer`/`onlyOwner` or it's front-runnable.
- **V-024 Unprotected selfdestruct** — never leave `selfdestruct` reachable without strict access control (and consider EIP-6780 semantics — see E-26).
- **V-025 Missing ownership check in custom NFT transfer** — don't hand-roll transfer logic without verifying `msg.sender` owns/is-approved-for the token.
- **V-030/V-036 Spot-price / LP-token oracle manipulation** — never price anything directly off AMM reserves or LP token supply in the same transaction; use TWAP or an external oracle.
- **V-049 First-depositor share inflation (ERC4626)** — seed an initial deposit or use virtual shares/offset; don't let the first depositor set an exploitable exchange rate.
- **V-055 `msg.value` in a loop / payable multicall** — `msg.value` doesn't decrease per iteration; never treat it as "per-item" inside a loop or multicall.
- **V-084 Uninitialized implementation contract** — call `_disableInitializers()` in the implementation's constructor for any UUPS/proxy pattern.
- **V-086 Unprotected `_authorizeUpgrade`** — UUPS upgrade authorization needs an explicit access check; it's not optional.
- **V-087 Storage collision across upgrades** — never reorder/retype/remove existing state variables in an upgradeable contract; only append, and use storage gaps.
- **V-090 `selfdestruct`/`delegatecall` in an implementation contract** — an implementation contract must never be `selfdestruct`-able or expose raw `delegatecall`; it can brick every proxy pointing at it.
- **V-119 Cross-chain signature/bridge replay** — always bind signatures to `block.chainid` and a domain separator (EIP-712); never accept a signature valid on another chain.
- **V-131/V-132 Arbitrary call / unsafe `delegatecall`** — never let user input choose the target or calldata of a `call`/`delegatecall` without an allowlist; `delegatecall` to an untrusted address gives it your storage.
- **V-134 Unauthenticated flash-loan/swap callback** — callback functions (`onFlashLoan`, `uniswapV3SwapCallback`, ...) must verify `msg.sender` is the expected pool/lender, not just trust the call.
- **V-147 Write to arbitrary storage location** — never let user input control a raw storage slot index in assembly.
- **V-172 Improper validation on aggregated/multicall routes** — validate every leg of a batched/aggregated call individually; don't assume validating the outer call is enough.
- **V-175 Flash-loan governance manipulation** — voting power must use a snapshot (checkpoint), never live balance, or a flash loan buys a proposal.
- **V-179 Timelock bypass/misconfiguration** — critical parameter changes need a timelock with no admin-only bypass path.
- **V-186 Cross-chain message validation failure** — verify source chain, source contract, and message authenticity explicitly for every bridge/CCIP/LayerZero receiver; never trust an unauthenticated `lzReceive`-style entrypoint.
- **V-190 Hidden backdoor via inline assembly** — treat all inline assembly as high-scrutiny; it can bypass every Solidity-level safety check silently.
- **E-22 Delegatecall context semantics** — `delegatecall` runs the callee's code in the caller's storage/`msg.sender`/`msg.value` context; a mismatched storage layout between caller and callee corrupts state, not just permissions.

## Router — category → file → when it applies

| Category | File | Applies when you're writing... | Entries |
|---|---|---|---|
| Reentrancy | `coding/reentrancy.md` | any function with an external call, token transfer, or NFT hook before/around a state update | 9 |
| Access Control | `coding/access-control.md` | admin functions, ownership, roles, initializers, `tx.origin`/`msg.sender` checks | 16 |
| Oracle | `coding/oracle.md` | any price/exchange-rate read (Chainlink, AMM, TWAP, LP pricing) | 11 |
| Math & Rounding | `coding/math-and-rounding.md` | arithmetic, `unchecked`, casts, division, share/vault math | 16 |
| Accounting & Fees | `coding/accounting-and-fees.md` | balances, rewards/staking accrual, fee logic, internal ledgers vs `balanceOf` | 14 |
| Token Standards | `coding/token-standards.md` | integrating/accepting arbitrary ERC20/721/1155 tokens, `permit`, approvals | 15 |
| DeFi Mechanics | `coding/defi-mechanics.md` | staking pools, MasterChef-style reward distributors, lending interest/liquidation | 6 |
| Proxy & Upgradeability | `coding/proxy-and-upgradeability.md` | any UUPS/transparent proxy, upgradeable base contract, storage layout | 10 |
| DoS | `coding/dos.md` | loops over dynamic arrays/mappings, push payments, queues | 12 |
| MEV & Front-running | `coding/mev-and-front-running.md` | swaps, liquidations, claims, mints, anything with a mempool-visible advantage | 13 |
| Signature & Replay | `coding/signature-and-replay.md` | `ecrecover`, EIP-712, meta-transactions, permits, nonces | 10 |
| External Calls | `coding/external-calls.md` | any low-level `.call`/`.delegatecall`/`.staticcall`, return-value handling | 21 |
| Storage | `coding/storage.md` | assembly touching storage slots, mappings, struct deletion, upgradeable layout | 30 |
| Logic Error | `coding/logic-error.md` | general business logic, conditionals, loops, invariants, `assert` | 24 |
| Governance | `coding/governance.md` | voting, proposals, quorum, timelocks, checkpoints | 8 |
| Cross-Chain & Multichain | `coding/cross-chain-and-multichain.md` | any contract deployed to/interacting across multiple chains | 4 |
| Other | `coding/other.md` | compiler/pragma choices, dependency versions, randomness, unused vars | 12 |
| Language Pitfall | `coding/language-pitfall.md` | general Solidity-language gotchas (visibility, `tx.origin`, gas stipends) | 10 |
| Gas | `coding/gas.md` | gas-sensitive code: loops, ether transfer, memory expansion | 8 |
| ABI & Encoding | `coding/abi-and-encoding.md` | `abi.encode`/`encodePacked`, calldata decoding, hashing over encoded data | 24 |
| EVM & Compiler | `coding/evm-and-compiler.md` | inline assembly, `viaIR`, storage slot packing, low-level EVM assumptions | 20 |

## Version-specific bugs (Part III, KB-01..KB-59)

Not a coding-time checklist item by pattern — these are compiler bugs tied to a specific solc version range. The one coding-time action: **pin a recent, non-floating solc version** (`pragma solidity 0.8.x;` exact, not `^0.8.x`) and check it against `../reference/INDEX.md` Part III rows before locking it in, especially if the project uses `viaIR`, inline assembly, or transient storage (EIP-1153) — several KB entries (KB-01..KB-11) are `viaIR`/assembly-specific and recent (fixed in 0.8.32-0.8.36).

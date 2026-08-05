# Vulnerability Database — Compact Index (all 293 entries)

Auto-derived index of every entry in `Smart-contract-vulnerability-database_v1.md`. Nothing is summarized here except the pointer — full Description/Detection/Aliases/Sources text lives at the line number given. This file exists so audit mode can scan IDs, titles, severities, and categories cheaply before opening the full entry.

Counts: Part I (application-level) V-001..V-197 = 197; Part II (EVM/compiler/language) E-01..E-37 = 37; Part III (known compiler bugs) KB-01..KB-59 = 59. Total = 293.

Full source of truth: `../Smart-contract-vulnerability-database_v1.md` (relative to this file's `reference/` directory).


## Part I — Application-Level Vulnerability Catalog

| ID | Title | Severity | Category | Line |
|---|---|---|---|---|
| V-001 | Classic Reentrancy (State Change After External Call) | Critical | Reentrancy | 29 |
| V-002 | Cross-Function & Cross-Contract Reentrancy | Critical/High | Reentrancy | 37 |
| V-003 | Read-Only Reentrancy | High | Reentrancy | 45 |
| V-004 | NFT Hook Reentrancy (`_safeMint` / `safeTransferFrom` Callbacks) | High | Reentrancy | 53 |
| V-005 | ERC777 / Token-Hook Callback Reentrancy | Critical/High | Reentrancy | 61 |
| V-006 | Flash Loan / Flash Mint Callback Reentrancy | High | Reentrancy | 69 |
| V-007 | Pitfalls in Reentrancy Solutions (Indirect Calls & Broken Mutexes) | High | Reentrancy | 77 |
| V-008 | External Calls in Modifiers (Modifier Reentrancy) | Medium | Reentrancy | 85 |
| V-009 | Gas-Stipend Reentrancy Fragility (Constantinople Lesson, historical) | Medium (historical) | Reentrancy | 93 |
| V-010 | Missing Access Control / Incorrect Visibility on Sensitive Function | Critical/High | Access Control | 103 |
| V-011 | Unprotected Ether Withdrawal / Constructor-Name Bug | Critical | Access Control | 111 |
| V-012 | `tx.origin` Used for Authentication | High | Access Control | 119 |
| V-013 | Unprotected / Front-Runnable Initializer | Critical/High | Access Control | 127 |
| V-014 | Single-Step Ownership Transfer & Overpowered Admin | Medium | Access Control | 135 |
| V-015 | Privilege Escalation via Flawed Role/Permission Transfer | High | Access Control | 143 |
| V-016 | Missing Ability to Revoke Access ("Can't Remove Access Control") | Medium | Access Control | 151 |
| V-017 | `msg.sender` Confusion (Meta-Tx / Multicall / ERC2771) | High | Access Control | 159 |
| V-018 | Hardcoded Address / Role | Medium | Access Control | 167 |
| V-019 | Admin Rug-Pull / Centralization Risk (incl. recoverERC20 Backdoor) | High | Access Control | 175 |
| V-020 | Instant Critical Parameter Changes (No Timelock) | Medium | Access Control | 183 |
| V-021 | Missing Validation in Privileged Setters | Medium | Access Control | 191 |
| V-022 | Authentication vs Authorization Mismatch | High | Access Control | 199 |
| V-023 | Bypass `isContract` / extcodesize EOA-Check | Medium/High | Access Control | 207 |
| V-024 | Unprotected Selfdestruct | Critical | Access Control | 215 |
| V-025 | Missing Ownership Check in Custom NFT Transfer | Critical | Access Control | 223 |
| V-026 | Stale Price Usage | High | Oracle | 233 |
| V-027 | Missing L2 Sequencer Uptime Check | High | Oracle | 241 |
| V-028 | Deprecated Chainlink API (`latestAnswer` / No Round Validation) | Medium | Oracle | 249 |
| V-029 | Zero / Negative Price Not Validated | Medium | Oracle | 257 |
| V-030 | Spot Price Manipulation via AMM Reserves (Flash-Loan Oracle Attack) | Critical | Oracle | 265 |
| V-031 | TWAP Window Too Short / Manipulable | High | Oracle | 273 |
| V-032 | Oracle Decimal / Feed-Pair Mismatch | Medium/High | Oracle | 281 |
| V-033 | Wrong Feed for the Asset (Depeg / Min-Max Bounds) | High | Oracle | 289 |
| V-034 | Oracle Update Front-Running | Medium/High | Oracle | 297 |
| V-035 | Single-Oracle Dependency / Unhandled Oracle Revert | Medium/High | Oracle | 305 |
| V-036 | LP Token Pricing Manipulation | Critical | Oracle | 313 |
| V-037 | Integer Overflow / Underflow (pre-0.8 and `unchecked`) | High | Math & Rounding | 323 |
| V-038 | Division Before Multiplication (Precision Loss) | Medium | Math & Rounding | 331 |
| V-039 | Precision Loss — Rounding Down to Zero | Medium | Math & Rounding | 339 |
| V-040 | Wrong Rounding Direction (Vaults/Shares, ERC4626) | High | Math & Rounding | 347 |
| V-041 | Division by Zero | Medium | Math & Rounding | 355 |
| V-042 | Unsafe Downcasting / Truncation | High | Math & Rounding | 363 |
| V-043 | Solidity Upcasting Trap (Small-Type Multiplication) | Medium | Math & Rounding | 371 |
| V-044 | Signed/Unsigned Conversion Errors (incl. MIN_INT Negation) | Medium | Math & Rounding | 379 |
| V-045 | Time-Unit Literal Overflow / Wrong Time Math | Medium | Math & Rounding | 387 |
| V-046 | Off-by-One / Wrong Comparison / Boundary Errors | Medium/High | Math & Rounding | 395 |
| V-047 | Extreme Input (0 / type.max) Edge Cases | Medium | Math & Rounding | 403 |
| V-048 | AMM Rounding / Invariant (k) Violations | High | Math & Rounding | 411 |
| V-049 | First Depositor / Share Inflation Attack (ERC4626 & Lending Markets) | Critical/High | Accounting & Fees | 421 |
| V-050 | Donation Attack / `balanceOf` vs Internal-Ledger Accounting | High | Accounting & Fees | 429 |
| V-051 | Stale Reward Accrual / Staking Reward Accounting Errors | Medium/High | Accounting & Fees | 437 |
| V-052 | Reward Rate Manipulation / Premature or Delayed Claims (JIT Sniping) | Medium | Accounting & Fees | 445 |
| V-053 | Fee Bypass via Rounding / Dust Amounts | Medium | Accounting & Fees | 453 |
| V-054 | Self-Transfer / `src == dst` Inflates Balances | High | Accounting & Fees | 461 |
| V-055 | `msg.value` Used Inside a Loop / Payable Multicall Double-Spend | Critical/High | Accounting & Fees | 469 |
| V-056 | Same-Block Deposit/Withdraw Arbitrage | Medium/High | Accounting & Fees | 477 |
| V-057 | Zero-Amount Withdraw/Claim Abuse ("Withdraw 0") | Medium | Accounting & Fees | 485 |
| V-058 | Received-Amount vs Requested-Amount Mismatch (No Pre/Post Balance Check) | High | Accounting & Fees | 493 |
| V-059 | Force-Feeding ETH Breaks Accounting (Unexpected Ether) | Medium | Accounting & Fees | 501 |
| V-060 | Emergency Withdraw Bypasses Fees/Accounting | Medium | Accounting & Fees | 509 |
| V-061 | Migration Loss | High | Accounting & Fees | 517 |
| V-062 | Incorrect Fee / Treasury Accounting (Double-Count, Rounding Drift) | Medium | Accounting & Fees | 525 |
| V-063 | Fee-on-Transfer Token Breaks Accounting | High | Token Standards | 535 |
| V-064 | Rebasing / Elastic-Supply Token Breaks Accounting | High | Token Standards | 543 |
| V-065 | Non-Standard ERC20 Return Values (USDT void / ZRX false) | High | Token Standards | 551 |
| V-066 | Tokens Reverting on Zero-Amount Transfers | Medium | Token Standards | 559 |
| V-067 | Blacklist / Pausable Token DoS | Medium | Token Standards | 567 |
| V-068 | Multiple-Address ("Two-Address") Tokens | High | Token Standards | 575 |
| V-069 | Non-18-Decimals Token Mishandling | High | Token Standards | 583 |
| V-070 | ERC20 Approval Race Condition | Medium | Token Standards | 591 |
| V-071 | Unlimited / Misused Approvals & Approval Scams | High | Token Standards | 599 |
| V-072 | Incorrect `supportsInterface` (ERC165) | Low/Medium | Token Standards | 607 |
| V-073 | NFT Approval Theft | High | Token Standards | 615 |
| V-074 | Flash Mint Inflates Balance Mid-Transaction | High | Token Standards | 623 |
| V-075 | CryptoPunks / Non-Standard NFT Handling | Medium | Token Standards | 631 |
| V-076 | Token Transfer to Zero Address / Contract's Own Address | Medium | Token Standards | 639 |
| V-077 | Phantom Function — Fake `permit` | High | Token Standards | 647 |
| V-078 | Rewards Locked / Lost Before First Staker | Medium | DeFi Mechanics | 657 |
| V-079 | Duplicate / Misconfigured Pool Entries (MasterChef) | High | DeFi Mechanics | 665 |
| V-080 | Lending Interest Accrual Order Bugs | High | DeFi Mechanics | 673 |
| V-081 | Liquidation Logic Flaws (Self-Liquidation, Close-Factor, Bad Debt) | High | DeFi Mechanics | 681 |
| V-082 | Withdrawal Queue / Share-Burn Ordering Bugs | High | DeFi Mechanics | 689 |
| V-083 | Reward Rate Manipulation via Donation (`notifyRewardAmount` Inflation) | High | DeFi Mechanics | 697 |
| V-084 | Uninitialized Implementation Contract Takeover | Critical | Proxy & Upgradeability | 707 |
| V-085 | Missing `initializer` / Re-initialization | High | Proxy & Upgradeability | 715 |
| V-086 | Unprotected UUPS `_authorizeUpgrade` | Critical | Proxy & Upgradeability | 723 |
| V-087 | Storage Collision / Layout Mismatch Between Versions | Critical | Proxy & Upgradeability | 731 |
| V-088 | Missing Storage Gaps in Upgradeable Base Contracts | Medium/High | Proxy & Upgradeability | 739 |
| V-089 | Constructor Used in Implementation / Immutable Variables Lost on Upgrade | Medium | Proxy & Upgradeability | 747 |
| V-090 | `selfdestruct` / `delegatecall` in Implementation Contract | Critical/High | Proxy & Upgradeability | 755 |
| V-091 | Wrong OpenZeppelin Branch for Proxies | Medium | Proxy & Upgradeability | 763 |
| V-092 | Proxy Selector Clash / Transparent Proxy Admin Confusion | High | Proxy & Upgradeability | 771 |
| V-093 | General Upgradeability Governance Risks | High | Proxy & Upgradeability | 779 |
| V-094 | Unbounded Loop / Array Growth Gas DoS | High | DoS | 789 |
| V-095 | Push Payments to Reverting Receiver | High | DoS | 797 |
| V-096 | Queue Clogging / Forced Queue Processing | Medium/High | DoS | 805 |
| V-097 | Griefing via Shared-State Manipulation (Delay Resetting) | Medium | DoS | 813 |
| V-098 | Insufficient-Gas Griefing (63/64 Rule / Relayer Censorship) | Medium | DoS | 821 |
| V-099 | External Call Consuming All Gas / Returnbomb | Medium | DoS | 829 |
| V-100 | Revert-Inside-Hook Griefing | Medium | DoS | 837 |
| V-101 | Permanently Locked Funds (No Withdrawal Path) | High | DoS | 845 |
| V-102 | Broken Loop (Missing Exit Condition) | Medium | DoS | 853 |
| V-103 | Pause-Induced DoS (Liquidations/Repayments Blocked) | Medium | DoS | 861 |
| V-104 | Dust-Driven Revert Griefing | Medium | DoS | 869 |
| V-105 | Call Depth Attack (deprecated) | Low (historical) | DoS | 877 |
| V-106 | Missing Slippage Protection (Sandwich Attack) | High | MEV & Front-running | 887 |
| V-107 | Missing Deadline Check | Medium | MEV & Front-running | 895 |
| V-108 | Get-or-Create / First-Come Front-Running (Displacement) | Medium/High | MEV & Front-running | 903 |
| V-109 | Two-Transaction Action Front-Running (No Commit-Reveal) | Medium | MEV & Front-running | 911 |
| V-110 | Merkle Claim Front-Running / Merkle Proof Misuse | Medium/High | MEV & Front-running | 919 |
| V-111 | Back-Running / Oracle Arbitrage | Medium | MEV & Front-running | 927 |
| V-112 | Liquidation Front-Running / Self-Liquidation Gaming | Medium | MEV & Front-running | 935 |
| V-113 | Liquidity Provider Racing / JIT Liquidity | Medium | MEV & Front-running | 943 |
| V-114 | Exchange-Rate Repricing Sandwich (LSD/Vaults) | High | MEV & Front-running | 951 |
| V-115 | Timestamp Dependence / Miner-Manipulable Time | Medium | MEV & Front-running | 959 |
| V-116 | NFT Mint via Exposed Metadata (Rare-Sniping, CVE-2022-38217) | Medium | MEV & Front-running | 967 |
| V-117 | VRF / Randomness-Oracle Front-Running & Reorg Rerolls | Medium | MEV & Front-running | 975 |
| V-118 | Frontrunning — Suppression Attack (Block Stuffing) | High | MEV & Front-running | 983 |
| V-119 | Cross-Chain Signature / Bridge Message Replay (Missing Chain ID / Domain) | Critical/High | Signature & Replay | 993 |
| V-120 | Missing Nonce / Signature Replay | High | Signature & Replay | 1001 |
| V-121 | Signature Malleability (ecrecover) | Medium/High | Signature & Replay | 1009 |
| V-122 | ecrecover Returns address(0) / Improper Signature Verification | High | Signature & Replay | 1017 |
| V-123 | Missing Signature Deadline / Expiry | Medium | Signature & Replay | 1025 |
| V-124 | Signature Not Bound to Intended Context | High | Signature & Replay | 1033 |
| V-125 | Failed-Transaction Replay | Medium | Signature & Replay | 1041 |
| V-126 | `abi.encodePacked` Hash Collision | High | Signature & Replay | 1049 |
| V-127 | EIP-1271 Contract Signatures Not Handled | Medium | Signature & Replay | 1057 |
| V-128 | Permit2 / Infinite-Allowance Drain via Signature | High | Signature & Replay | 1065 |
| V-129 | Unchecked Low-Level Call Return Value | High | External Calls | 1075 |
| V-130 | Call to Address Without Code | Medium/High | External Calls | 1083 |
| V-131 | Arbitrary Call From User Input (Call Injection) | Critical | External Calls | 1091 |
| V-132 | Unsafe `delegatecall` | Critical | External Calls | 1099 |
| V-133 | Unvalidated Return Data (Returndata Bomb / Truncated Data) | Medium | External Calls | 1107 |
| V-134 | Unauthenticated Callback / Flash-Loan & Swap Callback Spoofing | Critical | External Calls | 1115 |
| V-135 | Fixed Gas Stipends (`transfer`/`send`) Break on Gas Repricing | Medium | External Calls | 1123 |
| V-136 | try/catch Silent Failure & Gas Shortage | Medium | External Calls | 1131 |
| V-137 | Fallback Function Pitfalls | Low | External Calls | 1139 |
| V-138 | Interface Types Instead of Raw Addresses | Low | External Calls | 1147 |
| V-139 | Memory Pointer Aliasing | High | Storage | 1157 |
| V-140 | Sensitive Data Exposure On-Chain | Medium | Storage | 1165 |
| V-141 | Stale Mapping Entries / Struct Deletion Oversight | Medium | Storage | 1173 |
| V-142 | Stale Cached Storage/Memory Values (Data Location Confusion) | Medium/High | Storage | 1181 |
| V-143 | Inter-Related Storage Corruption | High | Storage | 1189 |
| V-144 | Uninitialized Storage Pointer / Uninitialized Proxy State | High | Storage | 1197 |
| V-145 | State Variable Default Visibility | Low | Storage | 1205 |
| V-146 | Shadowing State Variables | Medium | Storage | 1213 |
| V-147 | Write to Arbitrary Storage Location | Critical | Storage | 1221 |
| V-148 | Array Deletion Oversight (Ghost Entries) | Medium | Storage | 1229 |
| V-149 | DirtyBytes (Dirty High-Order Bits in Storage) | Medium | Storage | 1237 |
| V-150 | Transient Storage Misuse (EIP-1153) | High | Storage | 1245 |
| V-151 | Business Logic Flaw | High | Logic Error | 1255 |
| V-152 | Missing Logic / Unimplemented Path | High | Logic Error | 1263 |
| V-153 | Wrong Formula / Wrong Math | High | Logic Error | 1271 |
| V-154 | Typo / Copy-Paste / Parameter-Order Errors | Medium/High | Logic Error | 1279 |
| V-155 | Incorrect Conditional / Logical Operator | Medium | Logic Error | 1287 |
| V-156 | Uninitialized / Default State Variables | Medium | Logic Error | 1295 |
| V-157 | Non-Idempotent Functions (Repeat Invocation With Same Params) | Medium/High | Logic Error | 1303 |
| V-158 | Split vs Aggregate Operation Inequivalence | Medium | Logic Error | 1311 |
| V-159 | Array Removal Breaks Indexing (Swap-and-Pop) / Array Reorder | Medium | Logic Error | 1319 |
| V-160 | Duplicate Entries in Arrays | Medium | Logic Error | 1327 |
| V-161 | First/Last Iteration Edge Cases | Medium | Logic Error | 1335 |
| V-162 | Missing State Update After Admin Action | Medium | Logic Error | 1343 |
| V-163 | API / Semantic Inconsistency Between Functions | Medium | Logic Error | 1351 |
| V-164 | Global State Updated Incorrectly | High | Logic Error | 1359 |
| V-165 | Broken Invariants / Misused `assert` | Medium | Logic Error | 1367 |
| V-166 | Requirement Violation (Overly Strict / Mismatched Input Domains) | Low | Logic Error | 1375 |
| V-167 | Code With No Effects | Medium | Logic Error | 1383 |
| V-168 | Incorrect Inheritance Order (C3 Linearization) | Medium | Logic Error | 1391 |
| V-169 | Empty Loop / Empty Array Validation Bypass | High | Logic Error | 1399 |
| V-170 | `return` vs `break` in Loops | Medium | Logic Error | 1407 |
| V-171 | `tx.gasprice` Manipulation | Medium | Logic Error | 1415 |
| V-172 | Improper Input Validation on Aggregated Routes/Params | Critical | Logic Error | 1423 |
| V-173 | Ambiguous Evaluation Order | Low | Logic Error | 1431 |
| V-174 | Payability Quirk (Internal Calls) | Low | Logic Error | 1439 |
| V-175 | Flash-Loan Governance / Voting Power Manipulation | Critical | Governance | 1449 |
| V-176 | Double Voting via Token Transfer | High | Governance | 1457 |
| V-177 | Vote Checkpoint Errors (Double Writing / Wrong Snapshot) | High | Governance | 1465 |
| V-178 | Quorum / Threshold Miscalculation | Medium/High | Governance | 1473 |
| V-179 | Timelock Bypass / Misconfiguration & Execution Bypass | High/Critical | Governance | 1481 |
| V-180 | Veto / Guardian Abuse | Medium | Governance | 1489 |
| V-181 | Sybil Attack on User-Count Mechanisms | Medium | Governance | 1497 |
| V-182 | Governance Griefing / Proposal Spam | Medium | Governance | 1505 |
| V-183 | Cross-Chain Semantics Assumptions (block.number / timestamp / tx.origin) | Medium | Cross-Chain & Multichain | 1515 |
| V-184 | Opcode Incompatibility (PUSH0, EVM Diffs) | Medium | Cross-Chain & Multichain | 1523 |
| V-185 | Hardcoded Settings Not Portable Across Chains | Medium | Cross-Chain & Multichain | 1531 |
| V-186 | Cross-Chain Message Validation Failure (Bridge/CCIP/LayerZero) | Critical | Cross-Chain & Multichain | 1539 |
| V-187 | Outdated Compiler / Floating Pragma / Known Compiler Bugs in Pinned Versions | Medium/High | Other | 1549 |
| V-188 | Vulnerable Dependency Versions (OpenZeppelin CVEs) | High | Other | 1557 |
| V-189 | Weak Randomness From Block Properties | High | Other | 1565 |
| V-190 | Hidden Backdoor via Inline Assembly | Critical | Other | 1573 |
| V-191 | Right-To-Left-Override Control Character (U+202E) | Medium | Other | 1581 |
| V-192 | Presence of Unused Variables | Low | Other | 1589 |
| V-193 | Use of Deprecated Solidity Functions | Low | Other | 1597 |
| V-194 | Arbitrary Jump with Function Type Variable | High | Other | 1605 |
| V-195 | Missing Events / Audit Trail for Critical Changes | Low | Other | 1613 |
| V-196 | Gnosis Safe Module/Guard Nonce Issues | Medium | Other | 1621 |
| V-197 | Chain Reorganization Attack (CREATE / Same-Block Assumptions) | Medium | Other | 1629 |

## Part II — EVM, Compiler-Level & Language Pitfalls

| ID | Title | Severity | Category | Line |
|---|---|---|---|---|
| E-01 | Private Information and Randomness (On-Chain Visibility) | High | Language Pitfall | 1643 |
| E-02 | Reentrancy (Language-Level View) | Critical | External Calls | 1651 |
| E-03 | Gas Limit and Loops | High | Gas | 1659 |
| E-04 | Sending and Receiving Ether (Pitfall Cluster) | High | Gas | 1667 |
| E-05 | Call Stack Depth | Low | External Calls | 1675 |
| E-06 | Authorized Proxies (Arbitrary-Call Identity Assumption) | High | External Calls | 1683 |
| E-07 | tx.origin (Language Pitfall) | High | Language Pitfall | 1691 |
| E-08 | Two's Complement / Underflows / Overflows (Language Semantics) | High | Math & Rounding | 1699 |
| E-09 | Clearing Mappings | Medium | Storage | 1707 |
| E-10 | Internal Function Pointers in Upgradeable Contracts | Medium | Storage | 1715 |
| E-11 | Dirty Higher Order Bits / msg.data Malleability | Medium | ABI & Encoding | 1723 |
| E-12 | Official Recommendations (Warnings, Fail-Safe, Scope Limits) | Low | Other | 1731 |
| E-13 | Storage Slot Packing and Ordering | High | Storage | 1739 |
| E-14 | Mapping/Dynamic Array Slot Derivation (keccak256-based) | Medium | Storage | 1747 |
| E-15 | bytes/string Short-vs-Long Storage Encoding | Medium | Storage | 1755 |
| E-16 | Packing Can Increase Gas / Partial-Slot Writes | Low | Gas | 1763 |
| E-17 | Custom Storage Layout (`layout at N`) | High | Storage | 1771 |
| E-18 | Memory Layout Reserved Slots (0x00-0x7f) | Medium | ABI & Encoding | 1779 |
| E-19 | Free Memory Not Guaranteed Zeroed | Medium | ABI & Encoding | 1787 |
| E-20 | Memory vs Storage Layout Differences | Medium | ABI & Encoding | 1795 |
| E-21 | Memory Expansion Quadratic Gas Cost | Medium | Gas | 1803 |
| E-22 | Delegatecall Context Semantics | Critical | External Calls | 1811 |
| E-23 | Calls to Non-Existent Contracts Succeed | High | External Calls | 1819 |
| E-24 | 63/64 Gas Forwarding Rule and Practical Call Depth | Medium | Gas | 1827 |
| E-25 | 2300 Gas Stipend Not Guaranteed Forever | Medium | Gas | 1835 |
| E-26 | SELFDESTRUCT Semantics Post-Cancun (EIP-6780/6049) | High | EVM & Compiler | 1843 |
| E-27 | Force-Sent Ether Breaks Balance Invariants | High | EVM & Compiler | 1851 |
| E-28 | Transient Storage (EIP-1153) Semantics | High | Storage | 1859 |
| E-29 | Transient Storage via Yul Before/Outside Native Support | Medium | Storage | 1867 |
| E-30 | ecrecover Failure and Malleability (Precompile Behavior) | High | External Calls | 1875 |
| E-31 | Precompiled Contracts Range and Chain Differences | Medium | External Calls | 1883 |
| E-32 | CREATE / CREATE2 and Contract Under Construction | Medium | EVM & Compiler | 1891 |
| E-33 | Immutable/Constant Variables Live in Code, Not Storage | Low | Storage | 1899 |
| E-34 | Block Properties as Inputs (timestamp/prevrandao/blockhash) | Medium | EVM & Compiler | 1907 |
| E-35 | Checked vs Unchecked Arithmetic Modes and Panic Codes | Medium | Math & Rounding | 1915 |
| E-36 | Exception Bubbling and Low-Level Call Return Values | High | External Calls | 1923 |
| E-37 | Calldata Decoding: Eager vs Lazy | Low | ABI & Encoding | 1931 |

## Part III — Official Solidity Known Compiler Bugs

| ID | Title | Severity | Category | Line |
|---|---|---|---|---|
| KB-01 | InheritanceOrderReversalOnStorageEndWarning — introduced 0.8.29, fixed 0.8.36 | Medium | EVM & Compiler | 1945 |
| KB-02 | UnsoundSpillInMutualRecursion — introduced 0.7.2, fixed 0.8.36 | Medium | EVM & Compiler | 1950 |
| KB-03 | TransientStorageClearingHelperCollision — introduced 0.8.28, fixed 0.8.34 | High | EVM & Compiler | 1955 |
| KB-04 | LostStorageArrayWriteOnSlotOverflow — introduced 0.1.0, fixed 0.8.32 | Low | Storage | 1960 |
| KB-05 | VerbatimInvalidDeduplication — introduced 0.8.5, fixed 0.8.23 | Low | EVM & Compiler | 1965 |
| KB-06 | FullInlinerNonExpressionSplitArgumentEvaluationOrder — introduced 0.6.7, fixed 0.8.21 | Low | EVM & Compiler | 1970 |
| KB-07 | MissingSideEffectsOnSelectorAccess — introduced 0.6.2, fixed 0.8.21 | Low | Language Pitfall | 1975 |
| KB-08 | StorageWriteRemovalBeforeConditionalTermination — introduced 0.8.13, fixed 0.8.17 | High | Storage | 1980 |
| KB-09 | AbiReencodingHeadOverflowWithStaticArrayCleanup — introduced 0.5.8, fixed 0.8.16 | Medium | ABI & Encoding | 1985 |
| KB-10 | DirtyBytesArrayToStorage — introduced 0.0.1, fixed 0.8.15 | Low | Storage | 1990 |
| KB-11 | InlineAssemblyMemorySideEffects — introduced 0.8.13, fixed 0.8.15 | Medium | EVM & Compiler | 1995 |
| KB-12 | DataLocationChangeInInternalOverride — introduced 0.6.9, fixed 0.8.14 | Low | Language Pitfall | 2000 |
| KB-13 | NestedCalldataArrayAbiReencodingSizeValidation — introduced 0.5.8, fixed 0.8.14 | Low | ABI & Encoding | 2005 |
| KB-14 | AbiEncodeCallLiteralAsFixedBytesBug — introduced 0.8.11, fixed 0.8.13 | Low | ABI & Encoding | 2010 |
| KB-15 | UserDefinedValueTypesBug — introduced 0.8.8, fixed 0.8.9 | Low | Storage | 2015 |
| KB-16 | SignedImmutables — introduced 0.6.5, fixed 0.8.9 | Low | ABI & Encoding | 2020 |
| KB-17 | ABIDecodeTwoDimensionalArrayMemory — introduced 0.4.16, fixed 0.8.4 | Low | ABI & Encoding | 2025 |
| KB-18 | KeccakCaching — introduced before 0.4.x, fixed 0.8.3 | Medium | EVM & Compiler | 2030 |
| KB-19 | EmptyByteArrayCopy — introduced before 0.4.x, fixed 0.7.4 | Medium | Storage | 2035 |
| KB-20 | DynamicArrayCleanup — introduced before 0.4.x, fixed 0.7.3 | Medium | Storage | 2040 |
| KB-21 | FreeFunctionRedefinition — introduced 0.7.1, fixed 0.7.2 | Low | Language Pitfall | 2045 |
| KB-22 | UsingForCalldata — introduced 0.6.9, fixed 0.6.10 | Low | Language Pitfall | 2050 |
| KB-23 | MissingEscapingInFormatting — introduced 0.5.14, fixed 0.6.8 | Low | ABI & Encoding | 2055 |
| KB-24 | ArraySliceDynamicallyEncodedBaseType — introduced 0.6.0, fixed 0.6.8 | Low | ABI & Encoding | 2060 |
| KB-25 | ImplicitConstructorCallvalueCheck — introduced 0.4.5, fixed 0.6.8 | Low | Gas | 2065 |
| KB-26 | TupleAssignmentMultiStackSlotComponents — introduced 0.1.6, fixed 0.6.6 | Low | Language Pitfall | 2070 |
| KB-27 | MemoryArrayCreationOverflow — introduced 0.2.0, fixed 0.6.5 | Low | ABI & Encoding | 2075 |
| KB-28 | YulOptimizerRedundantAssignmentBreakContinue (+0.5 backport) — introduced 0.6.0, fixed 0.6.1; backport 0.5.8→0.5.16 | Medium | EVM & Compiler | 2080 |
| KB-29 | privateCanBeOverridden — introduced 0.3.0, fixed 0.5.17 | Low | Language Pitfall | 2085 |
| KB-30 | ABIEncoderV2LoopYulOptimizer — introduced 0.5.14, fixed 0.5.15 | Low | ABI & Encoding | 2090 |
| KB-31 | ABIEncoderV2CalldataStructsWithStaticallySizedAndDynamicallyEncodedMembers — introduced 0.5.6, fixed 0.5.11 | Low | ABI & Encoding | 2095 |
| KB-32 | SignedArrayStorageCopy — introduced 0.4.7, fixed 0.5.10 | Medium | Storage | 2100 |
| KB-33 | ABIEncoderV2StorageArrayWithMultiSlotElement — introduced 0.4.16, fixed 0.5.10 | Low | ABI & Encoding | 2105 |
| KB-34 | DynamicConstructorArgumentsClippedABIV2 — introduced 0.4.16, fixed 0.5.9 | Low | ABI & Encoding | 2110 |
| KB-35 | UninitializedFunctionPointerInConstructor (+0.4.x variant) — introduced 0.5.0, fixed 0.5.8; 0.4.x variant 0.4.5→0.4.26 | Low | Language Pitfall | 2115 |
| KB-36 | IncorrectEventSignatureInLibraries (+0.4.x variant) — introduced 0.5.0, fixed 0.5.8; 0.4.x variant 0.3.0→0.4.26 | Low | ABI & Encoding | 2120 |
| KB-37 | ABIEncoderV2PackedStorage (+0.4.x variant) — introduced 0.5.0, fixed 0.5.7; 0.4.x variant 0.4.19→0.4.26 | Low | ABI & Encoding | 2125 |
| KB-38 | IncorrectByteInstructionOptimization — introduced 0.5.5, fixed 0.5.7 | Low | EVM & Compiler | 2130 |
| KB-39 | DoubleShiftSizeOverflow — introduced 0.5.5, fixed 0.5.6 | Low | Math & Rounding | 2135 |
| KB-40 | ExpExponentCleanup — introduced before 0.4.x, fixed 0.4.25 | High | Math & Rounding | 2140 |
| KB-41 | EventStructWrongData — introduced 0.4.17, fixed 0.4.25 | Low | ABI & Encoding | 2145 |
| KB-42 | OneOfTwoConstructorsSkipped — introduced 0.4.22, fixed 0.4.23 | Low | Language Pitfall | 2150 |
| KB-43 | NestedArrayFunctionCallDecoder — introduced 0.1.4, fixed 0.4.22 | Medium | ABI & Encoding | 2155 |
| KB-44 | ZeroFunctionSelector — introduced before 0.4.x, fixed 0.4.18 | Low | ABI & Encoding | 2160 |
| KB-45 | DelegateCallReturnValue — introduced 0.3.0, fixed 0.4.15 | Low | External Calls | 2165 |
| KB-46 | ECRecoverMalformedInput — introduced before 0.4.x, fixed 0.4.14 | Medium | EVM & Compiler | 2170 |
| KB-47 | SkipEmptyStringLiteral — introduced before 0.4.x, fixed 0.4.12 | Low | ABI & Encoding | 2175 |
| KB-48 | ConstantOptimizerSubtraction — introduced before 0.4.x, fixed 0.4.11 | Low | EVM & Compiler | 2180 |
| KB-49 | IdentityPrecompileReturnIgnored — introduced before 0.4.x, fixed 0.4.7 | Low | External Calls | 2185 |
| KB-50 | OptimizerStateKnowledgeNotResetForJumpdest — introduced 0.4.5, fixed 0.4.6 | Medium | EVM & Compiler | 2190 |
| KB-51 | HighOrderByteCleanStorage — introduced 0.1.6, fixed 0.4.4 | High | Storage | 2195 |
| KB-52 | OptimizerStaleKnowledgeAboutSHA3 — introduced before 0.4.x, fixed 0.4.3 | Medium | EVM & Compiler | 2200 |
| KB-53 | LibrariesNotCallableFromPayableFunctions — introduced 0.4.0, fixed 0.4.2 | Low | External Calls | 2205 |
| KB-54 | SendFailsForZeroEther — introduced before 0.4.x, fixed 0.4.0 | Low | Gas | 2210 |
| KB-55 | DynamicAllocationInfiniteLoop — introduced before 0.3.x, fixed 0.3.6 | Low | EVM & Compiler | 2215 |
| KB-56 | OptimizerClearStateOnCodePathJoin — introduced before 0.3.x, fixed 0.3.6 | Low | EVM & Compiler | 2220 |
| KB-57 | CleanBytesHigherOrderBits — introduced before 0.3.x, fixed 0.3.3 | High | ABI & Encoding | 2225 |
| KB-58 | ArrayAccessCleanHigherOrderBits — introduced before 0.3.x, fixed 0.3.1 | High | Storage | 2230 |
| KB-59 | AncientCompiler — all versions before 0.3.0 | High | EVM & Compiler | 2235 |

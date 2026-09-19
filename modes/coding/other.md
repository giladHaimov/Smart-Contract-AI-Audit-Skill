# Coding-Mode Checklist — Other (12 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Other` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-187: Outdated Compiler / Floating Pragma / Known Compiler Bugs in Pinned Versions — Medium/High
- Aliases: Solodit "Known Compiler (SOLC) Bugs in Pinned Versions" (SOL-Basics-VI-SVI-1..48), SWC-102, SWC-103, DVL-94, CR-9
- **Don't:** Specific compiler versions have known codegen bugs (optimizer memory side effects 0.8.13/14, ABIEncoderV2 issues, Yul optimizer bugs); outdated or floating pragmas let contracts deploy under untested compilers inheriting those bugs. Floating pragmas are acceptable only for libraries.
- **Do / Detection:** Pin an exact pragma for deployable contracts; cross-reference the version against the official Solidity bug list — see Part III of this document for the complete known-bug list. Slither `solc-version`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1549`

### V-188: Vulnerable Dependency Versions (OpenZeppelin CVEs) — High
- Aliases: Solodit "Vulnerable Dependency Versions" (SOL-Basics-VI-OVI-1..14+)
- **Don't:** Using library versions with known vulnerabilities (ERC2771Context, GovernorCompatibilityBravo, ECDSA, MerkleProof multicall-proofs, TransparentUpgradeableProxy, etc.) imports audited-but-buggy code.
- **Do / Detection:** Diff `package.json`/imports against known OZ version-issue lists.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1557`

### V-189: Weak Randomness From Block Properties — High
- Aliases: Solodit "Weak Randomness From Block Properties" (SOL-AM-MA-2, SOL-Integrations-Chainlink-VRF-1/3/4), DVL-13, SWC-120 (see also E-01, E-34 in Part II)
- **Don't:** `block.timestamp`, `blockhash`, `block.number`, or `prevrandao` used as randomness is predictable/manipulable by validators and searchers, rigging lotteries/mints; `blockhash(n)` also returns 0 for n older than 256 blocks.
- **Do / Detection:** Grep randomness sources (keccak over block attributes, `gasleft`, caller-visible values) feeding lottery/NFT-trait/reward decisions; check outcomes can't be simulated off-chain before submitting; require VRF with adequate confirmations or commit-reveal. Slither `weak-prng`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1565`

### V-190: Hidden Backdoor via Inline Assembly — Critical
- Aliases: DVL-10
- **Don't:** Obfuscated inline assembly can overwrite arbitrary storage slots (owner, balances) behind innocuous-looking function names, giving deployers a covert backdoor invisible to casual review.
- **Do / Detection:** Audit every `assembly {}` block; decode hardcoded slot constants (`sstore(0x..)`), unexpected `mstore`/`sstore`, or `delegatecall` inside assembly; verify storage writes map to declared variables via the layout (`forge inspect ... storage-layout`).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1573`

### V-191: Right-To-Left-Override Control Character (U+202E) — Medium
- Aliases: SWC-130
- **Don't:** Invisible Unicode direction-control characters in source/comments make displayed argument order differ from actual execution order, hiding malicious logic from reviewers.
- **Do / Detection:** Scan source for U+202E/U+202D and other bidi control chars before manual review (grep/CI lint); be suspicious when rendered code and compiler behavior diverge.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1581`

### V-192: Presence of Unused Variables — Low
- Aliases: SWC-131
- **Don't:** Unused state/local variables waste gas, add code noise, and often signal a bug or incomplete logic.
- **Do / Detection:** Compiler warnings + Slither `unused-state`; treat each unused variable as a prompt to ask "was this supposed to do something?".
- Full entry: `Smart-contract-vulnerability-database_v1.md:1589`

### V-193: Use of Deprecated Solidity Functions — Low
- Aliases: SWC-111
- **Don't:** Deprecated constructs (`suicide`, `sha3`, `block.blockhash`, `callcode`, `throw`, `msg.gas`, `constant`, `var`) reduce code quality and may break or change semantics on newer compilers.
- **Do / Detection:** Grep for the deprecated aliases. Slither `deprecated-standards`, compiler warnings.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1597`

### V-194: Arbitrary Jump with Function Type Variable — High
- Aliases: SWC-127
- **Don't:** Function-type variables plus inline assembly (`mstore` on the variable) let an attacker redirect the function pointer to arbitrary code offsets, bypassing validation and executing privileged functions.
- **Do / Detection:** Flag any `assembly` block touching function-type variables or performing pointer arithmetic; heavy `mstore`-based code mixing with function pointers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1605`

### V-195: Missing Events / Audit Trail for Critical Changes — Low
- Aliases: Solodit "Missing Events / Audit Trail" (SOL-Basics-Event-1, SOL-CR-5)
- **Don't:** Critical state changes (admin setters, fee changes, withdrawals) emit no events, hiding attacks from monitoring and complicating incident response.
- **Do / Detection:** Diff state-changing functions against emitted events; admin setters especially.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1613`

### V-196: Gnosis Safe Module/Guard Nonce Issues — Medium
- Aliases: Solodit "Gnosis Safe Module/Guard Nonce Issues" (SOL-Integrations-GS-1/2)
- **Don't:** Safe modules bypassing Guard hooks or `execTransactionFromModule` paths that don't increment nonces allow replayed or unguarded transactions from Safe-based treasuries.
- **Do / Detection:** Module execution paths vs Guard hooks; nonce consumption on module txs.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1621`

### V-197: Chain Reorganization Attack (CREATE / Same-Block Assumptions) — Medium
- Aliases: Solodit "Chain Reorganization Attack" (SOL-Basics-BR-1)
- **Don't:** Factory contracts using CREATE (nonce-based addresses) can see a reorg replace a deployed child with a different contract at the same address; "finalized" assumptions break.
- **Do / Detection:** Reliance on deployment addresses/events without confirmations; prefer CREATE2 with salt binding.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1629`

### E-12: Official Recommendations (Warnings, Fail-Safe, Scope Limits) — Low
- Aliases: SC-12
- **Don't:** Take compiler warnings seriously; restrict the amount of Ether held; keep contracts small/modular; use Checks-Effects-Interactions; include a fail-safe mode; ask for peer review; use the latest compiler.
- **Do / Detection:** Check compilation has zero warnings, solc is current, an emergency pause/fail-safe exists, and value-at-risk is bounded.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1731`


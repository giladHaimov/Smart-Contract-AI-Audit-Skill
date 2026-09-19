# Coding-Mode Checklist — Cross-Chain & Multichain (4 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Cross-Chain & Multichain` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-183: Cross-Chain Semantics Assumptions (block.number / timestamp / tx.origin) — Medium
- Aliases: Solodit "Cross-Chain Semantics Assumptions" (SOL-McCc-1/4/11)
- **Don't:** `block.number` cadence, timestamp meaning, and `tx.origin`/`msg.sender` behavior differ across L2s; time-based logic calibrated for Ethereum misfires on fast/slow chains.
- **Do / Detection:** Block-based time math deployed multichain; per-chain constant review.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1515`

### V-184: Opcode Incompatibility (PUSH0, EVM Diffs) — Medium
- Aliases: Solodit "Opcode Incompatibility (PUSH0, EVM Diffs)" (SOL-McCc-3/10/12)
- **Don't:** Solidity >=0.8.20 emits `PUSH0`, reverting on chains without Shanghai; other opcode/behavior differences (e.g., zkSync Era) break deployed bytecode or assumptions.
- **Do / Detection:** Compiler version vs target chain EVM support matrix.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1523`

### V-185: Hardcoded Settings Not Portable Across Chains — Medium
- Aliases: Solodit "Hardcoded Settings Not Portable Across Chains" (SOL-Integrations-Uniswap-3/10, SOL-McCc-2)
- **Don't:** Fee tiers, pool addresses, token orders (token0/token1 differ per chain), or timing constants hardcoded for one chain are wrong on another deployment.
- **Do / Detection:** Constants audit for multichain targets; Uniswap token-order checks.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1531`

### V-186: Cross-Chain Message Validation Failure (Bridge/CCIP/LayerZero) — Critical
- Aliases: Solodit "Cross-Chain Message Validation Failure" (SOL-Integrations-Chainlink-CCIP-1..8, SOL-Integrations-LayerZero-1..7), DVL-77 (see also V-119)
- **Don't:** Bridge receivers that don't validate source chain selector, sender address, or router accept forged cross-chain messages; bridges trusting user-supplied token addresses let attackers register a worthless token mapping to a canonical wrapped asset and withdraw real reserves (Qubit). Wrong gas estimation or blocking-mode design also bricks delivery.
- **Do / Detection:** `_ccipReceive`/`_lzReceive` allowlist checks for sourceChain+sender+router; trace how `token` params map to canonical assets (whitelist vs user input); verify deposit event/payload authenticity; zero-amount/zero-address deposit edge cases; failure handling after execution windows.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1539`


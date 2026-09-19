# Coding-Mode Checklist — Signature & Replay (10 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Signature & Replay` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-119: Cross-Chain Signature / Bridge Message Replay (Missing Chain ID / Domain) — Critical/High
- Aliases: Solodit "Cross-Chain Signature Replay (Missing/Fixed Chain ID)" (SOL-AM-ReplayAttack-2, SOL-Signature-1), DVL-76
- **Don't:** Signatures/EIP-712 permits and bridge messages without (or with hardcoded) chainId in the domain separator — or omitting source/destination chain, nonce, or emitter — are replayable on forks and other deployed chains, minting or releasing assets twice.
- **Do / Detection:** Domain separators built once in constructor with hardcoded chain id (verify `block.chainid` recomputed); cross-chain verifiers must check domain separation (both chain IDs), message nonce/consumed-hash tracking, emitter whitelist per source chain; test replaying a mainnet proof on a fork/testnet.
- Full entry: `Smart-contract-vulnerability-database_v1.md:993`

### V-120: Missing Nonce / Signature Replay — High
- Aliases: Solodit "Missing Nonce / Signature Replay" (SOL-Signature-1), DVL-19, SWC-121
- **Don't:** Signed messages without per-signer nonces or used-hash tracking can be submitted repeatedly to repeat claims, withdrawals, or votes (NBA, Optimism airdrop cases).
- **Do / Detection:** For every `ecrecover`/EIP-712 verification check: nonce or `used[hash]` mapping, `block.chainid` and `address(this)` in the domain, deadline, and that the signer — not the caller — controls the action; test replaying the same signature twice.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1001`

### V-121: Signature Malleability (ecrecover) — Medium/High
- Aliases: Solodit "Signature Malleability" (SOL-Signature-2, SOL-Basics-VI-OVI-3), DVL-74, SWC-117 (see also E-30 in Part II)
- **Don't:** ECDSA signatures are malleable ((r, s) → (r, n−s) with flipped v); if the raw signature is used as a unique identifier or included in the replay-protection hash, the mirrored signature bypasses used-signature tracking. Letting users supply the *hash* to verify (instead of hashing on-chain) enables reusing any seen signature/hash pair. OZ ECDSA <4.7.3 had a related bug.
- **Do / Detection:** Enforce low-`s` (`s <= n/2`) and `v in {27,28}` via OZ `ECDSA`; never use raw signatures as unique identifiers; ensure message hashing happens in-contract; test replay with the mirrored (v,s).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1009`

### V-122: ecrecover Returns address(0) / Improper Signature Verification — High
- Aliases: DVL-33, SWC-122 (see also E-30 in Part II)
- **Don't:** `ecrecover` returns `address(0)` (not a revert) for invalid `v` or malformed signatures; comparing against an uninitialized signer (default address(0)) authenticates the attacker. Signature-validation shortcuts (`if (msg.sender == signer) return true` reachable by contracts, fake validator schemes) forge authorization via relayers/proxies.
- **Do / Detection:** Every raw `ecrecover` followed by `require(recovered != address(0))`; expected signer can never be zero; review every custom `isValidSignature`/validator branch.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1017`

### V-123: Missing Signature Deadline / Expiry — Medium
- Aliases: Solodit "Missing Signature Deadline / Expiry" (SOL-Signature-5)
- **Don't:** Permits and signed orders valid forever can be executed long after the signer changed their mind, at stale prices or after state changes.
- **Do / Detection:** Signed structs without `deadline`/`expiry` fields; deadline not checked against `block.timestamp`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1025`

### V-124: Signature Not Bound to Intended Context — High
- Aliases: Solodit "Signature Not Bound to Intended Context" (SOL-Signature-3/4)
- **Don't:** Signatures missing verifying contract, function, amount, or recipient binding are portable across contracts/functions within the same protocol (cross-function replay).
- **Do / Detection:** Hashed payloads omitting `address(this)`, function selector, or full parameter set; EIP-712 struct completeness.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1033`

### V-125: Failed-Transaction Replay — Medium
- Aliases: Solodit "Failed-Transaction Replay" (SOL-AM-ReplayAttack-1)
- **Don't:** Payloads/messages that failed once (bridge message, order) remain submittable; when conditions change the old payload executes unexpectedly.
- **Do / Detection:** Failure paths that don't mark message IDs consumed or expired.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1041`

### V-126: `abi.encodePacked` Hash Collision — High
- Aliases: Solodit "abi.encodePacked Hash Collision", DVL-37, SWC-133
- **Don't:** `keccak256(abi.encodePacked(a, b))` with multiple dynamic types collides across different inputs (`("a","bc")` vs `("ab","c")`), forging proofs/signatures/unique IDs.
- **Do / Detection:** `encodePacked` with >=2 dynamic arguments feeding signature/merkle verification or IDs; remediation: `abi.encode`, length-prefixing, or single dynamic argument; test boundary-shifting input pairs.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1049`

### V-127: EIP-1271 Contract Signatures Not Handled — Medium
- Aliases: Solodit "EIP-1271 Contract Signatures Not Handled" (SOL-Basics-VI-OVI-12)
- **Don't:** Contracts expecting only EOA signatures reject (or wrongly accept) smart-wallet signatures; conversely skipping EIP-1271 checks lets contracts "sign" unintentionally.
- **Do / Detection:** Signature flows without `SignatureChecker`/`isValidSignature` support; OZ SignatureChecker <4.7.1 issues.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1057`

### V-128: Permit2 / Infinite-Allowance Drain via Signature — High
- Aliases: DVL-81 (see also V-071)
- **Don't:** Permit2-style signature transfers or batched permits can be replayed/phished if nonce management, deadline, or spender binding is wrong — draining all tokens a user has permitted to Permit2.
- **Do / Detection:** Permit-consuming contracts: unordered nonce bitmap usage, `sigDeadline` enforcement, `transferFrom` spending exactly the permitted amount/owner; flag contracts requesting blanket Permit2 approvals.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1065`

# Coding-Mode Checklist — Gas (8 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Gas` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### E-03: Gas Limit and Loops — High
- Aliases: SC-3 (see also V-094 in Part I)
- **Don't:** Loops over storage-dependent (unbounded) iteration counts can exceed the block gas limit and permanently stall a contract; even `view` functions can stall on-chain callers.
- **Do / Detection:** Flag loops over growable arrays/mappings, unbounded batch operations; require pagination or pull-patterns; check `view` functions called on-chain.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1659`

### E-04: Sending and Receiving Ether (Pitfall Cluster) — High
- Aliases: SC-4 (see also V-059, V-095, V-135 in Part I)
- **Don't:** Ether can be force-sent via selfdestruct or block rewards, breaking balance invariants; receive/fallback get only the 2300-gas stipend (subject to change); `.call{value}` forwards all gas enabling reentrancy; `transfer`/`send` can fail and let recipients block the sender; wei rounding loses value.
- **Do / Detection:** Don't rely on `address(this).balance` equality checks; check receive/fallback gas usage; flag `transfer`/`send` without return checks; prefer withdrawal (pull) pattern.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1667`

### E-16: Packing Can Increase Gas / Partial-Slot Writes — Low
- Aliases: EV-4
- **Don't:** Writing one value in a multi-value slot requires read-modify-write of the whole slot; tight packing only helps when values are accessed together. Function args and memory values are never packed.
- **Do / Detection:** Check struct/variable ordering for actual access patterns; don't assume packing always saves gas.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1763`

### E-21: Memory Expansion Quadratic Gas Cost — Medium
- Aliases: EV-9 (see also V-099 in Part I)
- **Don't:** Memory expands per 256-bit word with gas cost growing quadratically; attacker-influenced memory sizes enable gas-griefing/DoS.
- **Do / Detection:** Check user-controlled memory allocations (dynamic arrays, return-data copies) for unbounded size.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1803`

### E-24: 63/64 Gas Forwarding Rule and Practical Call Depth — Medium
- Aliases: EV-12 (see also V-098 in Part I)
- **Don't:** Only 63/64 of remaining gas can be forwarded per message call, giving a practical call-depth limit below 1000; gas assumptions embedded in code break across hard-fork repricings.
- **Do / Detection:** Flag hard-coded gas values and gas-dependent branching; prefer pull patterns.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1827`

### E-25: 2300 Gas Stipend Not Guaranteed Forever — Medium
- Aliases: EV-13 (see also V-135 in Part I)
- **Don't:** `transfer`/`send` forward a 2300-gas stipend that might change with future hard forks; receive/fallback logic heavier than the stipend breaks payments.
- **Do / Detection:** Flag `transfer`/`send` usage and receive/fallback functions that write storage.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1835`

### KB-25: ImplicitConstructorCallvalueCheck — introduced 0.4.5, fixed 0.6.8 — Low
- **Don't:** A contract without its own constructor inheriting one did not revert when created with non-zero value.
- **Do / Detection:** Check solc >= 0.6.8; for new code, verify non-payable constructors reject value.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2065`

### KB-54: SendFailsForZeroEther — introduced before 0.4.x, fixed 0.4.0 — Low
- **Don't:** `send` didn't provide gas to the recipient when zero Ether was transferred.
- **Do / Detection:** Check solc >= 0.4.0; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2210`

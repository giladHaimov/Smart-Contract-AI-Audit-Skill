# Coding-Mode Checklist — Language Pitfall (10 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Language Pitfall` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### E-01: Private Information and Randomness (On-Chain Visibility) — High
- Aliases: SC-1 (see also V-140, V-189 in Part I)
- **Don't:** Everything on-chain is publicly visible, including `private` state variables; on-chain randomness can be manipulated by block builders.
- **Do / Detection:** Flag any secret stored in a contract (keys, seeds, unrevealed commitments) and randomness derived from `block.timestamp`/`blockhash`/`block.prevrandao` alone; require commit-reveal or VRF.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1643`

### E-07: tx.origin (Language Pitfall) — High
- Aliases: SC-7 (see also V-012 in Part I)
- **Don't:** Using `tx.origin` for authorization lets a malicious intermediary contract drain wallets, because `tx.origin` remains the original EOA through the whole call chain.
- **Do / Detection:** Flag every `tx.origin` usage in `require`/auth checks; require `msg.sender`-based authorization.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1691`

### KB-07: MissingSideEffectsOnSelectorAccess — introduced 0.6.2, fixed 0.8.21 — Low
- **Don't:** In legacy codegen, accessing `.selector` on a complex expression (e.g. `this.f().selector`) left the expression unevaluated — its side effects never happened.
- **Do / Detection:** Check solc >= 0.8.21; flag `.selector` accessed on non-trivial expressions rather than plain identifiers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1975`

### KB-12: DataLocationChangeInInternalOverride — introduced 0.6.9, fixed 0.8.14 — Low
- **Don't:** Overriding internal/public functions while changing parameter/return data location (calldata↔memory) was allowed and generated invalid code for virtual internal calls.
- **Do / Detection:** Check solc >= 0.8.14; flag overrides that change data location vs the base function signature.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2000`

### KB-21: FreeFunctionRedefinition — introduced 0.7.1, fixed 0.7.2 — Low
- **Don't:** Duplicate free functions with identical name+params in one source unit (or shadowing via import alias) were not flagged as errors.
- **Do / Detection:** Check solc >= 0.7.2; flag duplicated free-function names across imports.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2045`

### KB-22: UsingForCalldata — introduced 0.6.9, fixed 0.6.10 — Low
- **Don't:** Internal library functions with calldata parameters invoked via `using for` could read invalid data.
- **Do / Detection:** Check solc >= 0.6.10; flag `using L for T` where L functions take calldata args.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2050`

### KB-26: TupleAssignmentMultiStackSlotComponents — introduced 0.1.6, fixed 0.6.6 — Low
- **Don't:** Tuple assignments containing multi-stack-slot components (nested tuples, external function pointers, dynamic calldata array references) could yield invalid values.
- **Do / Detection:** Check solc >= 0.6.6; flag complex nested tuple unpacking.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2070`

### KB-29: privateCanBeOverridden — introduced 0.3.0, fixed 0.5.17 — Low
- **Don't:** `private` functions could silently be overridden by inheriting contracts.
- **Do / Detection:** Check solc >= 0.5.17; don't assume `private` prevents name collisions in derived contracts.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2085`

### KB-35: UninitializedFunctionPointerInConstructor (+0.4.x variant) — introduced 0.5.0, fixed 0.5.8; 0.4.x variant 0.4.5→0.4.26 — Low
- **Don't:** Calling uninitialized internal function pointers set in the constructor did not always revert.
- **Do / Detection:** Check solc >= 0.5.8; audit that function-pointer state variables are initialized before use (Panic revert can itself be a DoS).
- Full entry: `Smart-contract-vulnerability-database_v1.md:2115`

### KB-42: OneOfTwoConstructorsSkipped — introduced 0.4.22, fixed 0.4.23 — Low
- **Don't:** A contract defining both new-style `constructor()` and old-style same-named function had one silently ignored.
- **Do / Detection:** Check solc >= 0.4.23; flag any function sharing the contract's name.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2150`


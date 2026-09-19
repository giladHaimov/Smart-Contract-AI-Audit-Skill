# Coding-Mode Checklist — EVM & Compiler (20 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `EVM & Compiler` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### E-26: SELFDESTRUCT Semantics Post-Cancun (EIP-6780/6049) — High
- Aliases: EV-14 (see also V-024, V-090 in Part I)
- **Don't:** From EVM >= Cancun, `selfdestruct` only sweeps Ether; code/storage persist unless called in the same transaction as contract creation. Behavior depends on the chain's EVM version, not compile-time `--evm-version`.
- **Do / Detection:** Flag any `selfdestruct` use; flag contracts relying on code removal, metamorphic redeploys, or storage deletion; still treat selfdestruct as a force-Ether vector.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1843`

### E-27: Force-Sent Ether Breaks Balance Invariants — High
- Aliases: EV-15 (see also V-059 in Part I)
- **Don't:** Ether can arrive without any code execution via `selfdestruct(x)` beneficiary or block-builder payments; contracts cannot refuse it.
- **Do / Detection:** Flag `require(address(this).balance == x)`-style accounting; track deposits internally.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1851`

### E-32: CREATE / CREATE2 and Contract Under Construction — Medium
- Aliases: EV-20 (see also V-023 in Part I)
- **Don't:** During creation a contract's code is empty (`extcodesize == 0`), so callbacks into it run no code; addresses derive from creator+nonce (or salt+init-code for CREATE2), enabling address precomputation and metamorphic redeployment.
- **Do / Detection:** Flag `extcodesize > 0` used as a security check; check constructor-time external calls.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1891`

### E-34: Block Properties as Inputs (timestamp/prevrandao/blockhash) — Medium
- Aliases: EV-22 (see also V-115, V-189 in Part I)
- **Don't:** `block.timestamp`, `block.prevrandao`, `blockhash` (only last 256 blocks, zero beyond), and gas limits are influenced by block builders; using them for randomness or precise timing is unsafe.
- **Do / Detection:** Flag randomness or time-critical logic on these globals; flag `blockhash(block.number - n)` for n > 256.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1907`

### KB-01: InheritanceOrderReversalOnStorageEndWarning — introduced 0.8.29, fixed 0.8.36 — Medium
- **Don't:** Emitting the "storage base too close to storage end" warning reversed the `linearizedBaseContracts` annotation in place, reversing inheritance resolution order — can miscompile state-var init order, constructor invocation, virtual function/modifier resolution.
- **Do / Detection:** Require pragma/solc >= 0.8.36; flag contracts compiled with 0.8.29–0.8.35 using `layout at` custom storage layout near the storage end with inheritance.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1945`

### KB-02: UnsoundSpillInMutualRecursion — introduced 0.7.2, fixed 0.8.36 — Medium
- **Don't:** With `viaIR`, local variables of mutually recursive functions could be moved ("spilled") to fixed memory offsets and overwritten across recursive calls.
- **Do / Detection:** Check solc >= 0.8.36; flag mutual recursion (f calls g, g calls f) compiled via IR pipeline on earlier versions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1950`

### KB-03: TransientStorageClearingHelperCollision — introduced 0.8.28, fixed 0.8.34 — High
- **Don't:** With `viaIR` and EVM >= Cancun, clearing both storage and transient storage variables in the same contract could reuse the same clearing helper, so only one location was actually cleared.
- **Do / Detection:** Check solc >= 0.8.34; flag contracts using `transient` state variables plus `delete` on both storage and transient variables compiled 0.8.28–0.8.33.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1955`

### KB-05: VerbatimInvalidDeduplication — introduced 0.8.5, fixed 0.8.23 — Low
- **Don't:** The Yul deduplicator treated all `verbatim_*` blocks as identical and could unify different verbatim blocks surrounded by identical opcodes, producing wrong bytecode.
- **Do / Detection:** Check solc >= 0.8.23; flag Yul `verbatim` usage in inline assembly compiled 0.8.5–0.8.22.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1965`

### KB-06: FullInlinerNonExpressionSplitArgumentEvaluationOrder — introduced 0.6.7, fixed 0.8.21 — Low
- **Don't:** Yul optimizer FullInliner step did not preserve argument evaluation order for inlined calls; side-effectful argument expressions could run in the wrong order.
- **Do / Detection:** Check solc >= 0.8.21; flag Yul functions whose call arguments have side effects, compiled with optimizer on older versions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1970`

### KB-11: InlineAssemblyMemorySideEffects — introduced 0.8.13, fixed 0.8.15 — Medium
- **Don't:** The Yul optimizer could remove memory writes from inline assembly blocks that do not reference Solidity variables, treating them as side-effect free.
- **Do / Detection:** Check solc >= 0.8.15; flag assembly blocks doing raw `mstore` without touching Solidity variables.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1995`

### KB-18: KeccakCaching — introduced before 0.4.x, fixed 0.8.3 — Medium
- **Don't:** The bytecode optimizer incorrectly reused previously computed Keccak-256 hashes across changed memory.
- **Do / Detection:** Check solc >= 0.8.3; flag `keccak256` computed in inline assembly with optimizer enabled on older compilers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2030`

### KB-28: YulOptimizerRedundantAssignmentBreakContinue (+0.5 backport) — introduced 0.6.0, fixed 0.6.1; backport 0.5.8→0.5.16 — Medium
- **Don't:** The Yul optimizer removed essential assignments to variables declared inside for-loops when Yul `continue`/`break` was used.
- **Do / Detection:** Check solc >= 0.6.1; flag assembly loops using `break`/`continue` with optimizer on old compilers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2080`

### KB-38: IncorrectByteInstructionOptimization — introduced 0.5.5, fixed 0.5.7 — Low
- **Don't:** The optimizer mishandled the `byte` opcode whose second argument is (or evaluates to) 31.
- **Do / Detection:** Check solc >= 0.5.7; flag assembly byte extraction at index 31.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2130`

### KB-46: ECRecoverMalformedInput — introduced before 0.4.x, fixed 0.4.14 — Medium
- **Don't:** `ecrecover()` returned garbage instead of zero for malformed input.
- **Do / Detection:** Check solc >= 0.4.14; still verify `ecrecover` result != address(0) and check malleability.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2170`

### KB-48: ConstantOptimizerSubtraction — introduced before 0.4.x, fixed 0.4.11 — Low
- **Don't:** The constant optimizer replaced some numeric literals with routines computing different numbers.
- **Do / Detection:** Check solc >= 0.4.11; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2180`

### KB-50: OptimizerStateKnowledgeNotResetForJumpdest — introduced 0.4.5, fixed 0.4.6 — Medium
- **Don't:** Optimizer didn't reset internal state at jump destinations, potentially corrupting data.
- **Do / Detection:** Check solc >= 0.4.6; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2190`

### KB-52: OptimizerStaleKnowledgeAboutSHA3 — introduced before 0.4.x, fixed 0.4.3 — Medium
- **Don't:** Optimizer's stale SHA3 knowledge caused some hashes (incl. storage variable positions) to be computed incorrectly.
- **Do / Detection:** Check solc >= 0.4.3; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2200`

### KB-55: DynamicAllocationInfiniteLoop — introduced before 0.3.x, fixed 0.3.6 — Low
- **Don't:** Dynamic allocation of an empty memory array looped forever, burning all gas.
- **Do / Detection:** Check solc >= 0.3.6; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2215`

### KB-56: OptimizerClearStateOnCodePathJoin — introduced before 0.3.x, fixed 0.3.6 — Low
- **Don't:** Optimizer failed to reset state where code paths join, risking data corruption.
- **Do / Detection:** Check solc >= 0.3.6; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2220`

### KB-59: AncientCompiler — all versions before 0.3.0 — High
- **Don't:** Placeholder entry: compilers older than 0.3.0 may contain undocumented/undiscovered bugs.
- **Do / Detection:** Never audit/deploy anything compiled with solc < 0.3.0 (practically: require >= 0.8.x for new contracts).
- Full entry: `Smart-contract-vulnerability-database_v1.md:2235`


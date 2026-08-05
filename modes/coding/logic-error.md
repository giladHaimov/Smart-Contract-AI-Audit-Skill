# Coding-Mode Checklist — Logic Error (24 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Logic Error` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-151: Business Logic Flaw — High
- Aliases: Solodit "Business Logic Flaw"
- **Don't:** Implementation matches code intent but the underlying design is exploitable — wrong incentive math, gameable rules, unintended arbitrage paths.
- **Do / Detection:** Write protocol invariants first; model attacker profit across multi-step flows; compare against spec/economic assumptions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1255`

### V-152: Missing Logic / Unimplemented Path — High
- Aliases: Solodit "Missing Logic / Unimplemented Path"
- **Don't:** A required branch is absent — e.g., no handling when oracle fails, no path to withdraw a token type, unchecked case that silently proceeds.
- **Do / Detection:** Enumerate state machine transitions; look for empty `else`, TODOs, and unchecked enum values.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1263`

### V-153: Wrong Formula / Wrong Math — High
- Aliases: Solodit "Wrong Formula / Wrong Math"
- **Don't:** Correct-looking code computes the wrong quantity — swapped numerator/denominator, wrong interest exponent, inverted price ratio.
- **Do / Detection:** Dimensional analysis; unit-test against independent reference implementations and boundary cases.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1271`

### V-154: Typo / Copy-Paste / Parameter-Order Errors — Medium/High
- Aliases: Solodit "Typo / Copy-Paste Errors" (SOL-Heuristics-1/16), SWC-129, SF-4
- **Don't:** Duplicated code blocks where one identifier wasn't updated (wrong variable, token, or sign); typos that are still valid operators (`=+` instead of `+=`, `==` vs `=`); arguments passed in the wrong order to functions with same-typed consecutive parameters — sometimes deliberately hidden (see V-191).
- **Do / Detection:** Diff similar-looking adjacent blocks; asymmetric deposit/withdraw or token0/token1 paths; look for `=+`, `=-`, assignments inside conditions; cross-check argument order at call sites with adjacent same-type params.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1279`

### V-155: Incorrect Conditional / Logical Operator — Medium
- Aliases: Solodit "Incorrect Conditional / Logical Operator" (SOL-Heuristics-8)
- **Don't:** `&&` vs `||`, inverted booleans, or wrong comparison direction flips access or validation logic.
- **Do / Detection:** Truth-table test every guard; mutation testing.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1287`

### V-156: Uninitialized / Default State Variables — Medium
- Aliases: Solodit "Uninitialized / Default State Variables" (SOL-Basics-Initialization-1, SOL-Heuristics-11)
- **Don't:** State variables implicitly default to zero/false and are read before being set — first-user paths misbehave or checks pass vacuously.
- **Do / Detection:** Track "first use" of every state variable; constructor/initializer coverage.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1295`

### V-157: Non-Idempotent Functions (Repeat Invocation With Same Params) — Medium/High
- Aliases: Solodit "Non-Idempotent Functions" (SOL-Heuristics-12)
- **Don't:** Calling the same function twice with identical arguments double-counts (double claim, double registration) because completion isn't recorded.
- **Do / Detection:** Ask "what happens if called again with the same calldata?"; check executed/claimed flags.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1303`

### V-158: Split vs Aggregate Operation Inequivalence — Medium
- Aliases: Solodit "Split vs Aggregate Operation Inequivalence" (SOL-Heuristics-17)
- **Don't:** Calling a function N times with small amounts yields a different result than once with the total — exploited to bypass per-call limits, fees, or rounding.
- **Do / Detection:** Compare split vs single execution in tests; per-transaction caps without cumulative tracking.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1311`

### V-159: Array Removal Breaks Indexing (Swap-and-Pop) / Array Reorder — Medium
- Aliases: Solodit "Array Removal Breaks Indexing" (SOL-Basics-AL-4/5) (see also V-148)
- **Don't:** Removing array elements via swap-and-pop changes order and invalidates stored indexes; code assuming stable order/index misattributes entries.
- **Do / Detection:** Stored indices into mutable arrays; removal functions; order-dependent logic.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1319`

### V-160: Duplicate Entries in Arrays — Medium
- Aliases: Solodit "Duplicate Entries in Arrays" (SOL-Basics-AL-7)
- **Don't:** Arrays accepting duplicate entries (same token, same voter, same operator) double-count weights, rewards, or votes.
- **Do / Detection:** Push operations without existence checks.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1327`

### V-161: First/Last Iteration Edge Cases — Medium
- Aliases: Solodit "First/Last Iteration Edge Cases" (SOL-Basics-AL-1/8/13)
- **Don't:** Loop boundary handling (first and last cycle) miscomputes ranges, skips elements, or double-processes endpoints.
- **Do / Detection:** Test loops with 0, 1, 2 elements; check `break`/`continue` logic.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1335`

### V-162: Missing State Update After Admin Action — Medium
- Aliases: Solodit "Missing State Update After Admin Action"
- **Don't:** Admin changes a parameter but dependent derived state (cached indexes, totals, rates) isn't recomputed, leaving the protocol operating on stale values.
- **Do / Detection:** Setters that should trigger accrual/reindex but don't.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1343`

### V-163: API / Semantic Inconsistency Between Functions — Medium
- Aliases: Solodit "API / Semantic Inconsistency Between Functions"
- **Don't:** Similar functions behave differently (one takes shares, other assets; one validates, other doesn't), causing integrators to misuse the contract.
- **Do / Detection:** Side-by-side review of function families; consistent parameter semantics.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1351`

### V-164: Global State Updated Incorrectly — High
- Aliases: Solodit "Global State Updated Incorrectly" (SOL-Heuristics-13)
- **Don't:** Per-user updates occur but global aggregates (total deposits, total debt) drift, breaking solvency and rate computations.
- **Do / Detection:** Invariant: sum(per-user) == global total, tested with fuzzing/invariants.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1359`

### V-165: Broken Invariants / Misused `assert` — Medium
- Aliases: DVL-22, SWC-110
- **Don't:** `assert` is meant for invariants that can never fail; a reachable failing assert means either a real bug allows invalid state, or `assert` was misused for input validation (should be `require`). Pre-0.8.0, assert failures also consumed all gas.
- **Do / Detection:** Define protocol invariants (totalShares*pricePerShare ≈ totalAssets, sum(balances) == totalSupply, collateral >= debt) and test after every state transition; flag `assert` on user input. Mythril/MythX assertion analysis, Slither `assert-state-change`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1367`

### V-166: Requirement Violation (Overly Strict / Mismatched Input Domains) — Low
- Aliases: SWC-123
- **Don't:** `require()` validates external inputs; a violated requirement indicates either the caller contract supplies invalid inputs or the condition is too strong, rejecting valid inputs (liveness/integration bug rather than theft).
- **Do / Detection:** When integrating contracts, check parameter domains across the call boundary; flag overly strict requires that reject legitimate edge inputs.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1375`

### V-167: Code With No Effects — Medium
- Aliases: SWC-135
- **Don't:** Statements with no side effects compile silently — e.g., `balance[msg.sender] == amount;` (comparison instead of assignment) or `msg.sender.call.value(amount);` missing the trailing `("")` so no call executes.
- **Do / Detection:** Lint for expression statements that don't assign/call; compiler "statement has no effect" warnings; unit tests asserting state changes.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1383`

### V-168: Incorrect Inheritance Order (C3 Linearization) — Medium
- Aliases: SWC-125, CR-11
- **Don't:** With multiple inheritance, Solidity resolves conflicts by C3 linearization; declaring base contracts in the wrong order silently changes which overridden function runs.
- **Do / Detection:** Contracts with multiple bases sharing function names: compute the linearization and confirm the intended override wins; inherit most-general to most-specific; favor composition over deep hierarchies.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1391`

### V-169: Empty Loop / Empty Array Validation Bypass — High
- Aliases: DVL-30
- **Don't:** A function loops over an array validating each element (e.g., multisig signature checks); passing an empty array skips all validation and falls through to the privileged action.
- **Do / Detection:** For any `for` loop gating a privileged action, test `arr.length == 0` input; explicit `require(len > 0)`; length mismatches between parallel arrays (recipients vs amounts).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1399`

### V-170: `return` vs `break` in Loops — Medium
- Aliases: DVL-41
- **Don't:** Using `return` inside a loop where `break`/`continue` was intended terminates the whole function on the first matching element, skipping processing of remaining items (e.g., paying only the first payee).
- **Do / Detection:** Inspect every `return` inside a loop body; test multi-element inputs through batch functions and verify all elements processed.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1407`

### V-171: `tx.gasprice` Manipulation — Medium
- Aliases: DVL-40
- **Don't:** Logic using `tx.gasprice` (fee refunds, randomness inputs, payout computations) can be manipulated by the sender choosing their gas price, inflating refunds or skewing outcomes at protocol expense.
- **Do / Detection:** Grep `tx.gasprice` in refund/reward formulas; refunds can't exceed actual cost; flag it in any entropy mix; verify L2 semantics.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1415`

### V-172: Improper Input Validation on Aggregated Routes/Params — Critical
- Aliases: DVL-96
- **Don't:** Router/aggregator functions trusting user-constructed path/route payloads let attackers inject attacker-created pools or fake tokens into the swap path, making the protocol trade against a manipulated pool (SushiSwap RouteProcessor exploit).
- **Do / Detection:** Any `bytes route`/array param describing pools/tokens must be validated against a registry (factory-derived addresses, token whitelist); check first/last tokens match user intent; test injecting a freshly deployed malicious pool into the route.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1423`

### V-173: Ambiguous Evaluation Order — Low
- Aliases: SF-1
- **Don't:** Solidity does not guarantee operand evaluation order in some expressions; code depending on side-effect ordering within one expression behaves unexpectedly.
- **Do / Detection:** Expressions with multiple side-effecting sub-expressions (increments, external calls) in one statement; split into sequenced statements.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1431`

### V-174: Payability Quirk (Internal Calls) — Low
- Aliases: CR-7
- **Don't:** The `payable` modifier is only enforced on external entry points; an internal call from a payable function to a non-payable one succeeds while `msg.value` is still set, so value-accounting inside non-payable helpers can be silently wrong.
- **Do / Detection:** Non-payable internal functions that read `msg.value` or handle funds; verify value flows only through explicitly payable paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1439`


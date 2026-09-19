# Coding-Mode Checklist — Math & Rounding (16 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Math & Rounding` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-037: Integer Overflow / Underflow (pre-0.8 and `unchecked`) — High
- Aliases: Solodit "Over/Underflow in `unchecked` or Inline Assembly" (SOL-Basics-Math-9/11), DVL-1, SWC-101, CP-9 (see also E-08, E-35 in Part II)
- **Don't:** In Solidity <0.8 arithmetic silently wraps (BEC `batchTransfer`); in >=0.8, `unchecked{}` blocks and inline assembly bypass the overflow checks — bad assumptions wrap balances, allowances, counters, tick math.
- **Do / Detection:** Pragma <0.8 without SafeMath on user-influenced values; review every `unchecked` block and Yul arithmetic for provable no-wrap invariants; subtraction before an adequacy `require`; fuzz invariants like totalSupply == sum(balances).
- Full entry: `Smart-contract-vulnerability-database_v1.md:323`

### V-038: Division Before Multiplication (Precision Loss) — Medium
- Aliases: Solodit "Division Before Multiplication" (SOL-Basics-Math-4), DVL-24
- **Don't:** `a / b * c` truncates before scaling, losing precision; with small amounts the result can round to zero and break accounting or fees.
- **Do / Detection:** Any division preceding multiplication in the same expression chain; reorder to `(a * c) / b` and check overflow headroom; fuzz with small numerators < denominator.
- Full entry: `Smart-contract-vulnerability-database_v1.md:331`

### V-039: Precision Loss — Rounding Down to Zero — Medium
- Aliases: DVL-35, CR-3
- **Don't:** When numerator < denominator, integer division yields 0 — small deposits mint 0 shares, small swaps pay 0 fees, low-amount users get nothing while the contract keeps their input. All Solidity integer division rounds down; in fee/interest/share math this leaks value.
- **Do / Detection:** Every division where the numerator can plausibly be smaller than the denominator (fee on tiny amounts, per-second reward rates, decimal re-scaling); test minimum-amount flows; multiply-first ordering and scaling multipliers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:339`

### V-040: Wrong Rounding Direction (Vaults/Shares, ERC4626) — High
- Aliases: Solodit "Wrong Rounding Direction (Vaults/Shares)" (SOL-Basics-Math-5), DVL-57
- **Don't:** Share/asset conversions rounded in the user's favor (instead of the protocol's) let attackers repeatedly extract wei-level value; EIP-4626 mandates rounding against the user (shares round down on deposit, assets round up on withdrawal).
- **Do / Detection:** `mulDiv` rounding flags in `convertToShares/Assets`, mint/redeem; fuzz deposit/withdraw round-trips asserting the vault never loses value to rounding.
- Full entry: `Smart-contract-vulnerability-database_v1.md:347`

### V-041: Division by Zero — Medium
- Aliases: Solodit "Division by Zero" (SOL-Basics-Math-6, SOL-LL-5)
- **Don't:** Denominators derived from user-controlled state (totalSupply, totalAssets, time delta) can be zero, reverting core functions or enabling edge-case exploits.
- **Do / Detection:** Divisions without zero checks; empty-pool and first-operation paths (`totalSupply() == 0` scenarios).
- Full entry: `Smart-contract-vulnerability-database_v1.md:355`

### V-042: Unsafe Downcasting / Truncation — High
- Aliases: Solodit "Unsafe Downcasting / Truncation", DVL-31
- **Don't:** Casting `uint256`→`uint128/64/32` or int types silently truncates (pre-0.8, in assembly, or casting libs), corrupting packed structs, timestamps, amounts, prices (Union Finance case).
- **Do / Detection:** Grep `uint128(`, `uint64(`, `uint32(`, `intX(` casts on user-derived values; require SafeCast or explicit `require(x <= type(uintN).max)`; fuzz with values above the target-type max.
- Full entry: `Smart-contract-vulnerability-database_v1.md:363`

### V-043: Solidity Upcasting Trap (Small-Type Multiplication) — Medium
- Aliases: DVL-87
- **Don't:** `uint8 a * uint8 b` computes in uint8 and reverts/wraps even when assigned to uint256; packed-struct multiplication (uint64 x uint64 rates) silently overflows mid-calculation.
- **Do / Detection:** Arithmetic where all operands are sub-256-bit types (common with packed structs storing rates/timestamps); require explicit upcast before multiply; fuzz near-max small-type values.
- Full entry: `Smart-contract-vulnerability-database_v1.md:371`

### V-044: Signed/Unsigned Conversion Errors (incl. MIN_INT Negation) — Medium
- Aliases: Solodit "Signed/Unsigned Conversion Errors" (SOL-Basics-Math-8), CR-4
- **Don't:** Assigning negative `int256` results to `uint256` reverts (or wraps in assembly/older versions), DoS-ing PnL/rate flows. Two's-complement edge: `-type(intN).min` equals itself; MIN_INT * -1 or / -1 misbehaves.
- **Do / Detection:** Mixed int/uint arithmetic, especially `int` results cast to `uint` without sign checks; negation on signed integers without `x != type(intN).min` guard.
- Full entry: `Smart-contract-vulnerability-database_v1.md:379`

### V-045: Time-Unit Literal Overflow / Wrong Time Math — Medium
- Aliases: Solodit "Time-Unit Literal Overflow / Wrong Time Math" (SOL-Basics-Math-3, SOL-Basics-Type-2)
- **Don't:** Expressions like `uint24 x = 1 days` overflow the target type; time-based accrual math (per-second rates, rounding to day boundaries) miscalculates interest or vesting.
- **Do / Detection:** Time literals assigned to small integer types; rate x time computations with rounding to block/day granularity.
- Full entry: `Smart-contract-vulnerability-database_v1.md:387`

### V-046: Off-by-One / Wrong Comparison / Boundary Errors — Medium/High
- Aliases: Solodit "Off-by-One / Wrong Comparison Operator" (SOL-Basics-Math-10, SOL-Heuristics-7), DVL-95, DVL-47
- **Don't:** `<` vs `<=` errors let attackers bypass caps, deadlines, or thresholds by exactly one unit. A critical variant: flawed time comparisons (or missing claimed flags) letting users unlock/withdraw vesting tokens repeatedly before the lock elapses.
- **Do / Detection:** Boundary tests at limit, limit-1, limit+1 for caps, timestamps, quorum, array bounds; test time comparisons at exactly `lockEnd`; verify unlock functions can't be called twice for the same lock ID; trace epoch indexing (0- vs 1-based).
- Full entry: `Smart-contract-vulnerability-database_v1.md:395`

### V-047: Extreme Input (0 / type.max) Edge Cases — Medium
- Aliases: Solodit "Extreme Input Edge Cases" (SOL-Basics-Function-5, SOL-Basics-Math-12)
- **Don't:** Functions behave incorrectly for zero amounts or `type(uintN).max` inputs — zero-amount ops mint rewards or bypass checks; max inputs overflow or get interpreted as "unlimited".
- **Do / Detection:** Fuzz with 0, 1, max values; check special-casing of `type(uint256).max` as infinite approval/deadline.
- Full entry: `Smart-contract-vulnerability-database_v1.md:403`

### V-048: AMM Rounding / Invariant (k) Violations — High
- Aliases: Solodit "Constant-Product / AMM Rounding Exploit" (SOL-Defi-AS-5), DVL-78
- **Don't:** Rounding in `x*y=k` swap and liquidity math accumulates value leakage or lets small swaps drain the pool; custom AMM code that fails to enforce `reserve0 * reserve1 >= k` (after fees) or mis-handles fee growth leaks value to arbitrageurs.
- **Do / Detection:** Forked AMM code with modified fee/rounding logic; post-swap invariant with fee-adjusted balances; fuzz swap/add/remove sequences checking `k` never decreases excluding fees; verify `sync()`/`skim()` can't reset reserves mid-flow.
- Full entry: `Smart-contract-vulnerability-database_v1.md:411`

### E-08: Two's Complement / Underflows / Overflows (Language Semantics) — High
- Aliases: SC-8 (see also V-037 in Part I)
- **Don't:** Integer types wrap in `unchecked` blocks and revert in checked mode; even checked mode can leave a contract stuck if an unavoidable overflow always reverts. Signed two's-complement has extra edge cases.
- **Do / Detection:** Flag `unchecked` blocks without justification, `type(int).min` negation, narrowing casts; use the SMT checker for overflow paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1699`

### E-35: Checked vs Unchecked Arithmetic Modes and Panic Codes — Medium
- Aliases: EV-23 (see also V-037 in Part I)
- **Don't:** Since 0.8.0 arithmetic reverts (Panic 0x11) by default; `unchecked` re-enables wrapping. Panic codes also cover asserts (0x01), invalid storage encodings (0x22), OOB array access (0x32), bad enum conversions. An always-reverting overflow can permanently brick a function.
- **Do / Detection:** Audit every `unchecked` block for wrap safety; check arithmetic on external inputs can't permanently revert (DoS).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1915`

### KB-39: DoubleShiftSizeOverflow — introduced 0.5.5, fixed 0.5.6 — Low
- **Don't:** Double bitwise shifts by large constants whose shift amounts summed past 256 bits produced wrong values.
- **Do / Detection:** Check solc >= 0.5.6; flag chained constant shifts `x << a << b`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2135`

### KB-40: ExpExponentCleanup — introduced before 0.4.x, fixed 0.4.25 — High
- **Don't:** The `**` operator with an exponent type shorter than 256 bits used an uncleaned exponent, producing wrong results.
- **Do / Detection:** Check solc >= 0.4.25; in new code verify exponentiation uses wide types for base and exponent.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2140`


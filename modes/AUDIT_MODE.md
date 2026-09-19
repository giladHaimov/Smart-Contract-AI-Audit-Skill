# Audit Mode

Goal: walk a Solidity codebase (a repo, a directory, or a diff) against all 293 cataloged entries and produce a findings report. Depth over speed — this mode is for when correctness of the report matters more than turnaround time.

## Workflow

1. **Scope.** Enumerate `.sol` files in scope (`find . -name '*.sol'` or the diff's changed files). Note the compiler version(s) pinned in `pragma solidity` / `foundry.toml` / `hardhat.config` — you'll need it for Part III.

2. **Walk the database top-down, one category at a time**, using `../reference/INDEX.md` to see all IDs/titles/severities per category at a glance, then opening the full entry in `../Smart-contract-vulnerability-database_v1.md` at the given line for the exact Detection check, Aliases, and Sources before deciding a finding is real. Don't rely on the compressed `modes/coding/*.md` phrasing alone for a formal audit finding — always confirm against the full entry, since that's what has the complete Detection guidance and Slither detector names.

   Order (matches the database's own structure — front-load what actually gets exploited):
   - Part I categories in doc order: Reentrancy → Access Control → Oracle → Math & Rounding → Accounting & Fees → Token Standards → DeFi Mechanics → Proxy & Upgradeability → DoS → MEV & Front-running → Signature & Replay → External Calls → Storage → Logic Error → Governance → Cross-Chain & Multichain → Other.
   - Part II (E-01..E-37): EVM/language-level pitfalls — apply these to any code the Part I pass touched, they're the same bugs viewed at the language level and often catch what a category-scoped read misses (e.g. E-02 reentrancy is the general case of every V-00x reentrancy entry).
   - Part III (KB-01..KB-59): **version-gated, not code-pattern-gated.** For each KB entry, check whether the project's pinned solc version falls inside `introduced..fixed`. Only flag KB entries whose range covers the actual pinned version — don't flag a bug that was fixed in a version older than what's pinned. If the pragma floats across a range that straddles a vulnerable window, flag it as V-187-style floating-pragma risk in addition to the specific KB — unless the entry's Detection line explicitly carves out the case (e.g. V-187 carves out libraries). The entry's own Detection wins over the general rule.

3. **Apply each entry's Detection line as a concrete check** against the code: grep for the pattern it names, read the relevant function, decide if the precondition is real (external call before state update, missing modifier, unchecked return value, etc.) or a false positive given the surrounding code (e.g. a `nonReentrant`-guarded function calling an external contract is not V-001). Use Slither where a Detection line names a detector (`slither . --detect reentrancy-eth` etc.) to cross-check, but don't stop at tool output — several categories here (accounting, business logic, governance) are not mechanically detectable and need actual reading.

4. **Record every finding** with its entry ID as you go rather than batching at the end — it's easy to lose track across 293 checks.

## Finding format

One block per finding, most severe first:

```
### [<ID>] <Title> — <Severity>

**Location:** <file>:<line-range>, `<function name>`

**Issue:** <1-3 sentences specific to THIS code — what the code does, why it matches
the entry's failure mode. Not a copy of the database description; describe the
actual instance.

**Reference:** <ID> — <full entry line pointer, e.g. Smart-contract-vulnerability-database_v1.md:29>
**External refs:** <resolved source URL(s) from the Sources table in SKILL.md,
matched to the entry's Aliases/Sources line — e.g. "SWC-107" → https://swcregistry.io/docs/SWC-107,
"Solodit" → https://solodit.cyfrin.io">

**Suggested fix:** <concrete change — code-level, not just "add a check">
```

For KB (compiler-bug) findings, also state the pinned version and the entry's `introduced/fixed` range so the fix ("bump solc to ≥X") is unambiguous.

## Report structure

1. **Summary** — table of findings by severity (Critical/High/Medium/Low), total count, scope (files/commit reviewed).
2. **Findings** — one block per finding as above, grouped by severity.
3. **Coverage note** — state explicitly that all 293 entries were walked (or which categories were skipped and why, e.g. "no governance contracts in scope → Governance category not applicable").
4. **Non-findings worth noting** — patterns that looked suspicious but were ruled out, with the one-line reason (protects against re-litigating the same question later, and shows the check was actually done, not skipped).

## Notes

- Severity in the database is sometimes a range (`Critical/High`, `Medium/Low`) — pick the concrete severity for the instance found, don't just copy the range into the report.
- An entry with `(see also V-NNN in Part I)` in Part II, or an entry that's an alias-merge of multiple sources, is still ONE finding if it's the same root cause in the code — don't double-report E-02 and V-001 for the same unguarded external call.
- If the codebase is large, it's fine to checkpoint: report findings for the categories walked so far rather than holding everything until all 293 are done.

## Progress logging (required for multi-contract / corpus runs)

Emit short, structured progress lines so batch runs are debuggable and resumable. Prefer these exact prefixes:

```
[audit] scope: N .sol files | solc: <pragma or "mixed">
[audit] category start: <CategoryName> (Part I/II/III)
[audit] category done: <CategoryName> | findings_so_far: K
[audit] finding: <ID> <Severity> <function or file:line>
[audit] skip: <ID or Category> | reason: <one line>
[audit] file done: <path> | critical: C high: H medium: M low: L
[audit] complete: files=N findings=K duration_hint=<optional>
```

Rules:
- Log **category start/done** even when zero findings (proves the walk happened).
- Log **every finding** at decision time, not only in the final report.
- On false-positive candidates you considered and rejected, prefer a one-line `[audit] skip:` with reason over silence.
- For corpus jobs (many contracts), one `[audit] file done:` line per contract is the minimum operational signal.
- Do not spam token-level traces; keep logs one line each.

## Model-strength note (from 2026-08 evaluation)

When running this skill at scale:

- **Weaker / faster models** tend to over-report (high recall, high noise — e.g. systematic V-187 hits). Treat their Critical/High output as a **candidate set**.
- **Stronger models** on that filtered set drop most noise and retain a smaller, higher-precision residue.
- Always keep a **human pass** on surviving Critical/High items before treating them as confirmed.

See `../evaluation/` and the [Medium write-up](https://medium.com/@giladha/what-happened-when-we-stress-tested-an-ai-solidity-auditor-on-230-contracts-d45e0973e5fe).

# Independent re-run verification — 2026-09-19

On 2026-09-19 every test contract was re-audited from scratch (fresh runs,
`~/workspace/sc-audit-biz/rerun-compare/`, kept outside the repo) and each fresh
report was compared finding-by-finding against the committed report in
`test-contracts-audit/`. A coding-mode smoke test (CEI-violating withdrawal,
unguarded `_authorizeUpgrade`) was run in parallel.

## Result

- **33 of 35 committed findings reproduced with identical IDs and severities.**
- **Zero new Critical or High findings** in any fresh run.
- Every hand-edit from the 2026-09-18/19 fix pass was independently re-derived
  and confirmed correct: the MasterChef V-129 → V-065 re-ID (fixture contains
  zero low-level calls, so V-129 cannot apply), all count corrections
  (MasterChef 10 = 1C/3H/3M/3L; WETH9 7), and the absent-import disclosures.
- Coding-mode smoke test: **PASS** — both planted bugs caught with correct
  guidance, plus one genuine bonus catch (missing `_disableInitializers()`).

## Why the committed reports were kept

The fresh runs are raw tool output and skew noisier: six fresh-only Lows across
four contracts (most flagged cuttable by the workers themselves), one severity
up-rating the worker conceded was worse-calibrated than the committed call, and
one strictly weaker run (WETH9 fresh: 5 findings vs 7 committed — a genuine
recall miss, not a committed count error). The committed reports carry the
noise-reduction pass the fresh runs lack, so they were kept as the
better-calibrated artifacts. Two factual refinements were applied to report 02
only (import-disclosure re-framing; KB version-gating count 3 → 7).

## Caveats

- Slither was not installed; all runs note it as optional/absent per spec.
- The 03 (Uniswap V2 Pair) worker read the committed report before its walk —
  a reproduction check, not a blind audit.
- MasterChef V-001 (Critical) is scenario-conditional in both versions
  (requires an owner-added hook-capable LP token); the label alone does not
  convey that.

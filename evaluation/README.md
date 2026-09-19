# Evaluation results (2026-08)

Measured runs of this skill against a large production corpus and deliberately trapped contracts.

**Write-up:** [What Happened When We Stress-Tested an AI Solidity Auditor on ~220 Contracts](https://medium.com/@giladha/what-happened-when-we-stress-tested-an-ai-solidity-auditor-on-230-contracts-d45e0973e5fe)

## Pipeline

1. **Weak pass** (Composer 2.5) on ~219 production contracts → 327 findings
2. **Filter** — keep only contracts with Weak Critical or High → 36 contracts
3. **Strong pass** (Opus 5 High + Grok 4.6 High) on those 36 → 23 findings
4. **Human review** of 6 Critical+High survivors → 2 confirmed · 2 qualified · 2 rejected
5. **Trap set** (12 contracts, planted bugs, no filename hints) → 11/12 caught under Weak pass

## Headline numbers

| Stage | Result |
|-------|--------|
| Weak completed | 218 / 219 |
| Weak findings | 327 (4 Crit · 43 High · 147 Med · 119 Low · 14 Info) |
| Weak zero-finding contracts | 109 (50%) |
| Top weak FP pattern (V-187) | 66 contracts (30.3%) |
| Strong set size | 36 |
| Strong findings kept | 23 (1 Crit · 5 High · 10 Med · 7 Low) |
| Strong cleaned to zero | 21 / 36 (58%) |
| Weak Criticals still Critical under Strong | 1 / 4 |
| Human-confirmed from 6 survivors | 2 fully confirmed · 2 qualified · 2 rejected |
| Traps | 11 / 12 planted issue groups caught (Weak) |

## Practical takeaway

- **Weak models** are candidate generators — high recall, high noise. Do not treat as final audit.
- **Strong models** are the usable review layer for this skill — sharp drop in false positives.
- Keep a **human** on Critical/High residue.
- Trap detection shows solid recall on community-standard, DB-listed bug classes (access control, reentrancy, unchecked calls).

## Folders

- `weak-pass/` — summary of the ~219-contract run
- `strong-pass/` — the 36-contract filtered set and outcomes
- `traps/` — planted-bug methodology and results

## Limits

- Strong pass was run only on Weak Critical/High candidates, not the full corpus.
- Traps cover a small, known subset of the 293 DB classes (bugs the author understands).
- Code-level issues only — not economic design / incentive-layer risk.

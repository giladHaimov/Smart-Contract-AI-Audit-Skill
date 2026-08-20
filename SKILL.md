---
name: Smart-Contract-AI-Audit-Skill
description: Solidity smart-contract security database (293 entries — SWC, Solodit, DeFiVulnLabs, official solc known-bugs) with two modes — AUDIT (scan an existing Solidity codebase for known vulnerability classes and produce a findings report with description, external references, and suggested fix) and CODING (real-time guardrails consulted while writing or editing Solidity, so new code doesn't reintroduce a cataloged bug). Use whenever reviewing/auditing Solidity contracts for security issues, or whenever generating/modifying Solidity source (functions handling ETH/tokens, external calls, access control, oracles, proxies, signatures, loops, math) where known-vulnerability classes should be avoided by construction.
---

# Smart-Contract-AI-Audit-Skill

Two modes share one lossless source database. Pick the mode that matches what's happening right now; both can apply in the same session (e.g. write code in CODING mode, then AUDIT the finished diff).

| Signal | Mode | Load |
|---|---|---|
| "audit/review/scan this contract/repo for vulnerabilities", reviewing an existing diff or codebase for security bugs | **AUDIT** | `modes/AUDIT_MODE.md` |
| Writing, generating, or editing `.sol` code (new function, new contract, modifying an existing one) | **CODING** | `modes/CODING_MODE.md` |

Do not skip straight to the database file itself in either mode — `AUDIT_MODE.md` and `CODING_MODE.md` contain the workflow, report format, and routing that make the database usable; they in turn point into the database on demand.

## What's in this skill

```
SKILL.md                                    — this file, mode router
Smart-contract-vulnerability-database_v1.md — FULL source of truth, 293 entries, untouched (lossless)
reference/
  INDEX.md                                  — compact ID/Title/Severity/Category/Line table, all 293 rows
modes/
  AUDIT_MODE.md                             — audit workflow + findings-report template
  CODING_MODE.md                            — real-time router into 21 per-category checklists
  coding/<category>.md                      — 21 files, one per category, Don't/Do per entry (derived, verbatim text, nothing paraphrased)
```

## Coverage

293 unique entries, deduplicated from 411 raw items across four sources (see `Smart-contract-vulnerability-database_v1.md` → "Sources" and "Statistics & Methodology" for full provenance and the no-drop verification pass):

| Source | URL | Covers |
|---|---|---|
| Solodit (Cyfrin) | https://solodit.cyfrin.io | Largest curated real-audit-finding database + auditor checklist |
| SWC Registry + ConsenSys/SCSFG | https://swcregistry.io / https://consensys.github.io/smart-contract-best-practices / https://scsfg.io | SWC-100..136 canonical weakness classification + best practices |
| DeFiVulnLabs (SunWeb3Sec) | https://github.com/SunWeb3Sec/DeFiVulnLabs | PoC-backed DeFi vulnerability catalog, Foundry-reproducible |
| Official Solidity docs | https://docs.soliditylang.org | Known-compiler-bugs list (bugs.json) + official security considerations / EVM layout docs |

- **Part I — V-001..V-197** (197): application-level vulnerabilities, 21 categories (Reentrancy, Access Control, Oracle, Math & Rounding, Accounting & Fees, Token Standards, DeFi Mechanics, Proxy & Upgradeability, DoS, MEV & Front-running, Signature & Replay, External Calls, Storage, Logic Error, Governance, Cross-Chain & Multichain, Other, plus the Part II/III categories below).
- **Part II — E-01..E-37** (37): EVM/compiler/language-level pitfalls straight from the official docs.
- **Part III — KB-01..KB-59** (59): official Solidity known compiler bugs, each with an `introduced X, fixed Y` version range.

## Losslessness guarantee

`Smart-contract-vulnerability-database_v1.md` is the single source of truth and is never edited by either mode — only read. Everything else in this skill (`reference/INDEX.md`, `modes/coding/*.md`) is a **derived view**: INDEX.md is the same 293 rows with only the descriptive prose stripped out (title/severity/category/line survive); the per-category coding checklists reuse each entry's Description and Detection text verbatim, just relabeled Don't/Do, and link back to the exact line (`Smart-contract-vulnerability-database_v1.md:<line>`) for Aliases and Sources. If a derived file and the main database ever disagree, the main database wins — treat that as a bug in the derived file, not a reason to trust the summary.

Entry IDs (`V-`, `E-`, `KB-`) are stable identifiers — use them in audit findings and commit/PR references so they're traceable back to this database.

## Evaluation

Large-corpus results (weak vs strong models, trap contracts) live in `evaluation/`.
Measured write-up: https://medium.com/@giladha/what-happened-when-we-stress-tested-an-ai-solidity-auditor-on-230-contracts-d45e0973e5fe

# Weak pass — production corpus (~219 contracts)

**Model:** Composer 2.5  
**Skill / prompt:** standard AUDIT_MODE

## Aggregate

| Metric | Value |
|--------|-------|
| Completed | 218 / 219 |
| Failures | 1 (API resource exhaustion on one contract) |
| Total findings | 327 |
| Critical | 4 |
| High | 43 |
| Medium | 147 |
| Low | 119 |
| Info | 14 |
| Zero findings | 109 (50%) |
| Distinct patterns hit | 80 / 293 |
| Top FP pattern | V-187 (pinned compiler / known-bug window) on 66 contracts (30.3%) |

## Note on noise

Weak mode systematically over-triggers on design-level and pragma/compiler-window patterns. Use it to **generate candidates**, then run Strong mode on Critical/High contracts only.

# Trap contracts

Trap = production-style contract with deliberately planted bugs.  
Filenames left unchanged (no `_trap` suffix, no labels) so the agent gets no hint.

## Planted classes

- Missing access control (`onlyOwner` / controller / publisher / guardian-style checks removed)
- Removed reentrancy / mutex guards
- Unchecked external calls / return values
- Open execution paths (channel / caller allowlists removed)

## Results (Weak pass — Composer 2.5)

| Metric | Value |
|--------|-------|
| Trap contracts | 12 |
| Planted issue groups caught | **11 / 12** |
| Clear miss | 1 (GovernorAlpha `__acceptAdmin` access gap) |

## Example plant

```solidity
// Original (safe)
function setFeeTo(address _feeTo) external {
    require(msg.sender == feeToSetter, 'UniswapV2: FORBIDDEN');
    feeTo = _feeTo;
}

// Trap version used in the test
function setFeeTo(address _feeTo) external {
    // TRAP: access control removed
    feeTo = _feeTo;
}
```

## Limits

- Small sample.
- Only DB-listed classes the author knows; small portion of the 293 entries.
- Strong pass was not re-run exclusively on the trap set (Strong budget went to the 36-contract filtered corpus).

**Interpretation:** 11/12 is a strong but preliminary validity signal for community-standard, code-level bug classes encoded in this skill.

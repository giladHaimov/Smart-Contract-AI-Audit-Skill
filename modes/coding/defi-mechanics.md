# Coding-Mode Checklist — DeFi Mechanics (6 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `DeFi Mechanics` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-078: Rewards Locked / Lost Before First Staker — Medium
- Aliases: DVL-61
- **Don't:** Rewards streamed while `totalStaked == 0` (division skipped or rewards accrued to nobody) are permanently locked or later claimed by the first staker who didn't earn them.
- **Do / Detection:** Check `notifyRewardAmount`/emission behavior when no stakers exist; simulate emission window with zero stake then a late deposit; verify unclaimable rewards are rolled over/burned/extended — not gifted.
- Full entry: `Smart-contract-vulnerability-database_v1.md:657`

### V-079: Duplicate / Misconfigured Pool Entries (MasterChef) — High
- Aliases: DVL-62
- **Don't:** Adding the same LP token twice to a MasterChef-style farm (or wrong allocPoint) lets a user stake once and earn double rewards, diluting all other farmers.
- **Do / Detection:** Check `add()` for duplicate-token guards; verify `totalAllocPoint` consistency; test that sum(shares) matches staked balances per token across pools.
- Full entry: `Smart-contract-vulnerability-database_v1.md:665`

### V-080: Lending Interest Accrual Order Bugs — High
- Aliases: DVL-65
- **Don't:** Compound-style markets must call `accrueInterest()` before any borrow/redeem/liquidate changes balances; operating on stale `borrowIndex` lets users borrow or withdraw at outdated rates, stealing interest from suppliers.
- **Do / Detection:** Verify every state-changing market function accrues first; check `borrowBalanceStored` vs current index usage in liquidations; test two-block-separated borrow/repay sequences for skipped accrual.
- Full entry: `Smart-contract-vulnerability-database_v1.md:673`

### V-081: Liquidation Logic Flaws (Self-Liquidation, Close-Factor, Bad Debt) — High
- Aliases: DVL-66 (see also V-112 liquidation front-running)
- **Don't:** Liquidations that don't verify the account is actually underwater, allow self-liquidation for free collateral swaps, miscalculate close factor/incentive, or fail to socialize/write off bad debt leave the protocol insolvent or unfairly punish solvent users.
- **Do / Detection:** Invariants: `liquidate` must revert if healthFactor >= 1 (after oracle freshness); incentive math can't exceed seized collateral; trace the collateral < debt (bad-debt) path; test dust-position liquidations.
- Full entry: `Smart-contract-vulnerability-database_v1.md:681`

### V-082: Withdrawal Queue / Share-Burn Ordering Bugs — High
- Aliases: DVL-93
- **Don't:** Burning shares at request time vs claim time, or pricing claims at request-time vs fulfillment-time exchange rates, lets attackers join/exit around loss events (request withdrawal right before a strategy loss is socialized, or vice versa).
- **Do / Detection:** Trace share burn and rate snapshot points in async withdrawal flows; invariant: requester bears the PnL between request and fulfillment; test sandwiching a loss-report transaction.
- Full entry: `Smart-contract-vulnerability-database_v1.md:689`

### V-083: Reward Rate Manipulation via Donation (`notifyRewardAmount` Inflation) — High
- Aliases: DVL-98
- **Don't:** In StakingRewards-style contracts, if `rewardRate` is computed from `balanceOf(rewardToken)` including donations, or `notifyRewardAmount` can be frontrun/extended to dilute or spike rates, an attacker reshapes the emission schedule to their advantage.
- **Do / Detection:** rewardRate derived from tracked internal balances, not raw `balanceOf`; who can call notify functions; leftover-rollover math (`rewardRate = (amount + leftover) / duration`) gamed by topping up mid-period; test notify → donate → notify sequences.
- Full entry: `Smart-contract-vulnerability-database_v1.md:697`


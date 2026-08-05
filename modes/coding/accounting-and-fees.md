# Coding-Mode Checklist — Accounting & Fees (14 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Accounting & Fees` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-049: First Depositor / Share Inflation Attack (ERC4626 & Lending Markets) — Critical/High
- Aliases: Solodit "First Depositor / Share Inflation Attack (ERC4626)" (SOL-Defi-General-4), DVL-29, DVL-67
- **Don't:** First depositor mints 1 wei of shares then donates a large sum to inflate share price, so subsequent depositors' shares round to zero and the attacker redeems everything. Applies equally to lending markets using internal share tokens for collateral/borrows (multiple Compound-fork exploits).
- **Do / Detection:** Vault/share contracts without virtual shares/decimals offset (`_decimalsOffset`), minimum initial liquidity, or dead-share minting; check `totalSupply == 0` handling; simulate: deposit 1 wei, `transfer` donation, then victim deposit — verify victim receives nonzero shares; check empty-market handling and whether direct transfers move the exchange rate.
- Full entry: `Smart-contract-vulnerability-database_v1.md:421`

### V-050: Donation Attack / `balanceOf` vs Internal-Ledger Accounting — High
- Aliases: Solodit "Donation Attack via `balanceOf` Accounting" (SOL-AM-DA-1, SOL-Defi-General-3), DVL-58, DVL-59
- **Don't:** Protocol computes shares/rewards/rates from raw token `balanceOf(this)` instead of internal accounting, so anyone can "donate" tokens to skew rates, steal rewards, or force liquidations. Using both `balanceOf` and an internal `totalDeposits` interchangeably desyncs on donations, fee-on-transfer, rebases, or forced ETH.
- **Do / Detection:** `balanceOf(address(this))` in rate/share/reward math without internal tracking; strict equality checks (`require(balanceOf == total)`) outsiders can break; pick one source of truth and reconcile explicitly; simulate a donation between two operations and measure rate shift.
- Full entry: `Smart-contract-vulnerability-database_v1.md:429`

### V-051: Stale Reward Accrual / Staking Reward Accounting Errors — Medium/High
- Aliases: Solodit "Stale Reward Accrual (Rewards Not Updated Before Balance Change)" (SOL-Defi-Staking-2/3), DVL-60
- **Don't:** MasterChef/Synthetix-style accumulators break when `rewardPerTokenStored`/user indexes aren't settled before deposit/withdraw/transfer, when `rewardDebt` rounding favors early claimers, or when `allocPoint` math lets late stakers harvest past rewards.
- **Do / Detection:** `updateReward`/`accrue`/`updatePool` modifiers on every balance-changing path including share-token transfers; invariant: `pendingReward` only from rewards accrued after the user's last stake; test stake→claim→restake cycles and first-staker divide-by-zero.
- Full entry: `Smart-contract-vulnerability-database_v1.md:437`

### V-052: Reward Rate Manipulation / Premature or Delayed Claims (JIT Sniping) — Medium
- Aliases: Solodit "Reward Rate Manipulation / Premature or Delayed Claims" (SOL-Defi-Staking-2, SOL-Defi-General-8), DVL-63
- **Don't:** Attackers time deposits around reward distributions (deposit just before `notifyRewardAmount`/snapshot, withdraw after — often flash-loan funded), extracting disproportionate rewards for zero risk.
- **Do / Detection:** Reward distribution callable by anyone; deposit+claim in same block; rewards time-weighted per-second vs snapshot/instant; missing vesting/lock or cooldown; a large deposit immediately preceding `distribute()` should earn ~nothing.
- Full entry: `Smart-contract-vulnerability-database_v1.md:445`

### V-053: Fee Bypass via Rounding / Dust Amounts — Medium
- Aliases: Solodit "Fee Bypass via Rounding / Dust Amounts" (SOL-AM-DOSA-2)
- **Don't:** Fees computed with integer division round to zero on small amounts, so attackers split operations into dust transactions to avoid fees entirely.
- **Do / Detection:** Fee formulas without minimum fee or rounding-up; missing minimum transaction amount.
- Full entry: `Smart-contract-vulnerability-database_v1.md:453`

### V-054: Self-Transfer / `src == dst` Inflates Balances — High
- Aliases: Solodit "Self-Transfer / src == dst Inflates Balances" (SOL-Heuristics-3), DVL-44
- **Don't:** When sender equals receiver, code that debits then credits from a cached balance (or snapshots naively) double-counts, inflating user balance or voting power; burn/mint ordering can net to zero or double funds.
- **Do / Detection:** Test `transfer(self, amount)` on custom token/accounting logic; separate debit and credit reads of the same mapping entry; checkpoint logic applying both sides; vote-escrow and staking moves with `from == to`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:461`

### V-055: `msg.value` Used Inside a Loop / Payable Multicall Double-Spend — Critical/High
- Aliases: Solodit "msg.value Used Inside a Loop" (SOL-Basics-AL-11), Solodit "Multicall `msg.value` Reuse" (SOL-EC-2), DVL-50
- **Don't:** `msg.value` is constant per call frame, so crediting it per loop iteration (or per sub-call in a payable `multicall(bytes[])`) lets one ETH payment be counted N times — mint N NFTs while paying for 1 (Opyn hack).
- **Do / Detection:** `msg.value` referenced inside `for` loops in payable batch functions or in any function reachable via multicall; per-call value tracking (`valueSpent += msg.value; require(valueSpent <= msg.value)`); test batch-mint with a single payment.
- Full entry: `Smart-contract-vulnerability-database_v1.md:469`

### V-056: Same-Block Deposit/Withdraw Arbitrage — Medium/High
- Aliases: Solodit "Same-Block Deposit/Withdraw Arbitrage" (SOL-Defi-General-8, SOL-Defi-FlashLoan-1)
- **Don't:** Depositing and withdrawing in one transaction harvests yield or exploits exchange-rate updates with zero risk, often flash-loan amplified.
- **Do / Detection:** No same-block/cooldown restrictions; withdraw immediately after deposit profitable under rate updates.
- Full entry: `Smart-contract-vulnerability-database_v1.md:477`

### V-057: Zero-Amount Withdraw/Claim Abuse ("Withdraw 0") — Medium
- Aliases: Solodit "Zero-Amount Withdraw/Claim Abuse"
- **Don't:** Withdrawing/claiming 0 triggers side effects (reward distribution, index updates, cooldown resets) that can be spammed or used to farm rewards without stake.
- **Do / Detection:** Zero-amount paths in withdraw/claim functions and their state side effects.
- Full entry: `Smart-contract-vulnerability-database_v1.md:485`

### V-058: Received-Amount vs Requested-Amount Mismatch (No Pre/Post Balance Check) — High
- Aliases: Solodit "Received-Amount vs Requested-Amount Mismatch"
- **Don't:** Beyond fee-on-transfer: tokens with burns, hooks, or proxy weirdness deliver less/more than requested; contracts crediting the nominal amount become insolvent.
- **Do / Detection:** Absence of `balanceBefore/After` deltas on deposits of arbitrary ERC20s.
- Full entry: `Smart-contract-vulnerability-database_v1.md:493`

### V-059: Force-Feeding ETH Breaks Accounting (Unexpected Ether) — Medium
- Aliases: Solodit "Force-Feeding ETH Breaks Accounting (selfdestruct)" (SOL-Basics-Payment-3), SWC-132, CP-14 (see also E-27 in Part II)
- **Don't:** ETH forced via `selfdestruct`, pre-funded deterministic deployment addresses, or coinbase rewards inflates `address(this).balance`; logic assuming ETH only arrives via tracked deposits misprices shares or locks funds permanently.
- **Do / Detection:** `==`/`!=` strict comparisons against `address(this).balance`; ETH accounting via balance instead of internal counters; reconcile with `>=` comparisons or internal bookkeeping. Slither `incorrect-equality`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:501`

### V-060: Emergency Withdraw Bypasses Fees/Accounting — Medium
- Aliases: Solodit "Emergency Withdraw Bypasses Fees/Accounting"
- **Don't:** Emergency exit paths skip fees, penalties, reward settlement, or debt checks, becoming the cheapest exit or a way to dodge protocol invariants.
- **Do / Detection:** Compare emergency vs normal withdraw math; check penalties and reward updates still apply.
- Full entry: `Smart-contract-vulnerability-database_v1.md:509`

### V-061: Migration Loss — High
- Aliases: Solodit "Migration Loss"
- **Don't:** Migrating users/LP positions between contract versions loses rewards, mis-prices positions, or leaves funds stranded in the old contract.
- **Do / Detection:** Migration functions; snapshot of accrued rewards; total-value invariants before/after migration.
- Full entry: `Smart-contract-vulnerability-database_v1.md:517`

### V-062: Incorrect Fee / Treasury Accounting (Double-Count, Rounding Drift) — Medium
- Aliases: Solodit "Inconsistent Aggregate vs Individual Accounting" (SOL-Basics-AL-6, SOL-Defi-General-7), DVL-92
- **Don't:** Fees taken on both deposit and withdrawal from the same cached amount, fee-on-fee compounding errors, or per-operation rounding that drifts the sum of user balances away from total holdings, causing last-withdrawer shortfalls. Related: aggregate vs per-user computation paths diverging (dust bricks final withdrawals).
- **Do / Detection:** Invariant: sum(user claims) + fees <= contract balance at all times; test many small operations and measure drift; compare loop-accumulated vs directly-computed totals; fee computed once on a clearly defined base (gross vs net); treasury sweep can't touch user principal.
- Full entry: `Smart-contract-vulnerability-database_v1.md:525`


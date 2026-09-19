# Coding-Mode Checklist — Governance (8 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Governance` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-175: Flash-Loan Governance / Voting Power Manipulation — Critical
- Aliases: Solodit "Flash-Loan Governance / Voting Power Manipulation", DVL-69
- **Don't:** Voting power measured by current balance (no snapshot/checkpoint) lets attackers borrow tokens, vote, and repay within one block to pass malicious proposals (Beanstalk ~$181M).
- **Do / Detection:** `balanceOf`-based voting without `getPastVotes`/checkpoints; snapshots taken at a *past* block (`block.number - 1` or earlier) so same-tx acquired tokens don't count; timelock on execution.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1449`

### V-176: Double Voting via Token Transfer — High
- Aliases: DVL-68
- **Don't:** Voting weight read from current `balanceOf` lets an attacker vote, transfer tokens to a fresh address, and vote again — multiplying voting power arbitrarily.
- **Do / Detection:** Voting power source must be snapshot-based (`getPriorVotes`, ERC20Votes checkpoints, snapshot block), not live balance; test vote → transfer → vote from second address.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1457`

### V-177: Vote Checkpoint Errors (Double Writing / Wrong Snapshot) — High
- Aliases: Solodit "Vote Checkpoint Errors (Double Voting / Wrong Snapshot)" (SOL-Basics-VI-OVI-8)
- **Don't:** Checkpoint logic bugs (self-transfers double-writing, lookup off-by-one, ERC721Checkpointable OZ bug 4.8.0-4.8.2) let users vote twice or with wrong historical power.
- **Do / Detection:** Checkpoint write/lookup math; token transfers updating both sides; OZ version check.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1465`

### V-178: Quorum / Threshold Miscalculation — Medium/High
- Aliases: Solodit "Quorum / Threshold Miscalculation" (SOL-Basics-VI-OVI-11)
- **Don't:** Quorum computed against the wrong denominator (e.g., GovernorVotesQuorumFraction <4.7.2 bug, or non-circulating supply) makes proposals pass or fail incorrectly.
- **Do / Detection:** Quorum formula and its supply source; OZ Governor version audit.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1473`

### V-179: Timelock Bypass / Misconfiguration & Execution Bypass — High/Critical
- Aliases: Solodit "Timelock Bypass / Misconfiguration" (SOL-Timelock-1), DVL-70
- **Don't:** TimelockController misconfiguration (zero min delay, open executor role, proposer=executor) or executors accepting arbitrary `target`/`calldata` unchecked let critical actions execute immediately or by anyone — instant proxy upgrades, oracle swaps, fund seizure.
- **Do / Detection:** Timelock role assignments and delays; mandatory delay between vote success and `execute`; restricted callable targets/selectors; guardian/cancel powers; test executing with crafted calldata.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1481`

### V-180: Veto / Guardian Abuse — Medium
- Aliases: Solodit "Veto / Guardian Abuse"
- **Don't:** A veto or guardian role intended for emergencies can unilaterally override community decisions or freeze governance permanently.
- **Do / Detection:** Veto scope, expiry, and revocation paths; guardian powers vs proposal lifecycle.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1489`

### V-181: Sybil Attack on User-Count Mechanisms — Medium
- Aliases: Solodit "Sybil Attack on User-Count Mechanisms" (SOL-AM-SybilAttack-1)
- **Don't:** Mechanisms depending on number of participants (quadratic voting, per-address rewards, referenda) are gamed with many sockpuppet addresses.
- **Do / Detection:** Per-address incentives without identity/stake weighting.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1497`

### V-182: Governance Griefing / Proposal Spam — Medium
- Aliases: Solodit "Governance Griefing / Proposal Spam"
- **Don't:** Cheap proposal creation, cancellation, or voting delays let attackers block governance throughput or repeatedly cancel others' proposals.
- **Do / Detection:** Proposal thresholds, deposits, and cancellation rights.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1505`

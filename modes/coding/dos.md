# Coding-Mode Checklist — DoS (12 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `DoS` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-094: Unbounded Loop / Array Growth Gas DoS — High
- Aliases: Solodit "Unbounded Loop / Array Growth Gas DoS" (SOL-Basics-AL-9/10, SOL-Defi-LSD-7), DVL-90, SWC-128, CP-11 (see also E-03 in Part II)
- **Don't:** Iterating over user-growable arrays (stakers, validators, operators) eventually exceeds block gas, permanently freezing withdrawals, distributions, or liquidations (GovernMental).
- **Do / Detection:** Loops over unbounded dynamic arrays, especially with external calls or transfers inside; deletion of huge arrays; estimate gas at 10k/100k entries; require pagination (`nextIndex` resume pattern) or pull payments. Slither `calls-loop`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:789`

### V-095: Push Payments to Reverting Receiver — High
- Aliases: Solodit "Push Payments to Reverting Receiver" (SOL-AM-DOSA-1, SOL-Basics-Payment-1/5), DVL-12, SWC-113, CP-10
- **Don't:** Batch payouts pushing ETH/tokens revert if any recipient is a contract that reverts (or is blacklisted), blocking everyone's withdrawals (Akutars, Charged Particles); a malicious auction leader whose fallback reverts bricks refunds.
- **Do / Detection:** `require(x.send(...))` / `.call{value:}` inside loops to user-supplied address lists where one failure reverts the batch; prefer pull-over-push (`withdraw()`) patterns.
- Full entry: `Smart-contract-vulnerability-database_v1.md:797`

### V-096: Queue Clogging / Forced Queue Processing — Medium/High
- Aliases: Solodit "Queue Clogging / Forced Queue Processing" (SOL-AM-DOSA-4)
- **Don't:** Attacker floods a processing queue (withdrawals, messages, orders) with dust entries, forcing the protocol to burn gas processing junk or delaying legitimate users.
- **Do / Detection:** Public enqueue functions without minimum size/cost; FIFO processing without skip/cancel.
- Full entry: `Smart-contract-vulnerability-database_v1.md:805`

### V-097: Griefing via Shared-State Manipulation (Delay Resetting) — Medium
- Aliases: Solodit "Griefing via Shared-State Manipulation" (SOL-AM-GA-1), CP-13
- **Don't:** Attacker cheaply modifies state another user's transaction depends on — e.g., depositing 1 wei for the victim to reset their `lastDeposit` timelock — blocking the victim's action indefinitely.
- **Do / Detection:** Permissionless functions that write state keyed by an arbitrary `_for`/beneficiary address; timers, epochs, or shared counters resettable by third parties at minimal cost.
- Full entry: `Smart-contract-vulnerability-database_v1.md:813`

### V-098: Insufficient-Gas Griefing (63/64 Rule / Relayer Censorship) — Medium
- Aliases: Solodit "Insufficient-Gas Griefing (63/64 Rule / Gas Stipend Manipulation)" (SOL-AM-GA-2, SOL-EC-6/7), SWC-126, CP-12 (see also E-24 in Part II)
- **Don't:** A malicious relayer/forwarder supplies just enough gas for the outer tx to succeed (marking the message executed, burning the nonce) but not enough for the sub-call (63/64 rule), silently censoring the user's transaction.
- **Do / Detection:** Meta-tx/relayer patterns that mark `executed[data] = true` before an unchecked sub-call; require forwarders to pass a `gasLimit` verified inside the target (`require(gasleft() >= _gasLimit)`) or whitelist relayers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:821`

### V-099: External Call Consuming All Gas / Returnbomb — Medium
- Aliases: Solodit "External Call Consuming All Gas" (SOL-EC-7), DVL-89
- **Don't:** A malicious callee returns huge returndata (quadratic memory-copy cost) or burns all forwarded gas in its fallback, reverting the parent transaction — DoS-ing batch liquidations, distributions, or keeper flows.
- **Do / Detection:** Low-level calls copying unbounded returndata to memory (default behavior); loops calling untrusted contracts with shared gas; use assembly calls with bounded `returndatacopy` or gas-limited calls for untrusted targets.
- Full entry: `Smart-contract-vulnerability-database_v1.md:829`

### V-100: Revert-Inside-Hook Griefing — Medium
- Aliases: Solodit "Revert-Inside-Hook Griefing", DVL-97
- **Don't:** A receiver contract deliberately reverts inside a callback hook (`onERC721Received`, `tokensReceived`) or burns gas in its fallback, making itself un-transferable-to and blocking batch distributions, forced exits, or liquidation settlements that must push tokens to it.
- **Do / Detection:** Push-style token/NFT transfers to user-controlled addresses in mandatory flows; verify settlement can fall back to escrow/pull; test with a receiver that reverts or burns all gas.
- Full entry: `Smart-contract-vulnerability-database_v1.md:837`

### V-101: Permanently Locked Funds (No Withdrawal Path) — High
- Aliases: Solodit "Permanently Locked Funds" (SOL-Basics-Payment-7)
- **Don't:** ETH or tokens sent to a contract with no/insufficient withdrawal functions (or withdraw bricked by logic bugs) are locked forever.
- **Do / Detection:** `receive()`/payable functions and token inflows vs available sweep/withdraw functions; needless `receive()` on contracts with no ETH exit.
- Full entry: `Smart-contract-vulnerability-database_v1.md:845`

### V-102: Broken Loop (Missing Exit Condition) — Medium
- Aliases: Solodit "Broken Loop (Missing Exit Condition)"
- **Don't:** Loops without proper exit conditions (or with attacker-influenced bounds) run until gas exhaustion, bricking the function.
- **Do / Detection:** `while` loops with complex conditions; user-controlled loop bounds.
- Full entry: `Smart-contract-vulnerability-database_v1.md:853`

### V-103: Pause-Induced DoS (Liquidations/Repayments Blocked) — Medium
- Aliases: Solodit "Pause-Induced DoS" (SOL-Defi-Lending-4/5)
- **Don't:** Pausing transfers also blocks repayments/top-ups while liquidations continue (or vice versa), unfairly liquidating users who cannot act; resumed liquidations then fire in bursts.
- **Do / Detection:** Map which functions the pause gates; check liquidation, repayment, and collateral top-up pause consistently.
- Full entry: `Smart-contract-vulnerability-database_v1.md:861`

### V-104: Dust-Driven Revert Griefing — Medium
- Aliases: Solodit "Dust-Driven Revert Griefing" (SOL-AM-FrA-3)
- **Don't:** Attacker sends dust tokens/ETH into a victim's position or a shared pool so a subsequent operation (share mint, full withdrawal, migration) reverts on rounding or min-amount checks.
- **Do / Detection:** Operations sensitive to exact balances; minimum-balance assumptions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:869`

### V-105: Call Depth Attack (deprecated) — Low (historical)
- Aliases: CP-15 (see also E-05 in Part II)
- **Don't:** Deprecated/historical: attacker built a 1024-deep call stack so the victim's sub-call automatically failed. Neutralized by EIP-150.
- **Do / Detection:** Not exploitable post-EIP-150; relevant only insofar as unchecked call returns (V-129) would have masked such failures.
- Full entry: `Smart-contract-vulnerability-database_v1.md:877`


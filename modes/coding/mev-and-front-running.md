# Coding-Mode Checklist — MEV & Front-running (13 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `MEV & Front-running` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-106: Missing Slippage Protection (Sandwich Attack) — High
- Aliases: Solodit "Missing Slippage Protection" (SOL-AM-SandwichAttack-1, SOL-Defi-AS-1/7/11/13/14), DVL-36, DVL-71, CP-6, SF-3
- **Don't:** Swaps/liquidity ops without user-specified `minAmountOut`/price bounds get sandwiched: attacker pushes price before the victim tx and dumps after, extracting the slippage — potentially the entire trade value.
- **Do / Detection:** Swap calls with `amountOutMin = 0` or slippage computed on-chain from manipulable spot prices; hardcoded slippage; check protocol-level (not just router-level) minimum-received enforcement, batch auctions/private mempool options, and user-signed slippage bounds on keeper-executed orders.
- Full entry: `Smart-contract-vulnerability-database_v1.md:887`

### V-107: Missing Deadline Check — Medium
- Aliases: Solodit "Missing Deadline Check" (SOL-Defi-AS-2); deadline aspect of DVL-36 (see V-106)
- **Don't:** Without a deadline, a pending transaction can be held and executed much later at a worse price (stale-tx execution by block builders).
- **Do / Detection:** Swap/liquidity calls with `block.timestamp` or max-uint deadline instead of user-provided expiry.
- Full entry: `Smart-contract-vulnerability-database_v1.md:895`

### V-108: Get-or-Create / First-Come Front-Running (Displacement) — Medium/High
- Aliases: Solodit "Get-or-Create / First-Come Front-Running" (SOL-AM-FrA-1), CP-5
- **Don't:** First-come-first-served patterns (registering a name/pair/position/subaccount, claiming a bounty, copying a bid) can be preempted from the mempool, letting attackers claim the slot or parameterize it maliciously.
- **Do / Detection:** "If not exists, create" logic keyed on predictable identifiers; CREATE2 address derivation from public inputs; order-dependent state transitions with financial value. Mitigations: commit-reveal, batch auctions, submarine sends.
- Full entry: `Smart-contract-vulnerability-database_v1.md:903`

### V-109: Two-Transaction Action Front-Running (No Commit-Reveal) — Medium
- Aliases: Solodit "Two-Transaction Action Front-Running" (SOL-AM-FrA-2/4)
- **Don't:** Two-step flows (reveal bid, claim after registration, redeem after epoch) leak intent in step one; attackers insert themselves between the steps.
- **Do / Detection:** Multi-tx user flows without user-bound commit-reveal; predictable reveal data.
- Full entry: `Smart-contract-vulnerability-database_v1.md:911`

### V-110: Merkle Claim Front-Running / Merkle Proof Misuse — Medium/High
- Aliases: Solodit "Merkle Claim Front-Running" (SOL-HMT-1/2/5), DVL-72, DVL-75
- **Don't:** Airdrop/allowlist claims fail when (a) the proof isn't bound to `msg.sender` (anyone front-runs with the victim's proof), (b) leaves aren't hashed (attacker submits leaf == root), or (c) the claimed flag is never set (unlimited claims). Generally: any permissionless profitable call whose benefit isn't bound to the caller can be copied from the mempool.
- **Do / Detection:** Merkle leaves constructed as `keccak256(abi.encode(addr, amount))` with double-hashing; claim functions verifying `msg.sender` against the leaf; `claimed` bitmap actually written; multi-leaf claims can't double-count an index; ask "can anyone profit by copying this calldata verbatim?".
- Full entry: `Smart-contract-vulnerability-database_v1.md:919`

### V-111: Back-Running / Oracle Arbitrage — Medium
- Aliases: Solodit "Back-Running / Oracle Arbitrage"
- **Don't:** Bots immediately exploit state changes (oracle update, fee change, large trade) by executing right after the triggering transaction, draining predictable value.
- **Do / Detection:** Deterministic profit opportunities created by public state transitions; missing keeper incentives/commit delays.
- Full entry: `Smart-contract-vulnerability-database_v1.md:927`

### V-112: Liquidation Front-Running / Self-Liquidation Gaming — Medium
- Aliases: Solodit "Liquidation Front-Running / Self-Liquidation Gaming" (SOL-Defi-Lending-3/6/7) (see also V-081)
- **Don't:** Borrowers front-run liquidations with dust collateral top-ups to stay just above thresholds, or self-liquidate to capture bonuses; liquidators race and steal each other's work.
- **Do / Detection:** Liquidation bonus to borrower-reachable addresses; tiny repay/top-up allowed to dodge liquidation; no liquidator commitment.
- Full entry: `Smart-contract-vulnerability-database_v1.md:935`

### V-113: Liquidity Provider Racing / JIT Liquidity — Medium
- Aliases: Solodit "Liquidity Provider Racing / JIT Liquidity"
- **Don't:** LPs race to add/remove concentrated liquidity around large swaps (just-in-time liquidity), extracting fees while diluting passive LPs or distorting reward accounting.
- **Do / Detection:** Reward/fee accounting per-liquidity without time weighting; deposit-swap-withdraw atomic profitability.
- Full entry: `Smart-contract-vulnerability-database_v1.md:943`

### V-114: Exchange-Rate Repricing Sandwich (LSD/Vaults) — High
- Aliases: Solodit "Exchange-Rate Repricing Sandwich" (SOL-Defi-LSD-2)
- **Don't:** Attackers sandwich an oracle/keeper transaction that updates an LSD or vault exchange rate — minting before the update and redeeming after to drain ETH.
- **Do / Detection:** Rate updates executable as public keeper calls; mint/redeem priced off a rate that changes within a block.
- Full entry: `Smart-contract-vulnerability-database_v1.md:951`

### V-115: Timestamp Dependence / Miner-Manipulable Time — Medium
- Aliases: Solodit "Timestamp/Block-Property Manipulation (Miner Attack)" (SOL-AM-MA-1/2/3), SWC-116, CP-8 (see also E-34 in Part II)
- **Don't:** `block.timestamp` is manipulable by block producers within tolerance and `block.number` assumes constant block time that reorgs/forks invalidate; using them for precise triggers, tight windows, or randomness is unsafe. Post-merge validators can also withhold or reorder blocks.
- **Do / Detection:** `block.timestamp` in `require`s for exact deadlines, randomness seeds, or interest accrual with tight tolerances; `block.number`-based time math; assess whether a validator gains by nudging the timestamp across a boundary. Slither `timestamp`, `weak-prng`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:959`

### V-116: NFT Mint via Exposed Metadata (Rare-Sniping, CVE-2022-38217) — Medium
- Aliases: DVL-23
- **Don't:** If `tokenURI` metadata is readable before/while minting (via `ERC721Enumerable` + predictable tokenIds), attackers enumerate rare NFTs and snipe exactly those IDs via mempool monitoring, leaving commons for honest minters.
- **Do / Detection:** Check whether `tokenURI(tokenId)` works for unminted IDs, whether IDs are sequential, and whether reveal happens after mint completes; flag deterministic ID assignment without randomized commit.
- Full entry: `Smart-contract-vulnerability-database_v1.md:967`

### V-117: VRF / Randomness-Oracle Front-Running & Reorg Rerolls — Medium
- Aliases: DVL-91 (see also V-189 weak randomness, V-197 chain reorganization)
- **Don't:** Acting on a VRF/randomness result in the same flow that receives it — or accepting results too close to the chain tip — lets attackers front-run the fulfillment knowing the outcome, or exploit reorgs to reroll.
- **Do / Detection:** Request → wait → act design with the action gated on fulfillment in a prior block with sufficient confirmations; no game state committable between result visibility and resolution.
- Full entry: `Smart-contract-vulnerability-database_v1.md:975`

### V-118: Frontrunning — Suppression Attack (Block Stuffing) — High
- Aliases: CP-7
- **Don't:** Attacker fills consecutive blocks with high-gas-price transactions to prevent others' txs from being included before a deadline (the Fomo3D win).
- **Do / Detection:** Contracts rewarding "last actor before time T" or requiring an action within a window; assess whether stuffing the required window is profitable vs the prize.
- Full entry: `Smart-contract-vulnerability-database_v1.md:983`

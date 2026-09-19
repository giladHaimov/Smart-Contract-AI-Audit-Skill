# Coding-Mode Checklist — Oracle (11 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Oracle` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-026: Stale Price Usage — High
- Aliases: Solodit "Stale Price Usage" (SOL-Defi-Oracle-3/7), DVL-34
- **Don't:** Price feeds consumed without checking `updatedAt`/heartbeat/`answeredInRound`; a stale or frozen feed lets attackers trade/liquidate/borrow against outdated prices during congestion or oracle downtime.
- **Do / Detection:** Every `latestRoundData()` consumer must validate `updatedAt >= block.timestamp - threshold`, `answer > 0`, `answeredInRound >= roundId`; compare threshold against feed heartbeat per asset/chain.
- Full entry: `Smart-contract-vulnerability-database_v1.md:233`

### V-027: Missing L2 Sequencer Uptime Check — High
- Aliases: Solodit "Missing L2 Sequencer Uptime Check" (SOL-Defi-Oracle-4), DVL-55
- **Don't:** On Arbitrum/Optimism, if the sequencer is down, Chainlink prices go stale while the protocol keeps operating; borrowers get liquidated (or avoid liquidation) unfairly.
- **Do / Detection:** Chainlink consumers on L2s must read the `sequencerUptimeFeed` and enforce a grace period after `startedAt` before using `latestRoundData`; absence of the check on an L2 lending protocol is the red flag.
- Full entry: `Smart-contract-vulnerability-database_v1.md:241`

### V-028: Deprecated Chainlink API (`latestAnswer` / No Round Validation) — Medium
- Aliases: Solodit "Deprecated Chainlink API" (SOL-Defi-Oracle-1)
- **Don't:** `latestAnswer()` returns stale data with no timestamp; missing checks on `roundId`/`answeredInRound` admit incomplete-round data.
- **Do / Detection:** Grep `latestAnswer(`; absence of `roundId`/`answeredInRound` validation.
- Full entry: `Smart-contract-vulnerability-database_v1.md:249`

### V-029: Zero / Negative Price Not Validated — Medium
- Aliases: Solodit "Zero / Negative Price Not Validated" (SOL-Defi-Oracle-2)
- **Don't:** A feed returning 0 (or negative int256 for exotic feeds) is used directly, causing free mints, broken liquidations, or division by zero.
- **Do / Detection:** Oracle wrappers without `require(price > 0)` or sanity bounds.
- Full entry: `Smart-contract-vulnerability-database_v1.md:257`

### V-030: Spot Price Manipulation via AMM Reserves (Flash-Loan Oracle Attack) — Critical
- Aliases: Solodit "Spot Price Manipulation via AMM Reserves" (SOL-AM-PMA-1/2, SOL-Defi-Oracle-13, SOL-Integrations-Uniswap-5/9), DVL-32, DVL-52, CP-4
- **Don't:** Pricing from DEX pool reserves/`slot0`/balance ratios is manipulable within one transaction (flash-loan-swap → borrow/liquidate → revert). The single largest DeFi loss category.
- **Do / Detection:** For every price read ask "can this be moved within one tx?"; flag `getReserves`, `balanceOf(pool)` ratios, `slot0.sqrtPriceX96`, low-liquidity pairs pricing high-value decisions; require TWAP with sufficient window, Chainlink, or manipulation-resistant median; test with a flash-loan-sized swap before the price read.
- Full entry: `Smart-contract-vulnerability-database_v1.md:265`

### V-031: TWAP Window Too Short / Manipulable — High
- Aliases: Solodit "TWAP Window Too Short / Manipulable" (SOL-Defi-Oracle-5), DVL-53
- **Don't:** A short TWAP period (or Uniswap V3 pool with low observation cardinality) lets an attacker move the average price within a few blocks — economically feasible via multi-block MEV or low-liquidity pools.
- **Do / Detection:** TWAP/consult/`observe` calls with tiny `secondsAgo`; cardinality sufficient for the window; compare manipulation cost vs extractable value; pool liquidity depth relative to value secured.
- Full entry: `Smart-contract-vulnerability-database_v1.md:273`

### V-032: Oracle Decimal / Feed-Pair Mismatch — Medium/High
- Aliases: Solodit "Oracle Decimal / Feed-Pair Mismatch" (SOL-Defi-Oracle-6/8/9), DVL-56
- **Don't:** Combining feeds with different decimals (8 vs 18), or pricing token A in B's units without scaling, misprices by orders of magnitude — enabling under-collateralized borrows or free mints.
- **Do / Detection:** Trace every multiplication/division of oracle outputs; `decimals()` hardcoded or assumed; feed addresses hardcoded but protocol multichain; test value conservation across the full pipeline.
- Full entry: `Smart-contract-vulnerability-database_v1.md:281`

### V-033: Wrong Feed for the Asset (Depeg / Min-Max Bounds) — High
- Aliases: Solodit "Wrong Feed for the Asset" (SOL-Defi-Oracle-12/14)
- **Don't:** Using an ETH/USD feed for stETH-like assets, or assuming a stablecoin is always $1, breaks during depegs; Chainlink feeds also clamp at min/maxAnswer (LUNA incident), returning stale floor prices.
- **Do / Detection:** Hardcoded 1e18 for "stable" assets; no handling of feed min/max clamping or asset depeg.
- Full entry: `Smart-contract-vulnerability-database_v1.md:289`

### V-034: Oracle Update Front-Running — Medium/High
- Aliases: Solodit "Oracle Update Front-Running" (SOL-Defi-Oracle-10)
- **Don't:** Attacker sees a pending price update in the mempool and trades/liquidates before or after it lands to capture the delta.
- **Do / Detection:** Price-sensitive actions (liquidations, redemptions) executable in the same block as a fresh oracle update; check for commit/timing protections.
- Full entry: `Smart-contract-vulnerability-database_v1.md:297`

### V-035: Single-Oracle Dependency / Unhandled Oracle Revert — Medium/High
- Aliases: Solodit "Unhandled Oracle Revert / DoS" (SOL-Defi-Oracle-11), DVL-54
- **Don't:** Relying on one oracle (or one DEX) for critical pricing means a single feed failure/manipulation is fatal; if the oracle call reverts (paused feed, circuit breaker, access-controlled feed), core protocol functions DoS entirely.
- **Do / Detection:** Count independent price sources per asset; raw external oracle calls without try/catch or fallback; missing deviation bounds, circuit breakers on price jumps, and graceful degradation.
- Full entry: `Smart-contract-vulnerability-database_v1.md:305`

### V-036: LP Token Pricing Manipulation — Critical
- Aliases: DVL-79
- **Don't:** Protocols that accept LP tokens as collateral and price them from the pool's own spot reserves are circularly manipulable: inflate reserves → LP "worth" more → borrow against fake value.
- **Do / Detection:** LP valuation must use fair-pricing formulas (e.g., Alpha Homora fair LP price = 2*sqrt(p0*p1), supply-adjusted) or independent oracles for both underlyings; red flag: `lpPrice = reserveIn * price / lpSupply` with spot reserves.
- Full entry: `Smart-contract-vulnerability-database_v1.md:313`


# Strong pass — filtered set (36 contracts)

**Models:** Claude Opus 5 High (20 contracts), Grok 4.6 High (16 contracts)  
**Filter rule:** contract had ≥1 Critical or High finding under Weak pass (Composer 2.5)  
**Prompt / skill:** identical to Weak pass

## Aggregate

| Metric | Value |
|--------|-------|
| Completed | 36 / 36 |
| Findings kept | 23 |
| Critical | 1 |
| High | 5 |
| Medium | 10 |
| Low | 7 |
| Cleaned to zero | 21 / 36 (58%) |
| Approx. internal claims considered then dropped | ~435 → 23 (~5% survival) |

## Contracts in the Strong set

1. bancorprotocol__contracts-solidity__BancorNetwork
2. reflexer-labs__geb__MultiSAFEEngine
3. compound-finance__compound-protocol__Vat
4. pooltogether__v4-core__EIP2612PermitAndDeposit
5. 1inch__limit-order-protocol__ChainlinkCalculator
6. bancorprotocol__contracts-solidity__LiquidityProtection
7. bancorprotocol__contracts-solidity__StandardPoolConverter
8. transmissions11__solmate__ERC4626
9. pooltogether__v4-core__DrawBeacon
10. Layr-Labs__eigenlayer-contracts__EmissionsController
11. Layr-Labs__eigenlayer-contracts__RewardsCoordinator
12. Uniswap__v2-core__UniswapV2Pair
13. Uniswap__v3-core__UniswapV3Pool
14. Uniswap__v3-periphery__BytesLib
15. aave__aave-v3-core__Pool
16. balancer__balancer-v2-monorepo__BasePool
17. balancer__balancer-v2-monorepo__ComposableStablePool
18. balancer__balancer-v2-monorepo__FeeDistributor
19. balancer__balancer-v2-monorepo__LinearPool
20. bancorprotocol__contracts-solidity__BancorX
21. bancorprotocol__contracts-solidity__ConverterRegistry
22. bancorprotocol__contracts-solidity__StakingRewards
23. compound-finance__compound-protocol__Comptroller
24. ensdomains__ens-contracts__NameWrapper
25. makerdao__dss__Clipper
26. makerdao__dss__Flipper
27. makerdao__dss__Vow
28. morpho-org__morpho-blue__Morpho
29. pooltogether__v4-core__PrizeDistributionBuffer
30. pooltogether__v4-core__PrizeDistributor
31. pooltogether__v4-core__PrizePool
32. pooltogether__v4-core__Reserve
33. reflexer-labs__geb__GlobalSettlement
34. reflexer-labs__geb__MultiIncreasingDiscountCollateralAuctionHouse
35. reflexer-labs__geb__MultiLiquidationEngine
36. transmissions11__solmate__MultiRolesAuthority

## Human review — 6 Critical + High survivors

| # | Finding | Verdict |
|---|---------|---------|
| 1 | MultiSAFEEngine `confiscateSAFECollateralAndDebt` — no auth | Confirmed Critical |
| 2 | MultiSAFEEngine `updateAccumulatedRate` — memory copy never stored | Confirmed High |
| 3 | Vat `add`/`sub` under `^0.8.x` | Qualified High |
| 4 | Solmate ERC4626 first-deposit inflation | Qualified (design risk) |
| 5 | LiquidityProtection `_totalPositionsValue` | Partial |
| 6 | ConverterRegistry `removeConverter` | Rejected / weak |

**Tally: 2 confirmed · 2 qualified · 2 rejected**

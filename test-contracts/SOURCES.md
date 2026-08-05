# Test Contract Sources

Real, open-source Solidity fetched verbatim (no edits) from public GitHub repos, used to exercise `Smart-Contract-AI-Audit-Skill` (audit-skill) in AUDIT mode. Mix of production contracts (clean-ish, battle-tested) and intentionally-vulnerable teaching contracts (positive controls — the audit should find their known bugs).

| # | File | Source | License | Why picked |
|---|---|---|---|---|
| 01 | `01-weth9.sol` | [gnosis/canonical-weth](https://github.com/gnosis/canonical-weth/blob/master/contracts/WETH9.sol) | GPL-3.0 | Canonical Wrapped Ether — small, old (`pragma >=0.4.22 <0.6`), real mainnet contract holding billions; tests floating pragma / old-compiler / `.transfer()` gas-stipend checks |
| 02 | `02-openzeppelin-erc20.sol` | [OpenZeppelin/openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) | MIT | Current, heavily-audited reference ERC20 — expected to be near-clean; tests the audit doesn't false-positive on good code |
| 03 | `03-uniswap-v2-pair.sol` | [Uniswap/v2-core](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol) | GPL-3.0 | Production AMM core, `pragma =0.5.16`, real reentrancy lock + reserve-based pricing; tests Oracle/DeFi Mechanics/Reentrancy categories on a real complex contract |
| 04 | `04-sushiswap-masterchef.sol` | [sushiswap/masterchef](https://github.com/sushiswap/masterchef/blob/master/contracts/MasterChef.sol) | MIT | Production staking/rewards distributor; tests Accounting & Fees / DeFi Mechanics reward-accrual categories (this contract family has real documented duplicate-pool and reward-math history) |
| 05 | `05-nssc-reentrancy.sol` | [crytic/not-so-smart-contracts](https://github.com/crytic/not-so-smart-contracts/blob/master/reentrancy/Reentrancy.sol) | (educational, Trail of Bits) | Deliberately vulnerable classic reentrancy teaching contract — positive control for V-001 |
| 06 | `06-ethernaut-reentrance.sol` | [OpenZeppelin/ethernaut](https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/src/levels/Reentrance.sol) | MIT | Deliberately vulnerable (old `SafeMath`, withdraw-before-effects) — positive control for V-001/E-02 |
| 07 | `07-ethernaut-delegation.sol` | [OpenZeppelin/ethernaut](https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/src/levels/Delegation.sol) | MIT | Deliberately vulnerable `delegatecall` storage-collision puzzle — positive control for V-132/E-22 |

Fetched 2026-08-05 via `curl` from each repo's `raw.githubusercontent.com` URL. Files are unmodified byte-for-byte copies of what was live at fetch time.

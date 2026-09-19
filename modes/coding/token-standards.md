# Coding-Mode Checklist — Token Standards (15 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Token Standards` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-063: Fee-on-Transfer Token Breaks Accounting — High
- Aliases: Solodit "Fee-on-Transfer Token Breaks Accounting" (SOL-Defi-AS-9, SOL-Token-FE-6), DVL-27
- **Don't:** Protocols crediting the *requested* amount instead of the *received* amount (after transfer tax) let users over-claim, draining pools integrated with deflationary tokens (Balancer 2020; Safemoon-style; USDT fee-toggle risk).
- **Do / Detection:** `deposit(amount)` patterns that don't measure `balanceAfter - balanceBefore`; AMMs/staking caching reserves updated by parameter rather than actual balance; test with a 1% tax token.
- Full entry: `Smart-contract-vulnerability-database_v1.md:535`

### V-064: Rebasing / Elastic-Supply Token Breaks Accounting — High
- Aliases: Solodit "Rebasing / Elastic-Supply Token Breaks Accounting" (SOL-Defi-AS-10, SOL-Integrations-LSD-stETH-1), DVL-64
- **Don't:** Tokens whose balances change exogenously (stETH, AMPL, aTokens) desync internal share accounting: positive rebases get stuck or skimmed by first claimers, negative rebases cause underflow or overpay early withdrawers.
- **Do / Detection:** Amount-caching (`balanceHeld[user] = amount`) for tokens that can rebase; share-based vs absolute accounting; test with a mock that rebases +/-10% between deposit and withdraw.
- Full entry: `Smart-contract-vulnerability-database_v1.md:543`

### V-065: Non-Standard ERC20 Return Values (USDT void / ZRX false) — High
- Aliases: Solodit "Non-Standard ERC20 Return Value (USDT-style)" (SOL-Token-FE-1/8), DVL-25, DVL-26
- **Don't:** Tokens like USDT return nothing from `transfer/approve`, so interfaces expecting a bool revert or mis-evaluate; others (ZRX) return `false` instead of reverting, so code assuming revert-on-failure credits deposits that never happened.
- **Do / Detection:** Direct `IERC20(t).transfer(...)` on arbitrary tokens without SafeERC20/SafeTransferLib; boolean results not required true; confirm per-whitelisted-token behavior (revert vs false vs void).
- Full entry: `Smart-contract-vulnerability-database_v1.md:551`

### V-066: Tokens Reverting on Zero-Amount Transfers — Medium
- Aliases: Solodit "Tokens Reverting on Zero-Amount Transfers" (SOL-Token-FE-10)
- **Don't:** Some tokens (e.g., LEND) revert on `transfer(0)`; protocols performing zero-value transfers in edge paths (claiming zero rewards) DoS.
- **Do / Detection:** Transfer calls reachable with amount 0 (empty claims, zero fees).
- Full entry: `Smart-contract-vulnerability-database_v1.md:559`

### V-067: Blacklist / Pausable Token DoS — Medium
- Aliases: Solodit "Blacklist / Pausable Token DoS" (SOL-AM-DOSA-3, SOL-Token-FE-4/15)
- **Don't:** USDC/USDT-style blacklists or pauses make transfers revert; a blacklisted user in a push-payment or withdrawal flow blocks the entire queue or pool.
- **Do / Detection:** Iterated transfers to arbitrary token holders; single-point transfer in shared flows; consider pull patterns.
- Full entry: `Smart-contract-vulnerability-database_v1.md:567`

### V-068: Multiple-Address ("Two-Address") Tokens — High
- Aliases: Solodit "Multiple-Address (Two-Address) Tokens" (SOL-Token-FE-5)
- **Don't:** Tokens reachable via two addresses (Synthetix ProxyERC20, TUSD legacy/new) can double-count balances or bypass per-token checks keyed on address.
- **Do / Detection:** Accounting keyed by token address; whitelist per address; same-token-different-address deposit paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:575`

### V-069: Non-18-Decimals Token Mishandling — High
- Aliases: Solodit "Non-18-Decimals Token Mishandling" (SOL-Defi-General-1, SOL-AM-DOSA-5, SOL-McCc-6)
- **Don't:** Assuming 18 decimals misprices 6-decimal USDC or 2-decimal tokens by orders of magnitude; low-decimal tokens also round shares to zero (DoS or free value).
- **Do / Detection:** Hardcoded `1e18` scaling; missing normalization by `token.decimals()`; cross-chain decimal differences.
- Full entry: `Smart-contract-vulnerability-database_v1.md:583`

### V-070: ERC20 Approval Race Condition — Medium
- Aliases: Solodit "ERC20 Approval Race Condition" (SOL-Token-FE-2/13/14), DVL-73, SWC-114
- **Don't:** Changing allowance N->M via a second `approve` lets the spender front-run and spend N, then M after the change lands (N+M total). Also a documented instance of transaction-order dependence.
- **Do / Detection:** Token implementations without `increaseAllowance/decreaseAllowance` or require-zero-then-set semantics; integrations approving non-zero from non-zero; note USDT reverts on non-zero->non-zero approve.
- Full entry: `Smart-contract-vulnerability-database_v1.md:591`

### V-071: Unlimited / Misused Approvals & Approval Scams — High
- Aliases: SF-2, DVL-18 (see also V-070)
- **Don't:** Unlimited (`type(uint256).max`) approvals to upgradeable/unverified spenders drain user funds when the spender is exploited; users are also phished into `approve`/`setApprovalForAll` to drainer contracts, and protocols with anyone-can-sweep `transferFrom` functions abuse standing approvals.
- **Do / Detection:** `approve(spender, type(uint256).max)` in protocol code; functions that `transferFrom` an arbitrary `from` using standing approvals; permit/permit2 signature flows that don't bind the spender.
- Full entry: `Smart-contract-vulnerability-database_v1.md:599`

### V-072: Incorrect `supportsInterface` (ERC165) — Low/Medium
- Aliases: Solodit "Incorrect supportsInterface" (SOL-Token-NfE1-5)
- **Don't:** NFT contracts misreporting interface IDs break marketplace/wallet integrations and can bypass receiver checks that rely on ERC165.
- **Do / Detection:** Verify `supportsInterface` covers all implemented interfaces including parents.
- Full entry: `Smart-contract-vulnerability-database_v1.md:607`

### V-073: NFT Approval Theft — High
- Aliases: Solodit "NFT Approval Theft" (SOL-Token-NfE1-4)
- **Don't:** Flawed approval/operator logic (approvals not cleared on transfer, operator edge cases, approval granted via exploited function) lets attackers transfer others' NFTs.
- **Do / Detection:** `_approve`/`setApprovalForAll` clearing semantics; approvals surviving ownership changes.
- Full entry: `Smart-contract-vulnerability-database_v1.md:615`

### V-074: Flash Mint Inflates Balance Mid-Transaction — High
- Aliases: Solodit "Flash Mint Inflates Balance Mid-Transaction" (SOL-Token-FE-9)
- **Don't:** ERC3156 flash-mintable tokens let an attacker inflate `totalSupply`/balances within a transaction, gaming balance-based voting, snapshots, or pricing.
- **Do / Detection:** Governance/accounting reading checkpoint-less balances; protocols accepting flash-mintable collateral.
- Full entry: `Smart-contract-vulnerability-database_v1.md:623`

### V-075: CryptoPunks / Non-Standard NFT Handling — Medium
- Aliases: Solodit "CryptoPunks / Non-Standard NFT Handling" (SOL-Token-NfE1-8)
- **Don't:** Collections predating ERC721 (CryptoPunks) lack standard `transferFrom`/approvals; wrappers mishandle them, causing stuck NFTs or stolen bids.
- **Do / Detection:** Special-case collections integrated via generic ERC721 paths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:631`

### V-076: Token Transfer to Zero Address / Contract's Own Address — Medium
- Aliases: CR-1
- **Don't:** Tokens sent to `0x0` or to the token contract itself are permanently stuck.
- **Do / Detection:** Check `transfer`/`transferFrom` validate destination: `require(to != address(0))` and `require(to != address(this))`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:639`

### V-077: Phantom Function — Fake `permit` — High
- Aliases: DVL-28
- **Don't:** Tokens with a fallback function (e.g., WETH9) accept calls to functions they don't define without reverting; `SafeERC20.safePermit` on such a token succeeds as a no-op, and code assuming an allowance was set proceeds to pull zero funds.
- **Do / Detection:** Any `permit` call where the token is user-selected; `token.code.length == 0` guards plus verifying allowance actually increased post-permit; flag flows combining `permit` + `transferFrom` without checking allowance.
- Full entry: `Smart-contract-vulnerability-database_v1.md:647`


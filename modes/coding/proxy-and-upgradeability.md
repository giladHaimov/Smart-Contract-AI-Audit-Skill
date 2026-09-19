# Coding-Mode Checklist — Proxy & Upgradeability (10 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Proxy & Upgradeability` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-084: Uninitialized Implementation Contract Takeover — Critical
- Aliases: Solodit "Uninitialized Implementation Contract Takeover" (SOL-Basics-PU-5), DVL-82
- **Don't:** The implementation (logic) contract itself is left uninitialized; an attacker initializes it directly, takes ownership, and selfdestructs it or abuses its delegatecall context (Parity/UUPS incidents).
- **Do / Detection:** Implementation contract without `_disableInitializers()` in constructor; live bytecode check; confirm implementation's privileged functions can't be invoked directly.
- Full entry: `Smart-contract-vulnerability-database_v1.md:707`

### V-085: Missing `initializer` / Re-initialization — High
- Aliases: Solodit "Missing initializer / Re-initialization" (SOL-Basics-PU-2/3)
- **Don't:** `initialize()` callable multiple times (or child initializers missing `onlyInitializing`) lets attackers reset owner/config after deployment.
- **Do / Detection:** All init functions for `initializer`/`onlyInitializing`; nested inheritance init chains.
- Full entry: `Smart-contract-vulnerability-database_v1.md:715`

### V-086: Unprotected UUPS `_authorizeUpgrade` — Critical
- Aliases: Solodit "Unprotected UUPS _authorizeUpgrade" (SOL-Basics-PU-4)
- **Don't:** If `_authorizeUpgrade` lacks access control, anyone upgrades the proxy to malicious logic and drains everything.
- **Do / Detection:** UUPS implementations; verify `onlyOwner`/role on `_authorizeUpgrade`. Slither `unprotected-upgrade`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:723`

### V-087: Storage Collision / Layout Mismatch Between Versions — Critical
- Aliases: Solodit "Storage Collision / Layout Mismatch Between Versions" (SOL-Basics-PU-9/10), DVL-17 (see also E-13, E-14 in Part II)
- **Don't:** Reordered, retyped, or inserted state variables in a new implementation shift storage slots; proxy and implementation declaring variables at the same slot differently clobber each other through delegatecall (Audius incident). Unstructured-storage proxies must keep implementation/admin slots at EIP-1967 locations.
- **Do / Detection:** Diff storage layouts across versions (`openzeppelin-upgrades`, `forge inspect storage-layout`); check inheritance order changes and EIP-1967 slot usage.
- Full entry: `Smart-contract-vulnerability-database_v1.md:731`

### V-088: Missing Storage Gaps in Upgradeable Base Contracts — Medium/High
- Aliases: Solodit "Missing Storage Gaps", DVL-84
- **Don't:** Upgradeable libraries without `uint256[N] __gap` can't add variables in future versions without colliding with child contract storage; inserting (not appending) variables shifts all child-layout slots.
- **Do / Detection:** Every upgradeable base contract ends with `__gap`; diff V1/V2 layouts with the OZ upgrades plugin.
- Full entry: `Smart-contract-vulnerability-database_v1.md:739`

### V-089: Constructor Used in Implementation / Immutable Variables Lost on Upgrade — Medium
- Aliases: Solodit "Constructor Used in Implementation / Immutable Variables Lost on Upgrade" (SOL-Basics-PU-1/7), DVL-85 (see also E-33 in Part II)
- **Don't:** Constructors don't execute in proxy context and `immutable` values live in implementation bytecode, not proxy storage — proxied state set that way is empty, or an upgraded implementation with different immutables silently changes behavior while storage stays.
- **Do / Detection:** Constructors setting state in logic contracts; `immutable` use in upgradeable contracts meant to preserve config; config should live in storage via initializers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:747`

### V-090: `selfdestruct` / `delegatecall` in Implementation Contract — Critical/High
- Aliases: Solodit "selfdestruct / delegatecall in Implementation Contract" (SOL-Basics-PU-6, SOL-Basics-VI-EAI-1) (see also V-024, E-26 in Part II)
- **Don't:** If an attacker gains control of the implementation, reachable `selfdestruct` destroys the logic contract and bricks every proxy; raw `delegatecall` enables storage hijack. EIP-6780 changes selfdestruct semantics post-Cancun.
- **Do / Detection:** `selfdestruct`/`delegatecall` reachable in implementation contracts.
- Full entry: `Smart-contract-vulnerability-database_v1.md:755`

### V-091: Wrong OpenZeppelin Branch for Proxies — Medium
- Aliases: Solodit "Wrong OpenZeppelin Branch for Proxies" (SOL-Basics-PU-8)
- **Don't:** Mixing regular OZ contracts with `-upgradeable` variants misaligns initializers and storage layouts (e.g., non-upgradeable `ERC20` inherited by upgradeable child).
- **Do / Detection:** Imports from `@openzeppelin/contracts` inside upgradeable hierarchies.
- Full entry: `Smart-contract-vulnerability-database_v1.md:763`

### V-092: Proxy Selector Clash / Transparent Proxy Admin Confusion — High
- Aliases: Solodit "Transparent Proxy Admin Clash / OZ Proxy Bugs" (SOL-Basics-VI-OVI-7), DVL-83
- **Don't:** A function in the implementation with the same 4-byte selector as a proxy admin function gets shadowed or misrouted, locking upgrades or exposing admin functions to users; known OZ proxy bugs (e.g., TransparentUpgradeableProxy <4.8.3 admin storage read) compound this.
- **Do / Detection:** Diff selector sets of proxy-admin functions vs implementation ABI; check transparent-proxy admin routing; verify UUPS `upgradeTo` is access-controlled; OZ version check.
- Full entry: `Smart-contract-vulnerability-database_v1.md:771`

### V-093: General Upgradeability Governance Risks — High
- Aliases: CR-12
- **Don't:** Upgradeable contracts add governance and migration risk: admin keys can swap logic maliciously, storage layouts can collide across versions, and uninitialized proxy logic contracts can be hijacked.
- **Do / Detection:** Identify the upgrade admin (EOA vs multisig vs timelock); check `_disableInitializers` in constructor, storage gaps, `delegatecall` to admin-settable implementation, no slot reordering between versions.
- Full entry: `Smart-contract-vulnerability-database_v1.md:779`


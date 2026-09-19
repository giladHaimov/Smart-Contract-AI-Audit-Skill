# Coding-Mode Checklist — Storage (30 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `Storage` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### V-139: Memory Pointer Aliasing — High
- Aliases: Solodit "Memory Pointer Aliasing"
- **Don't:** Assigning one `bytes memory`/array variable to another makes both point to the same memory; mutating one silently corrupts the other.
- **Do / Detection:** Memory-to-memory assignments of dynamic types later mutated independently.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1157`

### V-140: Sensitive Data Exposure On-Chain — Medium
- Aliases: Solodit "Sensitive Data Exposure On-Chain" (SOL-Heuristics-15), DVL-8, SWC-136, SF-5 (see also E-01 in Part II)
- **Don't:** "Private" variables and calldata are publicly readable via storage inspection and transaction traces; storing secrets (commitments, passwords, unrevealed bids, PII) on-chain leaks them and defeats the scheme.
- **Do / Detection:** Any `private` data relied upon for secrecy; commitments before reveals; recommend commit-reveal or off-chain storage with hashes.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1165`

### V-141: Stale Mapping Entries / Struct Deletion Oversight — Medium
- Aliases: Solodit "Stale Mapping Entries (Missing Deletion)" (SOL-Basics-Map-1), DVL-38 (see also E-09 in Part II)
- **Don't:** Removed users/orders/approvals aren't deleted from mappings, so "deleted" entities retain rights, rewards, or balances; `delete myStruct` doesn't clear nested mappings or dynamic arrays, whose residue corrupts logic when the entry is reused.
- **Do / Detection:** Removal flows that skip `delete` on all related mapping keys; `delete` on structs containing `mapping`/`[]` members; re-registration flows tested for stale residual state; use epoch-keyed mappings or explicit per-key cleanup.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1173`

### V-142: Stale Cached Storage/Memory Values (Data Location Confusion) — Medium/High
- Aliases: Solodit "Stale Cached Storage/Memory Values", DVL-20
- **Don't:** A memory copy of a storage struct is read after the storage was updated (or vice versa), so logic uses pre-update values — famously the Cover Protocol hack where a stale cache let attackers mint unbounded rewards.
- **Do / Detection:** Track `Struct storage s = ...` vs `memory` copies through deposit/withdraw/reward flows; cached structs/values crossing an external call or update; reward accounting that snapshots `rewardPerTokenStored` in memory.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1181`

### V-143: Inter-Related Storage Corruption — High
- Aliases: Solodit "Inter-Related Storage Corruption" (SOL-Defi-LSD-6)
- **Don't:** Multiple storage structures that must stay consistent (operators<->validators, positions<->indexes) are updated in one but not the other, corrupting protocol state.
- **Do / Detection:** Linked data structures with separate update paths; invariant tests across structures.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1189`

### V-144: Uninitialized Storage Pointer / Uninitialized Proxy State — High
- Aliases: DVL-16, SWC-109
- **Don't:** Uninitialized local `storage` pointers (pre-0.5.0) default to slot 0 and overwrite the contract's first state variables (CryptoRoulette honeypot); uninitialized proxy state (owner = address(0)) leaves the contract claimable. Compiler-deprecated since 0.5.0 but still relevant on legacy code.
- **Do / Detection:** On pre-0.5 code, flag local struct/array declarations without `memory`/`storage`; check proxy implementations have initialized critical vars and disabled initializers; zero-address comparisons against ecrecover/owner defaults.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1197`

### V-145: State Variable Default Visibility — Low
- Aliases: SWC-108
- **Don't:** State variables without explicit visibility default to `internal`; developers may wrongly assume `public` (missing getter) or wrongly assume hidden data is safe. Code-quality/correctness issue rather than a direct exploit.
- **Do / Detection:** Lint for state variables lacking explicit visibility; check intended getters exist and "hidden" sensitive data doesn't rely on non-public visibility.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1205`

### V-146: Shadowing State Variables — Medium
- Aliases: SWC-119, CR-10
- **Don't:** Inheritance or same-contract re-declaration of a variable name creates two separate variables; functions read different copies than intended, breaking caps/limits.
- **Do / Detection:** Diff state-variable names across the inheritance hierarchy; compiler shadowing warnings. Slither `shadowing-state`, `shadowing-local`.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1213`

### V-147: Write to Arbitrary Storage Location — Critical
- Aliases: SWC-124
- **Don't:** User-controlled index/length writes to dynamic arrays (`array.length--` underflow pre-0.8, or attacker-chosen index) let an attacker write to any storage slot, overwriting `owner` or balances.
- **Do / Detection:** `arr.length` decrements/increments under user influence; `arr[userControlledIndex] = value` without bounds checks; inline-assembly `sstore` with computed slots.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1221`

### V-148: Array Deletion Oversight (Ghost Entries) — Medium
- Aliases: DVL-39 (see also V-159 swap-and-pop reordering)
- **Don't:** Deleting array elements via `delete arr[i]` leaves a zero-value gap (not removal); length stays the same, so iteration/division over the array counts ghost entries, corrupting averages and reward splits.
- **Do / Detection:** `delete arr[i]` where `arr.length` is later used for division or iteration; check swap-and-pop vs delete semantics; test reward distribution after mid-array removal.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1229`

### V-149: DirtyBytes (Dirty High-Order Bits in Storage) — Medium
- Aliases: DVL-21 (see also E-11 in Part II)
- **Don't:** Copying `bytes` from memory/calldata to storage can leave dirty high-order bits in the slot; values read back via assembly that assumes clean padding behave incorrectly.
- **Do / Detection:** Assembly `mload`/`sstore` of byte arrays without masking; `bytes` packed into fixed-size slots where padding matters; compare `keccak256(bytes)` round-trips after storage writes.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1237`

### V-150: Transient Storage Misuse (EIP-1153) — High
- Aliases: DVL-48 (see also E-28, E-29 in Part II)
- **Don't:** `TSTORE`/`TLOAD` values persist for the whole transaction across calls; a callback mid-transaction can overwrite a transient-storage lock/flag, bypassing reentrancy guards or access checks built on it (sir.trading exploit).
- **Do / Detection:** For Solidity >=0.8.24 contracts using `tstore`/`tload`: check what external calls happen between set and read, whether callbacks (token hooks, ETH receive) can rewrite the slot, and that the guard can't be set to an attacker-chosen value.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1245`

### E-09: Clearing Mappings — Medium
- Aliases: SC-9 (see also V-141 in Part I)
- **Don't:** Mappings don't track assigned keys and can't be cleared; deleting a storage array/struct of mappings leaves mapping contents alive, which resurfaces if the array regrows.
- **Do / Detection:** Flag `delete` on arrays/structs containing mappings and any reuse of deleted storage regions; use iterable-mapping libraries when deletion is required.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1707`

### E-10: Internal Function Pointers in Upgradeable Contracts — Medium
- Aliases: SC-10 (see also V-089 in Part I)
- **Don't:** Code upgrades invalidate stored internal-function-type values; persisting them across upgrades breaks.
- **Do / Detection:** Flag internal function pointers stored in state variables in upgradeable/proxy systems; treat them as ephemeral.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1715`

### E-13: Storage Slot Packing and Ordering — High
- Aliases: EV-1 (see also V-087 in Part I)
- **Don't:** Variables <32 bytes pack into shared slots lower-order aligned; structs/arrays always start new slots; inheritance ordering follows C3 linearization most-base-first and slots can be shared across contracts. Wrong assumptions corrupt data in assembly or proxies.
- **Do / Detection:** In upgradeable contracts, diff the storage layout (slot+offset) between versions; flag any inline assembly using hard-coded slot numbers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1739`

### E-14: Mapping/Dynamic Array Slot Derivation (keccak256-based) — Medium
- Aliases: EV-2 (see also V-087, V-190 in Part I)
- **Don't:** Mapping values live at `keccak256(h(k) . p)`, dynamic array data at `keccak256(p)`; slot `p` holds array length. Hand-computed slots in assembly or delegatecalled code must match exactly.
- **Do / Detection:** Verify any manual slot computation against the docs formula; flag delegatecall targets with mismatched layout.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1747`

### E-15: bytes/string Short-vs-Long Storage Encoding — Medium
- Aliases: EV-3
- **Don't:** `bytes`/`string` shorter than 32 bytes store data + `length*2` in one slot; longer ones store `length*2+1` with data at `keccak256(p)`. Reading an invalidly encoded slot via IR panics with `Panic(0x22)`.
- **Do / Detection:** Flag assembly reading raw byte-array slots without checking the low bit; ensure storage written by assembly uses valid encoding.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1755`

### E-17: Custom Storage Layout (`layout at N`) — High
- Aliases: EV-5 (related compiler bug: KB-01 in Part III)
- **Don't:** `contract C layout at N` shifts all static storage slots of the whole inheritance tree; misplacement near the storage end or clashing with other contracts sharing storage is catastrophic.
- **Do / Detection:** Flag any `layout at` specifier; verify base slot, inheritance-wide offset arithmetic, and distance from the 2^256 end.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1771`

### E-28: Transient Storage (EIP-1153) Semantics — High
- Aliases: EV-16 (see also V-150 in Part I)
- **Don't:** Transient storage persists across all calls within one transaction (including reentrant and delegated calls) and resets at transaction end; only value types are supported; layout is independent of storage layout.
- **Do / Detection:** Verify reentrancy locks built on `transient` variables are reset on every exit path; flag assumptions that transient state is cleared between external calls within a tx.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1859`

### E-29: Transient Storage via Yul Before/Outside Native Support — Medium
- Aliases: EV-17 (see also V-150 in Part I)
- **Don't:** Native `transient` variables arrived in Solidity 0.8.28; before that, TSTORE/TLOAD were only reachable via Yul, and contracts compiled for pre-Cancun EVM versions behave differently across chains.
- **Do / Detection:** Check `evmVersion >= cancun` for contracts using transient storage; flag raw `tstore`/`tload` assembly and cross-chain deployments.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1867`

### E-33: Immutable/Constant Variables Live in Code, Not Storage — Low
- Aliases: EV-21 (see also V-089 in Part I)
- **Don't:** `immutable` and `constant` values are embedded in the deployed code region, occupy no storage slots; proxies/clones and storage-layout tooling must account for this.
- **Do / Detection:** Verify storage-layout comparisons exclude immutables; flag storage readers assuming slots for config.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1899`

### KB-04: LostStorageArrayWriteOnSlotOverflow — introduced 0.1.0, fixed 0.8.32 — Low
- **Don't:** Clearing/copying arrays straddling the end of the storage address space (slots near 2^256−1) could silently retain data.
- **Do / Detection:** Check solc >= 0.8.32; flag custom `layout at` bases or inline assembly writing storage near the slot-space end.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1960`

### KB-08: StorageWriteRemovalBeforeConditionalTermination — introduced 0.8.13, fixed 0.8.17 — High
- **Don't:** The Yul optimizer could wrongly remove prior storage writes when a function conditionally terminated the call via assembly `return(...)` or `stop()`.
- **Do / Detection:** Check solc >= 0.8.17; flag inline assembly `return`/`stop` combined with storage writes before it (e.g. EIP-1167 minimal proxies / assembly-terminating functions).
- Full entry: `Smart-contract-vulnerability-database_v1.md:1980`

### KB-10: DirtyBytesArrayToStorage — introduced 0.0.1, fixed 0.8.15 — Low
- **Don't:** Copying `bytes` arrays from memory/calldata to storage could leave dirty (non-zero) bits beyond the array length in storage.
- **Do / Detection:** Check solc >= 0.8.15; flag inline assembly that reads storage byte arrays' full slots assuming clean padding.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1990`

### KB-15: UserDefinedValueTypesBug — introduced 0.8.8, fixed 0.8.9 — Low
- **Don't:** User-defined value types with underlying type shorter than 32 bytes used an incorrect storage layout.
- **Do / Detection:** Check solc >= 0.8.9; flag `type X is uint128` declarations compiled with 0.8.8.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2015`

### KB-19: EmptyByteArrayCopy — introduced before 0.4.x, fixed 0.7.4 — Medium
- **Don't:** Copying an empty byte array/string from memory or calldata to storage corrupted data if the target array was later lengthened without storing new data.
- **Do / Detection:** Check solc >= 0.7.4; flag storage `bytes`/`string` assignments of possibly-empty values later pushed to.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2035`

### KB-20: DynamicArrayCleanup — introduced before 0.4.x, fixed 0.7.3 — Medium
- **Don't:** Assigning a shorter dynamic array of types ≤16 bytes over a longer one in storage left parts of deleted slots non-zeroed.
- **Do / Detection:** Check solc >= 0.7.3; flag shrinking storage array assignments of packed small types.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2040`

### KB-32: SignedArrayStorageCopy — introduced 0.4.7, fixed 0.5.10 — Medium
- **Don't:** Assigning an array of signed integers to a storage array of a different signed type corrupted data (improper sign extension).
- **Do / Detection:** Check solc >= 0.5.10; flag storage array assignments between differing signed int widths.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2100`

### KB-51: HighOrderByteCleanStorage — introduced 0.1.6, fixed 0.4.4 — High
- **Don't:** For short types, high-order bytes weren't cleaned before storage writes and could overwrite adjacent packed data.
- **Do / Detection:** Check solc >= 0.4.4; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2195`

### KB-58: ArrayAccessCleanHigherOrderBits — introduced before 0.3.x, fixed 0.3.1 — High
- **Don't:** Access to array elements of <32-byte types didn't clean higher-order bits, corrupting sibling elements.
- **Do / Detection:** Check solc >= 0.3.1; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2230`

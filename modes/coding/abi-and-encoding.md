# Coding-Mode Checklist — ABI & Encoding (24 entries)

Real-time guardrails for this category. Consult while writing or editing Solidity code that touches `ABI & Encoding` concerns, before considering the code done. Each item is reused verbatim from the full database (`../../Smart-contract-vulnerability-database_v1.md`), just relabeled Don't/Do for at-the-keyboard use — nothing paraphrased or dropped. Follow the line pointer for Aliases and exact Sources.

### E-11: Dirty Higher Order Bits / msg.data Malleability — Medium
- Aliases: SC-11 (see also V-149 in Part I)
- **Don't:** Types shorter than 32 bytes may carry dirty higher-order bits; `msg.data` is malleable — `f(uint8)` is callable with different padded forms producing different `keccak256(msg.data)`.
- **Do / Detection:** Flag any logic hashing or signing over raw `msg.data`/`calldata` (meta-transactions, signature schemes); re-encode canonical ABI instead.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1723`

### E-18: Memory Layout Reserved Slots (0x00-0x7f) — Medium
- Aliases: EV-6
- **Don't:** Solidity reserves 0x00-0x3f as hashing scratch space, 0x40 as free memory pointer, 0x60 as zero slot which must never be written. Assembly violating these corrupts Solidity-managed memory.
- **Do / Detection:** In inline assembly, verify the free memory pointer is read/updated via `mload(0x40)`/`mstore(0x40)`, scratch space isn't used across statements, and 0x60 stays zero.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1779`

### E-19: Free Memory Not Guaranteed Zeroed — Medium
- Aliases: EV-7
- **Don't:** Operations needing >64 bytes of temp memory write at the free pointer without updating it; memory there may not be zeroed. Using `msize` as a "zeroed area" pointer has unexpected results.
- **Do / Detection:** Flag assembly that assumes freshly pointed-to memory is zero; require explicit zeroing or proper allocation.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1787`

### E-20: Memory vs Storage Layout Differences — Medium
- Aliases: EV-8
- **Don't:** Memory arrays always occupy full 32-byte multiples per element (even `uint8[4]`: 1 storage slot but 128 bytes in memory); memory structs are also unpacked. Conflating layouts breaks low-level code.
- **Do / Detection:** Flag assembly or hashing routines that assume storage-style packing for memory objects.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1795`

### E-37: Calldata Decoding: Eager vs Lazy — Low
- Aliases: EV-25
- **Don't:** `memory` parameters are eagerly ABI-decoded at function entry, while `calldata` parameters are decoded lazily on access. Wrong location choices waste gas or enable subtle aliasing issues.
- **Do / Detection:** Prefer `calldata` for external read-only reference args; flag unnecessary `memory` parameters.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1931`

### KB-09: AbiReencodingHeadOverflowWithStaticArrayCleanup — introduced 0.5.8, fixed 0.8.16 — Medium
- **Don't:** ABI-encoding a tuple whose last component is a statically-sized calldata array corrupted the leading 32 bytes of the first dynamically encoded component.
- **Do / Detection:** Check solc >= 0.8.16; flag `abi.encode`/external calls with tuples mixing static calldata arrays and dynamic members.
- Full entry: `Smart-contract-vulnerability-database_v1.md:1985`

### KB-13: NestedCalldataArrayAbiReencodingSizeValidation — introduced 0.5.8, fixed 0.8.14 — Low
- **Don't:** ABI re-encoding of nested dynamic calldata arrays missed size validation and could read past `calldatasize()`.
- **Do / Detection:** Check solc >= 0.8.14; flag external functions taking nested dynamic calldata arrays (e.g. `uint[][] calldata`) re-encoded or forwarded.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2005`

### KB-14: AbiEncodeCallLiteralAsFixedBytesBug — introduced 0.8.11, fixed 0.8.13 — Low
- **Don't:** Literals passed as fixed-length `bytesNN` arguments to `abi.encodeCall` were encoded incorrectly.
- **Do / Detection:** Check solc >= 0.8.13; flag `abi.encodeCall(selector, (…, 0x1234, …))` with literal bytes arguments.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2010`

### KB-16: SignedImmutables — introduced 0.6.5, fixed 0.8.9 — Low
- **Don't:** Immutable variables of signed integer type shorter than 256 bits could have invalid higher-order bits when read via inline assembly.
- **Do / Detection:** Check solc >= 0.8.9; flag assembly reading of signed short-type `immutable` variables.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2020`

### KB-17: ABIDecodeTwoDimensionalArrayMemory — introduced 0.4.16, fixed 0.8.4 — Low
- **Don't:** `abi.decode` on memory byte arrays (2-D arrays) could depend on memory contents outside the decoded array.
- **Do / Detection:** Check solc >= 0.8.4; flag `abi.decode(...(T[][]))` where input came from memory rather than calldata.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2025`

### KB-23: MissingEscapingInFormatting — introduced 0.5.14, fixed 0.6.8 — Low
- **Don't:** String literals containing double backslashes passed directly to external/encoding calls were mis-encoded under ABIEncoderV2.
- **Do / Detection:** Check solc >= 0.6.8; flag literals with `\\` used directly in external call arguments.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2055`

### KB-24: ArraySliceDynamicallyEncodedBaseType — introduced 0.6.0, fixed 0.6.8 — Low
- **Don't:** Array slices of arrays with dynamically encoded base types (e.g. multi-dimensional arrays) read invalid data.
- **Do / Detection:** Check solc >= 0.6.8; flag `arr[start:end]` slicing on calldata multi-dimensional/dynamic-base arrays.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2060`

### KB-27: MemoryArrayCreationOverflow — introduced 0.2.0, fixed 0.6.5 — Low
- **Don't:** Creating very large memory arrays could produce overlapping memory regions and memory corruption.
- **Do / Detection:** Check solc >= 0.6.5; flag `new T[](length)` with user-controlled length (still audit for OOG/DoS).
- Full entry: `Smart-contract-vulnerability-database_v1.md:2075`

### KB-30: ABIEncoderV2LoopYulOptimizer — introduced 0.5.14, fixed 0.5.15 — Low
- **Don't:** With experimental ABIEncoderV2 and Yul optimizer both active, one optimizer component reused stale memory data.
- **Do / Detection:** Check solc >= 0.5.15; legacy-only issue.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2090`

### KB-31: ABIEncoderV2CalldataStructsWithStaticallySizedAndDynamicallyEncodedMembers — introduced 0.5.6, fixed 0.5.11 — Low
- **Don't:** Reading calldata structs containing dynamically encoded but statically sized members produced incorrect values.
- **Do / Detection:** Check solc >= 0.5.11; flag calldata struct parameters mixing static arrays with dynamic members.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2095`

### KB-33: ABIEncoderV2StorageArrayWithMultiSlotElement — introduced 0.4.16, fixed 0.5.10 — Low
- **Don't:** Storage arrays of structs/static arrays were misread when directly encoded in external calls or `abi.encode*`.
- **Do / Detection:** Check solc >= 0.5.10; flag passing storage arrays of structs directly to external calls.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2105`

### KB-34: DynamicConstructorArgumentsClippedABIV2 — introduced 0.4.16, fixed 0.5.9 — Low
- **Don't:** Constructors taking structs/arrays containing dynamically sized arrays reverted or decoded invalid data.
- **Do / Detection:** Check solc >= 0.5.9; flag complex dynamic struct constructor args on old deployments.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2110`

### KB-36: IncorrectEventSignatureInLibraries (+0.4.x variant) — introduced 0.5.0, fixed 0.5.8; 0.4.x variant 0.3.0→0.4.26 — Low
- **Don't:** Events declared in libraries using contract types hashed to an incorrect event signature (topic0), breaking log filtering.
- **Do / Detection:** Check solc >= 0.5.8; verify event topic hashes of library-declared events against off-chain indexers.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2120`

### KB-37: ABIEncoderV2PackedStorage (+0.4.x variant) — introduced 0.5.0, fixed 0.5.7; 0.4.x variant 0.4.19→0.4.26 — Low
- **Don't:** Encoding storage structs/arrays with sub-32-byte types directly from storage via experimental ABIEncoderV2 corrupted data.
- **Do / Detection:** Check solc >= 0.5.7; flag `abi.encode` of storage references to tightly packed structs.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2125`

### KB-41: EventStructWrongData — introduced 0.4.17, fixed 0.4.25 — Low
- **Don't:** Emitting events with struct arguments logged wrong data.
- **Do / Detection:** Check solc >= 0.4.25; verify off-chain decoding matches.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2145`

### KB-43: NestedArrayFunctionCallDecoder — introduced 0.1.4, fixed 0.4.22 — Medium
- **Don't:** Calling functions returning multi-dimensional fixed-size arrays could corrupt memory.
- **Do / Detection:** Check solc >= 0.4.22; flag external calls returning `uint[N][M]` fixed arrays on ancient contracts.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2155`

### KB-44: ZeroFunctionSelector — introduced before 0.4.x, fixed 0.4.18 — Low
- **Don't:** A function could be crafted with selector `0x00000000`, executed instead of the fallback in specific circumstances.
- **Do / Detection:** Check solc >= 0.4.18; ensure fallback/receive logic doesn't assume selectors are never zero.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2160`

### KB-47: SkipEmptyStringLiteral — introduced before 0.4.x, fixed 0.4.12 — Low
- **Don't:** Using `""` in a function call caused following arguments to be passed incorrectly.
- **Do / Detection:** Check solc >= 0.4.12; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2175`

### KB-57: CleanBytesHigherOrderBits — introduced before 0.3.x, fixed 0.3.3 — High
- **Don't:** Higher-order bits of short `bytesNN` types were not cleaned before comparison.
- **Do / Detection:** Check solc >= 0.3.3; legacy only.
- Full entry: `Smart-contract-vulnerability-database_v1.md:2225`


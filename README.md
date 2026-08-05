# Smart-Contract-AI-Audit-Skill

Okay, so what is this thing.

Short version: it's a Solidity vulnerability database — 293 known bug classes, pulled from four different security sources and merged into one document — wired up so an AI coding agent can actually use it while working, instead of it just sitting in a repo as a PDF nobody opens after week one. Two modes. Point it at code you already wrote and it audits; keep it open while you're writing new code and it nags you before you ship the bug in the first place.

I want to be upfront about something: this is not a linter, and I didn't try to make it one. Slither already does the mechanical pattern-matching well, and honestly the skill tells you to go run Slither in a bunch of places. What this is for is the stuff that isn't mechanically detectable — a reward function that's fine by itself but wrong the moment an admin calls a specific setter, an oracle read that's totally reasonable on mainnet but falls apart the second you're bridging across chains. That kind of thing needs someone (or something) to actually read the function, not just grep for `.call`.

## What's in the folder

```
SKILL.md                                     entry point, routes to one of the two modes
Smart-contract-vulnerability-database_v1.md the database — 293 entries, source of truth, nobody edits this except to add entries
reference/
  INDEX.md                                  all 293 entries, compact — id / title / severity / category / line number
modes/
  AUDIT_MODE.md                             the audit workflow + the report template
  CODING_MODE.md                            routes into 21 category checklists depending on what you're writing
  coding/<category>.md                      21 of these, Don't/Do per entry, text lifted straight from the DB
test-contracts/                             7 real contracts I used to sanity-check this actually works
test-contracts-audit/                       what the audits of those 7 contracts came back with
```

Nothing fancy. It's markdown and a set of instructions for reading the markdown in a sensible order. The database is the part that matters; the file structure is just there so an agent doesn't have to swallow the whole thing every time it wants to check one thing.

## The database, and where it actually came from

293 entries, three parts:

| Part | IDs | Count | Covers |
|---|---|---|---|
| I — App-level | V-001–V-197 | 197 | reentrancy, access control, oracles, math/rounding, accounting, token weirdness, DeFi mechanics, proxies, DoS, MEV, signatures, external calls, storage, plain logic bugs, governance, cross-chain, the rest |
| II — EVM/language | E-01–E-37 | 37 | straight out of the official Solidity docs' security-considerations section |
| III — Compiler bugs | KB-01–KB-59 | 59 | every entry in solc's own bugs.json, each with the exact version range it's live in |

Started from 411 raw items across Solodit's audit-finding taxonomy, DeFiVulnLabs' catalog, the SWC Registry plus ConsenSys/SCSFG, and the Solidity docs themselves. A lot of those 411 are the same bug described three different ways by three different people, so they got merged — classic reentrancy shows up once as V-001 but its Aliases line still lists Solodit's name for it, DVL-4, SWC-107, and CP-1, so you can trace it back to wherever you first heard about it. At the end I ran a check to make sure every single one of the 411 original names actually landed somewhere in the final doc, either as its own entry or folded into one — didn't want to quietly lose something in the merge.

One rule I stuck to the whole way through: the big database file never gets touched once it's built. Everything else — the index, the 21 checklists — is generated *from* it, and if any of those ever disagree with the source file, the source file is right and the derived one has a bug. That's not a nice-to-have, it's basically the only thing keeping this from turning into three slightly-different copies of the truth after the next edit.

## Audit mode

Point it at code that exists — one file, a diff, a whole repo — and it walks all 293 entries against it, category by category, and writes up what it finds: severity, exact location, what's actually wrong (not the generic database description, the specific thing happening in *this* function), a pointer back to the source entry, real external reference links, and a fix that's actually a fix and not "add proper validation."

I'm not going to just claim it works, here's an actual finding from auditing the real SushiSwap MasterChef contract:

> **[V-001] Classic Reentrancy — Critical**
>
> `emergencyWithdraw()` is the clearest instance: line 276 calls `pool.lpToken.safeTransfer(address(msg.sender), user.amount)` — an external call into a token contract whose address was supplied by the (trusted) pool owner — and only *after* that call, on lines 278-279, does it zero `user.amount` and `user.rewardDebt`. Any LP token with a transfer hook... lets the receiving contract re-enter `emergencyWithdraw` while `user.amount` still reflects the pre-withdrawal balance, allowing the same staked balance to be paid out repeatedly.

That's a genuine bug, found by walking the checklist and then actually reading the function instead of pattern-matching. What I liked more, honestly, was what it *didn't* flag: `emergencyWithdraw()` forgoing pending rewards looks sketchy at first glance but it's the documented, intended behavior of an escape hatch — and the report says exactly that instead of either staying quiet about it or padding the finding count to look thorough.

That habit of writing down what got ruled out and why isn't just for show. A report that only lists problems looks identical whether it checked the other 280 entries or skipped straight to the obvious one. So the template forces a coverage note and a non-findings section every time — otherwise you can't tell diligence from luck.

## Coding mode

Same database, opposite direction. You're writing Solidity right now and want to not introduce one of these 293 bugs to begin with. Instead of loading the whole thing, it routes off what you're actually doing — a withdraw function pulls in Reentrancy plus External Calls, wiring up a price feed pulls in Oracle, touching an upgradeable contract pulls in Proxy & Upgradeability and Storage. Each category file is short — Don't/Do per entry, wording lifted straight from the database so nothing gets lost paraphrasing it, with a line number back to the full entry if you need the exact aliases or source citation.

There's also a short always-on list, 28 items, just the Critical and Critical/High stuff worth knowing by default no matter what you're touching. Don't leave a state write sitting after an external call. Don't ship `_authorizeUpgrade` without a guard on it. Don't let user input pick a `delegatecall` target. Don't price anything off raw AMM reserves in the same transaction you read them in. That kind of thing.

## How you actually invoke this from another project

Small confession here: this skill isn't wired into Claude Code's automatic discovery. The entry file is correctly named `SKILL.md` — that part follows the actual Agent Skills standard, same as every other skill on this machine, Claude Code's own or Cursor's — but it isn't sitting inside a location Claude Code's discovery scans (`~/.claude/skills/` for every project, or a specific project's own `.claude/skills/`). It's just a folder under `~/dev/`, so nothing scans it automatically. That means it won't fire on its own off a bare "check my contract" — you have to name it (or its path) explicitly, every time.

So you just say the path.

```
use the audit skill at /Users/giladhaimov/dev/Smart-Contract-AI-Audit-Skill to review this contract
```

```
consult the coding-mode checklist at /Users/giladhaimov/dev/Smart-Contract-AI-Audit-Skill before you finish this withdraw function
```

Works from any project, any session, no setup on the other end, because every internal reference in here is a relative path anchored to this folder — the entry point points at `modes/AUDIT_MODE.md`, that points at `../Smart-contract-vulnerability-database_v1.md`, and none of it cares what directory you were actually sitting in when you asked. I did set up the auto-discovery version at one point — symlinked it into `~/.claude/skills/` so it'd trigger globally — and then undid it, because saying the path once per conversation felt like a smaller cost than maintaining a symlink I'd forget existed. Your call if you want it back; it's a five-minute change.

## Why it's split into an index, checklists, and the full file instead of just being one big file

Fair question, and the honest answer is: mostly it doesn't matter, but sometimes it really does.

The full database is around 33-40k tokens. Every serious coding agent running in 2026 has at least a 128k window, most run 200k or more. So a single full read of the whole thing fits comfortably, with room left over — in the strictest sense none of the tiering below is *necessary*.

I built it anyway because two situations aren't hypothetical:

Not everyone's running the big model with the big window. Cheaper pipelines, smaller local models, agents someone capped at 8-32k tokens to save money — those genuinely can't take a 40k-token read in one go, but they can work fine off the ~7k-token index plus one or two ~1-2k-token category files. The tiering is what makes the same skill usable on both ends of that range instead of only the expensive one.

And even on a big model, a real audit session isn't just the database — it's the database plus the contract plus the running list of findings plus everything else in the conversation, all at once, for as long as the review takes. Rereading the same 40k tokens fresh for every one of ten contracts in a repo is wasteful even when it technically fits every time.

This isn't me theorizing, either — it's the exact reason I ran the seven-contract validation as seven parallel agents instead of one long session. Each one's context stayed scoped to its own contract, and the whole thing finished in the time of roughly one audit instead of seven stacked end to end.

## Does it actually work, though

I didn't want to just trust my own design here, so I fetched seven real contracts off GitHub — a mix of heavily-audited production code and contracts that are *deliberately* broken as teaching tools, specifically so I'd have a way to check whether the audit finds the real bug or just produces something that sounds plausible.

| Contract | Type | Critical | High | Medium | Low | Total |
|---|---|---|---|---|---|---|
| WETH9 | production | 0 | 0 | 4 | 3 | 7 |
| OpenZeppelin ERC20 (current) | production | 0 | 0 | 0 | 1 | 1 |
| Uniswap V2 Pair | production | 0 | 1 | 0 | 2 (+1 info) | 4 |
| SushiSwap MasterChef | production | 1 | 3 | 3 | 4 | 11 |
| not-so-smart-contracts Reentrancy | intentionally broken | 1 | 0 | 2 | 3 | 6 |
| Ethernaut "Reentrance" | intentionally broken | 1 | 1 | 2 | 0 | 4 |
| Ethernaut "Delegation" | intentionally broken | 1 | 0 | 0 | 2 | 3 |

The raw counts aren't really the interesting part. What I actually cared about: the two most battle-tested production files (current OpenZeppelin ERC20, canonical WETH9) came back with zero Critical or High findings — not because it went easy on them, that's genuinely the state of that code, and it says so instead of inventing findings to look busy. All three intentionally-broken contracts had their real bug caught with the right line numbers, including a walkthrough of the storage-collision bug in Ethernaut's "Delegation" level that actually explains the mechanism for that specific code (both contracts happen to store `owner` at slot 0, so a delegatecall-executed write from one lands in the other) rather than reciting the general concept. And the one production contract that came back Critical, MasterChef, is flagged for something that's really there.

I checked that last part myself, by hand — read the actual source, matched every line number and quoted snippet the reports cited against it. All of it checked out, including a code comment quoted verbatim (`// XXX DO NOT add the same LP token more than once`) that had no business being right unless the report writer actually opened the file.

## Where this falls short, because it does

It's only as good as the reading, not just the checklist. Anything with a clean, mechanical Detection line gets caught reliably. The stuff that needs real judgment — most of Logic Error, most of Accounting & Fees — only gets caught if whoever's running this actually reads the function instead of grepping for a keyword. The workflow pushes toward that, but it can't force it.

It doesn't run Slither, or solc, or anything else for you. A bunch of the Detection lines literally name a Slither detector to run — that's a "go run this" instruction, not something the skill does on its own. Pair it with real tooling before you ship anything.

It won't trigger itself. Covered this above, it's a deliberate trade for a readable filename, not an oversight, but worth saying twice so nobody's surprised when "check this contract" does nothing on its own.

It's a snapshot. New bug classes get found all the time; this reflects what was known when I built it. It doesn't pull anything live from Solodit or the SWC Registry — keeping it current is on whoever maintains it, same as any reference doc.

## Adding to it later

New entry goes into `Smart-contract-vulnerability-database_v1.md` in the same format everything else uses — Aliases, Category, Severity, Description, Detection, Sources — with the next ID in sequence, and then `reference/INDEX.md` and the relevant category file under `modes/coding/` need regenerating so they stay in sync. If you're ever unsure whether to fix something in the big file or one of the derived ones: fix the big file, regenerate the rest. Never the other way.

## Where the source material came from

| Source | URL | Contributed |
|---|---|---|
| Solodit (Cyfrin) | https://solodit.cyfrin.io | the biggest curated set of real audit findings, plus their aggregated checklist |
| SWC Registry + ConsenSys/SCSFG | https://swcregistry.io · https://consensys.github.io/smart-contract-best-practices · https://scsfg.io | SWC-100 through SWC-136, plus the long-standing community best practices |
| DeFiVulnLabs (SunWeb3Sec) | https://github.com/SunWeb3Sec/DeFiVulnLabs | PoC-backed DeFi bugs, each reproducible in Foundry |
| Official Solidity docs | https://docs.soliditylang.org | the full known-compiler-bugs list and the official security-considerations writeup |

The seven contracts used for the validation run came from `gnosis/canonical-weth`, `OpenZeppelin/openzeppelin-contracts`, `Uniswap/v2-core`, `sushiswap/masterchef`, `crytic/not-so-smart-contracts`, and `OpenZeppelin/ethernaut` — license and attribution details for each are in `test-contracts/SOURCES.md`.

---

Built and stress-tested in one sitting: assembled the database, split it into the two modes, ran it against seven real contracts to see if it actually holds up, and decided (deliberately, not by accident) to favor an explicit path over automatic discovery. If any of that changes later, this is the file to update first.

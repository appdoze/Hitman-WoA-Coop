# D014: Licence

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-11-07 |
| **Who decided** | @appdoze |
| **Roadmap item** | [Part 1: Licence](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions) |
| **Issue or PR** | – |

## Question

Which licence should the mod, Peacock plugin, relay and tools use?

## Options

1. **AGPL-3.0:** the same as Peacock. Compatible with ZHMModSDK's GPL-3.0-or-later. Anyone who hosts a modified relay or Peacock for others must publish their source.
2. **GPL-3.0:** the same as ZHMModSDK. Code that ends up inside a Peacock fork would still need to be AGPL-3.0, so the project would have two licences.
3. **A permissive licence (MIT):** not possible for code that links ZHMModSDK or becomes part of Peacock, both of which are copyleft.

## Decision

AGPL-3.0 for the whole repository.

## Why

- One licence that is compatible with both upstream projects
- Server-side code (relay, Peacock plugin) is covered when someone hosts it for others, not only when they distribute it
- Matches Peacock, which makes contributing changes upstream simpler

## Consequences

- What this makes easier: moving code between this repo and a Peacock fork, and contributing upstream
- What this makes harder or rules out: using libraries whose licence isn't compatible with AGPL-3.0. Check every new dependency.
- Roadmap items, risks or estimates that change: none
- When to revisit this: if ZHMModSDK or Peacock change licence

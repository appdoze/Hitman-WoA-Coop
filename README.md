# Hitman WoA Co-op

A planned co-op and multiplayer mod for **HITMAN World of Assassination**, built on [ZHMModSDK](https://github.com/OrfeasZ/ZHMModSDK) (game-side hooks) and [Peacock](https://github.com/thepeacockproject/Peacock) (server for lobbies, scoring and progression).

> **Status:** planning and research. Nothing is playable yet. See the [roadmap](WOA_COOP_ROADMAP.md).

> **Access:** this repository is private for now while the first research spikes run. If you'd like to help with research, ask the maintainer (appdoze) for access. It will go public once there are spike results to share.

## What to expect

How much co-op is possible depends on research that hasn't been done yet. At the [Month 3 decision gate](WOA_COOP_ROADMAP.md#decision-gate-end-of-month-3), the project picks one of three [design tiers](WOA_COOP_ROADMAP.md#part-3-design-tiers):

- **Ghost co-op:** you each play the same contract in your own world, see your partner, and share target kills and a combined score
- **Shared world:** one world for both players, but guards may not react to the partner
- **Full co-op:** one world, and guards react to both players

Plan for the first, hope for the last.

## How it will work

- **ZHMModSDK mod (C++):** syncs players, NPCs and world state between the two games over a real-time network connection. The host's game controls the world; the client follows.
- **Peacock plugin (TypeScript):** co-op lobbies, contract selection, and scoring and progression for both players. If Peacock's plugin hooks aren't enough, this becomes a Peacock fork with the co-op code kept as separate as possible.

## Requirements (planned)

- A legitimate copy of HITMAN World of Assassination on Steam or Epic for **each** player
- [ZHMModSDK](https://github.com/OrfeasZ/ZHMModSDK) installed on each PC
- Peacock
- The same game build, mod version and installed content mods on both PCs
- Both players must own the location being played
- Playing together across Steam and Epic: not decided yet ([roadmap Part 1](WOA_COOP_ROADMAP.md#part-1-architecture-decisions))

## Privacy

Co-op connects two players' games directly or through a relay.

- A **direct connection** reveals each player's IP address to the other. The mod will ask before connecting directly.
- A **relay** is the default. It hides your IP address from your partner, but whoever runs the relay can see both players' IP addresses.
- The mod will never open router ports without asking.

## Your progress

Co-op is experimental. Back up your Peacock `userdata` folder before playing. The mod will also back up your profile automatically before each co-op mission.

## Contributing and security

- How to help: [CONTRIBUTING.md](CONTRIBUTING.md)
- Reporting security problems: [SECURITY.md](SECURITY.md)
- Research results: [docs/spikes](docs/spikes/)

## Licence

[AGPL-3.0](LICENSE). Peacock is AGPL-3.0 and ZHMModSDK is GPL-3.0 or later, and both are compatible with this licence.

This project isn't affiliated with or endorsed by IO Interactive. HITMAN and World of Assassination are trademarks of IO Interactive. No game files are included or distributed.

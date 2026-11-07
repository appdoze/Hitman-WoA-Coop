# Hitman WoA Co-op

A planned co-op and multiplayer mod for **HITMAN World of Assassination**, built on [ZHMModSDK](https://github.com/OrfeasZ/ZHMModSDK) (game-side hooks) and [Peacock](https://github.com/thepeacockproject/Peacock) (server for lobbies, scoring and progression).

> **Status:** planning and research. Nothing is playable yet. See the [roadmap](WOA_COOP_ROADMAP.md).

> **Help wanted:** the first research spikes haven't started yet. If you know Glacier 2, ZHMModSDK or Peacock, see [CONTRIBUTING.md](CONTRIBUTING.md) and the [spike list](docs/spikes/).

## What to expect

How much co-op is possible depends on research that hasn't been done yet. At the [Month 3 decision gate](WOA_COOP_ROADMAP.md#decision-gate-end-of-month-3), the project picks one of three [design tiers](WOA_COOP_ROADMAP.md#part-3-design-tiers):

- **Ghost co-op:** you each play the same contract in your own world, see your partner, and share target kills and a combined score
- **Shared world:** one world for both players, but guards may not react to the partner
- **Full co-op:** one world, and guards react to both players

Plan for the first, hope for the last.

## How it will work

- **ZHMModSDK mod (C++):** syncs players, NPCs and world state between the two games over a real-time network connection. The host's game controls the world; the client follows.
- **Peacock plugin (TypeScript):** co-op lobbies, contract selection, and scoring and progression for both players. If Peacock's plugin hooks aren't enough, this becomes a Peacock fork with the co-op code kept as separate as possible.

## Repository layout

Planned. Folders are created when code for them starts.

| Folder | Contents | Language |
|---|---|---|
| `mod/` | The ZHMModSDK mod: hooks, puppet, sync, network client | C++ (CMake) |
| `peacock-plugin/` | Co-op lobbies, contract selection, scoring and progression for Peacock | TypeScript |
| `relay/` | The relay server that forwards traffic between two players | To be decided |
| `protocol/` | The network message schema, shared by the mod, relay and tools. Drafted in [docs/protocol.md](docs/protocol.md) until D008 is decided. | Schema files |
| `fuzz/` | Fuzz tests for the message parser (from Month 4, R17). Seed inputs are committed; generated crashes aren't. | C++ (libFuzzer) |
| `tools/` | Echo mode helpers, fake client and fake host, message replay, profile backup script | To be decided |
| `docs/` | [Spike reports](docs/spikes/), [decision records](docs/decisions/) and the [glossary](docs/GLOSSARY.md) | Markdown |

## Target versions

The versions the project is built and tested against. Filled in by [#7](https://github.com/appdoze/Hitman-WoA-Coop/issues/7) and [#10](https://github.com/appdoze/Hitman-WoA-Coop/issues/10), and updated whenever one changes.

| Component | Version | Notes |
|---|---|---|
| HITMAN World of Assassination (Steam) | To be recorded | Exact build number from the game |
| HITMAN World of Assassination (Epic) | To be recorded | Exact build number from the game |
| ZHMModSDK | To be recorded | Release or commit |
| Peacock | To be recorded | Release or commit |
| Simple Mod Framework | To be recorded | Only if content mods are tested |

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
- ZHMModSDK, which the mod runs on, can send crash reports to the SDK author's Sentry service if you opt in through the SDK. That's separate from this mod's own crash dumps, which stay on your PC unless you share them.

## Your progress

Co-op is experimental. Back up your Peacock `userdata` folder before playing. The mod will also back up your profile automatically before each co-op mission.

## Contributing and security

- How to help: [CONTRIBUTING.md](CONTRIBUTING.md)
- Reporting security problems: [SECURITY.md](SECURITY.md)
- Research results: [docs/spikes](docs/spikes/)
- Decisions and why they were made: [docs/decisions](docs/decisions/)
- Terms used in this project: [glossary](docs/GLOSSARY.md)
- Network message draft: [docs/protocol.md](docs/protocol.md)
- Code of conduct: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

## Licence

[AGPL-3.0](LICENSE). Peacock is AGPL-3.0 and ZHMModSDK is GPL-3.0 or later, and both are compatible with this licence.

This project isn't affiliated with or endorsed by IO Interactive. HITMAN and World of Assassination are trademarks of IO Interactive. No game files are included or distributed.

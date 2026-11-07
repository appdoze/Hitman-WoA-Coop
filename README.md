# Hitman WoA Co-op

A planned co-op and multiplayer mod for **HITMAN World of Assassination**, built on [ZHMModSDK](https://github.com/OrfeasZ/ZHMModSDK) (game-side hooks) and [Peacock](https://github.com/thepeacockproject/Peacock) (server for lobbies, scoring and progression).

> **Status:** planning. Nothing is playable yet. See the [roadmap](WOA_COOP_ROADMAP.md).

## How it will work

- **ZHMModSDK mod (C++):** syncs players, NPCs and world state between the two games over a real-time network connection. The host's game controls the world; the client follows.
- **Peacock plugin (TypeScript):** co-op lobbies, contract selection, and scoring and progression for both players.

## Requirements (planned)

- A legitimate copy of HITMAN World of Assassination on Steam or Epic for **each** player
- Peacock
- The same game build, mod version and installed content mods on both PCs
- Both players must own the location being played

## Privacy

Co-op connects two players' games directly or through a relay. A direct connection reveals each player's IP address to the other, so the mod will use a relay by default and ask before connecting directly. It will never open router ports without asking.

## Your progress

Co-op is experimental. Back up your Peacock `userdata` folder before playing. The mod will also back up your profile automatically before each co-op mission.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[AGPL-3.0](LICENSE). Peacock is AGPL-3.0 and ZHMModSDK is GPL-3.0 or later, and both are compatible with this licence.

This project isn't affiliated with or endorsed by IO Interactive. HITMAN and World of Assassination are trademarks of IO Interactive. No game files are included or distributed.

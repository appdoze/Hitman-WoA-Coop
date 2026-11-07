# Glossary

Terms used in the [roadmap](../WOA_COOP_ROADMAP.md) and the rest of this repository.

## The game

| Term | Meaning |
|---|---|
| **WoA** | HITMAN World of Assassination: HITMAN 3 with the HITMAN 2016 and HITMAN 2 locations included. The only version this mod targets. |
| **H1, H2, H3** | HITMAN (2016), HITMAN 2 and HITMAN 3. Peacock also calls them `h1`, `h2` and `h3`. |
| **Glacier 2** | IO Interactive's game engine. |
| **Entity** | Any object in a level: an NPC, a door, an item, a trigger, a camera. Each has an ID within its level data. |
| **Brick** | A chunk of level data that adds entities to a scene, for example the entities for one mission story. Different missions on the same map load different bricks. |
| **RPKG** | The game's resource package files. Never commit these. |
| **Contract** | A playable mission setup: map, targets, conditions. Missions, Featured Contracts, Escalations and Elusive Targets are all contracts. |
| **Mission story** | A guided route to a target (called "opportunities" in older games), usually with scripted events and cutscenes. |
| **Silent Assassin (SA)** | The top rating. In Peacock (`calculateScore` in `components/scoreHandler.ts`) all of these must hold: no non-target kills (including crowd NPCs), never spotted, no kills noticed, no bodies found, and not caught on a security camera (or the recording destroyed). Objectives don't count towards SA. Spotting, noticed kills and found bodies are forgiven only if that witness is later killed, and killing them is itself a non-target kill unless they're a target. One exception: a *murdered* body seen by a non-target loses SA for good (`MurderedBodySeen` in `components/eventHandler.ts`), but bodies found after an accident don't. Easy to break by mistake in co-op (see Spike 6). |
| **Enforcer** | An NPC who can see through a disguise of their own type. |
| **Ghost Mode** | HITMAN 2's 1v1 mode where both players race to kill the same targets in separate copies of the world. |
| **Sniper Assassin** | Sniper-only maps from HITMAN 2, which had a co-op mode. |
| **Freelancer** | WoA's roguelike mode with Syndicates and its own currency, Merces. |

## Tools and projects

| Term | Meaning |
|---|---|
| **Peacock** | A replacement for IOI's online servers. Handles profiles, contracts, challenges, scoring and menus, but not what happens inside a running mission. |
| **ZHMModSDK** | A C++ modding framework that loads into the game and lets mods hook its functions. The co-op mod is built on it. |
| **Hook** | Code that runs when a game function is called, to read or change what it does. |
| **SMF** | Simple Mod Framework: installs content mods that change game files. |
| **GlacierKit, QuickEntity** | Tools for reading and editing level entity data. See roadmap Part 0. |
| **Pattern scan** | Finding a game function by a byte pattern instead of a fixed address, so hooks survive small game updates. |

## Networking and sync

| Term | Meaning |
|---|---|
| **Host** | The player whose game controls the shared world: NPCs, AI, physics and story state. |
| **Client** | The other player. Their game sends actions to the host and shows the host's results. |
| **Puppet** | The partner's character as shown in your game, moved by network messages rather than by the game's player controller. |
| **Sync** | Making the same thing happen in both games. |
| **Desync, out-of-sync** | When the two games disagree about the world, for example an NPC alive in one and dead in the other. |
| **Checksum** | A short fingerprint of key game state. The host sends it regularly; if the client's doesn't match, something has desynced. |
| **Authority** | Which game's version of something is final. In this design, the host has authority over the world. |
| **Reliable / best-effort channel** | Messages that must arrive (kills, doors) versus messages where only the latest matters (positions). |
| **Smoothing (interpolation)** | Filling in movement between network updates so the puppet doesn't jerk. |
| **Relay** | A small server that forwards traffic between two players so they don't see each other's IP address. |
| **P2P** | Peer-to-peer: the two games connect directly. |
| **Handshake** | The first messages when players connect, which check that game build, mod version, protocol version and installed mods match. |
| **Protocol version** | A number for the network message format. Players with different protocol versions can't connect. |
| **Fuzzing** | Feeding a program large amounts of random or broken input to find crashes. |

## This project

| Term | Meaning |
|---|---|
| **Spike** | A short, throwaway experiment that answers one risky question with pass or fail. See roadmap Part 2 and [docs/spikes](spikes/). |
| **Decision gate** | The end of Month 3, where spike results decide the design tier. |
| **Tier 1, 2, 3** | Ghost co-op, shared world, full co-op. See roadmap Part 3. |
| **Echo mode** | A debug mode that sends your own movements back to you as a puppet, for testing on one PC. |
| **Fake client / fake host** | Standalone programs that pretend to be the other player, for testing without a second person. |
| **Message log** | A timestamped record of every network message, used for debugging and replay tests. |
| **Decision record (DNNN)** | A file in [docs/decisions](decisions/) recording what was chosen for a Part 1 decision and why. |
| **R1, R2, ...** | Risk IDs in roadmap Part 7. |

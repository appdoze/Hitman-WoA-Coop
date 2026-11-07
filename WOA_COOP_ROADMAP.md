# Hitman WoA Co-op Mod (ZHMModSDK + Peacock)

New to the terms used here? See the [glossary](docs/GLOSSARY.md). Work is tracked in [issues](https://github.com/appdoze/Hitman-WoA-Coop/issues) and [milestones](https://github.com/appdoze/Hitman-WoA-Coop/milestones), one milestone per month in Part 4.

- [How this roadmap works](#how-this-roadmap-works)
- [Part 0: Existing tools, prior work and contacts](#part-0-existing-tools-prior-work-and-contacts)
- [Part 1: Architecture decisions](#part-1-architecture-decisions)
- [Part 2: Risk spikes (do these first)](#part-2-risk-spikes-do-these-first)
- [Part 3: Design tiers](#part-3-design-tiers)
- [Part 4: 12-month roadmap](#part-4-12-month-roadmap)
- [Part 5: Per-map checklist](#part-5-per-map-checklist)
- [Part 6: Year 2 and beyond](#part-6-year-2-and-beyond)
- [Part 7: Risk register](#part-7-risk-register)
- [Part 8: Definition of done and metrics](#part-8-definition-of-done-and-metrics)
- [Part 9: Solo development setup](#part-9-solo-development-setup)

---

## How this roadmap works

- **Risk first.** The parts that might be impossible come before the parts that are only tedious. Menus, Peacock and game modes are low risk, so they come late.
- **One map at a time.** One map fully working is worth more than ten maps half working. Each new map gets the per-map checklist in Part 5.
- **Decision gates.** At the end of Month 3 you pick a design tier (Part 3) based on what the tests actually showed, and the rest of the plan follows that tier.
- **Sizing.** These estimates assume one to three people working part time. With more people or full-time work, compress the timeline. Don't add scope.

### Where the work actually is

| Area | Tool | Share of effort |
|---|---|---|
| Live game state: players, NPCs, AI, world, physics | ZHMModSDK (C++ hooks into Glacier 2) | ~80% |
| Real-time networking | ENet, GameNetworkingSockets or Steam P2P | ~10% |
| Lobbies, contracts, scoring, progression, menus | Peacock (Node/TypeScript server) | ~10% |

Peacock never touches what happens inside a running mission. It only sees the events each game sends it and serves menu and contract data back.

---

## Part 0: Existing tools, prior work and contacts

Don't build what already exists. Check these in Month 1.

### Tools

| Tool | Use for this mod | Licence |
|---|---|---|
| [ZHMModSDK](https://github.com/OrfeasZ/ZHMModSDK) | The mod itself: C++ hooks into the game | GPL-3.0+ |
| ZHMModSDK built-in mods (`Mods/` folder) | Ready-made reference code for almost every spike. See [ZHMModSDK built-in mods](#zhmmodsdk-built-in-mods) below. | GPL-3.0+ |
| [Peacock](https://github.com/thepeacockproject/Peacock) | Server: lobbies, scoring, progression | AGPL-3.0 |
| [Simple Mod Framework](https://github.com/atampy25/simple-mod-framework) | Installing content changes; what most players use to install mods | LGPL-3.0 |
| [GlacierKit](https://github.com/atampy25/glacierkit) | Browsing level entities and bricks offline (Spike 2, per-map work) | LGPL-3.0 |
| [QuickEntity](https://github.com/atampy25/quickentity-rs) | Reading and diffing entity data as JSON | LGPL-3.0 |
| [RPKG Tool](https://github.com/glacier-modding/RPKG-Tool) | Extracting game resources for research | See repo |
| [Hitman Hashes](https://github.com/glacier-modding/Hitman-Hashes) | Resource and hash name lists | MIT |

- [x] Read through the ZHMModSDK `Mods/` folder before writing hooks from scratch. Done for all 22 mods at SDK commit `d98355d` (2026-11-02): see the table below.
- [ ] Try `Editor` on Paris and note how NPCs, doors and items are identified (input to Spike 2)
- [x] Check whether `NoPause` already stops the game pausing when its window loses focus. **Yes:** it forces the game's `PauseOnFocusLoss` option off and stops the window minimising on focus loss. Reuse it for Part 9 Method 4. The pause menu itself (R6) still needs its own hook.
- [ ] Try `OnlineTools` with a Peacock running on another PC (Part 1 option (a))

### ZHMModSDK built-in mods

What each built-in mod offers co-op, checked at SDK commit `d98355d` (2026-11-02). Reuse the approaches; copy code only where it meets the message rules in [docs/protocol.md](docs/protocol.md).

| Mod | What it gives co-op | Where |
|---|---|---|
| **`Hitmen`** | **A two-player prototype by the SDK's author.** Adds a second, fully working 47 ("Agent48", the game's `agent47_default` template) through a small SMF brick (`[assembly:/_sdk/hitmen.brick]`), and drives him by writing the other player's input state into his `ZCharacterInputProcessor`, so his animations come from the game. Sends NPC positions from host to client by `ActorManager` slot number. Uses GameNetworkingSockets. **Dormant:** commented out of the SDK build, its GameNetworkingSockets dependency is commented out too, and its network code is commented out. The author's comments call it "extremely incredibly unsafe": it copies raw memory straight off the network, which R17 forbids. Also ships `multiplayer_hitmen.entity.json`, HITMAN 2's Sniper Assassin co-op level data. | Spikes 1, 4, 5; D008 |
| **`Editor`** | A local JSON API over WebSocket on `127.0.0.1:46735` (`Mods/Editor/editor.d.ts`): list entities, get details, set properties and transforms, trigger entity pins (`signalEntityPin`), spawn entities, and "property changed" events. Identifies game entities by **(blueprint (TBLU) hash, entity ID)**. Scripts can drive it, so many spike questions can be tried before writing C++. Accepts connections from the same PC only: never expose it to the network. Also hooks `SignalInputPin`, `SignalOutputPin` and `SetPropertyValue`, the general way world state changes. | Spikes 2, 4; Part 9 tools |
| **`AdvancedRating`** | Hooks `ZAchievementManagerSimple_OnEventSent`, which receives each game event as JSON (name and value), and reads the game's own state flags in `ZGameStatsManager` (trespassing spotted, body found, dead body seen). Events reach Peacock in batches (`SaveAndSynchronizeEvents4` takes an array), not one by one, so what this hook sees may not have reached Peacock yet. | Spikes 3, 6 |
| **`Player`** | Hooks for weapon firing, getting hit (`ZActor_YouGotHit`) and security cameras. | Month 4 firing, Month 7 cameras |
| **`Player`, `Randomizer`, `Assets`, `Outfits`** | `ZHitman5_SetOutfit` (`Player`, `Randomizer`) and `ZActor_SetOutfit` (`Randomizer`, `Assets`) change outfits; item creation and item spawners (`ZItemSpawner`). `Outfits` doesn't change outfits itself: it loads a scene's outfit bricks so outfits from other maps can be used. | Spike 1 outfits, Month 4 disguises and items |
| **`WakingUpNpcs`, `FreeCam`, `Editor`** | `ZActor_ReviveActor` and `ZActor_KillActor`. | Spike 4, Month 5 kills, knockouts, waking |
| **`DebugMod`** | Inspects NPCs, AI behaviour and pathfinding (`ActorManager`, `BehaviorService`, `Pathfinder`). | Spikes 3, 4 |
| **`QuickSave`** | Hooks `ZSaveSlotManager_QuickSave`. | Month 9: saves off in co-op |
| **`Clumsy`** | Ragdoll functions (not related to the clumsy network tool in Part 9). | Body physics sync |
| **`World`** | Changes the game-speed multiplier (`GameTimeManager`). Co-op must lock or sync it so one player can't slow time. | D010 |
| **`FreelancerSeeder`** | Hooks Freelancer campaign generation, so both players could share one seed. | Year 2 Freelancer co-op |
| **`NoPause`, `OnlineTools`** | See the checkboxes above. | Part 9, Part 1 |
| `Noclip`, `TitaniumBullets`, `SkipIntro`, `MaxPatchLevel`, `DiscordRichPresence`, `DebugCheckKeyEntityEnabler` | Little or no use for co-op. | – |

**SDK crash reporting:** the SDK can optionally send crash reports to the SDK author's Sentry project, if the player opts in. Crashes in this mod would go there too, so keep our own local crash dumps (#13) and mention this in the privacy notes.

### Community mods (Nexus Mods)

Checked on 2026-11-07 by searching [Nexus Mods](https://www.nexusmods.com/games/hitman3/mods) for multiplayer, co-op, ghost, sniper, NPC, spawn, AI, Peacock, online, editor, SDK and scoring. Most of the 966 mods are visual or gameplay tweaks. Only the mod pages were read; no mod files were downloaded or inspected.

| Mod | What it shows | Where |
|---|---|---|
| [Ghost Mode](https://www.nexusmods.com/hitman3/mods/260) (MrMike and Anthony Fuller, updated Dec 2025) | **WoA's own multiplayer still works.** Ports HITMAN 2's Ghost Mode to WoA: two players in separate worlds, connected through Steam invites and a community server (LocalGhost or Peacock). **Steam only, no Epic.** Known issues: occasional crashes, and the opponent appears as a black silhouette. | Spike 5, Tier 1, D017 |
| [Sniper Co-op](https://www.nexusmods.com/hitman3/mods/444) (Jan 2023) | **Two players in one shared world** on the Sniper Assassin maps, in WoA. Needs Peacock, Steam only, both players on the same Peacock server (the host may need open ports), and the Ghost Mode mod to avoid a crash. | Spike 5, Spike 4, Tier 2 |
| [Ghost Mode - Haven Island](https://www.nexusmods.com/hitman3/mods/890) | Adds a new map to Ghost Mode through content changes and a modified server, so the game's own multiplayer isn't locked to its original maps. | Spike 5 |
| [Ghost Mode Emotes](https://www.nexusmods.com/hitman3/mods/558) | Restores HITMAN 2's Ghost Mode emote menu: a ready-made way for partners to signal each other. | Month 10 quick pings |
| [LocalGhost](https://gitlab.com/grappigegovert/localghost) (GitLab, zlib licence) | The open-source server Ghost Mode was first played on after IOI's servers closed, and where Peacock started. | Spike 5, Peacock side |
| [AI Sight Modifications](https://www.nexusmods.com/hitman3/mods/645), [Blind NPCs](https://www.nexusmods.com/hitman3/mods/363), [AI Playground](https://www.nexusmods.com/hitman3/mods/364) (succeeded by "Reality Control"), [All NPCs Follow Hitman](https://www.nexusmods.com/hitman3/mods/271) | Change NPC and camera vision and behaviour through level data alone, which shows where perception is set up. | Spike 3 |
| [Offline Scoring](https://www.nexusmods.com/hitman3/mods/68) | Custom scoring, including tracking of security camera recordings. | Spike 6 |

### Prior work
- [ ] Search the Peacock and Glacier modding Discords, GitHub and Nexus Mods for earlier co-op or multiplayer attempts on WoA, HITMAN 2 or HITMAN 2016

  Where to search, checked on 2026-11-07 (Nexus Mods is covered in [Community mods](#community-mods-nexus-mods)):

  | Site | What's there |
  |---|---|
  | [Glacier Modding](https://glaciermodding.org) and its [Discord](https://discord.gg/6UDtuYhZP6) | The main Glacier 2 modding hub: guides (`/docs`), featured mods, tool pages. The Discord is where the ZHMModSDK, Simple Mod Framework and GlacierKit authors are: **the best place to ask about earlier co-op attempts.** |
  | [HitmanDB](https://hitmandb.glaciermodding.org) | Searches game file hashes (useful for Spike 2). |
  | [hitman-resources.netlify.app](https://hitman-resources.netlify.app) and [TonyTools](https://tonytools.win) | HITMAN 3 modding resources, and docs for another set of Glacier tools. |
  | [The Peacock Project](https://thepeacockproject.org) | Peacock's site; its Discord is linked from the Peacock readme. |
  | [HITMAPS](https://www.hitmaps.com) | Maps and Roulette; runs the public Ghost Mode server `gm.hitmaps.com` and the [hitman-server-patcher](https://gitlab.com/hitmaps/hitman-server-patcher). |
  | [Hitman Forum](https://www.hitmanforum.com) | The long-running fan forum, with modding threads. |
  | GitHub: [glacier-modding](https://github.com/glacier-modding), [atampy25](https://github.com/atampy25), [OrfeasZ](https://github.com/OrfeasZ); GitLab: [LocalGhost](https://gitlab.com/grappigegovert/localghost) | Source for the tools and servers above. |
  | [GameBanana](https://gamebanana.com/games/8925) and [ModDB](https://www.moddb.com/games/hitman-3) | Almost nothing: GameBanana has 3 HITMAN 3 items, ModDB has 1 HITMAN 3 mod and 3 downloads. Not worth searching further. |

  There's no Steam Workshop for HITMAN. Discord can't be searched from outside, so ask there directly.
- [ ] For any you find, write down how far they got, what blocked them and whether their code or notes are available
- [x] **Found:** ZHMModSDK's own `Hitmen` mod, a dormant two-player prototype by the SDK's author (see [ZHMModSDK built-in mods](#zhmmodsdk-built-in-mods)). Ask the author how far it got and why it stopped (Contacts below).
- [x] Check the Ghost Mode and Sniper Assassin co-op research others have already done (input to Spike 5). **Found:** both run in WoA on Steam today through community mods and Peacock or LocalGhost. See [Community mods](#community-mods-nexus-mods).

### Contacts (before Month 4)
- [ ] Introduce the project to the Peacock developers (Discord, linked from the Peacock readme). Ask about plans for `RegisterToPreset`, multiplayer and plugin hooks, and whether co-op changes could go upstream.
- [ ] Introduce the project to the ZHMModSDK developer. Ask whether hooks you need (player spawning, AI perception, entity IDs) already exist or would be accepted upstream, and how far the `Hitmen` prototype got and why it stopped.
- [ ] Contact the authors of the Ghost Mode port (MrMike and Anthony Fuller) and of Sniper Co-op. Ask how the game's own multiplayer is connected (Steam networking, RakNet), what causes the crashes, whether it could carry a normal mission, and whether they'd share notes. **Do this in Month 1: it shapes Spike 5.**
- [ ] Share Spike results with both communities. Upstream hooks are maintained by others after game patches, which lowers R7.

---

## Part 1: Architecture decisions

Settle these before writing feature code. Record each answer in [docs/decisions](docs/decisions/) (one file per decision, numbered D001–D018 in the order below), then tick it here. Once a decision file exists, it is the source of truth for that decision, not this list or its issue.

- [ ] **Network model:** the host machine controls the world (NPCs, AI, physics, story state). The client sends its inputs and actions to the host and shows the host's results.
- [ ] **Two network channels:**
  - Real-time channel (P2P library) for positions, animations and world events.
  - Peacock for lobbies, contract selection and end-of-mission scoring. Peacock's push messages only reach the game when it sends its regular event updates, which takes seconds, so never use them for gameplay.
- [ ] **Peacock server setup.** Most players run Peacock on their own PC. Choose one:
  - (a) Both games connect to one shared Peacock run by the host. Simplest, but the host needs a reachable port, or you need a relay.
  - (b) Each player keeps a local Peacock, and the mod syncs only the match and results data between them.
- [ ] **Player identity:** two separate Peacock profiles, each with its own `unique_name`. Decide where the partner's profile ID comes from.
- [ ] **Event authority for scoring:** each game reports events from its own point of view. When the partner kills someone, the host's game sees an NPC dying to an unknown cause, which can break Silent Assassin, challenges and ratings. Decide:
  - Whose event stream Peacock trusts for which events
  - How you tag events with the player who caused them
  - How you remove duplicate events reported by both games
- [ ] **Version locking:** the mod checks the game build and refuses to run on a mismatch. Both players must run the same mod version, so check it in the handshake.
- [ ] **Protocol versioning:** a network protocol version number, separate from the mod version. Bump it whenever the message format changes, and reject mismatches in the handshake with a clear error.
- [ ] **Message format:** pick a compact binary schema (for example FlatBuffers, Protobuf or a hand-written versioned format). Never send raw C++ structs or pointers.
- [ ] **Randomness:** find out where the game uses random numbers (NPC routines, reactions, loot, crowds). Decide whether to sync the seed at mission start or send every random outcome from the host.
- [ ] **Clock and timing:** a shared mission clock (host time plus measured offset) for timestamping events, smoothing movement and timing the mission.
- [ ] **Pause behaviour:** the game must not pause in co-op. Opening the pause menu, map or inventory only covers the local screen. Decide what happens to saves, loading screens and the photo mode.
- [ ] **Difficulty and settings:** both players use the host's difficulty, mission settings and starting conditions. Starting location and loadout can differ for each player.
- [ ] **Crash recovery:** if either game crashes, the mission ends cleanly for the other player, and nobody loses progress already confirmed in Peacock.
- [x] **Licence:** *(Decided: [D014](docs/decisions/D014-licence.md).)* Peacock is AGPL-3.0 and ZHMModSDK is GPL-3.0+. A fork you release, or host for other people, must publish its source. This repo uses AGPL-3.0, which is compatible with both. Keep co-op server code in a separate plugin or module where possible, to make rebasing on Peacock updates easier.
- [ ] **Installed mods must match:** content mods (Simple Mod Framework) can change levels, NPCs and items, which breaks entity IDs and sync. Hash the list of installed mods and their versions, and compare it in the handshake. Decide whether to block mismatches or only warn.
- [ ] **DLC and location ownership:** both players must own the location being played. Use the entitlements Peacock already reads at login (`components/oauthToken.ts`, with the platform entitlement lists in `components/platformEntitlements.ts`; `components/ownership.ts` maps DLC packs to unlockable items) to grey out locations that either player doesn't own. Under server option (b), each player's Peacock only knows that player's entitlements, so the two players must exchange their owned locations, for example in the handshake.
- [ ] **Steam and Epic together:** decide whether a Steam player and an Epic player can play together. Anything tied to one platform (Steam P2P, Steam Datagram Relay, Steam invites) only works if both players are on that platform.
- [ ] **Network privacy:** direct P2P connections reveal each player's IP address to the other. Use a small self-hosted relay as the default, since it works for both Steam and Epic players. Steam Datagram Relay (through GameNetworkingSockets) is an option for Steam-only lobbies, but only once you've confirmed a mod can use it: it runs on Valve's network and may need Steamworks setup tied to the game's own app. Never open router ports (UPnP) without asking the player first, and show a warning before a direct connection. The relay must only forward traffic between two peers in the same match, never act as an open proxy (R18), and whoever runs it can see both players' IP addresses, so say so in the privacy notice.

---

## Part 2: Risk spikes (do these first)

Each test has a pass/fail outcome. Don't polish anything here. Throwaway code is fine.

Write each spike up in [docs/spikes](docs/spikes/) using [the template](docs/spikes/TEMPLATE.md), one file per spike.

**Where to track what:** this section is the *plan* for each spike: the steps and the pass condition. Progress lives in each spike's linked issue: tick the boxes there, not here, and discuss there. If a spike's plan changes, change it here first, then copy the change into the issue. When a spike is done, tick its whole section here in one go and link the write-up.

### Spike 1: Remote player puppet
Issue: [#1](https://github.com/appdoze/Hitman-WoA-Coop/issues/1)
- [ ] Spawn a second 47 (or an NPC wearing 47's outfit) in a level, controlled by your code, not by the game's player controller.
- [ ] Drive its position and rotation from recorded data.
- [ ] Play locomotion animations on it: walk, run, crouch.
- [ ] Change its outfit at runtime.
- **Start from:** `Hitmen` already adds a second 47 through a small SMF brick and drives him by writing input state into his `ZCharacterInputProcessor`, so animations come from the game. Reuse the approach, not the code: it copies raw memory from the network (R17). `ZHitman5_SetOutfit` (`Player`, `Randomizer`) for outfit changes; `Outfits` shows how to load outfits that aren't normally on the map. See [ZHMModSDK built-in mods](#zhmmodsdk-built-in-mods).
- **Pass:** a recorded walk is replayed by the puppet and looks right. **Fail:** you have no workable way to show the partner, so stop and rethink.

### Spike 2: Matching entities across machines
Issue: [#2](https://github.com/appdoze/Hitman-WoA-Coop/issues/2)
- [ ] Find how NPCs, doors and items are identified in the scene and bricks, starting from the entity IDs ZHMModSDK exposes.
- [ ] Prove the same NPC gets the same ID on two separate launches of the same mission.
- [ ] Find what changes this: different loadouts, starting locations, difficulty or mission story state.
- **Start from:** `Editor`'s local API identifies game entities by (blueprint (TBLU) hash, entity ID). A short script that calls `listEntities` on each PC and compares the results answers most of this spike. `Hitmen` used `ActorManager` slot numbers instead, which only works if both machines fill the list in the same order: test that too.
- **Pass:** a stable ID for NPCs, doors and pickups in Paris. **Fail:** you need your own ID scheme, such as spawn order or position, which is fragile. Budget extra time.

### Spike 3: Can the AI perceive a second player?
Issue: [#3](https://github.com/appdoze/Hitman-WoA-Coop/issues/3)
- [ ] Find what the AI checks for sight, hearing, disguise and enforcers. Is it hard-wired to the single local player?
- [ ] Try making an NPC notice, become suspicious of, or attack the puppet from Spike 1.
- [ ] Try forcing perception events by hand, for example "this NPC spotted the partner at position X".
- **Start from:** `DebugMod` for inspecting NPCs and AI behaviour. `AdvancedRating` reads the game's own `ZGameStatsManager` flags (trespassing spotted, body found, dead body seen), a quick way to see whether the AI noticed the puppet. The Nexus mods [AI Sight Modifications, Blind NPCs and AI Playground](#community-mods-nexus-mods) change NPC vision and behaviour through level data alone, which shows where perception is set up.
- **Pass:** guards react to the puppet in some controllable way. **Fail:** go with Tier 1 or Tier 2 (see Part 3).

### Spike 4: Can the client's world follow the host's?
Issue: [#4](https://github.com/appdoze/Hitman-WoA-Coop/issues/4)
- [ ] On the client, override one NPC's position and state with the host's values every frame. Does the client's own AI fight you?
- [ ] Kill or knock out an NPC on the host, and make the same thing happen on the client.
- [ ] Open a door on one machine and make it open on the other.
- [ ] Launch the same mission on both machines with no syncing at all, and log NPC positions over time. How quickly do they drift apart? This shows how much the game relies on randomness and frame timing.
- [ ] Try setting the same random seed on both machines. Does drift drop noticeably?
- **Start from:** `Editor`'s API can set properties and transforms and trigger entity pins (`signalEntityPin`, such as a door's open input) from a script, before any C++. `ZActor_KillActor` and `ZActor_ReviveActor` for kills and waking. `Hitmen`'s NPC position sync for the NPC override test.
- **Pass:** a single NPC and a single door stay in step for 10 minutes.

### Spike 5: Can the game's own multiplayer carry co-op?
Issue: [#5](https://github.com/appdoze/Hitman-WoA-Coop/issues/5)

**Re-planned on 2026-11-07.** Community mods show the game's own multiplayer still works in WoA on Steam: [Ghost Mode](#community-mods-nexus-mods) (separate worlds) and [Sniper Co-op](#community-mods-nexus-mods) (one shared world). So the question is no longer "is it there?" but "how far can it go?". **Run this spike first, from Month 1:** its result decides how much of Spikes 1 and 4 is needed.

- [ ] On two Steam PCs, install Ghost Mode and Sniper Co-op and play one match of each through Peacock. Write down what works, what crashes and what each player sees.
- [ ] Find how the two games connect: Steam networking, RakNet (`ZRakNetReplica` in the SDK's `ZPlayerRegistry.h`) or both, and what Peacock's role is (`components/multiplayer/`, and `components/menus/sniper.ts`, which returns loadouts for all three Sniper Assassin characters because single player and multiplayer share one request; official servers returned one).
- [ ] Find what is replicated in Sniper Co-op: player positions and animations, NPCs, kills, items, doors, scoring (the `ScoringEventReplicator` and `EntityPropertyReplica` entities, and entities that carry the engine's `zreplicableaspect`).
- [ ] Try loading a normal mission (Paris or ICA Facility) in a multiplayer session, for example by pointing a Sniper Co-op or Ghost Mode preset at a normal contract. Do both players appear? What stays in sync?
- [ ] Check whether any of it works on Epic.
- **Start from:** the SDK's `ZPlayerRegistry.h` maps 4 player slots, each with a `ZNetPlayerController` that has a RakNet replica and a `m_bConnectedToMultiplayer` flag (commented "true for both players when in multiplayer"). `Hitmen` ships `multiplayer_hitmen.entity.json`, HITMAN 2's Sniper Assassin co-op level data with Stone, Knight, a `ScoringEventReplicator` entity (class `zsniperchallengescoringeventreplicator`), an `EntityPropertyReplica` entity, and two entities made networked by adding the engine's `zreplicableaspect` to them (a `ScoringEventEmitter` and a "Weapon Fired" hero sensor). That aspect is likely how the game marks an entity for replication. The Ghost Mode and Sniper Co-op mods, [LocalGhost](https://gitlab.com/grappigegovert/localghost) and their authors (Part 0 Contacts).
- **Pass:** a normal mission loads with both players in the game's own multiplayer session and both 47s visible, and at least kills or NPC positions stay in sync. **Partial:** it only works in Ghost Mode or on Sniper Assassin maps; Tier 1 can still build on Ghost Mode. **Fail:** the game's own multiplayer can't be used outside its original modes; carry on with our own netcode (Spikes 1 and 4).
- **If it passes:** re-plan Spikes 1 and 4, Months 4–5 and D008 around the game's own netcode. This path is Steam only (R19, D017).

### Spike 6: Tagging and injecting Peacock events
Issue: [#6](https://github.com/appdoze/Hitman-WoA-Coop/issues/6)
- [ ] Log the full event stream each game sends to Peacock (`SaveAndSynchronizeEvents4` / `SaveEvents2` in `components/eventHandler.ts`) during a short mission.
- [ ] Try adding a custom field to outgoing events, such as the ID of the player who caused it. The game builds and sends these events itself, so this needs a hook on the game's event sending in the mod, or a small proxy between the game and Peacock. A Peacock plugin alone can't know who caused an event.
- [ ] Try sending a synthetic event, such as a partner's kill, into the host's session, and check that scoring and challenges react to it.
- [ ] Find which events the host's game sends when the *partner* kills or knocks out an NPC (likely an NPC death or "body found" with no clear cause). These are the events that would wrongly cost the host Silent Assassin or non-target kill penalties.
- [ ] Check `MurderedBodySeen` in particular. When a non-target in the host's game sees a murdered body, Peacock (`components/eventHandler.ts`) sets `silentAssassinLost` for the rest of the mission, and killing the witness afterwards doesn't undo it. The only exemption is an `AccidentBodyFound` event with the same timestamp. So one guard seeing a body the *partner* left costs the host Silent Assassin for good, and so does a partner's accident kill if the host's game doesn't report it as an accident. Test both, and make sure the partner-action filter (see Fail below) covers this event.
- **Start from:** `AdvancedRating` hooks `ZAchievementManagerSimple_OnEventSent`, which receives each game event as JSON. Check whether these are exactly the events sent to Peacock; if they are, that's the hook for tagging events with the player who caused them. The Nexus mod [Offline Scoring](#community-mods-nexus-mods) is a reference for custom scoring, including camera recordings.
- **Pass:** Peacock can tell which player caused each kill. **Fail:** score each player separately and merge the results at the end of the mission, with a filter rule: each player's session ignores deaths, knockouts and bodies that the mod says were caused by the partner. Without that rule, separate scoring still penalises the host for the partner's actions.

### Decision gate (end of Month 3)
- [ ] Write down each spike's result and pick a tier from Part 3.
- [ ] Use Spike 6's result to choose the scoring approach: tagged shared events, or separate scoring with the partner-action filter, merged at the end.
- [ ] Re-estimate Months 4–12 based on what the spikes actually took. If the spikes ran over, cut maps, not quality.

---

## Part 3: Design tiers

| | Tier 1: Ghost co-op | Tier 2: Shared world | Tier 3: Full co-op |
|---|---|---|---|
| Partner visible | Yes (puppet) | Yes (puppet) | Yes |
| NPC world | Each machine runs its own | Host controls it, client follows | Host controls it, client follows |
| Kills and knockouts sync | Targets only | All NPCs | All NPCs |
| Doors, items, bodies sync | No | Yes | Yes |
| AI reacts to the partner | No | Only through forced events or not at all | Yes |
| Needs which spikes to pass | 1 | 1, 2, 4 | 1–4 |
| Realistic in year 1 | Several maps | 2–4 maps | 1–2 maps, rough |

- **Tier 1:** like the old Ghost Mode, but working together. Both players play the same contract in separate worlds, the partner shows up as a ghost, target deaths are shared, and scores are combined. Least risk. You can ship it even if Spike 3 fails. Since the [Ghost Mode](#community-mods-nexus-mods) port shows the game's own separate-worlds multiplayer works on Steam, Tier 1 might be built on that instead of our own netcode (Spike 5).
- **Tier 2:** a real shared world, but guards are effectively blind to the partner. Gameplay works if you treat the partner as a "support" role, or add forced detection for actions that should obviously be seen, such as shooting in front of a guard.
- **Tier 3:** what the original roadmap assumed. Only commit to it if Spike 3 clearly passes.

---

## Part 4: 12-month roadmap

Months 4–12 assume Tier 2. Under Tier 1, skip the NPC and world sync items and use the time for more maps. Under Tier 3, add AI perception work to Months 5 and 7, and expect to finish fewer maps.

### Month 1: Foundation
Issues: [Month 1 milestone](https://github.com/appdoze/Hitman-WoA-Coop/milestone/1). Items marked `good first issue` there need no reverse-engineering experience.
- [ ] ZHMModSDK building and running on the current WoA build, with a hello-world mod
- [ ] Part 0: tools reviewed, prior work searched and written up
- [ ] Fork Peacock and run it locally
- [ ] Profile backup script: copy Peacock's `userdata` folder (profiles, contract sessions) to a timestamped backup before every test session. Restore it with one command.
- [ ] Two-PC (or two-instance) test setup, with a script that launches both
- [ ] Logging from the mod to a file, with timestamps, on both machines
- [ ] CI build for the mod and the Peacock fork, so every commit produces a test build
- [ ] Crash dump collection for the mod
- [ ] Echo mode switch in the mod (Part 9, Method 1), ready for Spike 1
- [ ] Start Spike 5 (play Ghost Mode and Sniper Co-op on two Steam PCs) and contact their authors
- [ ] Start Spikes 1 and 2

### Month 2: Spikes
- [ ] Settle the decisions that don't need spike results: D006, D007, D008, D010, D012, D013, D015, D017, D018 (issues #16–#24, see [docs/decisions](docs/decisions/))
- [ ] Finish Spikes 1 and 2
- [ ] Start Spikes 3 and 4
- [ ] Finish Spike 5 first, and adjust Spikes 1 and 4 to its result
- [ ] Spike 6 in parallel. Logging the event stream is Peacock-only work; tagging events with the player who caused them needs a mod-side hook (or a proxy in front of Peacock).

### Month 3: Spikes and decision gate
- [ ] Finish Spikes 3 and 4
- [ ] Decision gate: pick the tier
- [ ] Fill in every answer in Part 1
- **Milestone:** a written test report (all six files in [docs/spikes](docs/spikes/)) and a chosen tier

### Month 4: Network layer and player sync (Paris or ICA Facility)
- [ ] Handshake: game build, mod version, protocol version, installed-mods hash, contract ID, difficulty (fields drafted in [docs/protocol.md](docs/protocol.md))
- [ ] Connection choice: self-hosted relay by default, direct P2P only after a warning about IP addresses
- [ ] Handshake also exchanges each player's platform (Steam or Epic) and owned locations
- [ ] Reliable channel for events, best-effort channel for position
- [ ] Message checks built into the message format from the start: every incoming message is checked for size, field ranges, known message types and known entity IDs before anything uses it. The mod reads network data in C++ inside the game process, so a malformed message must never reach game code (R17).
- [ ] Fuzz-test the message parser: feed it large amounts of random and broken input (for example with libFuzzer) and fix every crash. Run it in CI.
- [ ] Sync position, rotation, speed, crouch and sneak at 20–30 Hz, with smoothing between updates
- [ ] Sync disguise changes
- [ ] Sync held and visible items
- [ ] Name tag over the partner
- [ ] Message log: write every network message to a file with its mission-clock timestamp, on both machines
- [ ] Fake client program (Part 9, Method 2) that joins as player 2 and walks a scripted path
- **Milestone:** two players walk around Paris together and see each other move smoothly

### Month 5: NPC and world basics (Paris)
- [ ] NPC kills and knockouts, on both machines
- [ ] Bodies: drag, hide in containers, discovery
- [ ] Doors, locks and keycards
- [ ] Item pickup, drop and throw
- [ ] Health and death of both players
- [ ] Message replay tool: feed a recorded host session's messages (from the Month 4 message log) into a fresh client instance, so you can retest the client without a second person. Replay the host's *messages*, not a player's *inputs*: the game isn't deterministic (see Spike 4), so replayed inputs drift and won't reproduce bugs.
- [ ] Out-of-sync detection: the host sends periodic checksums of key state, and the client logs any mismatch
- **Milestone:** you can clear Paris's targets together with basic actions

### Month 6: Peacock integration
- [ ] Peacock server setup from Part 1 working
- [ ] Lobbies built on Peacock's existing `RegisterToMatch` and `SetMatchData` (`components/multiplayer/multiplayerService.ts`), not from scratch
- [ ] Invite codes, which map to match IDs
- [ ] Ownership check: the lobby only offers locations both players own (using the owned locations exchanged in the handshake under server option (b))
- [ ] Co-op contract presets (Peacock's existing `MultiplayerPresets` config is the model)
- [ ] Event authority and duplicate removal from Part 1, so scoring, Silent Assassin and challenges are correct for both profiles
- [ ] XP and mastery for both profiles, with no duplication or loss
- **Milestone:** start a co-op Paris mission from the menu, finish it, and get correct results for both profiles

### Month 7: AI states and mission stories (Paris)
- [ ] Sync AI states from the host: alerted, searching, investigating, combat, fleeing
- [ ] Sync alarms, cameras and evidence recorders
- [ ] Paris mission stories and opportunities, including their triggers
- [ ] Cutscene rule: pick one (skip for both, or the host triggers and both play), and implement it
- [ ] Crowds: leave them on each machine; sync only panic and reaction events
- **Milestone:** Paris fully playable, including mission stories

### Month 8: Accidents and second map
- [ ] Accidents: fuses, explosions, falling objects, poisonings, drowning. The host's result is final.
- [ ] Second map, using the Part 5 checklist. Pick a small map: Sapienza or Hokkaido.
- **Milestone:** two maps fully playable

### Month 9: Network reliability
- [ ] Simulated lag and packet loss in testing: 100 ms, 150 ms, 2% loss
- [ ] Bandwidth: send only what changed; nearby NPCs get updated more often than distant ones
- [ ] Disconnect handling for v1: end the mission cleanly for both players and save progress up to that point. Rejoining and host migration are year 2.
- [ ] Mission restart and quitting handled for both players
- [ ] Manual saves turned off in co-op
- [ ] Pause behaviour from Part 1: menus don't stop the shared world
- [ ] Gameplay checks on incoming messages, on top of the Month 4 format checks: reject impossible values for the game state, such as teleporting across the map, killing an NPC that's already dead, or picking up an item that's somewhere else
- [ ] Rate limits: drop a peer that floods messages
- [ ] Fuzz tests extended to every message type added since Month 4
- **Milestone:** stable at 150 ms ping, with clean failure on disconnect

### Month 10: Interface and contracts
- [ ] Partner marker in the game and on the minimap
- [ ] Quick pings (the [Ghost Mode Emotes](#community-mods-nexus-mods) mod restores HITMAN 2's emote menu, a possible starting point)
- [ ] Shared notifications ("Partner spotted", "Body found")
- [ ] Loadout screen for each player
- [ ] Results screen showing both players
- [ ] Featured Contracts and user contracts on the supported maps
- **Milestone:** looks and feels like a finished mode on two maps

### Month 11: Third map and outside testing
- [ ] Third map, using the Part 5 checklist
- [ ] Closed test with people outside the team
- [ ] Debug overlay: ping, packet loss, out-of-sync count, entity being looked at
- [ ] Out-of-sync logs that testers can send you
- **Milestone:** outside testers finish missions on all three maps

### Month 12: Public beta
- [ ] Installer or patcher integration
- [ ] Automatic Peacock profile backup before every co-op mission, kept for the last 10 missions
- [ ] Privacy notice in the README and the lobby: what data is sent to the partner and to any relay
- [ ] Documentation, FAQ and known issues
- [ ] Patch plan: keep game-specific addresses and hooks in one place, and have a quick way to find them again after a game update
- [ ] Public beta on 2–4 maps, then a fixes cycle
- **Milestone:** public beta. Call it 0.x, not 1.0.

---

## Part 5: Per-map checklist

Copy this for each map. A map isn't "done" until every box is ticked.

- [ ] Entity IDs stable across launches (Spike 2 method still works)
- [ ] All targets: routes, deaths and exits synced
- [ ] Every mission story and opportunity completed together
- [ ] Every scripted sequence and cutscene handled
- [ ] Map-specific accidents tested
- [ ] Map-specific items: keycards, poisons and story items
- [ ] Crowd panic tested (busy maps only)
- [ ] All exits and the mission end screen work for both players
- [ ] Silent Assassin run completed and scored correctly for both profiles
- [ ] Full two-player run at 150 ms with no out-of-sync errors
- [ ] Both players start from different starting locations and with different loadouts
- [ ] Pause, map and inventory menus open on one machine without breaking sync
- [ ] Host message log of a full run saved as a regression test (replayed into a client with the Month 5 tool)
- [ ] Known issues written down

Suggested order, smallest and simplest first: Paris, Sapienza, Hokkaido, Miami, Dubai, Berlin, then the larger and busier maps (Marrakesh, Mumbai, Chongqing, New York, Mendoza). Expect about one map a month with a small team. Busy or very scripted maps take longer.

---

## Part 6: Year 2 and beyond

- [ ] Remaining maps, using the Part 5 checklist
- [ ] Escalations
- [ ] Elusive Targets
- [ ] Sniper Assassin co-op
- [ ] Freelancer co-op: shared campaign, Syndicates and Merces
- [ ] Matchmaking (fill in Peacock's `RegisterToPreset`, currently a `501` stub)
- [ ] Rejoin after a disconnect
- [ ] Host migration
- [ ] Versus mode in the style of Ghost Mode
- [ ] Co-op leaderboards and achievements
- [ ] Sessions that survive a Peacock restart (Peacock currently keeps matches in memory only)
- [ ] Groups larger than two
- [ ] Voice chat
- [ ] Spectator mode
- [ ] HITMAN 2016 and HITMAN 2 builds

---

## Part 7: Risk register

Review this at the decision gate and at the end of every month. Update the status column, add new risks as you find them.

| # | Risk | Likelihood | Impact | Early warning | Response | Status |
|---|---|---|---|---|---|---|
| R1 | AI can't be made to perceive a second player | High | Tier 3 impossible | Spike 3 | Drop to Tier 2 or Tier 1 | Open |
| R2 | No stable entity IDs across machines | Medium | Tier 2/3 much harder | Spike 2 | Own ID scheme (spawn order, position), budget extra time | Open |
| R3 | Client's own AI fights the host's state | Medium | Constant out-of-sync errors | Spike 4 | Suppress client AI for host-controlled NPCs; drop to Tier 1 if impossible | Open |
| R4 | Game randomness and frame timing cause fast drift | Medium | More bandwidth, more sync code | Spike 4 drift test | Sync seed at start, or send every random outcome from the host | Open |
| R5 | Peacock can't attribute events to a player | Medium | Wrong scoring, broken Silent Assassin | Spike 6 | Separate scoring with partner-action filter | Open |
| R6 | Pause menu or loading screens can't be decoupled from the shared world | Medium | Co-op breaks whenever someone opens a menu | Month 4 testing | Hook pause; block pause in co-op if needed | Open |
| R7 | Game patch breaks hooks | Low (WoA patches rarely now) | Mod stops working | Build number changes | Version lock; keep addresses in one place; pattern scans instead of fixed addresses | Open |
| R8 | Scripted sequences on big maps don't sync | High | Some mission stories unplayable | Per-map checklist | Mark as known issue; skip or force-sync cutscenes | Open |
| R9 | Team time drops (burnout, life) | High | Everything slips | Months running over by more than 25% | Cut maps, not quality; release what works | Open |
| R10 | IOI objects to the mod | Low | Distribution problems | Contact from IOI | Follow Peacock's approach: no game files shipped, offline only, no piracy support | Open |
| R11 | Players run different content mods | High | Entity IDs differ, instant out-of-sync | Handshake mods-hash mismatch | Compare mods hash in handshake; block or warn | Open |
| R12 | Partner doesn't own a location or DLC | Medium | Can't join, or game crashes on load | Ownership check in lobby | Only offer locations both players own | Open |
| R13 | Player IP addresses exposed over P2P | Medium | Privacy and harassment risk | Direct connection used | Self-hosted relay by default; warn before direct connection; no automatic port opening | Open |
| R14 | Co-op bug corrupts or loses a Peacock profile | Medium | Players lose progress, lose trust | Profile validation errors, tester reports | Automatic profile backup before every co-op mission; validate profile writes | Open |
| R15 | Duplicated work with another co-op project or upstream | Low | Wasted months | Part 0 search, community contacts | Search and make contact in Month 1; collaborate or reuse | Open |
| R16 | Steam-only networking excludes Epic players | Medium | Steam and Epic players can't play together | Choosing Steam P2P or Steam Datagram Relay | Platform-neutral relay by default; Steam networking only for Steam-only lobbies | Open |
| R17 | Malformed messages crash or exploit a player's game, Peacock or the relay | Medium | Crashes, remote code execution, players stop trusting the mod | Fuzz-test crashes, reports through SECURITY.md | Format checks from Month 4; fuzz tests in CI; gameplay checks and rate limits in Month 9; private reporting through SECURITY.md | Open |
| R18 | Relay abused as an open proxy | Low | Relay operator blamed for others' traffic; relay shut down | Unknown traffic through the relay | Relay only forwards between two peers in the same match, after the handshake; rate limits | Open |
| R19 | Building on the game's own multiplayer ties co-op to Steam and to its known crashes | Medium | Epic players left out; crashes we can't fix in our own code | Spike 5 results; Ghost Mode and Sniper Co-op known issues | Decide in D017; keep our own netcode as the fallback; talk to the Ghost Mode and Sniper Co-op authors about the crashes | Open |

---

## Part 8: Definition of done and metrics

### Definition of done for any feature
- [ ] Works on both host and client, tested on two separate PCs (not only two instances)
- [ ] Works at 150 ms ping and 2% packet loss
- [ ] Out-of-sync detection covers it (the host's checksum includes its state)
- [ ] Messages it sends are checked on arrival (malformed or impossible values rejected)
- [ ] Logged well enough that a tester's log explains a failure
- [ ] Known issues written down

### Metrics to track
Record these at the end of every month from your test runs. A milestone isn't met until its numbers are.

| Metric | How to measure | Beta target (Month 12) |
|---|---|---|
| Out-of-sync errors | Checksum mismatches per hour of play, from the Month 5 detection | Fewer than 1 per hour |
| Mission completion | Share of two-player test runs that reach the exit on supported maps | 90% or more |
| Crash rate | Crashes per hour of play, from crash dumps | Fewer than 1 per 5 hours |
| Bandwidth | Average and peak per player, from the message log | Under 100 KB/s average |
| Latency tolerance | Highest ping where test runs still meet the out-of-sync target | 150 ms |
| Scoring accuracy | Test runs where both profiles get the expected rating, Silent Assassin status and XP | 100% on Silent Assassin test runs |
| Disconnect handling | Test disconnects where both players end cleanly with progress kept | 100% |
| Map coverage | Maps with every Part 5 box ticked | 2–4 |

---

## Part 9: Solo development setup

How to build and test the mod on one PC, without a second person. Rough split: about 90% of development uses methods 1–3, method 4 is for milestone checks, and real people are only for the per-map checks and the beta.

### Method 1: Echo mode (from Month 1)
A debug switch in the mod that sends your own network messages back to yourself:
- [ ] Pack your own 47's position, animation state, disguise and actions into protocol messages
- [ ] Send them over loopback with an artificial delay (default 150 ms, adjustable)
- [ ] Drive the Spike 1 puppet from those messages, so a copy of you follows about a second behind
- Tests: Spike 1, Spike 3 (walk the puppet past guards), Month 4 player sync and smoothing, the message format, and lag handling, all in a single game.

### Method 2: Fake partner program (from Month 4)
A small standalone program that speaks the co-op protocol and pretends to be player 2:
- [ ] **Fake client:** joins the real game running as host, walks a recorded or scripted path, and sends scripted actions ("knocked out NPC #412", "opened door #88"). Tests the host side.
- [ ] **Fake host:** plays back a host message log (Month 4 log, Month 5 replay tool) into the real game running as client. Tests the client side.
- [ ] Scripted test scenarios that run without anyone at the keyboard, as regression tests
- This only works if the protocol has stable entity IDs and a clean, versioned message format (Part 1 and Spike 2). Keep it that way.

### Method 3: Peacock tests without the game (from Month 2)
- [ ] Scripted HTTP tests against the Peacock fork with two fake profiles: log in, `RegisterToMatch`, send events, check scoring and progression for both
- [ ] **Dev-only profile override.** Peacock keys each profile to the `steam_userid` or `epic_userid` the game sends at login (`components/oauthToken.ts`), so two copies of the game on one account become the same profile. Either run two Peacock servers on different ports, or add a dev flag that gives the second connection a different fake user ID. Never enable this outside development.

### Method 4: Two copies of the game on one PC (milestone checks only)
- [ ] Find out what stops a second launch (Steam, and any single-instance check in the game itself) and handle it in the mod for development builds
- [ ] Run the second copy windowed, at low resolution and low settings. Plan for 32 GB of RAM.
- [ ] Stop the game pausing or freezing when its window loses focus: enable ZHMModSDK's built-in `NoPause` mod on both copies (see Part 0)
- [ ] Each copy uses its own Peacock profile (Method 3 override)
- Alternative: a Hyper-V virtual machine with GPU partitioning (Windows 11 Pro; community scripts such as Easy-GPU-PV). More isolated, but fiddlier to set up.

### Method 5: Bad-connection testing on one PC
- [ ] **clumsy** (free Windows tool) to add lag, packet loss and reordering, including to loopback traffic between two programs on the same PC. Not to be confused with ZHMModSDK's built-in mod also called `Clumsy`, which has nothing to do with networking.
- [ ] If you use GameNetworkingSockets, its built-in fake lag and loss settings may be enough on their own

### Method 6: Real people (Month 11 onward)
- [ ] Per-map two-player runs (Part 5), outside testing (Month 11) and the beta
- [ ] Rent a cloud gaming PC by the hour and connect with Parsec, or recruit testers from the Peacock and Hitman modding communities, who already have Peacock set up

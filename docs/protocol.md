# Network protocol (draft)

> **Status: draft.** Nothing here is final. The format depends on [D008 (#18)](https://github.com/appdoze/Hitman-WoA-Coop/issues/18), versioning on [D007 (#17)](https://github.com/appdoze/Hitman-WoA-Coop/issues/17), timing on [D010 (#19)](https://github.com/appdoze/Hitman-WoA-Coop/issues/19) and entity IDs on [Spike 2 (#2)](https://github.com/appdoze/Hitman-WoA-Coop/issues/2). Update this file whenever one of those is decided.

This lists the messages the mod, relay and tools exchange, so that echo mode ([#15](https://github.com/appdoze/Hitman-WoA-Coop/issues/15)), Spike 1 and the fake client and host all use the same format from the start. The real schema will live in `protocol/` once D008 is decided.

## Rules for every message

- Starts with a header: protocol version, message type, length, sequence number and mission-clock timestamp.
- The protocol version is always the **first 2 bytes** of every message, in the same encoding, in every version of the protocol. It must never move, so any build can read it and reply "wrong version" before trying to parse the rest.
- Is checked before anything uses it: length within the limit for its type, known message type, every field in range, every entity ID known for the current mission. Anything that fails is dropped and logged, never passed to game code ([R17](../WOA_COOP_ROADMAP.md#part-7-risk-register)).
- Never contains raw C++ structs, pointers or memory addresses.
- Never contains IP addresses, Steam or Epic user IDs or account names. Players are identified by a per-match player number (0 = host, 1 = client).
- Every message type has a fuzz test (from Month 4).
- Traffic between the players is encrypted and authenticated, so the relay or anyone on the network can't read or change it. GameNetworkingSockets does this built in; ENet doesn't, so choosing ENet means adding encryption on top (D008, D018).

## Channels

| Channel | Delivery | Used for |
|---|---|---|
| **Reliable** | Guaranteed, in order | Handshake, world events, anything that changes game state |
| **Best-effort** | May be lost; newer replaces older | Positions and animation state, sent 20–30 times a second; clock sync messages |

## Handshake (reliable)

Sent by both sides on connect, before any other message. It has two stages, because the mission isn't chosen until after the players are connected.

### On connect

Any mismatch or failed check ends the connection with a clear error to both players.

| Field | Purpose | Decision |
|---|---|---|
| Protocol version | Message format must match (always the first 2 bytes, see above) | D007 |
| Match ID and join secret | From the Peacock lobby (invite code). Proves the peer belongs to this match. The relay checks them too, and forwards nothing until they match (R18). | D003, D018 |
| Mod version | Same mod build on both sides | D006 |
| Game build | Same game build | D006 |
| Platform (Steam or Epic) | Sent for information. Only a mismatch if D017 rules out playing across platforms. | D017 |
| Installed-mods hash | Content mods must match (R11) | D015 |
| Owned locations | Lobby only offers locations both players own | D016 |

### At mission load

Checked when the host picks a mission in the lobby and both games load it. A mismatch returns both players to the lobby with a clear error.

| Field | Purpose | Decision |
|---|---|---|
| Contract ID | Both load the same mission | – |
| Difficulty and mission settings | Both use the host's settings | D012 |

## Message types (first pass)

| Type | Channel | Sender | Contents | Needed from |
|---|---|---|---|---|
| `Hello` | Reliable | Both | On-connect handshake fields above | Month 4 (echo mode: Month 1) |
| `MissionLoad` | Reliable | Host → client, then client → host to confirm | At-mission-load fields above | Month 6 |
| `Goodbye` | Reliable | Both | Reason: quit, mission end, error | Month 4 |
| `ClockPing` / `ClockPong` | Best-effort | Both | Timestamps for offset measurement. Not reliable: resends and in-order delivery add random delay that spoils the measurement. Send several, and throw away samples with unusually long round trips. | Month 4 |
| `PlayerState` | Best-effort | Both | Position, rotation, speed, stance (stand, crouch, sneak), animation state | Month 1 (echo mode) |
| `PlayerOutfit` | Reliable | Both | Current disguise | Month 4 |
| `PlayerItem` | Reliable | Both | Held and visible items | Month 4 |
| `PlayerAction` | Reliable | Client → host | An action to apply in the host's world: knock out, kill, pick up, open door, drag body | Month 5 |
| `WorldEvent` | Reliable | Host → client | The host's result: NPC dead or unconscious, door state, item moved, body hidden or found, alarm | Month 5 |
| `NpcState` | Best-effort | Host → client | Position and AI state for NPCs near the client | Month 5 (Tier 2+) |
| `Checksum` | Reliable | Host → client | Fingerprint of key world state, for out-of-sync detection | Month 5 |
| `ScoreEvent` | Reliable | Host → client | Which player caused a kill, knockout or body, for scoring. Only the host sends it, because the host's world is the authority; sending it from both sides would bring back the duplicate events from Part 1 (D005). | Month 6 (depends on Spike 6) |

Every `PlayerAction` and `WorldEvent` carries the player number of whoever caused it, so scoring can tell the players apart ([Spike 6, #6](https://github.com/appdoze/Hitman-WoA-Coop/issues/6)).

## Open questions

- How entity IDs are encoded (Spike 2)
- Whether `NpcState` is needed at all, or `WorldEvent`s are enough (Spike 4, design tier)
- Maximum message sizes and rates for each type (Month 9 bandwidth work)
- Whether the relay needs to read anything beyond the match ID and join secret, or only forwards encrypted traffic (D018)
- How the join secret is generated, shared with both players by Peacock, and expired after the match (D003, D018)

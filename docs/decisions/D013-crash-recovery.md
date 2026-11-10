# D013: Crash recovery

| | |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-11-07 |
| **Who decided** | @painbaba |
| **Roadmap item** | [Part 1: Crash recovery](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions) |
| **Issue or PR** | [#20](https://github.com/appdoze/Hitman-WoA-Coop/issues/20) |

## Question

If either game crashes, how does the other game notice, what does the remaining player see, and what progress is kept — and how is what was kept confirmed with Peacock?

## Options

1. **Silence detection, then a clean mission end for the survivor; nothing new is written at crash time.** Both games already stream traffic continuously (`PlayerState` at 20–30 Hz on the best-effort channel, plus reliable-channel messages). A partner counts as gone after **5 s** with no best-effort traffic from them (100+ missed updates at 20–30 Hz — far beyond the 150 ms / 2% loss the beta targets, so a live partner is never misread as gone). On the 5 s mark the surviving game sends a probe on the reliable channel and shows an overlay; if nothing comes back within **5 more seconds** (10 s total), the partner is declared gone. The remaining player sees an overlay — "Partner's game closed unexpectedly — mission ending" with a short countdown — and then the mission ends through the same code path as a normal quit ([`Goodbye`](../protocol.md), lobby return), not by force-killing the session. Silence is measured on the networking thread — keep-alive answers and the reliable probe are sent and answered there, not on the game thread — so a frame stall (loading screen, shader compile, a paused debugger) never looks like silence; when a stall is expected, the mod sends `Goodbye` with a "loading" reason instead of letting the partner run detection. Progress: the game queues scoring events and delivers them to Peacock in batches — Peacock's `SaveAndSynchronizeEvents4` receives an array of events (`values`), sent every few seconds, and the game decides when to send, not the mod. So a crash loses everything the crashed game emitted since its **last sync** — seconds of events, not "sub-second" — plus any client-to-host actions that never reached the host's world. Everything Peacock already confirmed stays. "The survivor flushes its own pending queue" before ending is only possible if the mod can make the game sync immediately; that is an open question for [Spike 6](../../WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first) ([#6](https://github.com/appdoze/Hitman-WoA-Coop/issues/6)), so this flow is conditional on it. Which game talks to Peacock at all is [D005](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions)'s call, and D005 is still open: the protocol draft currently has only the host sending `ScoreEvent`, so if the host reports for both players, a host crash also loses the client's unsent progress.
2. **Freeze the shared world and wait for the partner to come back.** Friendlier in theory, but pausing the shared world is exactly the problem [R6](../../WOA_COOP_ROADMAP.md#part-7-risk-register) exists for — the pause menu freezes only the local game, and [D011](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions) still has to make menus cover just the local screen. Rejoining and host migration are explicitly year 2 ([Part 6](../../WOA_COOP_ROADMAP.md#part-6-year-2-and-beyond)), so a frozen world would sit there indefinitely in v1, holding a half-synced state that nothing can confirm to Peacock.
3. **The survivor keeps playing solo.** Asymmetric by construction: if the client crashes, the host's world is intact and could go on; if the host crashes, the client's world is only a follower ([Spike 4](../../WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first)) and has nothing to continue in. Handling both directions means two new feature paths (detaching the shared world, promoting the client), plus scoring rules for a mission that changed player count midway — all unplanned, while the roadmap's Month 9 disconnect item already commits to "end the mission cleanly for both players".

## Decision

Silence detection (5 s best-effort quiet, +5 s reliable-channel probe) marks the partner as gone; the surviving player sees a notification overlay and the mission ends through the normal quit path, with no writes to Peacock or profiles triggered by the crash itself.

## Why

- A crashed or hung process sends nothing, so silence is a dependable signal — no crash-detection hook inside the partner's game is needed on the survivor's side. Crashes are already the rare path (beta target: fewer than 1 per 5 hours, [Part 8](../../WOA_COOP_ROADMAP.md#part-8-definition-of-done-and-metrics)), so detection does not need to be fast, only correct; 10 s worst case is an acceptable wait for a mission that is ending anyway.
- "Nobody loses progress already confirmed in Peacock" holds by construction: events already delivered in the game's periodic sync batches are in Peacock, and the crash-time path performs **no** new profile or scoring writes — the only way the flow touches Peacock is the survivor asking the game to sync immediately (open until Spike 6 lands) and then ending the mission normally, so each player's session is rated exactly once from events Peacock already had ([R14](../../WOA_COOP_ROADMAP.md#part-7-risk-register) is not worsened; the Month 1 profile backups and the Month 12 automatic backups stay as the general corruption safety net).
- Ending through the normal quit path reuses code that must exist anyway (Month 9 "mission restart and quitting handled for both players"), so crash recovery and disconnect handling become one tested flow instead of two.
- What the remaining player sees stays boring on purpose: one overlay with the reason and a countdown, an entry in their message log (with mission-clock timestamps, [D010](D010-clock-and-timing.md)) covering the silence window, and the mod's crash-dump and log collection (Month 1) on the machine that crashed for the actual diagnosis.

## Consequences

- What this makes easier: one "partner lost" flow serves both crashes and disconnects; scoring stays Peacock's job with no new server-side behaviour; testing only needs a way to kill one game (or use the fake client, Part 9) plus the simulated-lag setup from Month 9.
- What this makes harder or rules out: no rejoin and no host migration in v1 (year 2, as already planned) — if your game crashes, your mission is over; the survivor cannot keep playing solo; progress since the crashed game's last sync is not scored. And when Peacock runs on the host's PC ([D003](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions), option a), the host's whole PC going down takes the client's Peacock with it — the client's unsent events cannot be confirmed anywhere.
- Roadmap items, risks or estimates that change: Month 9's "disconnect handling for v1" and this decision share one implementation and one test plan; the Part 8 metric "disconnect handling: 100%" now also covers crash tests. No risk register changes — [R14](../../WOA_COOP_ROADMAP.md#part-7-risk-register) coverage is unchanged and no new risk is introduced.
- When to revisit this: with the year 2 rejoin/host-migration work (a frozen-world or reconnection mode then replaces the immediate end), and if playtests show 10 s feels too long — the timeouts are constants, not design.
- Out of scope, tracked elsewhere: a Peacock restart mid-mission ([Part 6](../../WOA_COOP_ROADMAP.md#part-6-year-2-and-beyond)) and pausing via menus ([D011](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions)).

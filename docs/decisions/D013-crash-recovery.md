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

1. **Silence detection, then a clean mission end for the survivor; nothing new is written at crash time.** Both games already stream traffic continuously (`PlayerState` at 20–30 Hz on the best-effort channel, plus reliable-channel messages). A partner counts as gone after **5 s** with no best-effort traffic from them (100+ missed updates at 20–30 Hz — far beyond the 150 ms / 2% loss the beta targets, so a live partner is never misread as gone). On the 5 s mark the surviving game sends a probe on the reliable channel and shows an overlay; if nothing comes back within **5 more seconds** (10 s total), the partner is declared gone. The remaining player sees an overlay — "Partner's game closed unexpectedly — mission ending" with a short countdown — and then the mission ends through the same code path as a normal quit ([`Goodbye`](../protocol.md), lobby return), not by force-killing the session. Progress: the mod never batches scoring events for later — each game reports its own events to Peacock as the game emits them ([D005](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions), [Spike 6](../../WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first)). So everything already confirmed in Peacock stays; before ending, the survivor just flushes and waits for its own pending event queue, so nothing it did up to that moment is lost either. What is genuinely lost: the crashed game's in-flight events (sub-second) and any client-to-host actions that never reached the host's world.
2. **Freeze the shared world and wait for the partner to come back.** Friendlier in theory, but pausing the shared world is exactly the problem [R6](../../WOA_COOP_ROADMAP.md#part-7-risk-register) exists for — the pause menu freezes only the local game, and [D011](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions) still has to make menus cover just the local screen. Rejoining and host migration are explicitly year 2 ([Part 6](../../WOA_COOP_ROADMAP.md#part-6-year-2-and-beyond)), so a frozen world would sit there indefinitely in v1, holding a half-synced state that nothing can confirm to Peacock.
3. **The survivor keeps playing solo.** Asymmetric by construction: if the client crashes, the host's world is intact and could go on; if the host crashes, the client's world is only a follower ([Spike 4](../../WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first)) and has nothing to continue in. Handling both directions means two new feature paths (detaching the shared world, promoting the client), plus scoring rules for a mission that changed player count midway — all unplanned, while the roadmap's Month 9 disconnect item already commits to "end the mission cleanly for both players".

## Decision

Silence detection (5 s best-effort quiet, +5 s reliable-channel probe) marks the partner as gone; the surviving player sees a notification overlay and the mission ends through the normal quit path, with no writes to Peacock or profiles triggered by the crash itself.

## Why

- A crashed or hung process sends nothing, so silence is a dependable signal — no crash-detection hook inside the partner's game is needed on the survivor's side. Crashes are already the rare path (beta target: fewer than 1 per 5 hours, [Part 8](../../WOA_COOP_ROADMAP.md#part-8-definition-of-done-and-metrics)), so detection does not need to be fast, only correct; 10 s worst case is an acceptable wait for a mission that is ending anyway.
- "Nobody loses progress already confirmed in Peacock" holds by construction: events were already streamed to Peacock when they happened, and the crash-time path performs **no** new profile or scoring writes — the only way the flow touches Peacock is the survivor flushing its own pending queue and then ending the mission normally, so each player's session is rated exactly once from events Peacock already had ([R14](../../WOA_COOP_ROADMAP.md#part-7-risk-register) is not worsened; the Month 1 profile backups and the Month 12 automatic backups stay as the general corruption safety net).
- Ending through the normal quit path reuses code that must exist anyway (Month 9 "mission restart and quitting handled for both players"), so crash recovery and disconnect handling become one tested flow instead of two.
- What the remaining player sees stays boring on purpose: one overlay with the reason and a countdown, an entry in their message log (with mission-clock timestamps, [D010](D010-clock-and-timing.md)) covering the silence window, and the mod's crash-dump and log collection (Month 1) on the machine that crashed for the actual diagnosis.

## Consequences

- What this makes easier: one "partner lost" flow serves both crashes and disconnects; scoring stays Peacock's job with no new server-side behaviour; testing only needs a way to kill one game (or use the fake client, Part 9) plus the simulated-lag setup from Month 9.
- What this makes harder or rules out: no rejoin and no host migration in v1 (year 2, as already planned) — if your game crashes, your mission is over; the survivor cannot keep playing solo; progress made in the final second before a crash is not scored.
- Roadmap items, risks or estimates that change: Month 9's "disconnect handling for v1" and this decision share one implementation and one test plan; the Part 8 metric "disconnect handling: 100%" now also covers crash tests. No risk register changes — [R14](../../WOA_COOP_ROADMAP.md#part-7-risk-register) coverage is unchanged and no new risk is introduced.
- When to revisit this: with the year 2 rejoin/host-migration work (a frozen-world or reconnection mode then replaces the immediate end), and if playtests show 10 s feels too long — the timeouts are constants, not design.
- Out of scope, tracked elsewhere: a Peacock restart mid-mission ([Part 6](../../WOA_COOP_ROADMAP.md#part-6-year-2-and-beyond)) and pausing via menus ([D011](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions)).

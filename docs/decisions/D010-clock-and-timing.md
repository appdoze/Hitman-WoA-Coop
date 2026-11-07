# D010: Clock and timing

| | |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-11-07 |
| **Who decided** | @painbaba |
| **Roadmap item** | [Part 1: Clock and timing](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions) |
| **Issue or PR** | [#19](https://github.com/appdoze/Hitman-WoA-Coop/issues/19) |

## Question

How do the two games share one mission clock — how is the client's offset from the host measured, how often is it re-measured, and what tick rate do messages carry?

## Options

1. **NTP-style ping exchange on the best-effort channel, re-measured continuously.** The mission clock is the host's monotonic clock (milliseconds since mission load). The client measures the offset to it with `ClockPing`/`ClockPong` exchanges ([protocol draft](../protocol.md)): the client sends `T1`, the host answers with `T2` (arrival) and `T3` (send), the client completes the sample at `T4` and estimates `offset = ((T2 - T1) + (T3 - T4)) / 2`. The client keeps the sample with the lowest round-trip time (`T4 - T1`) out of the recent window, because queuing delay only ever inflates RTT and so biases the offset estimate. One exchange every 2 s during play, plus a burst of about 8 over the first seconds after mission load and after any long stall, so the offset is good before movement needs smoothing. Cost: one small message pair per 2 s — negligible against the 20–30 Hz `PlayerState` stream. Risk: an asymmetric path (different upload/download routes through the relay) biases the estimate by half the asymmetry; at the 150 ms target ping the worst-case error of the min-RTT sample is about ±75 ms, which the interpolation buffer absorbs.
2. **Measure the offset once at mission load.** Cheapest, and enough for a mission or two if both PCs keep good time. Risk: the two clocks are independent monotonic counters — they drift apart by however much their rates differ, and any error in the single measurement is permanent. A 30–60 minute mission gives drift time to accumulate, and there is no way to detect that it has.
3. **No shared clock: host stamps every message, client uses arrival time.** No offset exchange at all. Risk: the client cannot tell network delay from actual delay, so it cannot render the partner at a consistent mission time; interpolation (smoothing) has no time base and [Spike 4](../../WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first) drift comparisons and the timestamped [message log](../GLOSSARY.md) lose their common reference. Saves the 2-second ping traffic, breaks the roadmap requirement that movement be smoothed.

## Decision

The client keeps a continuously re-measured offset to the host's monotonic mission clock, estimated NTP-style from `ClockPing`/`ClockPong` exchanges on the best-effort channel and filtered by lowest round-trip time.

## Why

- The [protocol draft](../protocol.md) already reserves `ClockPing`/`ClockPong` on the best-effort channel with "send several and throw away samples with unusually long round trips", and puts a mission-clock timestamp in every message header. This decision only fixes the estimator (half the difference, min-RTT sample), the schedule and the units, so no protocol change is needed.
- The host is the authority for the world, so the host's clock is the one to sync to; per-message host timestamps would arrive too late to drive smoothing (option 3).
- Games are not deterministic across machines (the Spike 4 drift test exists to measure that), so the clock cannot be assumed equal at mission start — hence re-measuring (option 1) rather than one-shot (option 2).
- Every 2 s is far below the rate at which consumer PC monotonic clocks drift apart over a mission, while still cheap; the load-time burst gives smoothing a good offset before the first movement.
- Millisecond resolution in the message header is enough for 20–30 Hz updates (33–50 ms apart) and keeps timestamps within `u32` for missions up to ~49 days.

## Consequences

- What this makes easier: interpolation of the partner's movement has a single time base in both games; `WorldEvent`s can be ordered against each other; both machines' message logs (with mission-clock timestamps, Month 4) and the out-of-sync checksums (Month 5) share one reference; latency measurements in [Part 8](../../WOA_COOP_ROADMAP.md#part-8-definition-of-done-and-metrics) come free from the ping samples.
- What this makes harder or rules out: wall-clock (UTC) timestamps in messages — the mission clock is monotonic milliseconds since mission load, and anything that needs wall time maps once through the mission-load instant. Offset changes must be slewed in gradually except at mission load, or the puppet twitches when the estimate moves.
- Roadmap items, risks or estimates that change: none materially — Month 4's "sync position at 20–30 Hz with smoothing" and "message log with mission-clock timestamps" get their time base. This does not touch [R4](../../WOA_COOP_ROADMAP.md#part-7-risk-register) (the games stay non-deterministic; the clock only aligns timestamps, not world state).
- When to revisit this: if interpolation artifacts show up at 150 ms in Month 4 testing (may need host-timestamped input delay compensation), or if the relay turns out to make routes strongly asymmetric (then sample before and after the relay and compare).

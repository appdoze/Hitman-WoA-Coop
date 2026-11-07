# Architecture decisions

One file per decision from [roadmap Part 1](../../WOA_COOP_ROADMAP.md#part-1-architecture-decisions). The roadmap lists *what* has to be decided; these files record *what was chosen and why*, so nobody has to rediscover the reasoning later.

## How to use this folder

1. Pick an open decision from the table below.
2. Copy [TEMPLATE.md](TEMPLATE.md) to `DNNN-short-name.md`, using the number from the table.
3. Fill it in and open a pull request with the `decision` label. Set the status to **Proposed**.
4. Once it's agreed, set the status to **Accepted**, tick the item in roadmap Part 1, and update the table below.

**Where to track what:** the roadmap's Part 1 lists what has to be decided. Each decision's issue is for discussion until a decision file exists; from then on, the file in this folder is the source of truth. Change the file, not the issue, and don't copy the outcome back into the issue.

Don't edit an accepted decision to change it. Write a new one, and set the old one's status to **Replaced by DNNN**.

Decisions with a linked issue don't depend on spike results and can be settled in Months 1–2. The rest depend on [spike results](../spikes/). Decide them at the [Month 3 decision gate](../../WOA_COOP_ROADMAP.md#decision-gate-end-of-month-3) at the latest.

## Decisions

| # | Decision | Status | Outcome |
|---|---|---|---|
| D001 | Network model | Open | – |
| D002 | Two network channels | Open | – |
| D003 | Peacock server setup | Open | – |
| D004 | Player identity | Open | – |
| D005 | Event authority for scoring | Open | – |
| D006 | Version locking ([#16](https://github.com/appdoze/Hitman-WoA-Coop/issues/16)) | Open | – |
| D007 | Protocol versioning ([#17](https://github.com/appdoze/Hitman-WoA-Coop/issues/17)) | Open | – |
| D008 | Message format ([#18](https://github.com/appdoze/Hitman-WoA-Coop/issues/18)) | Open | – |
| D009 | Randomness | Open | – |
| D010 | Clock and timing ([#19](https://github.com/appdoze/Hitman-WoA-Coop/issues/19)) | Open | – |
| D011 | Pause behaviour | Open | – |
| D012 | Difficulty and settings ([#23](https://github.com/appdoze/Hitman-WoA-Coop/issues/23)) | Open | – |
| D013 | Crash recovery ([#20](https://github.com/appdoze/Hitman-WoA-Coop/issues/20)) | Open | – |
| D014 | [Licence](D014-licence.md) | Accepted | AGPL-3.0 for the whole repository |
| D015 | Installed mods must match ([#21](https://github.com/appdoze/Hitman-WoA-Coop/issues/21)) | Open | – |
| D016 | DLC and location ownership | Open | – |
| D017 | Steam and Epic together ([#24](https://github.com/appdoze/Hitman-WoA-Coop/issues/24)) | Open | – |
| D018 | Network privacy ([#22](https://github.com/appdoze/Hitman-WoA-Coop/issues/22)) | Open | – |

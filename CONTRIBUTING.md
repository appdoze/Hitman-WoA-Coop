# Contributing

Thanks for helping. The project is in the planning and research phase, so the most useful contributions right now are:

- **Research for the risk spikes** in [Part 2 of the roadmap](WOA_COOP_ROADMAP.md#part-2-risk-spikes-do-these-first): what you tried, what happened and how to reproduce it. Write it up with the [spike report template](docs/spikes/TEMPLATE.md).
- **Prior work:** links to earlier co-op or multiplayer attempts and what blocked them
- **Corrections** to the roadmap

## How to contribute

1. Open an issue first for anything bigger than a typo, so work isn't duplicated. Pick the template that fits: spike report, prior work or bug report.
2. Fork the repo and make your change on a branch.
3. Open a pull request that explains what changed and why, and links the issue. The pull request template has a short checklist.

Questions and ideas that aren't a specific task can go in an issue with the `question` label.

New to the project? The [glossary](docs/GLOSSARY.md) explains the terms used here. Issues labelled `help wanted` are open to anyone, and each [spike issue](https://github.com/appdoze/Hitman-WoA-Coop/issues?q=is%3Aissue+label%3Aspike) is a good place to start.

To propose an answer to one of the roadmap's architecture decisions, follow [docs/decisions](docs/decisions/).

Everyone taking part is expected to follow the [code of conduct](CODE_OF_CONDUCT.md).

## Ground rules

- **No game files.** Never commit extracted game resources, executables or anything copyrighted by IO Interactive. Commit tools and code that work against a player's own copy.
- **No decompiled game code.** Don't commit decompiled or disassembled game code, or IDA, Ghidra or Binary Ninja databases. Function names, byte patterns, offsets, struct layouts and your own notes are fine.
- **No piracy support.** Only legitimate Steam and Epic copies are supported.
- **No personal data.** Logs, screenshots and profiles can contain IP addresses, Steam or Epic user IDs and account names. Remove them before posting anything.
- **Back up profiles.** Back up your Peacock `userdata` folder before testing co-op builds.
- **Licence.** By contributing, you agree that your contribution is licensed under [AGPL-3.0](LICENSE).
- **Upstream first.** If a change belongs in Peacock or ZHMModSDK, propose it there too.
- **Security problems** go through [SECURITY.md](SECURITY.md), not public issues.

## Reporting bugs (once builds exist)

Use the bug report issue template. Include:
- Game build and platform (Steam or Epic)
- Mod version and Peacock version
- Installed content mods
- The map, what you did and what happened
- Mod logs and message logs from **both** players

Before attaching logs, remove IP addresses, Steam and Epic user IDs and account names. Never attach your whole Peacock `userdata` folder. If a profile file is needed to reproduce the bug, say so in the issue and the maintainer will ask for it privately.

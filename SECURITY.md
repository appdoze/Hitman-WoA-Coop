# Security

The co-op mod reads network messages inside the game process. A bug in how it handles those messages could let a partner, a relay or anything else that connects crash your game or worse. Please report problems like that privately so they can be fixed before anyone abuses them.

## What to report privately

- Malformed or unexpected network messages that crash the game, the mod, Peacock or the relay
- Anything that lets a partner or third party run code, read files or reach other machines through the mod or the relay
- Ways to use the relay as an open proxy
- Leaks of IP addresses, Steam or Epic user IDs or profile data beyond what [the README's privacy section](README.md#privacy) describes
- Ways to corrupt or overwrite another player's Peacock profile

Ordinary bugs, desyncs and crashes from normal play go in a public [bug report](CONTRIBUTING.md#reporting-bugs-once-builds-exist).

## How to report

Use GitHub's private vulnerability reporting: the **Security** tab of this repository, then **Report a vulnerability**. Include:

- What you found and what an attacker could do with it
- Steps or a small test case that reproduces it
- Game build, platform, mod version and Peacock version

Don't open a public issue, and don't post details in Discussions or Discord, until a fix is released.

## Supported versions

Only the latest release, and the latest commit on `main`, get security fixes. Nothing has been released yet.

## Scope

Problems in Peacock or ZHMModSDK themselves should be reported to those projects. If you're not sure where it belongs, report it here and it will be passed on.

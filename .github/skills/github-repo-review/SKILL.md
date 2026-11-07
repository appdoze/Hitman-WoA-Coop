---
name: github-repo-review
description: Review new changes in a GitHub repository (commits, PRs, issues, CI, settings), fact-check technical claims against upstream source code, fix errors and report. Use when the user says they updated a repo, asks to check or review a repo, or asks "anything else to change/add?" about a GitHub repository, especially docs, roadmap or planning repos that cite other projects' code.
---

# GitHub repo review

Review what changed in a repository since the last review, check every claim it makes against primary sources, fix what's wrong, and report briefly. Planning, roadmap and docs repos are the main target: they make many claims about other projects' code that go stale or were never true.

## Principles

- **Primary sources only.** Check claims against the actual upstream code at the stated commit, not READMEs, comments, memory or search snippets. A comment saying "returns one" can sit right above code that returns three.
- **Read the code, not just grep hits.** A grep match proves a name exists, not that it does what the doc says. Open the surrounding lines.
- **Exact names matter.** Check identifiers, file paths, counts, ports, licences, dates and commit SHAs exactly as written.
- **Owner decisions stay with the owner.** Don't merge outside PRs, accept proposals, close discussions or change settings that express a decision. Review, label and comment instead.
- **Minimal, surgical fixes.** Fix factual errors and broken consistency. Don't restyle the author's prose.
- **Respect branch protection.** If `main` requires PRs, open a PR. If a push bypasses a rule, tell the user.

## Workflow

### 1. Setup

```bash
R=owner/repo
cd /tmp && rm -rf review && git -c credential.helper='!gh auth git-credential' clone -q https://github.com/$R.git review && cd review
```

If git fails because of a broken global config (for example `/dev/null` isn't a device), use an empty config for every command:
`: > /tmp/e.gitconfig; export GIT_CONFIG_GLOBAL=/tmp/e.gitconfig GIT_CONFIG_SYSTEM=/tmp/e.gitconfig`. Environment variables don't persist between separate shell calls, so re-export each time.

### 2. Find everything that changed

Gather this in one batch. Use the last reviewed commit as `BASE`, from earlier in the conversation or a checkpoint.

```bash
git --no-pager log --oneline BASE..HEAD
git --no-pager diff BASE HEAD --stat
git --no-pager diff BASE HEAD > /tmp/review.diff   # read the whole thing; save it to a file if large
gh run list -R $R --limit 5                        # CI on the new commits; investigate any failure
gh pr list -R $R --state all --limit 20            # new PRs, especially from outside contributors
gh issue list -R $R --state all --limit 100 --json number,state,title,milestone
gh repo view $R --json hasWikiEnabled,hasDiscussionsEnabled,repositoryTopics
gh api repos/$R/branches/main --jq .protected; gh api repos/$R/rulesets --jq '.[]|"\(.name) \(.enforcement)"'
```

Notes:
- A failed CI run later fixed by a manual `workflow_dispatch` usually means the push trigger doesn't cover the changed file. Check the `paths:` filter.
- A PR check in `action_required` is a first-time contributor workflow waiting for maintainer approval. Tell the user.
- A 403 from settings, protection or project APIs means the session token can't access them. Report it as something the user must do; don't retry.

### 3. List the checkable claims

Go through the diff and list each claim that can be checked. Track them in a session SQL table if there are many:

| Kind | Examples |
|---|---|
| Code facts | function, hook or class names; which module or file contains what; what a function returns |
| Numbers | counts ("22 mods"), ports, slot counts, limits, sizes, line numbers |
| Provenance | commit SHAs and their dates, versions, licences, authors |
| Behaviour | "X forces Y off", "events are sent as they happen", "only works on the same PC" |
| External | URLs, third-party project status, platform support |
| Internal | references to other sections, issues, decisions, risk IDs, anchors |

### 4. Check them against upstream source

Clone each upstream project. Use a full clone if a specific commit must be checked, otherwise `--depth 1`.

```bash
git clone -q https://github.com/ORG/PROJECT.git /tmp/up && cd /tmp/up
git cat-file -t SHA && git --no-pager log -1 --format='%h %ci' SHA   # SHA exists, date matches
ls DIR | wc -l                                                        # counts
grep -rn "Identifier" path/ | head                                    # existence and location
grep -rhoE 'Hooks::[A-Za-z0-9_]+' Mods/X | sort -u                    # what a module really uses
sed -n 'START,ENDp' file                                              # read the real context
```

Checks that have caught real errors before:
- **Attribution:** the feature exists, but in a different module than the doc says. List what each named module actually uses.
- **Comments next to the code:** a nearby comment can contradict the code under it. Quote the code.
- **Names in data files:** an entity is named `ScoringEventReplicator`, while the doc invented a class name `Z…Replicator`. Use the name as it appears.
- **API scope:** check flags and options, for example whether "list" covers all entities or only some.
- **Data flow:** check whether something is sent one at a time or in batches, and who controls timing. Find the endpoint signature (for example `values: Event[]`).
- **"Commented out"/"dormant"/"unused":** confirm in the build files (CMake, package manifests) and in the code itself.
- **Security claims:** bind addresses (`127.0.0.1` vs `0.0.0.0`), opt-in vs opt-out telemetry, what crosses the network.

Sites outside GitHub may be unreachable from the sandbox. Note which claims you couldn't check rather than guessing.

### 5. Check internal consistency

- Renamed headings break `#anchor` links everywhere: docs, issue bodies, templates.
- Text copied into issue bodies, issue-template dropdowns, index tables and glossaries must match the source doc after edits.
- Numbered sequences (risks R1..Rn, decisions D001..Dn, milestones) need no gaps, duplicates or ordering errors, and new items must be referenced where needed.
- Glossary entries need new terms added and existing entries updated when facts change.
- Status tables must match reality (spike or decision status, ticked checkboxes).
- If the repo says "the file is the source of truth, not the issue", don't copy outcomes back into issues.

### 6. Check CI and repo hygiene

- Workflow triggers cover every file that affects the check, including ignore and config files like `.lycheeignore`.
- A required status check must run on every PR, with no `paths:` filter on `pull_request`, or merges get blocked.
- Actions are pinned to full commit SHAs with a version comment; `permissions:` is least-privilege.
- YAML is valid: `npx -y js-yaml file.yml`.
- Link checks pass locally where reachable. Sites that block bots belong in the ignore file with a comment.
- No secrets, personal data, IPs or user IDs, and no proprietary or decompiled files committed.

### 7. Review outside PRs

For each new PR from someone else:
1. Read the full diff (`gh pr diff N`) and the repo's own rules (CONTRIBUTING, decision or spike workflow, templates).
2. Check its factual claims against upstream source, as in step 4. Check its assumptions against open decisions: a proposal that silently assumes an undecided outcome is a finding.
3. Look for design gaps: wrong clock or time source, threads that stall, failure cases (host machine down, partial delivery), security.
4. Post **one** review comment with `gh pr review N --comment -F file.md`, numbered, specific, and citing file and function. Don't approve, request changes or merge unless asked.
5. Add any labels the contributor asked for but couldn't set from a fork.

### 8. Fix

Edit with exact-match replacement so the edit fails loudly instead of corrupting text:

```python
s = open(p).read()
for old, new in pairs:
    assert s.count(old) == 1, old
    s = s.replace(old, new)
open(p, "w").write(s)
```

Then:
- Validate (YAML, link check) before committing.
- Commit with a message that lists each fix, plus the required `Co-authored-by` trailer.
- Push. Use a branch and PR if protection requires it, or if the user wants to see changes first.
- Update mirrored copies (issue bodies with `gh issue view N --json body` → edit → `gh issue edit N -F file`, and any local copies).
- Wait for CI on the new commit and confirm it passed.

### 9. Report

Keep it short, in three groups:
- **Checked and correct:** one line, so the user knows what was checked.
- **Fixed:** each error and what it was changed to, with the commit SHA.
- **Needs you:** PR approvals or merges, settings blocked by 403, decisions, and rule bypasses.

Clean up temporary clones and files at the end.

## Answering "anything else to add or change?"

Suggest only things backed by what you found. Good sources:
- Claims you couldn't check, which need an experiment or a contact
- Gaps between linked docs (a risk with no mitigation, a decision with no issue, a spike with no decision it feeds)
- Hygiene missing for the repo's stage: templates, CODEOWNERS, Dependabot, branch protection, security policy, licence
- Upstream changes since the docs were written (new commits, releases, modules)

Give a short numbered list and ask whether to apply them, or apply them if autopilot is on.

---
name: changelog
description: >
  Generate changelogs and release notes from Git history.
  Trigger with: "generate changelog", "write release notes",
  "what changed since v1.2", "prepare release", "diff since last tag"
version: 1.0.0
author: jarne
tags: [git, changelog, release, devops]
---

# Changelog Generator Skill

When invoked, follow these steps exactly.

---

## Step 1 — Gather context

Run the following shell commands:

```bash
# List recent tags so the user can pick a range
git tag --sort=-creatordate | head -20

# Get the most recent tag
git describe --tags --abbrev=0 2>/dev/null || echo "NO_TAGS"

# Get remote URL (to build PR links)
git remote get-url origin 2>/dev/null || echo "NO_REMOTE"
```

Then, based on the results:
- If tags exist: run `git log --oneline --no-merges <last_tag>..HEAD`
- If no tags: run `git log --oneline --no-merges -50`
- If the user specified a range (e.g. "since v1.2"): use `git log --oneline --no-merges v1.2..HEAD`

For 200+ commits, do NOT truncate — process all of them.

Detect remote type:
- GitHub: remote URL contains `github.com` → PR link pattern: `https://github.com/<org>/<repo>/pull/<N>`
- GitLab: remote URL contains `gitlab.com` → `https://gitlab.com/<org>/<repo>/-/merge_requests/<N>`
- No remote: skip PR link construction entirely, do not error.

Determine the version label:
- If a `to_tag` was specified by the user, use it.
- Else if tags exist, suggest `<last_tag+1>` (e.g. v1.2.0 → v1.3.0) and ask the user to confirm.
- Else use `Unreleased`.

---

## Step 2 — Parse and categorise commits

Filter out:
- Merge commits (lines starting with "Merge")
- Bot commits (author or message contains: dependabot, renovate, github-actions, snyk)
  UNLESS the user explicitly said to include them.

Group remaining commits into buckets using conventional commit prefixes first,
then fuzzy keyword matching as fallback:

| Bucket             | Match rules (case-insensitive)                                          |
|--------------------|-------------------------------------------------------------------------|
| Breaking Changes   | `BREAKING CHANGE`, `feat!:`, `fix!:`, `!:` anywhere in subject         |
| New Features       | `feat:`, `feature:`, starts with `add `, `implement`, `introduce`      |
| Bug Fixes          | `fix:`, `bug`, `patch`, `resolve`, `closes #`, `fixes #`               |
| Performance        | `perf:`, `optimise`, `optimize`, `speed`, `cache`, `faster`            |
| Developer / Docs   | `docs:`, `chore:`, `refactor:`, `test:`, `ci:`, `build:`, `style:`     |
| Security           | `security`, `cve`, `vuln`, `auth`, `xss`, `injection`, `sanitize`      |
| Other Changes      | anything that did not match above                                       |

If a commit matches multiple buckets, assign to the highest-priority one
(order: Breaking > Security > Features > Bug Fixes > Performance > Developer > Other).

For each commit, extract:
- Short description (strip the conventional prefix, capitalise first letter)
- Commit hash (7 chars)
- PR number if mentioned in commit message as `(#NNN)` or `#NNN`

---

## Step 3 — Generate three output formats

Produce all three formats. Display them clearly separated so the user can copy each.

---

### FORMAT A — CHANGELOG.md style (GitHub standard)

```
## [<version>] — <YYYY-MM-DD>

### Breaking Changes
- <description> ([<hash>](<commit-url>))

### Security
- <description> ([<hash>](<commit-url>))

### New Features
- <description> ([<hash>](<commit-url>))

### Bug Fixes
- <description> ([<hash>](<commit-url>))

### Performance
- <description> ([<hash>](<commit-url>))

### Developer / Docs
- <description> ([<hash>](<commit-url>))

### Other Changes
- <description> ([<hash>](<commit-url>))
```

Rules:
- Omit any section that has zero commits.
- If no remote detected, omit the hyperlink — just show the hash in plain text.
- Sort commits within each section newest-first.

---

### FORMAT B — Customer-facing release notes (non-technical prose)

Write 1–3 short paragraphs. Rules:
- No commit hashes, no branch names, no jargon.
- Written for end-users, not developers.
- Group by theme (e.g. "Reliability improvements", "New capabilities").
- Highlight breaking changes prominently at the top if any exist.
- Tone: clear, positive, professional.

Example structure:
```
**Version <X> — <Month DD, YYYY>**

[If breaking changes]: ⚠️ This release contains breaking changes. <1-sentence summary of impact and migration>.

<Theme 1>: <2–4 sentences describing user-visible improvements>.

<Theme 2>: <2–4 sentences>. 
```

---

### FORMAT C — Internal summary (Slack / Notion post)

Rules:
- Max 10 bullets total. Pick the most impactful changes.
- Use emoji section headers.
- End with: `Full changelog: <link-to-CHANGELOG.md-or-release-page>`

Example:
```
🚀 *Release <version> — <date>*

💥 *Breaking*
• <item>

✨ *Features*
• <item>
• <item>

🐛 *Fixes*
• <item>

Full changelog: <url>
```

---

## Step 4 — Save outputs

1. **CHANGELOG.md** — prepend FORMAT A at the top of the file.
   - If CHANGELOG.md does not exist, create it with a standard header first:
     ```
     # Changelog
     All notable changes to this project will be documented here.
     Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
     ```
   - NEVER overwrite existing entries — always insert above the previous release block.

2. **release-notes.md** — overwrite entirely with FORMAT B.

3. **FORMAT C** — ask the user: "Would you like FORMAT C (Slack/Notion summary) copied to clipboard?"
   If yes, output it inside a code block so they can copy it easily.

---

## Step 5 — Offer follow-up actions

After all outputs are generated, always present this menu:

```
What would you like to do next?

(1) Open a GitHub/GitLab release draft with these notes
(2) Post the Slack summary (FORMAT C) to a channel
(3) Tag this version in git: git tag -a <version> -m "Release <version>"
(4) Nothing — we're done
```

Handle each choice:
- **(1)** Construct the release URL and instruct the user to open it, pre-filled with FORMAT B as the release description.
- **(2)** Ask for the webhook URL or channel name, then post FORMAT C via the Slack MCP or curl if available.
- **(3)** Run `git tag -a <version> -m "Release <version>"` — confirm with the user before executing.
- **(4)** End gracefully.

---

## Error handling

- **No git repo**: Output "This directory is not a git repository. Please run from inside a git project." and stop.
- **No commits in range**: Output "No commits found between <from> and <to>. Nothing to changelog." and stop.
- **Detached HEAD**: Use the full commit hash as the "to" reference instead of a tag.
- **Empty repo (no commits)**: Output "Repository has no commits yet." and stop.

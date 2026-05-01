# Changelog Generator — Claude Code Skill

Automatically generate changelogs, release notes, and Slack summaries from your Git history. Works with any repo that uses [Conventional Commits](https://www.conventionalcommits.org/) — or just plain commit messages.

## What it produces

| Format | Output | Audience |
|--------|--------|----------|
| A | `CHANGELOG.md` (Keep a Changelog style) | Developers / GitHub |
| B | `release-notes.md` (prose paragraphs) | End users |
| C | Slack / Notion bullet summary | Internal teams |

All three are generated every time. You pick what you need.

## Installation

### Option 1 — Drop into any project (recommended)

```bash
# from your project root
mkdir -p skills/changelog
curl -sL https://raw.githubusercontent.com/<your-username>/changelog-skill/main/skills/changelog/SKILL.md \
  -o skills/changelog/SKILL.md
```

Then just talk to Claude Code inside that project:

> "generate changelog since v1.2.0"

### Option 2 — Install as a plugin (Cowork / Claude Code with plugin support)

Download the latest `changelog-skill-distributable.plugin` from [Releases](../../releases) and run:

```bash
claude plugins install ./changelog-skill-distributable.plugin
```

### Option 3 — Clone this repo and copy manually

```bash
git clone https://github.com/<your-username>/changelog-skill.git
cp -r changelog-skill/skills/changelog /your-project/skills/changelog
```

## Usage

Trigger phrases Claude recognises:

- `generate changelog`
- `write release notes`
- `what changed since v1.2`
- `prepare release`
- `diff since last tag`

Claude will ask you to confirm the version, then write all three formats and offer to tag the release or open a GitHub/GitLab release draft.

## How commits are categorised

| Section | Matched by |
|---------|-----------|
| Breaking Changes | `BREAKING CHANGE`, `feat!:`, `fix!:` |
| Security | `security`, `cve`, `vuln`, `auth` |
| New Features | `feat:`, `feature:`, `add`, `implement` |
| Bug Fixes | `fix:`, `bug`, `patch`, `resolve` |
| Performance | `perf:`, `optimise`, `speed`, `cache` |
| Developer / Docs | `docs:`, `chore:`, `refactor:`, `test:`, `ci:` |
| Other | anything else |

Bot commits (dependabot, renovate, github-actions) are skipped by default.

## Repo structure

```
.
├── skills/
│   └── changelog/
│       └── SKILL.md                       # the skill — this is all you need
├── changelog-skill.plugin/                # plugin bundle (directory)
│   ├── manifest.json
│   └── skills/changelog/SKILL.md
├── changelog-skill-distributable.plugin  # installable zip
└── README.md
```

## Requirements

- Claude Code with shell access (`git` must be in PATH)
- A git repository with at least one commit
- No external dependencies — pure shell + markdown

## License

MIT

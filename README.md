# Automate-Publishing-Pre-Release-Qstrike-Builder

A Claude Code skill that guides you through publishing a pre-release version of QStrike Builder to npm from the `staging` branch.

```
To open a folder on MacOS: open ~/.claude 
```

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill — auto-loaded by Claude Code to guide the publish workflow |
| `GITHUB_CLI_SETUP.md` | One-time GitHub CLI setup (required for automated PR creation) |

## Install the Skill

### 1. Create the skills directory (if it doesn't exist)

```bash
mkdir -p ~/.claude/skills/publish-prerelease-qstrike-builder
```

### 2. Copy SKILL.md into it

If you received the file directly:

```bash
cp /path/to/SKILL.md ~/.claude/skills/publish-prerelease-qstrike-builder/SKILL.md
```

If you're copying from the repo:

```bash
cp .claude/skills/publish-prerelease-qstrike-builder/SKILL.md ~/.claude/skills/publish-prerelease-qstrike-builder/SKILL.md
```

### 3. Verify it's in place

```bash
ls ~/.claude/skills/publish-prerelease-qstrike-builder/
```

Expected output: `SKILL.md`

### 4. Restart Claude Code

The skill auto-loads at session start. Open a new Claude Code session — you should see `publish-prerelease-qstrike-builder` listed under available skills in the session-start message.

## GitHub CLI Setup

The skill uses `gh` to create the Release PR automatically. Install and authenticate once:

### 1. Install GitHub CLI

```bash
brew install gh
```

Verify:

```bash
gh --version
```

> If Homebrew is not installed, get it at https://brew.sh first.

### 2. Authenticate

```bash
gh auth login
```

When prompted:

| Prompt | Choose |
|--------|--------|
| Where do you use GitHub? | **GitHub.com** |
| Protocol | **HTTPS** |
| Authenticate | **Login with a web browser** |

The terminal will display a one-time code like `XXXX-XXXX`. Copy it, press Enter, and paste it into the browser page that opens (`github.com/login/device`).

### 3. Verify

```bash
gh auth status
```

Expected output includes `✓ Logged in to github.com account <your-username>`.

## Usage

Once the skill is installed, start a Claude Code session and say:

> "I need to publish a pre-release of QStrike Builder from the staging branch. The PR was just squash-and-merged. What do I do?"

Claude will follow the full 8-step publish workflow automatically.
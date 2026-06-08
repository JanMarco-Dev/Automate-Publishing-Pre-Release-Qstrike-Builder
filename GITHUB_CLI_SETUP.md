# GitHub CLI Setup for Claude Code

One-time setup so Claude Code can create PRs and interact with GitHub automatically.

## 1. Install GitHub CLI

```bash
brew install gh
```

Verify:

```bash
gh --version
```

> If Homebrew is not installed, get it at https://brew.sh first.

## 2. Authenticate

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

## 3. Verify

```bash
gh auth status
```

Expected output includes `✓ Logged in to github.com account <your-username>`.

## Done

Claude Code can now run `gh` commands on your behalf — creating PRs, listing issues, checking CI status, etc. No further setup needed.

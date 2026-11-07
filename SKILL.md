---
name: publish-to-github
description: Publish a local folder (skill, script, or project) to GitHub: initialize git, commit, create a public/private repo, push, add README and topics, and optionally create a release or fork upstream repos. Use when the user says "publish to github", "push this to github", "create a github repo for X", "fork and publish", or wants to share a local skill/project. Also covers publishing a patched fork of an upstream skill with attribution.
---

# Publish to GitHub

End-to-end workflow for turning a local folder into a published GitHub repository, including forks and patched re-publications.

## When to use

- User says "publish to GitHub", "push this to GitHub", "create a GitHub repo for X"
- User wants to share a local skill/project publicly
- User wants to fork an upstream repo and apply their own patches
- User wants to add README, topics, or a release after publishing

## Prerequisites

- `git` installed (`git --version`)
- `gh` CLI installed and authenticated: `gh auth status` must show "Logged in"
  - If missing: `sudo pacman -S github-cli` (Arch) / `brew install gh` (macOS) / `apt install gh`
  - Auth: `gh auth login --hostname github.com --git-protocol https --web` (device flow; user completes in browser)
- Git identity configured (set if empty):
  ```bash
  git config --global user.name "<name>"
  git config --global user.email "<email>"
  ```
  Or use the GitHub login/email from `gh api user`.

## Workflow A — Publish a local folder as a new repo

1. **Clean the folder.** Remove build artifacts, caches, secrets:
   ```bash
   rm -rf __pycache__ *.pyc node_modules .env
   ```
   Add a `.gitignore` for future protection.

2. **Initialize git.**
   ```bash
   cd <folder>
   git init -b main
   git config user.name "<name>"
   git config user.email "<email>"
   ```

3. **Stage and commit.**
   ```bash
   git add .
   git commit -m "Initial commit: <short description>"
   ```

4. **Create the GitHub repo and push.**
   ```bash
   gh repo create <repo-name> --public --source=. --remote=origin --push
   ```
   Use `--private` instead of `--public` if requested. The command prints the repo URL.

5. **Add README.md** (recommended for skills.sh indexing) and push:
   ```bash
   git add README.md && git commit -m "Add README.md" && git push origin main
   ```

6. **Set topics** for discoverability:
   ```bash
   gh repo edit <owner>/<repo> --add-topic <topic1> --add-topic <topic2>
   ```

7. **Create a release** (optional but recommended for versioning):
   ```bash
   gh release create v1.0.0 --title "v1.0.0 — initial release" --notes "<notes>"
   ```

## Workflow B — Fork an upstream repo

Use when the user wants to keep upstream credit or contribute back.

```bash
gh repo fork <owner>/<repo> --clone=false
```

This creates `phycomp/<repo>` (or the authenticated user's fork) on GitHub without a local clone. To apply local patches, clone the fork, apply changes, commit, push.

## Workflow C — Publish a patched third-party skill

When publishing a modified version of someone else's skill:

1. Copy the skill folder to a temp working dir.
2. Remove caches/secrets; add `.gitignore`.
3. Write a `README.md` that explicitly states:
   - Upstream source URL
   - What changed (the patch)
   - That credit and license belong to the original authors
4. Follow Workflow A steps 2–7.
5. Optionally fork the upstream repo too (Workflow B) so the patch can be PR'd back.

## Gotchas

- **No `gh` CLI?** Install via package manager first; do not use password auth (GitHub removed it).
- **Git identity missing** → commit fails with "Please tell me who you are"; set `user.name`/`user.email`.
- **`__pycache__` / `*.pyc`** must be removed and gitignored before committing Python skills.
- **Global install of skills** reports `PromptScript: does not support global skill installation` — harmless; OpenCode still gets the skill.
- **Device flow auth** expires (~15 min); rerun `gh auth login` if timed out.
- **Large monorepos** (e.g., awesome-copilot, baoyu-skills) take long to clone; prefer `--clone=false` for forks.
- **License:** always preserve upstream license and credit when re-publishing third-party skills.

## Example: publishing this very workflow

```bash
# (already done once)
gh repo create publish-to-github --public --source=. --remote=origin --push
gh repo edit phycomp/publish-to-github --add-topic agent-skill --add-topic github
gh release create v1.0.0 --title "v1.0.0" --notes "Initial release."
```
EOF
echo "written"
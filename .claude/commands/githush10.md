---
description: Push code to GitHub, wire up GitHub Pages via Actions, refresh the README, update the repo's About section with the live link, and run a secrets/PII scan before anything ships.
argument-hint: [commit message]
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git remote:*), Bash(git show:*), Bash(gh:*), Bash(curl:*), Read, Edit, Write, Grep, Glob
---

You are running the `githush10` release runbook for this repo. Execute the steps below **in order**. Do not skip the security scan and do not push anything that fails it.

Optional commit message from the user: $ARGUMENTS (if empty, write your own concise message from the actual diff).

## Step 0 — Orient

- `git status` and `git diff` to see what's changed.
- `git remote get-url origin` to get `OWNER/REPO` (parse it out of the URL; works for both `https://github.com/OWNER/REPO.git` and `git@github.com:OWNER/REPO.git`).
- Note whether `gh` (GitHub CLI) is on PATH (`command -v gh`). If it isn't, and no `GITHUB_TOKEN` env var is set, some steps below (Pages source, repo About) can't be done via API — fall back to printing the exact manual steps/URLs for the user instead of skipping silently.

## Step 1 — Security scan (BEFORE staging anything)

Scan the working tree changes (`git status`, `git diff`, and any new untracked files) for anything that should never leave the machine:

- Cloud/API credentials: AWS (`AKIA[0-9A-Z]{16}`), generic `api[_-]?key`, `secret`, `token`, `password` assignments with real-looking values
- Private key material: `-----BEGIN ... PRIVATE KEY-----`
- Service tokens: `ghp_`, `gho_`, `github_pat_`, `xox[baprs]-`, `sk-`, `AIza`
- `.env` files, `credentials.json`, `*.pem`, `*.key`, database connection strings with embedded passwords
- Real personal data that isn't the site's existing fictional placeholder content (this project's clinic/doctor/patient details are intentionally fake marketing copy — don't flag those; do flag anything that looks like a real person's data, internal URLs, or infrastructure details)
- Any file large/binary enough that it looks accidental (dumps, archives, `node_modules`, build output)

Use Grep across the diff/new files for the patterns above. If you find anything suspicious:

- **Stop.** Do not stage or push it.
- Report exactly what you found and where.
- Ask how to proceed (remove the file, add to `.gitignore`, or confirm it's actually safe) before continuing to Step 2.

If the scan is clean, say so briefly and continue.

## Step 2 — Push code to GitHub

- Stage files **by name** (never a blind `git add -A`/`.`) — review `git status` first and only add what's actually part of this change.
- Commit with a descriptive message (use `$ARGUMENTS` if provided) ending with:
  ```
  Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
  ```
- Push to `origin` on the current branch (this repo's default is `main`).

## Step 3 — GitHub Pages via GitHub Actions

- Check `.github/workflows/deploy.yml`. It should check out the repo, run `actions/configure-pages`, `actions/upload-pages-artifact` (path `.` since this is a static single-file site with no build step), and `actions/deploy-pages`. Create or fix it if missing/broken — don't add a build step, this project has none.
- Make sure the repo's Pages **source** is actually set to "GitHub Actions" (a one-time repo setting, separate from the workflow file):
  - If `gh` is available: `gh api -X PUT repos/OWNER/REPO/pages -f build_type=workflow` (if that 404s because Pages was never enabled, use `-X POST` instead).
  - Else if `GITHUB_TOKEN` is set: same call via `curl -X PUT -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/repos/OWNER/REPO/pages -d '{"build_type":"workflow"}'`.
  - Else: tell the user to open `https://github.com/OWNER/REPO/settings/pages` and set Source to "GitHub Actions" themselves, since this needs credentials you don't have.
- If `gh` is available, optionally confirm the latest workflow run succeeded (`gh run list --limit 1`).

## Step 4 — Professional README

Read the current `index.html` (and any other source files) so the README reflects what's actually there, then write/update `README.md` with:

- A short, accurate project description
- The live site link (see Step 5 for how to derive it) near the top
- A rundown of the page's sections/features
- Local preview instructions (there's no build step — opening `index.html` is enough)
- A short note on how deployment works (push to main → GitHub Actions → Pages)
- Tech stack / notable conventions
- A pointer to `CLAUDE.md` for contributors who want the deeper architecture notes

Keep it professional and concise — this is a portfolio/marketing-site repo, not a framework, so don't over-engineer the doc with sections that don't apply.

## Step 5 — Update repo About + live site link

- Determine the Pages URL: `https://OWNER.github.io/REPO/` unless a `CNAME` file in the repo root says otherwise.
- Update the GitHub repo's description and homepage ("About" panel):
  - If `gh` is available: `gh repo edit OWNER/REPO --description "..." --homepage "https://OWNER.github.io/REPO/"`.
  - Else if `GITHUB_TOKEN` is set: `curl -X PATCH -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/repos/OWNER/REPO -d '{"description":"...","homepage":"https://..."}'`.
  - Else: tell the user the exact description/homepage text to paste in via the pencil icon next to "About" on the repo's GitHub page.
- Verify the live URL actually resolves (e.g. `curl -s -o /dev/null -w "%{http_code}\n" https://OWNER.github.io/REPO/`) before telling the user it's live.

## Step 6 — Wrap up

- `git log --oneline -5` and `git status` to confirm everything pushed cleanly.
- Give the user a short summary: what was pushed, whether Pages/About were updated automatically or need a manual step, the live link, and the result of the security scan.

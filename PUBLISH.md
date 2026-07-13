# Publishing a New Daily Dispatch Edition

The site is served from GitHub Pages off the `main` branch (root) of the
`daily-dispatch` repo. `index.html` is what visitors see; `latest.html` holds
the newest edition you want to promote.

- **Repo:** https://github.com/Shensei/daily-dispatch
- **Remote name:** `origin`
- **Live URL:** https://shensei.github.io/daily-dispatch/

## Authentication (headless pushes)

Pushes authenticate via a repo-local credential store — **no `gh` login is
required** in the automation sandbox. It is configured via:

- Repo-local git config: `credential.helper = store --file=/Users/jackson/Desktop/daily-dispatch/.git-credentials`
- The file `/Users/jackson/Desktop/daily-dispatch/.git-credentials` holds the
  push token in **plaintext** (`chmod 600`). It is listed in `.gitignore` and
  must never be committed.

So the daily automation just needs to run `git add`/`commit`/`push` — git reads
the token from that file automatically. If pushes start failing with an auth
error, the token has likely expired or been rotated; regenerate it on a host
with `gh` via `gh auth token` and rewrite the file (see "Rotating the token").

## Daily publish steps

Run from `/Users/jackson/Desktop/daily-dispatch`:

```bash
cd /Users/jackson/Desktop/daily-dispatch

# 1. Promote the newest edition to the page visitors see.
cp latest.html index.html

# 2. Stage, commit, and push to main (auth comes from .git-credentials).
git add -A
git diff --cached --quiet || git commit -m "Publish Daily Dispatch edition $(date +%Y-%m-%d)"
git push origin main
```

GitHub Pages rebuilds automatically after each push to `main`. The new edition
is usually live within ~1 minute at https://shensei.github.io/daily-dispatch/.

> `git add -A` is safe here because `.git-credentials` is gitignored — it will
> not be staged. Confirm with `git check-ignore .git-credentials` if unsure.

## Rotating the token

On a host that has `gh` authenticated:

```bash
TOKEN=$(gh auth token)
umask 077
printf 'https://Shensei:%s@github.com\n' "$TOKEN" > /Users/jackson/Desktop/daily-dispatch/.git-credentials
chmod 600 /Users/jackson/Desktop/daily-dispatch/.git-credentials
```

## Notes for automation

- Ensure the new edition's HTML is written to `latest.html` before running the
  steps above (the copy to `index.html` is what actually goes live).
- If nothing changed, `git commit` exits non-zero ("nothing to commit"); guard
  for that in scripts, e.g. `git diff --cached --quiet || git commit -m ...`.
- Check build/deploy status any time with:
  `gh api repos/Shensei/daily-dispatch/pages/builds/latest --jq .status`

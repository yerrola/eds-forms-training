# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Full Agent Guidance

See `AGENTS.md` for the complete project overview, code style rules, block authoring conventions, three-phase loading architecture, and publishing process. Follow all instructions there.

## Commands

```sh
npm install                        # install dev dependencies (ESLint, Stylelint)
npm run lint                       # lint JS and CSS
npm run lint:fix                   # auto-fix lint issues
npm run branch -- <branch-name>    # create branch from master, commit, and push (see below)
aem up                             # start local dev server at http://localhost:3000
npx -y @adobe/aem-cli up --no-open --forward-browser-logs  # same, without opening browser
```

There is no build step — the project ships vanilla JS/CSS directly.

### `/branch` — Claude slash command (recommended)

```
/branch feature/my-new-block
/branch fix/header-nav
```

Invoke inside Claude Code. Claude will: detect master/main, stash local changes, create the branch, restore changes, analyse the diff to write a conventional commit message, confirm with you, commit, push, and print the AEM preview URL.

The command file is at `.claude/commands/branch.md`.

### `npm run branch` — PowerShell fallback

```sh
npm run branch -- <branch-name>                          # auto-generate commit message
npm run branch -- <branch-name> -Message "feat(x): y"   # provide your own message
npm run branch -- <branch-name> -NoPush                  # skip push to remote
```

The script (`scripts/git-branch.ps1`):
1. Stashes any local changes, updates `master`/`main`, then creates the new branch.
2. Restores stashed changes and stages everything with `git add -A`.
3. Generates a commit message — tries `claude` CLI first, falls back to heuristics based on changed paths (block name, `scripts/`, `styles/`).
4. Prompts to accept / reject / edit the message, then commits and pushes.
5. Prints the AEM preview URL on success.

## Architecture

- **`scripts/aem.js`** — core AEM library; handles page decoration, block loading, section/icon helpers. **Never modify this file.**
- **`scripts/scripts.js`** — main entry point; calls `loadPage()` which runs three phases: eager (LCP), lazy (rest of page), delayed (martech/analytics via `delayed.js`). Also defines `decorateMain`, `buildAutoBlocks`, and `decorateButtons`.
- **`blocks/{name}/{name}.js`** — each block exports a single `decorate(block)` function; loaded on-demand by `aem.js` when the block appears on the page.
- **`blocks/{name}/{name}.css`** — scoped styles for that block; loaded alongside the JS.
- **`styles/styles.css`** — critical/above-fold global styles (LCP path).
- **`styles/lazy-styles.css`** — non-critical global styles loaded in the lazy phase.

## Key Patterns

**Button authoring:** Links wrapped in `**bold**` become `.button.primary`, `*italic*` become `.button.secondary`, `***bold+italic***` become `.button.accent`.

**Auto-blocking:** `buildAutoBlocks` in `scripts.js` auto-wraps hero images and inlines fragment links (`/fragments/` paths). Add new auto-block rules there.

**CSS scoping:** All block selectors must be prefixed with `.{blockname}` (e.g. `.cards .item`, not `.item`). Avoid `.{blockname}-wrapper` and `.{blockname}-container` — those are reserved by the framework.

**Draft content:** Place static HTML test pages under `drafts/` and pass `--html-folder drafts` to `aem up` to serve them locally without CMS content.

## Environments

- Local: `http://localhost:3000`
- Preview: `https://{branch}--{repo}--{owner}.aem.page/`
- Live: `https://main--{repo}--{owner}.aem.live/`

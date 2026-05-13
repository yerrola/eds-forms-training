Create a new git branch from the default branch of https://github.com/yerrola/eds-forms-training, commit all current changes to it with an appropriate commit message, and push it to remote.

The branch name to create is: $ARGUMENTS

Follow these steps exactly:

## Step 1 — Validate input
If $ARGUMENTS is empty, stop and tell the user: "Usage: /branch <branch-name>"

## Step 2 — Confirm this is the right repo
Run `git remote get-url origin` and verify it points to `github.com/yerrola/eds-forms-training`. If it points elsewhere, warn the user and ask whether to continue.

## Step 3 — Check for pending changes
Run `git status --porcelain` to capture what files are modified, added, or deleted. Save this list — you will need it to write the commit message.

Also run `git diff HEAD` (or `git diff` if nothing is staged) to get the full diff. You will analyse this diff to write the commit message.

## Step 4 — Detect the default branch
Run `git symbolic-ref refs/remotes/origin/HEAD` to detect whether the default branch is `main` or `master`. If the command fails, check `git branch -r` for `origin/main` or `origin/master`.

## Step 5 — Abort if branch already exists
Run `git branch --list $ARGUMENTS`. If output is non-empty, stop and tell the user the branch already exists.

## Step 6 — Stash local changes
If Step 3 found pending changes, run `git stash push --include-untracked -m "branch-cmd stash"` so you can safely switch branches.

## Step 7 — Update default branch and create new branch
```
git checkout <default-branch>
git pull origin <default-branch>
git checkout -b $ARGUMENTS
```

## Step 8 — Restore changes
If you stashed in Step 6, run `git stash pop`.

## Step 9 — Stage all changes
Run `git add -A`, then show `git diff --staged --stat` so the user can see what will be committed.

## Step 10 — Write the commit message
Analyse the staged diff from Step 3. Write a single conventional commit message following this format:

  <type>(<scope>): <short imperative description>

Rules:
- **type**: `feat` (new block/feature/file), `fix` (bug or correction), `style` (CSS-only), `refactor` (restructure without behaviour change), `chore` (config/tooling), `docs` (docs only)
- **scope**: the block name if files are under `blocks/<name>/`; `scripts` if `scripts/` changed; `styles` if only `styles/` changed; `config` for package.json/eslint/stylelint changes
- **description**: imperative mood, lowercase, max 72 chars total
- If multiple unrelated areas changed, use the most prominent one as scope and mention others in an optional body line

Examples:
- `feat(cards): add cards block with image and text layout`
- `fix(header): correct mobile nav z-index overlap`
- `style(hero): adjust font sizes for responsive breakpoints`
- `chore(config): update eslint rules for import extensions`

Do NOT invent changes. Base the message entirely on the actual diff content.

## Step 11 — Confirm with the user
Show the proposed commit message and ask:
"Commit with this message? [Y]es / [n]o / [e]dit"

- **Yes** (default): proceed
- **No**: unstage changes (`git restore --staged .`), tell the user changes remain on the branch uncommitted, and stop
- **Edit**: ask the user to type a replacement message, then use that

## Step 12 — Commit
```
git commit -m "<confirmed message>"
```

## Step 13 — Push to remote
```
git push -u origin $ARGUMENTS
```

If push succeeds, print:
- The branch name
- The AEM feature preview URL:  `https://$ARGUMENTS--eds-forms-training--yerrola.aem.page/`
- A reminder: "Open a PR when ready: gh pr create --base <default-branch>"

If push fails, show the exact error and suggest the user check their GitHub credentials or run `gh auth status`.

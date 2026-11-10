---
name: init-brief
description: >
  Creates or updates the project brief (.ravenwatch.md) that tells the ravenwatch-lite reviewers what the project
  is, who uses it, what data it handles, which checks to run (tests, type checks, linters) and which risks the
  owner has accepted, and has the owner confirm it. Use when the user types /ravenwatch-lite:init-brief, or asks
  to "set up the project brief", "add checks to the review", "accept this risk permanently" or "update the brief".
argument-hint: "[--update]"
allowed-tools: Read, Grep, Glob
---

# Project brief

The brief is a short Markdown file, `.ravenwatch.md`, at the repository root. Every reviewer reads it, so it cuts false alarms and repeated findings. Its frontmatter can list **checks**: the owner's own commands (tests, type checks, linters) that the review offers to run and treats as hard evidence.

The brief can skip reviewers, accept risks and name commands to run, so the owner must see and confirm every part of it. Never write content the owner didn't confirm, and never present confirmation as a formality. `/ravenwatch-lite:review` also shows the whole brief and asks the owner to confirm it on the first review in each Claude session (and again whenever it changes), and asks before running any check.

Arguments received: `$ARGUMENTS`

## 1. Look before asking

1. Find the repository root (`git rev-parse --show-toplevel`). Before reading or writing `.ravenwatch.md` in any way, make sure it is an ordinary file or doesn't exist: a repository can commit it as a symlink to a device or to a private file elsewhere (`~/.ssh/…`, `~/.zshrc`), and reading or writing through the link would expose or overwrite that file. If `test -L '<root>/.ravenwatch.md'` succeeds, or `git -C '<root>' ls-files -s -- .ravenwatch.md` shows mode `120000`, or the path exists and `test -f` fails, stop: tell the owner `.ravenwatch.md` is not a regular file, and that they should remove it themselves (`rm` on a symlink removes the link, not what it points to) before running this again. Write nothing.
   If `.ravenwatch.md` is a regular file, read it (only the first 64 KB: `head -c 65536 '<root>/.ravenwatch.md'`; a longer file isn't a real brief, so stop and tell the owner); this is an update, so keep everything the owner wrote unless they ask to change it. If it came from someone else (for example it was already in a cloned repository), point out its check commands, skip list and accepted risks to the owner.
2. Work out the stack and the commands that already exist. Read `package.json` scripts, `pyproject.toml`/`setup.cfg`/`tox.ini`/`noxfile.py`, `Makefile`, `Cargo.toml`, `go.mod`, `deno.json`, CI workflows, and config files for test runners, type checkers and linters (vitest/jest/mocha/playwright, pytest, tsc/`tsconfig.json`, eslint/biome, ruff/mypy, cargo test/clippy, go test/vet).
3. Draft check commands that already work in this repo and finish on their own:
   - Prefer the project's own scripts (`npm test`, `npm run lint`, `make test`) over raw tool calls.
   - They must not wait for input or watch files. The review sets `CI=1`, which stops vitest and jest watch mode; for other tools add the non-watch flag (`--run`, `--watch=false`).
   - They must not deploy, publish, migrate a real database, push, send email, or call paid services. Leave out anything that touches production. Say why if you leave a script out.
   - Mark a check `blocking: true` only if its failure means the code is broken (tests, type check). Lint and formatting are usually `blocking: false`.
   - Never invent a command the project doesn't have; if there are no tests, say so.
   - Never draft a command that downloads and runs a package: `npx` or `npm exec` without `--no-install`, `pnpm dlx`, `yarn dlx`, `bunx`, `uvx`, `pipx run`, `go run <pkg>@<version>`. Checks run non-interactively, where `npx` installs a missing package without asking (and `npx tsc` installs an unrelated package named `tsc`). Use the project's scripts or its installed binaries, e.g. `npx --no-install tsc --noEmit`.
4. Look for what the app does and who uses it: README, routes, page titles, `wrangler.toml`/IaC (what is deployed and where), forms and fields (what personal data it collects), outside services it calls (email, payments, analytics, AI APIs).

## 2. Ask the owner (one round, AskUserQuestion)

Ask only what you can't tell from the code, at most four questions. Usually:

- **Audience**: who uses it and where (e.g. "public, EU", "only me", "internal team"). This sets how strict the privacy review is.
- **Distribution**: open source (which licence), closed-source commercial, or private/personal. This drives the licence checks.
- **Checks**: show the drafted commands and ask which to include (multi-select), noting which would block.
- **Keep the brief private?** If the repository is or will be public, recommend private: a committed brief tells everyone what you've accepted as risk. Private means the owner adds `/.ravenwatch.md` to `.gitignore` (or to `.git/info/exclude` themselves, for this clone only).

## 3. Write the brief

Write `.ravenwatch.md` with this structure (omit empty sections; keep it under ~80 lines):

```markdown
---
audience: <who uses it, where>
distribution: <open source (MIT) | closed-source commercial | private>
checks:
  - name: tests
    run: npm test
    timeout: 600
    blocking: true
  - name: types
    run: npx --no-install tsc --noEmit
    blocking: true
  - name: lint
    run: npm run lint
    blocking: false
skip: []          # optional; only impact-analyzer, ux-reviewer, test-reviewer, design-reviewer, privacy-reviewer
---
# <Project name>

## About
<2–4 sentences: what it does, main user flows, how it is deployed.>

## Public and private parts
<Which routes/pages are public, which need login, which are admin-only, and how admin access works.>

## Data
<Personal data collected, where it is stored, how long, which outside services receive it.>

## Conventions
<Anything reviewers should respect: e.g. "money is integer cents", "all DB access goes through src/lib/db.js".>

## Accepted risks
- <What is accepted, where (file or area), why, and the date. Be specific so it can't hide new issues.>
```

Rules for **Accepted risks**: add an entry only when the owner explicitly says to accept something, in their words. Each entry must be a one-line `- ` bullet that names the specific issue and place. Never write broad entries ("XSS is fine", "ignore tests"). Confirmed leaked credentials can never be accepted.

## 4. Confirm with the owner

Show the owner the whole file you wrote, then list separately every check command (exactly as it will run), the skip list, the audience and every accepted risk, and point out anything that came from someone other than the owner. Ask them to confirm it. If they want changes, make them and show the result again. If they don't confirm, delete the draft (or, for an update, put back the previous version) and say that reviews will run without it.

Finish with one line: the brief's status, how many checks it has, and that `/ravenwatch-lite:review` will now use it. If the brief is not private, remind them to commit it.

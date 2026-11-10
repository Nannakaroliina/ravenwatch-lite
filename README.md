# ravenwatch-lite

A code reviewer for people who work alone. Before anything reaches GitHub, a team of specialist agents reviews your unpushed commits with an emphasis on **bugs, loopholes and security**. They also check the blast radius, privacy, user experience, tests and design, and can run your own tests. A skeptical verifier throws out false alarms before you get a PASS, WARN or BLOCK verdict.

This is the lite edition of [ravenwatch](https://github.com/Nannakaroliina/ravenwatch). It is made only of agents and skills, with no scripts or hooks, so it installs from the Claude plugin directory. It reviews exactly like ravenwatch. What it leaves out is the enforcement: ravenwatch's push gate, git pre-push hook, brief approval by hash, sandboxed check runner and recorded verdicts. Here the verdict is advice. If you want `git push` to stay locked until a review passes, use the full edition.

## How it works

```
/ravenwatch-lite:review
   │
   ├─ 1. scope        what's about to be pushed (unpushed commits, or full history on a first push),
   │                  or with --full the whole repository at HEAD, split into review groups;
   │                  found with read-only git commands
   │                  + secret sweep: sensitive files added anywhere in the range, gitleaks if installed
   │                  + reads your project brief (.ravenwatch.md) and offers to run its checks
   │
   ├─ 2. triage       picks specialists from what changed
   │
   ├─ 3. specialists  run in parallel ───────────────────────────────────────────────┐
   │    core (can block)                                                              │
   │      bug-hunter            logic errors, edge cases, broken caller contracts     │
   │      security-auditor      untrusted input → dangerous sinks, authn/authz, crypto│
   │      loophole-hunter       business-logic abuse: client trust, races, replays,   │
   │                            limit bypass, workflow skipping, captcha bypass       │
   │      resilience-reviewer   failure paths, partial writes, timeouts, leaks, DoS   │
   │      supply-chain-auditor  new deps, typosquats, install scripts, CVEs, CI pins  │
   │      infra-config-auditor  secrets in every commit, wrangler/AWS/IaC, Actions    │
   │    broader (skipped with --quick)                                                │
   │      impact-analyzer       blast radius: dependents, breaking API/schema changes,│
   │                            migrations, env vars, deploy order, rollback  (blocks)│
   │      privacy-reviewer      personal data in logs/URLs/3rd parties, consent,      │
   │                            retention, special categories (GDPR-minded) (≤ HIGH)  │
   │      ux-reviewer           flows, states, accessibility, mobile, copy  (≤ HIGH)  │
   │      test-reviewer         risky code without tests, tests that can't fail,      │
   │                            weakened/skipped tests, flakiness         (≤ MEDIUM)  │
   │      design-reviewer       architecture fit, duplication, coupling, data model,  │
   │                            over-engineering                          (≤ MEDIUM)  │
   │                                                                                  │
   ├─ 4. finding-verifier  tries to disprove every MEDIUM+ finding, and sets aside ◄─┘
   │                       ones matching your brief's accepted risks
   │
   ├─ 5. verdict      BLOCK (critical/high) · WARN (medium) · PASS
   │
   └─ 6. report       short summary in chat, full report in a private folder in your temporary folder
```

## Components

| Component | What it does |
|---|---|
| `/ravenwatch-lite:review` | Runs the review. Arguments: none (unpushed commits, measured against the upstream or the remote's default branch), `--base <ref>`, `--worktree` (uncommitted changes), `--full` (audit the whole repository at HEAD, not just what changed). Add `--quick` to any of these to run only the core reviewers, and `--previous <report>` after a BLOCK so the verifier checks each earlier blocking finding is fixed. |
| `/ravenwatch-lite:init-brief` | Drafts the project brief from your repo (stack, commands, routes, data), asks you a few questions, and has you confirm it. `--update` edits an existing brief, for example to accept a risk permanently. |
| 12 agents | The eleven specialists above plus `finding-verifier`. All are read-only and never run your code. Only the brief's checks run code, and only the ones you pick each time. |

## Severity

| Reviewer | Highest severity | Effect |
|---|---|---|
| bug, security, loophole, resilience, deps, config, impact | CRITICAL | can BLOCK |
| ux-reviewer | HIGH, only when a user can't complete a core task (including accessibility blockers) | can BLOCK |
| privacy-reviewer | HIGH (personal data leaking by design, special categories, real personal data committed) | can BLOCK |
| your checks (from the brief) | a `blocking: true` check that fails or times out is HIGH; the same for a non-blocking check is MEDIUM | can BLOCK |
| licence findings | MEDIUM | can WARN, never BLOCK |
| test-reviewer, design-reviewer | MEDIUM | can WARN, never BLOCK |

BLOCK means "don't push this yet". Nothing stops the push itself. To enforce it, protect the branch on your git host with required status checks or reviews.

## Project brief

`.ravenwatch.md` tells the reviewers what your project is, so they raise fewer false alarms and stop repeating things you've decided to accept. Create it with `/ravenwatch-lite:init-brief`.

```markdown
---
audience: public, EU
distribution: closed-source commercial
checks:
  - name: tests
    run: npm test
    blocking: true
  - name: lint
    run: npm run lint
    blocking: false
skip: []            # optional: impact-analyzer, ux-reviewer, test-reviewer, design-reviewer, privacy-reviewer
---
# Cabin booking
## About
## Public and private parts
## Data
## Conventions
## Accepted risks
- WINTER discount code is public on purpose (marketing), public/book.js — 2026-10-27
```

The brief can skip reviewers, accept risks and name commands, so nothing in it counts until you confirm it. `init-brief` shows you every check command, skipped reviewer and accepted risk before you confirm it. Because this edition stores no approval, a review also shows you the whole brief and asks you to confirm it on the first review in each Claude session and whenever it changes. That covers a brief that arrived with a clone or a pull, too. Before running checks, a review always asks which ones to run and quotes each command exactly. Checks run the committed code at HEAD with your permissions and `CI=1`. They need a clean working tree. Only pick commands you'd run yourself. Accepted risks only cover the specific issue and place they name, never CRITICAL findings or leaked credentials.

For public repos, keep the brief private by adding `/.ravenwatch.md` to `.gitignore`: a committed brief tells everyone which risks you've accepted.

## Setup

Requirements: `git`. Optional: if [gitleaks](https://github.com/gitleaks/gitleaks) or `osv-scanner` is installed, the review uses it.

Install from the Claude plugin directory, or add this repository as a marketplace in Claude Code:

```
/plugin marketplace add Nannakaroliina/ravenwatch-lite
/plugin install ravenwatch-lite@ravenwatch-lite
```

Then restart Claude Code (or run `/reload-plugins`).

## Day-to-day

1. Commit as usual.
2. Run `/ravenwatch-lite:review`.
3. BLOCK → ask Claude to fix the findings, commit, then `/ravenwatch-lite:review --previous <report>` (Claude gives you the exact command).
4. Push.

## Limits

- The reviewers are AI models: they reduce risk, they don't certify code as safe. Keep tests, dependency updates and secret scanning in CI too.
- There is no deterministic secret scanner. `infra-config-auditor` searches every commit in the range for credentials, and gitleaks runs if you have it. Installing gitleaks is the best way to make the sweep reliable.
- Nothing is enforced. The verdict isn't recorded anywhere, and nothing stops `git push`, from you or from Claude. Protect the branch on your git host for real enforcement, or use the [full edition](https://github.com/Nannakaroliina/ravenwatch) for a local push gate.
- Claude Code asks your permission for the git commands a review runs, as for any other command. If you allow read-only git commands for the session, reviews run without prompts. Don't allow the check commands that way: picking them each time is your confirmation.

## Privacy and terms

ravenwatch-lite collects no data: no server, no account, no telemetry. See the [Privacy Policy](PRIVACY.md) and [Terms of Use](TERMS.md).

## License

MIT © 2026 Nanna Karoliina

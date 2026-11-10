# Changelog

## 0.1.0 — 2026-11-10

First release of the lite edition, made from [ravenwatch](https://github.com/Nannakaroliina/ravenwatch) 0.5.4 for the Claude plugin directory.

- Same twelve agents and review flow: triage, parallel specialists, skeptical verifier, PASS / WARN / BLOCK verdict.
- No scripts or hooks. The review finds the range with read-only `git` commands, and `infra-config-auditor` plus gitleaks (if installed) take the place of the scripted secret scan.
- Left out: the push gate, `install-git-hook`, recorded verdicts, `--since-last`, brief approval by hash and the sandboxed check runner. The review asks you to confirm a changed brief and to pick which checks to run each time.
- Skills declare only `Read`, `Grep` and `Glob` in `allowed-tools`, so Claude Code asks your permission for every command.
- Without an upstream, the range is measured against the default branch of the remote the branch pushes to; if that doesn't settle it, the review asks you. It never measures against all remote refs, so commits fetched from a fork are still reviewed and a merged-in main isn't.
- The brief is shown in full and confirmed on the first review in each Claude session and whenever its bytes change (`hash-object --no-filters`), including a brief that came with a clone or pull.
- Every agent, the verifier included, reads files with uncommitted changes at HEAD (`git show 'HEAD:<path>'`); `--worktree` lists every untracked file and runs gitleaks on just the changed files.
- Per-commit and secret sweeps include what was changed while resolving merges, octopus merges included (`--cc`, as in the full edition).
- The push remote comes from git itself (`%(push:remotename)`).
- Git commands, the agents' included, run against the repository root, with refs and paths single-quoted (`'` written as `'\''`). The brief is always read from disk, the version you confirmed.
- Reports go to a private `mktemp -d` folder. `--previous <report>` has the verifier check earlier blocking findings that weren't accepted.
- The brief template no longer suggests commands that download packages (`npx --no-install`); supply-chain-auditor checks a repository's `osv-scanner.toml`.

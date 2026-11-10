# Changelog

## 0.1.0 — 2026-11-10

First release of the lite edition, made from [ravenwatch](https://github.com/Nannakaroliina/ravenwatch) 0.5.4 for the Claude plugin directory.

- Same twelve agents and review flow: triage, parallel specialists, skeptical verifier, PASS / WARN / BLOCK verdict.
- No scripts or hooks. The review finds the range with read-only `git` commands, and `infra-config-auditor` plus gitleaks (if installed) take the place of the scripted secret scan.
- Left out: the push gate, `install-git-hook`, recorded verdicts, `--since-last`, brief approval by hash and the sandboxed check runner. The review asks you to confirm a changed brief and to pick which checks to run each time.
- Skills declare only `Read`, `Grep` and `Glob` in `allowed-tools`, so Claude Code asks your permission for every command.

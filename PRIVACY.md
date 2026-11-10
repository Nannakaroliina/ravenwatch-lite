# Privacy Policy

Last updated: 10 November 2026

ravenwatch-lite is a free, open-source plugin for Claude Code, made by Nanna Karoliina ("I", "me"). This policy explains what happens to your data when you use it.

**Short version: ravenwatch-lite collects nothing. It has no server, no account, no analytics and no telemetry. It is a set of instructions and agents that run inside Claude Code on your machine. The code it reviews is read by Claude, so it goes to Anthropic like any code you ask Claude to read. The only files it writes are review reports and copies of your project brief in your system's temporary folder and, if you ask for one, the project brief in your repository.**

## What ravenwatch-lite does with your data

ravenwatch-lite reviews code in a git repository you choose. To do that, Claude:

- reads your repository's files and commits on your machine with read-only `git` commands, and reviews diffs, file contents and commit subjects;
- writes each review report, and a copy of the project brief taken before you're asked to confirm it, to a private `ravenwatch-*` folder (readable only by you, with a random name) in your system's temporary folder (`$TMPDIR` on macOS, usually `/tmp` on Linux);
- runs the check commands (tests, type checks, linters) from your project brief, but only the ones you pick for that review;
- runs `gitleaks` and `osv-scanner` only if you already have them installed.

`/ravenwatch-lite:init-brief` writes the project brief, `.ravenwatch.md`, in your repository's root folder, where it is committed like any other file unless you git-ignore it.

I never receive any of this. ravenwatch-lite contains no code that sends data to me or to any server I run.

## Where your data does go

ravenwatch-lite works through other software, and that software has its own terms:

- **Claude Code and Anthropic.** Claude carries out the review through Claude Code. The code, diffs, commit subjects and reports Claude reads are sent to Anthropic (or to the cloud provider you have configured for Claude Code) as part of your normal Claude Code session, exactly like any other code you ask Claude to read. That is governed by your agreement with Anthropic and [Anthropic's Privacy Policy](https://www.anthropic.com/legal/privacy), not by ravenwatch-lite.
- **osv-scanner.** If it is installed, a review may run it on your lockfiles. It sends the package names and versions listed there to the public OSV vulnerability database. It is a separate program with its own privacy policy. `gitleaks` works offline.
- **Your own checks.** Commands you run from the project brief run as you; what they send anywhere is up to them.

## Personal data

ravenwatch-lite does not ask for, collect or store personal data about you, and it never sends anything to me.

Your repository may contain personal data, for example names or emails in code, test fixtures or commit messages. A review reads it like any other content. Whatever Claude reads is sent to Anthropic, as described above, so check that this is allowed before you review a repository that holds other people's personal data. Reports in the temporary folder may quote the code they discuss.

## Deleting your data

- Delete the `ravenwatch-*` folders in your system's temporary folder to remove review reports and brief copies. Most systems also clear that folder on their own.
- Delete `.ravenwatch.md` if you no longer want the project brief.
- Uninstalling the plugin from Claude Code removes the plugin itself.

## Children

ravenwatch-lite is a developer tool and is not directed at children.

## Changes

If this policy changes, the new version will be published in this file in the repository, with a new date at the top. The repository's history keeps every earlier version.

## Contact

Questions about this policy: open an issue at <https://github.com/Nannakaroliina/ravenwatch-lite/issues>.

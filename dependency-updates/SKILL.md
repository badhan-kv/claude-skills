---
name: dependency-updates
description: Check whether the project's dependencies, libraries, frameworks, base image, host kernel or upstream forks have updates or fixes; list each with an explanation of its impact on this project (performance, security, bug fix, breaking change, irrelevant), then let the user choose what to do. Use when the user invokes /dependency-updates, asks "any updates to our dependencies/libraries?", "what's new upstream?", or wants to triage the Dependency updates inbox or the `Deps:` line in CHECKPOINT.md.
---

# Dependency updates

Goal: turn raw "something changed upstream" findings into a short, decision-ready list, then act
only on what the user chooses. Never change a pin, lock file or image without that choice.

## 1. Gather

1. Read `CHECKPOINT.md` (`Deps:` line) and the "Dependency updates inbox" section of `BACKLOG.md`.
2. Refresh: `python3 tools/dependency_watch.py --force` (about 2 minutes; the GitHub search is
   rate-limited, so use a long timeout). It appends only findings not seen before. If it fails
   or reports errors, say which checks failed and what is therefore missing; do not present a
   partial list as complete.
3. The untriaged items are the `- [ ]` lines of the inbox. If there are none, say so and stop.

## 2. Assess each item (do the lookup, don't guess)

For every item, read the primary source: release notes or changelog (PyPI/GitHub release page),
the PR or issue (`gh pr view` / `gh issue view`, or WebFetch), the commit message. Then check
against this project:

- **Is it used?** Grep the code (`learner/`, `engine/`, `trainer/`, `tools/`, `docker/`) for the
  API or module the item touches. Read `requirements-fork-torchrl.in` for the current pin.
- **Does it remove a workaround or reopen a tracked story?** Search `BACKLOG.md` and `backlog/`
  for the item's issue number or topic (e.g. TR-13b, TR-13h).
- **Security:** for a newer package release, check advisories for the pinned version, e.g.
  `gh api "/advisories?ecosystem=pip&affects=<pkg>@<pin>"`. Note CVEs fixed by the update.
- **Kernel (host OS):** for a kernel package or Ubuntu security notice (USN), read the notice
  (`https://ubuntu.com/security/notices/<id>`) and the package changelog
  (`apt-get changelog linux-image-generic`). Say whether the fixes are exposed on this machine
  (network-facing, local privilege escalation, GPU / driver, container or namespace issues) or
  irrelevant to it (hardware or drivers it does not have). A kernel install needs a reboot:
  check for a running training or render job first, confirm the NVIDIA driver rebuilds for the
  new kernel (`nvidia-smi` afterwards), and note that containers share the host kernel.
  Never install or reboot without the user's choice.
- **Breaking risk:** deprecations, removed APIs, changed defaults, new minimum versions of
  torch / tensordict / CUDA / driver. A newer CUDA image needs the driver on `ew-sim` and the
  cloud hosts to support it.
- **Reproducibility cost:** moving a pin means recompiling the hashed lock, rerunning the
  verification the stories name (golden values, parity checks, step benchmark) and possibly
  rebuilding the image.

Classify each item as one or more of: `bug fix`, `performance`, `security`, `kernel`, `feature`,
`breaking`, `maintenance`, `irrelevant`. Tag every impact claim `verified (source)` or
`unverified`; say plainly when a changelog was not found or not read.

## 3. Present

One table, most relevant first, then one line per irrelevant group (e.g. "12 unrelated
pytorch/rl PRs on LLM tooling"):

| Item | What changed | Impact on this project | Class | Risk / effort | Suggested action |

Keep "Impact on this project" concrete: name the file, story or workaround affected and the
direction (faster / safer / fixes X / needs code change in Y). Lead with a two-line summary of
what matters most. Do not pad with items that do not touch the project.

## 4. Let the user choose

Use AskUserQuestion. Group items so there are at most four questions, each item or group with
options such as: adopt now, make it a backlog story for later, ignore (with reason), dig
deeper first. Put the recommended option first. Do not ask a question whose answer the table
already makes obvious (security fix on an actively used package: recommend adopt).

## 5. Act and record

- **Ignore / later:** tick the `- [ ]` line in `BACKLOG.md` and append the decision and reason
  after it (`- [x] ... -- ignore (2026-MM-DD): not used by this project`). For "later", add a
  story to `backlog/` and the index in `BACKLOG.md`, and link it from the line.
- **Adopt:** make it a story (per the project's agile workflow), then bump the pin in
  `requirements-fork-torchrl.in`, recompile the lock with the command in that file's header,
  and verify with the project's tests plus a smoke test before marking it done. Append a dated
  change entry that quotes the old pin (appended, never overwritten).
- Update the `Deps:` line in `CHECKPOINT.md` (or remove it when no `- [ ]` lines remain).
- Project files must not mention the assistant or AI tooling; write the decisions neutrally
  ("decided: adopt", not "Claude suggested").

# spec-and-proof

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![Python 3](https://img.shields.io/badge/python-3-blue.svg)

Two [Agent Skills](https://agentskills.io/specification) that make AI coding agents **plan before they build** and **prove what they claim**.

| Skill | What it does |
|---|---|
| **spec-sprint** | Turns a vague idea into a short specification (`SPEC.md`) and an ordered task plan (`PLAN.md`) before any code is written. |
| **proof-build** | Implements one task at a time, runs the checks that exist, and reports each one as `PASS`, `FAIL`, `NOT RUN`, or `BLOCKED`. It never claims a result it did not observe. |

```
idea -> SPEC.md -> PLAN.md -> one task -> tests -> evidence-based report
        (spec-sprint)                     (proof-build)
```

Each skill works alone. Together they share one small contract: every task in `PLAN.md` has a **Done when** condition and a **Verify with** check, and proof-build reads them.

## Why this exists

Coding agents fail in two repeatable ways:

1. **They start building before the goal is clear**, so the first hour goes to the wrong thing.
2. **They finish with "all tests pass"** when they ran nothing, ran something unrelated, or changed the tests until they went green.

These skills are plain Markdown instructions aimed at those two failures. There is no runtime, no API key, no network call, and no model call anywhere in the repository.

## Quick start

A skill is a folder whose name matches the `name` in its `SKILL.md`. Copy the whole folder, because the templates in `assets/` travel with it. Each skill is self-contained, so you can install one without the other.

### Claude Code

**Windows (PowerShell)**

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse skills\spec-sprint "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse skills\proof-build "$env:USERPROFILE\.claude\skills\"
```

**macOS / Linux**

```bash
mkdir -p ~/.claude/skills
cp -r skills/spec-sprint ~/.claude/skills/
cp -r skills/proof-build ~/.claude/skills/
```

To install for a single project instead (commit it and your team gets it too), use `.claude/skills` inside that project. These locations come from [Claude Code's skills documentation](https://code.claude.com/docs/en/skills). If the skills directory did not exist when your session started, restart Claude Code once.

### Other agents

The folders follow the open Agent Skills format, so any client that implements it should accept them. Copy them to the directory that client scans and check its documentation for the path. This repository makes no compatibility claim about any client other than Claude Code.

### No agent at all

Each `SKILL.md` is ordinary Markdown. You can paste it into any AI chat and say "follow these instructions". The AI cannot run your tests for you in that setting, so you run the commands and paste the results back.

## Usage

Ask in plain language. The `description` in each `SKILL.md` is what lets an agent pick the skill up on its own.

```
Use spec-sprint on this idea: "I keep losing track of what I spend..."
```

```
Use proof-build to do the next unchecked task in PLAN.md.
```

In Claude Code, a skill's name also works as a slash command: `/spec-sprint` and `/proof-build`.

## Worked example: expense predictor

To try the workflow on a real task, the skills were applied to a small project: **a browser app that predicts next month's spending**.

- **spec-sprint** produced a spec with numbered acceptance criteria and a four-task plan.
- **proof-build** then built it one task at a time, writing tests first and confirming they failed before writing code.
- The result is a single-page app with a per-category trend forecast, an add-expense form, and 45 browser-runnable tests.

Source and write-up: [expense-predictor](https://github.com/AditiiJ8/expense-predictor)

This is **one worked example, not a measurement.** The skills' instructions were followed manually in a chat session, not by an agent with the skills installed.

A smaller example, with sample output, lives in this repository under [`examples/`](examples/).

## Repository layout

```
skills/
  spec-sprint/
    SKILL.md
    assets/SPEC.template.md, PLAN.template.md
  proof-build/
    SKILL.md
    assets/REPORT.template.md
examples/
  sample-project-idea.md          a vague request
  sample-spec.md, sample-plan.md  what spec-sprint output looks like
  expense-tracker/                the small program the plan describes, with 20 tests
  sample-verification-report.md   what proof-build output looks like
tests/
  validate_skills.py              checks skills against the Agent Skills spec
  test_skill_structure.py         unit tests (validator, skills, examples, links)
DEMO.md                           a five-minute walkthrough
```

The only addition to the minimal skill layout is `assets/`, which the specification defines for templates. Agents load those files when needed, so each `SKILL.md` stays short.

## Validate

Python 3 and nothing else. (On macOS and Linux, use `python3`.)

```bash
python -m unittest discover -s tests -v
python tests/validate_skills.py
python -m unittest discover -s examples/expense-tracker -v
```

1. The first command runs the structural tests.
2. The second prints `[OK]` or `[FAIL]` for each skill.
3. The third runs the example program's own 20 tests.

The structural tests check that:

- both `SKILL.md` files exist, with valid frontmatter, a valid `name` that matches its folder, and a non-empty `description` under 1024 characters
- there are no unknown frontmatter fields, and strings are quoted where YAML would otherwise read a number
- every workflow stage appears as a heading, in order, plus key phrases such as the four status words
- every relative file reference resolves and stays inside its skill folder
- the two skills agree on the plan format and the status vocabulary
- the validator rejects bad input (uppercase names, hyphen rules, wrong folder name, broken links, and so on), so a passing run means something
- the examples exist, and links in the README, DEMO, and examples resolve

The rules come from the specification page at agentskills.io. The project's own validator is a lightweight stand-in for the official `skills-ref` library, which is the authority. To cross-check against it (needs network access, so it is not part of the test suite):

```bash
python3 -m venv /tmp/refenv
/tmp/refenv/bin/pip install "git+https://github.com/agentskills/agentskills.git#subdirectory=skills-ref"
/tmp/refenv/bin/skills-ref validate skills/spec-sprint
/tmp/refenv/bin/skills-ref validate skills/proof-build
```

Both skills passed this check on 2026-10-03, and a deliberately broken copy failed it.

## What the tests do not cover

Passing tests mean the files are well formed and consistent. They do **not** show that an agent follows the instructions well, that the instructions improve results, or that the skills are production-ready. Nobody has measured that yet.

The honest way to test it is to run an agent on real tasks with and without the skills and compare. That has not been done, and contributions that do it are welcome.

## Design choices

- **Two skills, not one**, because planning and building are different moments with different failure modes. A shared plan format connects them.
- **Four status words**, because "not checked" and "could not check" are different facts and a report should not blur them.
- **Approval gates.** proof-build does not commit, push, deploy, or delete without the user's say-so, and spec-sprint never overwrites an existing `SPEC.md` or `PLAN.md`.
- **At most three questions**, each with a default, so planning cannot stall on a user who wants to move.
- **A real demo program**, so the sample report contains captured results instead of invented ones.

## Limitations

- The instructions are tested for structure only. Agent behavior is unmeasured.
- The skills have not been run inside a live agent session.
- The frontmatter parser reads the small YAML subset the specification uses. It rejects anything else instead of guessing, so a valid but unusual YAML file could be flagged.
- Name characters are checked with Python's `isalnum` and `lower`. The official library may treat unusual Unicode letters differently.
- The sample spec, plan, and report are examples written by hand. Only the commands and outputs listed in the report were captured from real runs.
- The `skills-ref` cross-check was run once, at build time. Later edits to the skills need a rerun.

## Author

**Aditi Jaiswal**, [@AditiiJ8](https://github.com/AditiiJ8)

## Attribution and license

Released under the [MIT License](LICENSE).

This project shares a general idea with [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills), which was suggested as inspiration: agent workflows that specify first, work in small steps, and verify. No text or code from that project is used here, and this project is not affiliated with it. The file format follows the public [Agent Skills specification](https://agentskills.io/specification).

# Plan: al-fixes-1

## What Sean asked for
"Start the next repo on the shortlist" (basnijholt/adaptive-lighting), "respecting any CONTRIBUTING.md in the repo as a standing rule": fix real bugs and send each fix to the original project as its own pull request.

## Contribution rules
- **Written rules:** there is no CONTRIBUTING.md, pull request template or code of conduct, on any branch or at the owner's account level. The project sets its rules through its checks and files:
  - **Welcome text for new pull requests** (`.github/config.yml`): the context must be clear, with the related discussion linked, and the change itself clear and self-reviewed.
  - **Tests** (`tests/README.md`): "Add tests for observable behavior: emitted light commands, final states, manual-control events, and timer expiry."
  - **Test check:** the full test suite on every supported Home Assistant version (2025.9 to 2026.9, plus dev), with at least 89% line and 80% branch coverage.
  - **Style check:** pre-commit (ruff, black, whitespace and line-ending checks).
  - **Generated files:** parts of the README, `services.yaml` and the settings text are built by `./scripts/update-generated-content`. A check fails if they are out of date. They are never edited by hand.
  - **Version and release notes:** written by the owner at release time. Contributors don't touch the version or add changelog entries.
- **What the owner does in practice:**
  - Small, well-tested bug fixes get merged. New features are often closed as out of scope.
  - He asks for regression tests that exercise the real code, not a copy of it.
  - Merged outside pull requests use a plain sentence title and a description covering what, why and how it was tested. He squashes them into one commit.
  - He reviews in batches, so expect weeks before a reply. About 30 pull requests are waiting.
- **What we'll follow:** all of the above. One small fix, one pull request. Nothing touches the version, workflows, translations or generated files.

## What we found
- **#1655 is a real bug on today's code, with no pull request yet.** When a light is turned on, the integration sends it new settings with a short fade. If the integration's regular update runs before the light reports those settings, it compares them with the light's old values and wrongly decides that someone changed the light by hand. The light then stops adapting. This was reproduced with the project's own test setup.
- **#1665 is already fixed** by an open pull request from someone else (#1664). We leave it alone.
- **#1568 is already fixed** on the current code, by the owner's own change in September (#1587). The issue was just never closed.
- No other bug without an existing pull request was found.

## What we'll change
1. **#1655 Light wrongly marked as manually controlled right after turning on:** start the "light is still changing" timer as soon as the integration sends the new settings, not only once the light reports them. The regular update already skips a light while this timer runs. This is the change the reporter suggested in the issue.
   - **File:** `custom_components/adaptive_lighting/switch.py` (one added call, with a short comment).
   - **Checks:**
     - a new test in `tests/test_switch.py` that fails on the current code: a regular update runs before the light reports, and the light must not be marked as manually controlled;
     - a second test: once the timer has run out and the light really shows different values, it is still marked as manually controlled, so real manual changes are still detected;
     - the full test suite on the newest and the oldest supported Home Assistant versions;
     - the style check, the coverage floor, and the generated files staying unchanged.
2. **Send upstream, only after Sean's final approval on this PR:**
   - The fix gets its own clean branch from the original project's latest `main`, with no plan file.
   - Sean opens the pull request to basnijholt/adaptive-lighting from his own account. Its full title and description are shown here first.
   - Draft title: "Fix lights marked as manually controlled right after turning on"

## Who's on it
Researcher (done), Engineer, QA, Security Officer, Auditor.

## Risks
- **Slow review:** the owner hasn't merged an outside pull request since 22 September. The fix may wait weeks.
- **Someone else may get there first:** the person who reported #1655 offered to send this fix. If they open a pull request before we send ours, we'll drop ours.
- **Not every light is covered:** lights that can't fade, or changes sent with no fade, don't get this timer, so a smaller race remains for them. The pull request will say so instead of trying to fix that too.
- **Neighbouring changes:** another open pull request (#1640) edits lines close to this one. Our change is one line, so a conflict would be easy to resolve.

## Out of scope
- #1665 (open pull request #1664) and #1568 (already fixed by #1587).
- The remaining race for lights without fades.
- #1636 (similar symptom on one brand of light, but it couldn't be reproduced).
- Feature requests, and issues that already have an open pull request.

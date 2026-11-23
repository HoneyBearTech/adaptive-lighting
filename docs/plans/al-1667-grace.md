# Plan: al-1667-grace

## What Sean asked for
Follow up on the upstream pull request basnijholt/adaptive-lighting#1667: give lights an extra second to report when the command is sent, make the new tests stop depending on the real clock, and reply to the person who reported the issue, accepting their offer to test on their lights. Use the same standing rules as the pyscript work: follow the project's contribution rules; check every comment on the pull request since our last reply, and again just before the final approval; write the description and replies in full, in plain English, for Sean to post himself; and re-check any edit made after the reviews.

## Contribution rules
- **Written rules:** still none (no CONTRIBUTING.md, pull request template or code of conduct). Checked again today; nothing changed since the first job (al-fixes-1).
- **What applies, from the project's files and checks:**
  - Tests for observable behaviour: light commands sent, final states, manual-control events, timer expiry (`tests/README.md`).
  - The full test suite on every supported Home Assistant version (2025.9 to 2026.9, plus dev), with at least 89% line and 80% branch coverage.
  - pre-commit (ruff, black, whitespace checks).
  - Generated files (`./scripts/update-generated-content`) stay unchanged. No version or release-note changes.
- **How we deliver:** the upstream pull request already exists, so the change goes on its branch as a **new commit** (no rewrite, no force push), so the owner sees only what changed. Sean posts the reply and the updated description upstream himself.

## What we found
- **The fix is right, but its time limit is too short.** The pull request starts the "light is still changing" timer when the command is sent, but the timer only lasts as long as the fade (1 second in the reporter's case). The reporter logged 215 commands with a 1-second fade over 10 days: 203 took longer than 1 second for the light to report back, about a quarter by 100 ms or more, the slowest by 376 ms. In all those cases the timer runs out before the light reports, so the bug can still happen.
- **The reporter's suggestion avoids the downside we worried about.** We left out a longer timer because it would apply everywhere. Adding the extra second only when the command is sent keeps the normal length after the light reports, because the timer restarts then (`start_transition_timer`, called from the state-change listener).
- **Only one place reads this timer:** the regular update that skips lights still changing. Nothing else depends on its length.
- **Open review comment:** the automated review on the pull request points out that the new tests use a real 1-second timer. On a slow test machine the first test could fail at random, and the second always waits a full second.
- **All comments checked:** besides the automated review, the only comment since the pull request was opened is the reporter's (2026-11-23). The owner hasn't commented. Upstream `main` hasn't moved since the pull request was opened.

## What we'll change
1. **Extra second only when the command is sent.** In `custom_components/adaptive_lighting/switch.py`, the timer started when the integration sends new settings lasts the fade plus 1 second. The timer started when the light reports is unchanged (fade length only). The extra second is a named constant with a comment pointing to #1655.
   - **Checks:**
     - a test that the timer started at send time lasts fade + 1 second, and that it goes back to the fade length once the light reports;
     - the existing test (a regular update before the light reports doesn't mark the light as manual) still passes, and now also covers a light that is slower than the fade itself (for example 1.4 seconds with a 1-second fade). It must fail without the extra second.
2. **Tests no longer depend on the real clock.** Rework the two new tests in `tests/test_switch.py` so the test decides when the timer ends, instead of waiting for a real timer. No real waits; the result must not depend on how fast the test machine is.
   - **Checks:** both tests pass 20 runs in a row; the second test (once the timer has ended and the light still shows values different from what was sent, it is marked as manually controlled) still passes.
3. **Updated description and reply, for Sean to post upstream (after final approval here):**
   - a full new description for #1667 (not line edits), same layout as now: what, why, notes, how tested. It drops the "few milliseconds" reasoning and credits the reporter's data and idea;
   - a reply to the reporter that thanks them by their GitHub handle, explains the change, and accepts their offer to test it on their Hue lights;
   - a short reply on the automated review comment, saying how the tests now control the timer.
4. **Overall checks:** here, the full test suite on the oldest and newest pinned Home Assistant versions, coverage above the floor, pre-commit, and generated files unchanged. The project's own checks run every supported version (plus dev) on the pull request, but only after the owner allows them to run for a new contributor; today they are waiting for that. We'll say so in the reply.

## Who's on it
Engineer, QA, Security Officer, Auditor. A plan check before this is sent, and a plain-English check on the description and replies. No Researcher: this is a small follow-up and the code was read while planning.

## Risks
- **Lights that never report a change:** if a light is already at the new settings, regular updates skip it for fade + 1 second instead of the fade alone. They are still adapted on the next update after that. The description will say so.
- **Very slow lights:** a light that takes more than a full extra second to report can still hit the bug. The reporter's slowest was 376 ms over, so 1 second leaves a clear margin. We won't make it configurable.
- **The owner may prefer a different length** or a setting. The reply leaves that open.

## Out of scope
- Lights that can't fade, and changes sent with no fade: they still get no timer, as stated in the pull request.
- Any change to how long the timer lasts after the light reports.
- Rewriting the existing commit or rebasing the pull request.
- Merging: this plan pull request on the fork is for approvals only and will be closed, not merged.

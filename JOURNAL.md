# PathReview — Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/codepath-ai201/pathreview/issues/64

**Issue title:** Prompt injection defense doesn't sanitize newline characters in user-supplied resume text

**Tier:** [ ] Tier 1 [x] Tier 2 [ ] Tier 3

**Problem summary:**
PathReview's prompt-injection defense in `safety/prompt_defense.py` sanitizes
resume text before it is inserted into the LLM system prompt, but it only strips
the characters `<`, `>`, and `{`. Because it never handles newline-based
sequences, an adversarial user can embed patterns like `\n---\n` or `\nSystem:`
in their resume to visually terminate the system prompt and inject their own
instructions, hijacking the agent's behavior. The current sanitizer therefore
gives a false sense of safety while leaving the most practical injection vector
open. A successful fix would detect and neutralize these newline/delimiter and
role-label patterns (in addition to the existing character filtering) so that
attacker-controlled resume text can no longer break out of its intended context,
backed by unit tests covering the `\n---\n` and `\nSystem:` cases.

**Branch name:** fix/64-prompt-injection-newline-sanitizer

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/edombelayneh/pathreview/commit/31978e4436f2eca89bd6a4c19bfa81b7e9b8ba58

**Reproduction summary:**
Ran user-supplied resume strings containing `\nSystem:` and `\n---\n` through
`PromptDefense.sanitize()` and observed the output was byte-for-byte identical to
the malicious input — so `is_injection_attempt(sanitize(x))` still returns `True`.
Captured this as two failing unit tests (`TestNewlineSanitizationRepro` in
`tests/unit/test_prompt_defense.py`) that assert the expected post-fix behavior.

**PLAN.md link:** https://github.com/edombelayneh/pathreview/blob/fix/64-prompt-injection-newline-sanitizer/PLAN.md

**Blockers or open questions:**
While reproducing, I found a _second_, pre-existing gap: `is_injection_attempt()`
misses role labels with a space before the colon (e.g. `"System :"`), so the
existing `test_whitespace_variations_detected` test already fails independent of
my change. Open question for Week 9: fix that in the same PR or scope it out.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All implementation sub-tasks from PLAN.md are done. I extended
`PromptDefense.sanitize()` in `safety/prompt_defense.py` to (1) normalize newline
variants (`\r\n`, `\r`, U+2028, U+2029) to `\n`, then (2) neutralize the
newline-anchored injection sequences the detector already flags — separator lines
(`\n---\n`), role labels (`\nSystem:`/`Human:`/`Assistant:`), and override lines
(`\nIgnore`/`Forget`/`Disregard`/`Override`). The two reproduction tests from
Week 8 now pass, and I added `TestNewlineSanitizationFix` (9 tests) covering
CRLF/CR, Unicode separators, case-insensitivity, stacked injections, idempotency,
and a false-positive guard so a normal multi-paragraph resume stays readable and
un-flagged. Scoped `ruff`/`black`/`mypy` are clean on both changed files, and
`pytest tests/unit/test_prompt_defense.py` is 42 passed / 1 failed (only the
pre-existing, unrelated `test_whitespace_variations_detected`). Draft PR is open.

**Next steps:**
Post the draft PR in the cohort Slack channel for peer review, address any
feedback, then mark the PR "Ready for review." Finalize Check-in 2 and submit the
branch URL via the course portal.

**Blockers:**
Resolved the Week 8 open question: the pre-existing `test_whitespace_variations_detected`
failure is a separate bug in `is_injection_attempt()` (not `sanitize()`), so I
scoped it out of this PR and documented it in the PR's "Notes for Reviewers"
rather than expanding scope. No active blockers.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/602

**Branch:** `fix/64-prompt-injection-newline-sanitizer`

**What you built:**
Extended `PromptDefense.sanitize()` to neutralize newline-based prompt-injection
sequences (separator lines, role labels, override lines) after normalizing newline
variants, so `is_injection_attempt(sanitize(x))` is `False` for attacker-controlled
resume text. The change is surgical (legitimate multi-paragraph resumes survive
intact) and idempotent, and the sanitizer patterns mirror the detector's to prevent
the two from drifting apart again.

**Tests added or updated:**
`tests/unit/test_prompt_defense.py` — the two Week 8 reproduction tests
(`TestNewlineSanitizationRepro`) now pass as regression tests, plus a new
`TestNewlineSanitizationFix` class (9 tests) covering CRLF/bare-CR, Unicode line
separators, case-insensitive role labels, override lines, stacked injections,
idempotency, empty/whitespace safety, and a legitimate-resume false-positive guard.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
_(In this codebase there are documented pre-existing failures on the base commit —
repo-wide `make check` reports 182 ruff / 52 black / 5 mypy issues and `make
test-unit` reports 55 pre-existing failures. "Passes" here means my change
introduces **no new** failures; scoped to my two files, ruff/black/mypy are clean
and the only failing test is the pre-existing `test_whitespace_variations_detected`.
See the PR's "Notes for Reviewers.")_

**Draft PR feedback received from:** _[pending — draft PR posted in cohort Slack for peer review; update with reviewer name/handle before final submission]_

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

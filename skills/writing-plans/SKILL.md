---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain.

**Verification is tiered, not uniform.** The project's AGENTS.md is the authority for which code falls into which tier (e.g., the "验证分档: TDD vs run-and-verify" section). Apply TDD only where the tier demands it; use lighter verification for research scripts and decision memos.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `aq-workflow:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Verification Tiers (per task)

Every task is classified into one of three verification tiers. The project's AGENTS.md is the authority for which code falls into which tier.

| Tier | Applies to | Default verification |
|------|-----------|---------------------|
| **Decision memo** | No code; markdown is the deliverable | None — markdown is the artifact |
| **Shared-library TDD** | Code imported by ≥2 callers, or spec pins formulas to field level | Strict TDD: failing test → impl → pass |
| **Research-script run-and-verify** | One-off diagnostic / execution scripts | compile check → quick run → output sanity |

**Tier is per-task, not per-plan.** A single plan commonly mixes tiers (e.g., Task 1 builds a shared lib via TDD, Tasks 2-5 consume it via run-and-verify). Label each task's tier in its header.

### Step granularity by tier

- **TDD tier:** one action per step (2-5 min): write failing test / run-fail / implement / run-pass. This is the only tier using the red-green rhythm.
- **run-and-verify tier:** larger steps OK (10-30 min): create script / compile check / quick run + sanity. 3 steps.
- **Decision memo tier:** write paragraph / update status doc. 2 steps.

Commit is NOT a per-task step in any tier — commit at logical milestones or per user preference.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use aq-workflow:subagent-driven-development (recommended) or aq-workflow:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Verification tiers used:** [e.g., Task 1 TDD, Tasks 2-5 run-and-verify — so the executor sees the full verification distribution at a glance]

---
```

## Task Structure

Each task header states its verification tier. Use the matching template below.

### Shared-library TDD task

````markdown
### Task N: [Component Name]  *(tier: shared-library TDD)*

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS
````

### Research-script run-and-verify task

````markdown
### Task N: [Script Name]  *(tier: research-script run-and-verify)*

**Files:** Create `scripts/<sector>/expNN.py` (ref: `scripts/<sector>/expMM.py` — reference prior implementation, do not reinvent)

- [ ] **Step 1: Create script**

- [ ] **Step 2: Compile check**

Run: `<venv python> -m py_compile scripts/<sector>/expNN.py`
Expected: no output (success)

- [ ] **Step 3: Quick run + sanity check**

Run: `<venv python> scripts/<sector>/expNN.py --quick`
Sanity: row-count in spec-expected range; returns within ±10% band; direction consistent with known baseline. (See AGENTS.md "研究脚本 run-and-verify 的 sanity 判据" for the full criteria.)
````

### Decision-memo task

````markdown
### Task N: [Decision]  *(tier: decision memo)*

- [ ] **Step 1: Write decision paragraph**, citing evidence / data
- [ ] **Step 2: Update relevant README / status doc
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- DRY, YAGNI
- Verification tiered per task — match the tier to the code's nature (see AGENTS.md)
- Commit cadence is NOT prescribed per-task; commit at logical milestones or per user preference

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Tier consistency:** Does each task's stated tier match AGENTS.md's classification rules? A task labeled "run-and-verify" but touching `src/` shared modules is a mismatch — either reclassify to TDD or justify the exception. Also check that the plan header's "Verification tiers used" line matches the per-task labels.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**"Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use aq-workflow:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use aq-workflow:executing-plans
- Batch execution with checkpoints for review

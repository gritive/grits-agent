---
name: done
description: "Complete current work: verify → commit → update Grits → suggest next. Triggers on: 'done', 'task complete', 'finish work', 'grits done'."
---

# Grits Complete Work Workflow

## Step 1: Verify Changes

If there are code changes, run the project's verification commands (lint, build, test).
Check the project's README or CI config for the exact commands.

Fix and retry on failure. Never mark as complete without passing verification.

## Step 2: Commit (On User Request)

If the user wants to commit, commit the changes. Reflect the work in the commit message.
**Only push when the user explicitly requests it.**

## Step 3: Collect Session Context + Decision Extraction (for KPA quality)

Before updating Grits, collect session context AND extract decision rationale.
이 단계의 **primary objective**는 의사결정 맥락 추출입니다. KPA가 "작업 로그"가 아닌 "판단 지도"를 생성하려면 결정의 "왜"가 필요합니다.

```bash
# Commits from this work session
git log --oneline -10 --no-merges

# Changed files summary
git diff --stat HEAD~N..HEAD  # N = number of commits in this session
```

From the above + conversation history, build:

### A. Session Context (기본)
- **Changed files**: key files modified (max 10)
- **Commits**: commit messages from this session

### B. Decision Records (핵심, 0-3개)

이 세션에서 내린 **비자명한 결정**을 구조화하여 추출합니다.

**추출 대상** (대안을 비교하고 하나를 고른 것):
- "A 방식 vs B 방식 중 A를 선택한 이유"
- "기존 구조를 변경한 이유와 대안"
- "특정 도구/라이브러리/패턴을 채택한 근거"

**추출하지 않는 것** (자명한 작업):
- 단순 버그 수정 (수정 방법이 자명한 경우)
- 타이포 수정, 린트 경고 수정
- **0개도 유효** — typo fix 세션에서 결정을 억지로 만들지 않음

**형식** (각 결정당):
```markdown
- **결정**: [무엇을 결정했는가]
  - **트리거**: [왜 이 결정이 필요했는가]
  - **대안**: [고려한 다른 방법]
  - **근거**: [왜 이걸 선택했는가]
```

## Step 4: Update Grits (Required)

Mark the in-progress Grits task as complete. **Always use `work_done`** (not `task_done`) — it updates the description with results before closing.

```
work_done(
  task_id: "<TASK_ID>",
  message: "One-line summary of what was done",
  description: "<existing description>\n\n## Results\n- Changes: ...\n- Files modified: ...\n- Verification: ...\n\n## 의사결정\n- **결정**: ...\n  - **트리거**: ...\n  - **대안**: ...\n  - **근거**: ..."
)
```

**Description rules:**
1. Preserve existing sections (## Problem, ## Approach, etc.) as-is
2. Append a `## Results` section with changes, files, verification outcome
3. Append a `## 의사결정` section with decision records (0-3개) extracted from the session
4. 의사결정 섹션이 KPA의 "판단 지도" 생성 품질을 결정 — 결정의 "왜"가 없으면 KPA 출력은 작업 로그 수준

**Good message examples:**
- "Fixed redirect URL mismatch between email link and actual route path"
- "Batched N+1 dashboard queries: 100+ → 3 queries for 20 projects"
- "Added smart_view filter to task context. Excludes other users' IN_PROGRESS tasks"

**Bad message examples:**
- "Fixed"
- "done"
- "Bug fix"

## Step 4b: Update Related Tasks (Batch Work)

If the completed work addresses **multiple existing Grits tasks** (e.g., batch TODO processing, sprint cleanup), each individual task must also be updated — not just the umbrella task.

**How to detect:** The umbrella task description references multiple task IDs, or the session involved fixes for tasks listed in `task_list()`.

**For each related task**, use `task_update()`:
```
task_update(
  id: "<TASK_ID>",
  title: "Refined title if vague/informal",
  description: "<existing>\n\n## 분석\n- Root cause / investigation findings\n\n## 수정\n- What was changed and why",
  status: "DONE"
)
```

**Rules:**
1. Preserve the original description — append analysis, don't replace
2. **## 분석**: root cause, what was investigated, what was found
3. **## 수정**: what was changed (files, approach). Use **## 결론** instead if no code change was needed (already implemented, design decision deferred, etc.)
4. Refine vague titles to be clear and actionable
5. Batch updates in groups of 5 parallel calls for efficiency

**Never skip this step** — batch work that only completes the umbrella task leaves individual tasks stale in the backlog. The individual task descriptions become the team's knowledge base.

## Step 5: Suggest Next Task

```
task_next()
```

Show the next recommended task and ask if the user wants to continue.

## Important Notes
- Never mark as complete without passing verification (test/lint)
- Commit only when the user explicitly requests it
- Push only when the user explicitly requests it
- **Never skip the Grits update** — always call task_done after code changes
- Completion comments must include **what, why, and how** — others should be able to read it later

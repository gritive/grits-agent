---
description: "현재 작업 완료 처리 + 검증 + 커밋 + Grits 업데이트 + 다음 추천. '작업 완료', '끝', 'grits done' 등으로 호출."
---

# Grits 작업 완료 워크플로우

## 1단계: 변경사항 검증

코드 변경이 있으면 검증을 실행합니다:

```bash
# Backend
make vet && make build && make test

# Frontend (frontend/ 변경이 있을 때)
cd frontend && bun run lint && bun run test
```

검증 실패 시 수정 후 재시도합니다. 검증 통과 없이 완료하지 않습니다.

## 2단계: 커밋 (사용자 요청 시)

사용자가 커밋을 원하면 변경사항을 커밋합니다. 커밋 메시지에 작업 내용을 반영합니다.
**푸시는 사용자가 명시적으로 요청할 때만 합니다.**

## 3단계: Grits 업데이트 (필수)

진행중인 Grits 작업을 완료 처리합니다. MCP 도구를 사용합니다:

```
# 단일 작업
task_done(task_id: "<TASK_ID>", message: "원인/해결방법/영향을 포함한 구체적 코멘트")

# 또는
work_done(task_id: "<TASK_ID>", message: "구체적 코멘트")
```

**좋은 코멘트 예시:**
- "이메일 링크 /a/verify-email → /verify-email 수정. SvelteKit 라우트와 불일치 해소"
- "Dashboard N+1 쿼리 배치화: 프로젝트 20개 기준 100+ → 3 쿼리"
- "task context에 smart_view=my_tasks 추가. 다른 사용자 IN_PROGRESS 제외"

**나쁜 코멘트 예시:**
- "수정 완료"
- "done"
- "버그 수정"

## 4단계: 다음 작업 추천

```
task_next()
```

다음 추천 작업을 보여주고, 계속할지 물어봅니다.

## 주의사항
- 검증(test/lint) 통과 없이 완료 표시하지 않음
- 커밋은 사용자가 명시적으로 요청할 때만
- 푸시도 사용자가 명시적으로 요청할 때만
- **Grits 업데이트를 절대 빠뜨리지 않음** — 코드 수정 후 반드시 task_done 호출
- 완료 코멘트에는 **무엇을 왜 어떻게 했는지** 포함 — 다른 사람이 나중에 읽을 수 있어야 함

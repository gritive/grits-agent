---
name: workflow
description: "Grits workflow guide for AI agents. Use when starting work, tracking tasks, or learning how to integrate Grits into your development workflow. Triggers on: 'how to use grits', 'grits workflow', 'task tracking', 'work tracking'."
---

# Grits Workflow

Grits는 AI 에이전트의 작업을 자동으로 추적하는 Agentic PM & OKR 플랫폼입니다.
이 스킬은 Grits MCP 도구를 사용한 작업 추적 워크플로우를 안내합니다.

## Core Principle

**모든 비자명한 작업은 Grits에 기록한다.** 코드를 수정하기 전에 반드시 작업을 등록한다.

건너뛸 수 있는 경우: 단순 질문 응답, 1줄 수정, 읽기 전용 탐색.

## Standard Workflow

### 1. 현황 파악

```
task_context
```

현재 진행 중인 작업과 오늘 일정을 확인한다. 사용자의 요청이 기존 태스크에 해당하는지 판단한다.

### 2. 작업 시작

**기존 태스크가 있을 때:**
```
work_start(task_id: "<id>", description: "수정 내용 설명")
```

**새 작업일 때:**
```
work_start(title: "명확하고 액션 가능한 제목", description: "무엇을 왜 하는지")
```

**제목 규칙:**
- 비격식적이면 다듬는다: "이것도 좋은 접근이군" → "작업 컨텍스트 기반 프롬프트 생성 기능 검토"
- 기존 태스크도 제목이 모호하면 개선된 제목을 함께 전달한다

**설명 규칙:**
- 항상 포함한다. 기존 태스크에 설명이 없으면 추가한다.

### 3. Linkage 확인

`work_start` 응답에 linkage 섹션이 포함된다:

```json
{
  "linkage": {
    "has_milestone": false,
    "has_kr": false,
    "available_milestones": [...],
    "available_krs": [...]
  }
}
```

- `has_milestone` 또는 `has_kr`이 `false`이면 추천 목록을 확인한다
- `recommended: true`이고 `reason`이 합리적이면 연결한다:
  ```
  milestone_link(task_id: "<id>", milestone_id: "<id>")
  kr_link(task_id: "<id>", kr_id: "<id>")
  ```
- 관련 없으면 건너뛴다

### 4. 작업 수행

코드를 수정하고, 테스트하고, 검증한다. 중간에 진행 기록이 필요하면:

```
task_comment(task_id: "<id>", message: "중간 진행 상황")
```

### 5. 작업 완료

```
task_done(task_id: "<id>", message: "완료 내용 요약")
```

또는:

```
work_done(task_id: "<id>", message: "완료 내용 요약")
```

## Hook Enforcement

Grits plugin은 **PreToolUse hook**으로 코드 수정 전 작업 등록을 강제한다.

- 코드 파일 수정 시 `work_start`가 호출되지 않았으면 `[Grits]` 메시지가 표시된다
- 이 메시지를 받으면 **즉시 `work_start`를 호출**한 후 작업을 계속한다
- 설정/문서 파일(`.md`, `.json`, `.yaml` 등)은 대상에서 제외된다
- `work_start` 호출 후에는 hook이 방해하지 않는다

## Available Tools

| 도구                 | 용도                                   |
| -------------------- | -------------------------------------- |
| `task_context`       | 현재 진행 중 작업 + 오늘 일정          |
| `work_start`         | 작업 시작 (생성 또는 IN_PROGRESS 전환) |
| `work_done`          | 작업 완료 + 코멘트                     |
| `task_create`        | 태스크 생성 (시작하지 않음)            |
| `task_batch_create`  | 여러 태스크 한번에 생성                |
| `task_update`        | 상태/필드 변경                         |
| `task_done`          | 완료 처리                              |
| `task_get`           | 태스크 상세 조회                       |
| `task_list`          | 태스크 목록 조회                       |
| `task_next`          | 다음 추천 작업                         |
| `task_comment`       | 코멘트 추가                            |
| `task_report`        | 작업 리포트                            |
| `milestone_link`     | 마일스톤 연결                          |
| `kr_link`            | KR 연결                                |
| `tag_list`           | 태그 목록                              |
| `tag_create`         | 태그 생성                              |
| `dashboard_insights` | 대시보드 인사이트                      |
| `share_link_create`  | 공유 링크 생성                         |

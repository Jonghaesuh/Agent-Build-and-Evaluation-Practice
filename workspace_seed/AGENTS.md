# AGENTS.md

이 저장소는 고객 PRD를 입력받아 데이터 분석/ML·DL 모델 개발과 성능 평가 보고서 작성을
자동화하는 멀티 에이전트 시스템을 구현합니다. 이 파일은 이 저장소에서 작업하는
코딩 에이전트(Claude Code 등)를 위한 안내 문서입니다.

## 프로젝트 개요

- **목적**: PRD → TASK 구체화 → 데이터 전처리 → 후보 모델 개발 → 성능 평가 → 보고서
  생성까지의 워크플로우를 Orchestrator + 2개 Sub-agent 구조로 지원한다.
- **에이전트 구조**
  - `Orchestrator Agent`: 워크플로우 상태 관리, 서브 에이전트 호출, Human Checkpoint 관리
  - `Data & Model Agent`: 요구사항 분석 → 데이터 전처리 → 모델 개발 → Fallback 코드까지 통합 담당
  - `Reporting Agent`: 성능 평가 → 결과 분석 → PPT/Markdown 보고서 생성 담당
- 각 에이전트의 시스템 프롬프트와 작업 절차는 `skills/<skill-name>/SKILL.md` 파일을
  기준으로 참고한다. 현재 워크스페이스에는 `data-model-agent`와 `reporting-agent` 두
  개의 스킬 디렉터리가 존재한다.

## 디렉토리 구조

```
workspace_seed/
├── AGENTS.md
└── skills/
    ├── data-model-agent/
    │   └── SKILL.md
    └── reporting-agent/
        └── SKILL.md
```

## 셋업 / 실행 명령 (플랫폼 확정 전 임시 규약)

> 현재 이 작업 디렉터리에는 실제 실행 스크립트가 포함되어 있지 않으므로, 각 스킬의
> 세부 절차는 해당 `SKILL.md` 파일을 우선 참고한다.

> 실제 구현 플랫폼(Claude Code 자동화 / 사내 파이프라인 툴 / LangGraph 등)이 확정되면
> 이 섹션을 구체적인 명령으로 교체할 것. 그 전까지는 아래 관례를 따른다.

```bash
# 의존성 설치
pip install -r requirements.txt

# 전처리 코드 실행 (검증용)
python -m src.preprocessing.run --task <task_id>

# 후보 모델 학습/튜닝
python -m src.models.<model_name>.train --config <config_path>

# 성능 평가
python -m evaluation.metrics.run --model-version <version> --data-version <version>
```

## 코드 스타일 / 컨벤션

- Python 기준, PEP 8 준수. 함수/클래스에는 docstring 필수.
- 모든 후보 모델은 공통 인터페이스(`fit(X, y)`, `predict(X)`, `evaluate(X, y)`)를 구현한다.
- 전처리 로직은 원본 데이터를 직접 변형하지 않고, 별도 함수/파이프라인 객체로 구현한다.
- 매직 넘버 금지 — 하이퍼파라미터/임계값은 config 파일 또는 상수로 분리한다.
- 커밋 메시지 형식: `[<agent>] <TASK/체크포인트 참조> - <변경 요약>`
  예: `[data-model-agent] TASK-3 결측치 처리 로직 추가`

## 에이전트 워크플로우 요약

1. **PRD 입력** → Orchestrator가 Data & Model Agent 호출
2. Data & Model Agent가 TASK 구체화 → **[Checkpoint: TASK 확정]**
3. 데이터 프로파일링 및 전처리 설계 → **[Checkpoint: 전처리 로직 확정]**
4. 전처리 코드 구현 → 후보 모델(2~3개) 개발 → **[Checkpoint: 후보 모델 선정]**
5. train/val/test·튜닝 코드, Fallback 코드 완성
6. Orchestrator가 Reporting Agent 호출 → 성능 평가 → 보고서(PPT/MD) 생성
7. **[Checkpoint: 보고서 최종 승인]** → 고객 공유

각 Checkpoint에서는 반드시 실행을 멈추고 사람의 명시적 승인을 받아야 다음 단계로
진행할 수 있다. (자세한 규칙은 `system_prompts.md`의 각 에이전트 섹션 참고)

## 테스트 / 검증 규칙

- 새로 생성된 전처리/모델/평가 코드는 병합 전 반드시 sandbox에서 최소 1회 실행하고
  성공/실패 로그를 PR(또는 산출물 리포트)에 첨부한다.
- 모델 코드는 최소한 다음을 검증한다: (1) 더미/샘플 데이터로 학습-추론 파이프라인이
  에러 없이 동작하는지, (2) Fallback 케이스(결측/이상치/미지 카테고리)가 정의된
  대로 동작하는지.
- 평가 코드는 baseline 값과의 비교 결과가 함께 출력되는지 확인한다.

## 보안 / 데이터 취급 규칙

- 고객 데이터 및 파생 데이터는 승인되지 않은 외부 API/서비스로 전송하지 않는다.
- LLM 호출 시 원본 데이터의 민감 필드는 마스킹하거나 스키마/샘플 수준으로만 전달한다.
- `data/raw/`는 읽기 전용으로 취급하며, 어떤 에이전트도 직접 덮어쓰지 않는다.

## PR / 산출물 제출 규칙

- 모든 산출물(코드/문서)은 "draft" 라벨과 함께 제출하고, 담당 Human Checkpoint 승인 후
  라벨을 제거(최종화)한다.
- PR 설명에는 (1) 어떤 TASK/체크포인트에서 발생한 변경인지, (2) 실행 검증 로그 요약,
  (3) 참조한 PRD 문장 위치 또는 근거 데이터를 포함한다.
- 성능 관련 주장은 항상 baseline 대비 수치로 표기하고, 정성적 과장 표현을 쓰지 않는다.

## 미결정 사항 (Open Questions)

- 구현 플랫폼: Claude Code 기반 자동화 / 사내 파이프라인 툴 / 별도 프레임워크(LangGraph 등)
- Data & Model Agent 내부 단계 분리 여부 (단일 긴 실행 흐름 vs. 멀티 스텝 호출)
- Human Checkpoint 승인 방식 (Slack 알림 / 사내 툴 연동 / CLI 프롬프트 등)

> 원본 기획 문서: `my_agent_plan.md` 참고

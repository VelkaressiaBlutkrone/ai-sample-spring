# AI Guide

> **Workflow 중심 개발**: 개발자는 명확한 순서를 설계하고, 코드는 AI가 작성한다.

---

## 1. 자동 동기화 (최우선 실행)

이 문서를 읽는 반드시 `.ai/skills/*/SKILL.md` 파일을 스캔하여 name, description을 추출하고, 아래 테이블을 실제 파일 내용과 일치하도록 자동 갱신해야 한다.

| 분류 | 이름 | 트리거/용도 | 주요 능력 |
| :--- | :--- | :--- | :--- |
| **Skill** | `deep-interview` | `/deep-interview`, `요구사항 정리`, `기능 정의`, `문제 분석` | 소크라테스식 인터뷰를 통해 모호한 요구사항을 구조화된 명세로 변환 (코딩 전 문제 정의) |
| **Skill** | `deepinit` | `/deepinit`, `코드베이스 문서화해줘`, `AI 가이드 만들어줘`, `AI-CONTEXT.md 생성해줘` | 코드베이스를 인덱싱하고 계층형 AI-CONTEXT.md 파일을 생성하여 프로젝트 구조와 지침을 문서화 |

---

## 2. 참조 규칙 (Memory Load)
아래 파일을 읽고 규칙을 반드시 준수한다.

- **컨벤션**: `.ai/rules/common-rule.md`
- **스킬상세**: `.ai/skills/{스킬명}/SKILL.md`

## 3. AI-CONTEXT
코드 작업 전 해당 디렉토리에 `AI-CONTEXT.md`가 있으면 먼저 읽어라.
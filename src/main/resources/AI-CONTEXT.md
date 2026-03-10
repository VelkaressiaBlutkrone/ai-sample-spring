<!-- Parent: ../../../AI-CONTEXT.md -->

# resources

## 목적
애플리케이션의 설정 정보, 정적 리소스 및 뷰 템플릿을 관리하는 디렉토리입니다.

## 주요 파일
| 파일명 | 설명 |
|--------|------|
| `application.properties` | Spring Boot 애플리케이션의 핵심 설정 파일 (DB, 서버 포트 등) |

## 하위 디렉토리
- `db/` - 데이터베이스 관련 스크립트
- `static/` - 정적 리소스 (JS, CSS, 이미지)
- `templates/` - Mustache 뷰 템플릿

## AI 작업 지침
- **설명 추가**: 설정값 변경 시 해당 설정의 목적을 주석으로 명시합니다.
- **경로 관리**: 리소스 경로는 상대 경로를 사용하여 이식성을 보장합니다.

## 의존성
- 내부: `db`, `templates` 하위 디렉토리 참조
- 외부: Spring Boot Configuration

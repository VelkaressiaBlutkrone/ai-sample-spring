# Demo Project

## 목적
Spring Boot 3.3.4와 Java 21을 기반으로 한 JWT 인증 및 게시판 기능을 제공하는 REST API 프로젝트입니다.

## 주요 파일
| 파일명 | 설명 |
|--------|------|
| `build.gradle` | 프로젝트 의존성 및 빌드 설정 (Spring Boot 3.3.4, Java 21) |
| `AI-GUIDE.md` | AI 개발 가이드 및 스킬 정의 |
| `GEMINI.md` | Gemini CLI 동작을 위한 파운데이션 가이드 |
| `README.md` | 프로젝트 기본 설명 |

## 하위 디렉토리
- `src/main/java/com/example/demo/` - 핵심 도메인 로직 및 컨트롤러
- `src/main/resources/` - DB 설정, Mustache 템플릿, 정적 리소스
- `.ai/` - AI 관련 규칙 및 스킬 정의

## AI 작업 지침
- **Java 21/Spring Boot 3.x**: 최신 자바 문법(var, 레코드 등)과 스프링 부트 기능을 적극 활용한다.
- **DTO 평탄화**: 응답 및 요청 DTO는 가능한 한 평탄한 구조(Flat DTO)를 유지한다.
- **한글 주석**: 모든 중요한 로직에는 친절한 한글 주석을 작성한다.
- **불필요한 코드 제거**: 사용하지 않는 임포트나 중복된 코드를 엄격히 제거한다.

## 테스트
- `./gradlew test` 명령을 통해 전체 테스트를 수행할 수 있습니다.

## 의존성
- 내부: `board`, `reply`, `user`, `_core` 패키지 간 상호 작용
- 외부: Spring Boot Starter (Data JPA, Web, Mustache), H2 Database, Lombok

<!-- Parent: ../AI-CONTEXT.md -->

# templates

## 목적
Mustache 템플릿 엔진을 사용하여 서버 사이드 렌더링(SSR)을 처리하기 위한 뷰 파일들을 관리합니다.

## 주요 파일
| 파일명 | 설명 |
|--------|------|
| `home.mustache` | 메인 페이지 화면 템플릿 |
| `join-form.mustache` | 회원가입 폼 화면 템플릿 |

## 하위 디렉토리
- 없음

## AI 작업 지침
- **시맨틱 마크업**: HTML5 표준 시맨틱 태그를 준수합니다.
- **데이터 바인딩**: Mustache 문법을 사용하여 컨트롤러에서 전달된 데이터를 정확히 표시합니다.
- **주석**: 복잡한 템플릿 로직이나 UI 구성 요소를 한글 주석으로 설명합니다.

## 의존성
- 내부: `UserController`, `BoardController` 등에서 지정된 뷰 이름과 일치해야 함
- 외부: Mustache Template Engine

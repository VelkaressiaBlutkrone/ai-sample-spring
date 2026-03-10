<!-- Parent: ../AI-CONTEXT.md -->

# reply

## 목적
게시글에 달리는 댓글에 대한 생성, 삭제 등의 로직을 담당하는 도메인 패키지입니다.

## 주요 파일
| 파일명 | 설명 |
|--------|------|
| `Reply.java` | 댓글 엔티티 클래스 |
| `ReplyController.java` | 댓글 관련 HTTP 요청을 처리하는 컨트롤러 |
| `ReplyService.java` | 댓글 비즈니스 로직을 처리하는 서비스 |
| `ReplyRepository.java` | 댓글 데이터 접근을 위한 인터페이스 |
| `ReplyRequest.java` | 댓글 요청 관련 DTO |
| `ReplyResponse.java` | 댓글 응답 관련 DTO |

## 하위 디렉토리
- 없음

## AI 작업 지침
- **Modern Java**: Java 21의 `var` 키워드를 적극적으로 사용하여 가독성을 높입니다.
- **DTO 설계**: 계층 간 데이터 전송을 위해 평탄한 구조(Flat DTO)를 유지합니다.
- **주석**: 모든 로직에 친절하고 상세한 한글 주석을 작성합니다.
- **청결한 코드**: 사용하지 않는 임포트와 중복 코드를 즉시 제거합니다.

## 테스트
- `./gradlew test --tests com.example.demo.reply.*`를 통해 댓글 관련 테스트를 실행합니다.

## 의존성
- 내부: `com.example.demo.user.User`, `com.example.demo.board.Board` 참조, `com.example.demo._core.utils.Resp` 사용
- 외부: Spring Data JPA

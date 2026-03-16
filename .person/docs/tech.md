# 기술 명세서: 심플 다중 사용자 블로그

## 1. 기술 스택

| 구분 | 기술 | 버전 |
|------|------|------|
| Language | Java | 21 |
| Framework | Spring Boot | 3.3.4 |
| ORM | Spring Data JPA (Hibernate) | - |
| Template Engine | Mustache | - |
| Database | H2 (In-Memory) | - |
| Build Tool | Gradle | - |
| Utility | Lombok | - |

---

## 2. 아키텍처

### 2.1 패키지 구조 (도메인 기반 플랫 구조)

```
com.example.demo/
├── DemoApplication.java
├── _core/
│   └── utils/
│       └── Resp.java              ← 통일 응답 래퍼
├── board/
│   ├── Board.java                 ← 엔티티
│   ├── BoardController.java       ← SSR (@Controller)
│   ├── BoardApiController.java    ← REST (@RestController, /api)
│   ├── BoardService.java          ← 비즈니스 로직
│   ├── BoardRepository.java       ← JPA 리포지토리
│   ├── BoardRequest.java          ← 요청 DTO (Save, Update)
│   └── BoardResponse.java         ← 응답 DTO (Max, Detail)
├── reply/
│   ├── Reply.java
│   ├── ReplyController.java
│   ├── ReplyApiController.java
│   ├── ReplyService.java
│   ├── ReplyRepository.java
│   ├── ReplyRequest.java          ← 요청 DTO (Save)
│   └── ReplyResponse.java         ← 응답 DTO (Max)
└── user/
    ├── User.java
    ├── UserController.java
    ├── UserApiController.java
    ├── UserService.java
    ├── UserRepository.java
    ├── UserRequest.java            ← 요청 DTO (Join, Login)
    └── UserResponse.java           ← 응답 DTO (Min)
```

### 2.2 레이어 흐름

```
[Client] → Controller/ApiController → Service → Repository → DB
              │                          │
              │ DTO만 주고받음            │ Entity → DTO 변환
              │                          │
              └── Session 인증 확인       └── @Transactional 관리
```

---

## 3. 핵심 설계 결정

### 3.1 인증: HttpSession

| 항목 | 결정 |
|------|------|
| 방식 | HttpSession (Spring Security 미사용) |
| 세션 키 | `sessionUser` |
| 저장 객체 | `UserResponse.Min` (username, email) |
| 주입 방식 | 생성자 주입 (`HttpSession session`) |

**인증 검증 흐름:**
1. Controller에서 `session.getAttribute("sessionUser")`로 로그인 여부 확인
2. null이면 401 응답 또는 로그인 페이지 redirect
3. 본인 확인이 필요한 경우 세션의 username과 리소스 작성자 비교

### 3.2 OSIV: false

- `spring.jpa.open-in-view=false` 고정
- 모든 연관관계 `FetchType.LAZY`
- Service 레이어 내에서 필요한 데이터를 DTO로 변환 후 반환
- N+1 방지: `default_batch_fetch_size=10` 또는 fetch join 사용

### 3.3 DTO 전략

**Request DTO**: 기능명으로 명명
| 도메인 | 클래스 | 필드 |
|--------|--------|------|
| User | `Join` | username, password, email, postcode, address, detailAddress, extraAddress |
| User | `Login` | username, password |
| Board | `Save` | title, content |
| Board | `Update` | title, content |
| Reply | `Save` | comment |

**Response DTO**: 데이터 범위로 명명
| 도메인 | 클래스 | 용도 | 필드 |
|--------|--------|------|------|
| User | `Min` | 세션 저장, 간단 표시 | username, email |
| Board | `Max` | 목록 조회, 작성/수정 응답 | id, title, content, username, createdAt |
| Board | `Detail` | 상세 조회 (댓글 포함) | id, title, content, username, userId, createdAt, isOwner, replies[] |
| Reply | `Max` | 댓글 표시 | id, comment, username, boardId, createdAt |

### 3.4 응답 래퍼

```java
// 성공
Resp.ok(dto)          // → { status: 200, msg: "성공", body: dto }

// 실패
Resp.fail(HttpStatus.BAD_REQUEST, "에러 메시지")
                      // → { status: 400, msg: "에러 메시지", body: null }
```

---

## 4. 데이터베이스 설정

| 항목 | 값 |
|------|-----|
| Driver | org.h2.Driver |
| URL | jdbc:h2:mem:test |
| Username | sa |
| Password | (빈 문자열) |
| H2 Console | 활성화 (`/h2-console`) |
| DDL | JPA 자동 생성 (create-drop) |
| 초기 데이터 | `classpath:db/data.sql` |
| defer-datasource-initialization | true (DDL 후 data.sql 실행) |

---

## 5. 도메인별 구현 상세

### 5.1 User 도메인

**Entity (user_tb)**
| 필드 | 타입 | 제약 | 비고 |
|------|------|------|------|
| id | Integer | PK, IDENTITY | - |
| username | String | UNIQUE | 로그인 ID |
| password | String | NOT NULL, length=100 | 평문 저장 (Spring Security 미사용) |
| email | String | - | - |
| postcode | String | - | 우편번호 |
| address | String | - | 기본 주소 |
| detailAddress | String | - | 상세 주소 |
| extraAddress | String | - | 참고 주소 |
| createdAt | LocalDateTime | @CreationTimestamp | - |

**Repository 메서드**
- `findByUsername(String username)` → `Optional<User>`

**Service 메서드**
| 메서드 | 트랜잭션 | 설명 |
|--------|----------|------|
| `join(UserRequest.Join)` | @Transactional | 회원가입 (중복 체크 후 저장) |
| `login(UserRequest.Login)` | readOnly | 로그인 (username 조회 + 비밀번호 비교) |
| `withdraw(int userId)` | @Transactional | 회원 탈퇴 (댓글 → 게시글 → 사용자 삭제) |

### 5.2 Board 도메인

**Entity (board_tb)**
| 필드 | 타입 | 제약 | 비고 |
|------|------|------|------|
| id | Integer | PK, IDENTITY | - |
| title | String | - | 게시글 제목 |
| content | String | - | 게시글 본문 |
| user | User | FK, @ManyToOne(LAZY) | 작성자 |
| createdAt | LocalDateTime | @CreationTimestamp | - |

**Repository 메서드**
- `findAll(Pageable)` → 기본 페이징
- `findByTitleContainingOrContentContaining(String, String, Pageable)` → 키워드 검색

**Service 메서드**
| 메서드 | 트랜잭션 | 설명 |
|--------|----------|------|
| `findAll(int page)` | readOnly | 최신순 페이징 목록 |
| `findById(int id)` | readOnly | 상세 조회 (댓글 포함) |
| `save(BoardRequest.Save, User)` | @Transactional | 게시글 작성 |
| `update(int id, BoardRequest.Update, int userId)` | @Transactional | 게시글 수정 (본인 검증) |
| `delete(int id, int userId)` | @Transactional | 게시글 삭제 (본인 검증) |
| `search(String keyword, int page)` | readOnly | 제목+본문 키워드 검색 |

### 5.3 Reply 도메인

**Entity (reply_tb)**
| 필드 | 타입 | 제약 | 비고 |
|------|------|------|------|
| id | Integer | PK, IDENTITY | - |
| comment | String | - | 댓글 내용 |
| user | User | FK, @ManyToOne(LAZY) | 작성자 |
| board | Board | FK, @ManyToOne(LAZY) | 게시글 |
| createdAt | LocalDateTime | @CreationTimestamp | - |

**Repository 메서드**
- `findByBoardId(int boardId)` → 게시글별 댓글 목록
- `deleteByUserId(int userId)` → 회원 탈퇴 시 일괄 삭제
- `deleteByBoardId(int boardId)` → 게시글 삭제 시 댓글 일괄 삭제

**Service 메서드**
| 메서드 | 트랜잭션 | 설명 |
|--------|----------|------|
| `save(int boardId, ReplyRequest.Save, User)` | @Transactional | 댓글 작성 |
| `delete(int id, int userId)` | @Transactional | 댓글 삭제 (본인 검증) |

---

## 6. 프론트엔드 (Mustache)

### 6.1 템플릿 구조

```
src/main/resources/templates/
├── home.mustache              ← 게시글 목록 (홈)
├── user/
│   ├── join-form.mustache     ← 회원가입 폼
│   └── login-form.mustache    ← 로그인 폼
├── board/
│   ├── detail.mustache        ← 게시글 상세 (댓글 포함)
│   ├── save-form.mustache     ← 게시글 작성 폼
│   └── update-form.mustache   ← 게시글 수정 폼
└── layout/
    ├── header.mustache        ← 공통 헤더 (네비게이션)
    └── footer.mustache        ← 공통 푸터
```

### 6.2 폼 제출 방식
- 기본: `<form>` 태그 + `name` 속성 → POST 제출 (페이지 이동)
- Ajax: 중복체크 등 부분 갱신이 필요한 경우만 fetch 사용

### 6.3 세션 활용
- `spring.mustache.servlet.expose-session-attributes=true`
- 템플릿에서 `{{#sessionUser}}` / `{{^sessionUser}}`로 로그인 상태 분기

---

## 7. 설정 요약 (application.properties)

| 설정 | 값 | 이유 |
|------|-----|------|
| server.port | 8080 | 기본 포트 |
| encoding | UTF-8 (force) | 한글 지원 |
| spring.jpa.open-in-view | false | 트랜잭션 밖 Lazy 로딩 방지 |
| hibernate.default_batch_fetch_size | 10 | N+1 문제 완화 |
| spring.jpa.show-sql | true | 개발 중 쿼리 확인 |
| hibernate.format_sql | true | SQL 가독성 |
| spring.sql.init.data-locations | classpath:db/data.sql | 초기 더미 데이터 |
| spring.jpa.defer-datasource-initialization | true | JPA DDL 후 data.sql 실행 |

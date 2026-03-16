# ERD: 심플 다중 사용자 블로그

## 1. ER 다이어그램

```
┌──────────────────────────┐
│        user_tb           │
├──────────────────────────┤
│ PK  id          INTEGER  │──┐
│     username    VARCHAR  │  │  (UNIQUE)
│     password    VARCHAR  │  │  (NOT NULL, length=100)
│     email       VARCHAR  │  │
│     postcode    VARCHAR  │  │
│     address     VARCHAR  │  │
│     detail_address VARCHAR│  │
│     extra_address VARCHAR│  │
│     created_at  DATETIME │  │
├──────────────────────────┤  │
│ IDENTITY 자동 생성        │  │
└──────────────────────────┘  │
              │                │
              │ 1              │
              │                │
              ├────────────────┤
              │                │
         ┌────┴────┐     ┌────┴────┐
         │    N    │     │    N    │
         ▼         │     ▼         │
┌──────────────────┤  ┌───────────────────┐
│    board_tb      │  │    reply_tb        │
├──────────────────┤  ├───────────────────┤
│ PK  id   INTEGER │  │ PK  id    INTEGER │
│     title VARCHAR│  │     comment VARCHAR│
│     content TEXT  │  │ FK  user_id INTEGER│──▶ user_tb.id
│ FK  user_id INT  │──▶ user_tb.id        │
│     created_at   │  │ FK  board_id INTEGER│──▶ board_tb.id
│       DATETIME   │  │     created_at     │
├──────────────────┤  │       DATETIME     │
│ IDENTITY 자동 생성│  ├───────────────────┤
└──────────────────┘  │ IDENTITY 자동 생성  │
         │            └───────────────────┘
         │ 1                   ▲
         │                     │ N
         └─────────────────────┘
```

## 2. 테이블 정의

### 2.1 user_tb (사용자)

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | INTEGER | PK, AUTO_INCREMENT | 사용자 식별자 |
| username | VARCHAR(255) | UNIQUE, NOT NULL | 로그인 아이디 |
| password | VARCHAR(100) | NOT NULL | 비밀번호 |
| email | VARCHAR(255) | | 이메일 주소 |
| postcode | VARCHAR(255) | | 우편번호 |
| address | VARCHAR(255) | | 기본 주소 |
| detail_address | VARCHAR(255) | | 상세 주소 |
| extra_address | VARCHAR(255) | | 참고 주소 |
| created_at | DATETIME | AUTO (CreationTimestamp) | 가입일시 |

### 2.2 board_tb (게시글)

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | INTEGER | PK, AUTO_INCREMENT | 게시글 식별자 |
| title | VARCHAR(255) | | 게시글 제목 |
| content | TEXT | | 게시글 본문 |
| user_id | INTEGER | FK → user_tb.id | 작성자 |
| created_at | DATETIME | AUTO (CreationTimestamp) | 작성일시 |

### 2.3 reply_tb (댓글)

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | INTEGER | PK, AUTO_INCREMENT | 댓글 식별자 |
| comment | VARCHAR(255) | | 댓글 내용 |
| user_id | INTEGER | FK → user_tb.id | 작성자 |
| board_id | INTEGER | FK → board_tb.id | 게시글 |
| created_at | DATETIME | AUTO (CreationTimestamp) | 작성일시 |

## 3. 연관관계

| 관계 | 방향 | 타입 | 설명 |
|------|------|------|------|
| user_tb → board_tb | 1:N | @ManyToOne (LAZY) | 사용자 한 명이 여러 게시글 작성 |
| user_tb → reply_tb | 1:N | @ManyToOne (LAZY) | 사용자 한 명이 여러 댓글 작성 |
| board_tb → reply_tb | 1:N | @ManyToOne (LAZY) | 게시글 하나에 여러 댓글 |

## 4. PRD 요구사항 매핑

| PRD 요구사항 | ERD 반영 | 비고 |
|-------------|----------|------|
| 회원가입 (아이디, 비밀번호) | user_tb.username, password | username UNIQUE 제약 |
| 로그인 | user_tb.username, password | Session 기반 인증 |
| 회원 탈퇴 (비노출 처리) | 물리 삭제 + CASCADE | 탈퇴 시 관련 게시글/댓글 함께 삭제 |
| 글 CRUD | board_tb | user_id FK로 작성자 추적 |
| 글 검색 (제목/본문) | board_tb.title, content | LIKE 쿼리 또는 JPQL 검색 |
| 글 목록 (최신순, 페이징) | board_tb.created_at | ORDER BY created_at DESC + Pageable |
| 댓글 작성/삭제 | reply_tb | 1단 구조, 대댓글 없음 |
| 본인 리소스만 관리 | FK user_id 비교 | 세션 사용자 == 작성자 검증 |

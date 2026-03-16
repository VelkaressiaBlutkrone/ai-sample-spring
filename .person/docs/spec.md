# API 명세서: 심플 다중 사용자 블로그

## 1. 공통 사항

### 1.1 응답 형식
모든 REST API 응답은 `Resp<T>` 래퍼를 사용한다.

```json
{
  "status": 200,
  "msg": "성공",
  "body": { ... }
}
```

### 1.2 에러 응답

```json
{
  "status": 400,
  "msg": "에러 메시지",
  "body": null
}
```

### 1.3 인증
- 방식: HttpSession 기반
- 세션 키: `sessionUser` (UserResponse.Min 객체 저장)
- 인증 필요 API에 미로그인 접근 시: `401 Unauthorized`
- 본인 리소스가 아닌 경우: `403 Forbidden`

---

## 2. 회원 API (User)

### 2.1 회원가입
| 항목 | 내용 |
|------|------|
| SSR | `POST /join` → form 제출 → 로그인 페이지 redirect |
| API | `POST /api/join` |

**Request Body (UserRequest.Join)**
```json
{
  "username": "ssar",
  "password": "1234",
  "email": "ssar@example.com",
  "postcode": "12345",
  "address": "서울시 강남구",
  "detailAddress": "101호",
  "extraAddress": "(역삼동)"
}
```

**Response (UserResponse.Min)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "username": "ssar",
    "email": "ssar@example.com"
  }
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 아이디 중복 | 400 | 이미 존재하는 아이디입니다 |
| 필수값 누락 | 400 | username과 password는 필수입니다 |

---

### 2.2 로그인
| 항목 | 내용 |
|------|------|
| SSR | `POST /login` → form 제출 → /home redirect |
| API | `POST /api/login` |

**Request Body (UserRequest.Login)**
```json
{
  "username": "ssar",
  "password": "1234"
}
```

**Response (UserResponse.Min)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "username": "ssar",
    "email": "ssar@example.com"
  }
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 아이디 불일치 | 401 | 아이디 또는 비밀번호가 일치하지 않습니다 |
| 비밀번호 불일치 | 401 | 아이디 또는 비밀번호가 일치하지 않습니다 |

---

### 2.3 로그아웃
| 항목 | 내용 |
|------|------|
| SSR | `GET /logout` → 세션 무효화 → /home redirect |

---

### 2.4 회원 탈퇴
| 항목 | 내용 |
|------|------|
| SSR | `POST /withdraw` → 세션 무효화 → /home redirect |
| API | `DELETE /api/users` |
| 인증 | 필수 |

**동작**: 해당 사용자의 댓글 → 게시글 → 사용자 순서로 물리 삭제 (CASCADE)

**Response**
```json
{
  "status": 200,
  "msg": "성공",
  "body": null
}
```

---

## 3. 게시글 API (Board)

### 3.1 글 목록 조회 (페이징)
| 항목 | 내용 |
|------|------|
| SSR | `GET /` 또는 `GET /home` → 목록 페이지 렌더링 |
| API | `GET /api/boards?page=0` |
| 인증 | 불필요 (비회원 접근 가능) |

**Query Parameters**
| 파라미터 | 타입 | 기본값 | 설명 |
|----------|------|--------|------|
| page | int | 0 | 페이지 번호 (0부터) |

**Response (Page of BoardResponse.Max)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "content": [
      {
        "id": 3,
        "title": "세 번째 게시글",
        "content": "안녕하세요.",
        "username": "cos",
        "createdAt": "2026-03-16T12:00:00"
      }
    ],
    "totalPages": 1,
    "totalElements": 3,
    "number": 0,
    "last": true
  }
}
```

---

### 3.2 글 상세 조회
| 항목 | 내용 |
|------|------|
| SSR | `GET /boards/{id}` → 상세 페이지 렌더링 |
| API | `GET /api/boards/{id}` |
| 인증 | 불필요 |

**Response (BoardResponse.Detail)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "id": 1,
    "title": "첫 번째 게시글",
    "content": "안녕하세요. ssar의 첫 번째 글입니다.",
    "username": "ssar",
    "userId": 1,
    "createdAt": "2026-03-16T12:00:00",
    "isOwner": true,
    "replies": [
      {
        "id": 1,
        "comment": "첫 번째 댓글입니다.",
        "username": "ssar",
        "userId": 1,
        "isOwner": true,
        "createdAt": "2026-03-16T12:00:00"
      }
    ]
  }
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 게시글 없음 | 404 | 게시글을 찾을 수 없습니다 |

---

### 3.3 글 작성
| 항목 | 내용 |
|------|------|
| SSR | `POST /boards` → form 제출 → 상세 페이지 redirect |
| API | `POST /api/boards` |
| 인증 | 필수 |

**Request Body (BoardRequest.Save)**
```json
{
  "title": "새 게시글",
  "content": "게시글 내용입니다."
}
```

**Response (BoardResponse.Max)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "id": 4,
    "title": "새 게시글",
    "content": "게시글 내용입니다.",
    "username": "ssar",
    "createdAt": "2026-03-16T12:00:00"
  }
}
```

---

### 3.4 글 수정
| 항목 | 내용 |
|------|------|
| SSR | `POST /boards/{id}/update` → form 제출 → 상세 페이지 redirect |
| API | `PUT /api/boards/{id}` |
| 인증 | 필수 (본인만) |

**Request Body (BoardRequest.Update)**
```json
{
  "title": "수정된 제목",
  "content": "수정된 내용입니다."
}
```

**Response (BoardResponse.Max)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "id": 1,
    "title": "수정된 제목",
    "content": "수정된 내용입니다.",
    "username": "ssar",
    "createdAt": "2026-03-16T12:00:00"
  }
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 게시글 없음 | 404 | 게시글을 찾을 수 없습니다 |
| 권한 없음 | 403 | 본인이 작성한 게시글만 수정할 수 있습니다 |

---

### 3.5 글 삭제
| 항목 | 내용 |
|------|------|
| SSR | `POST /boards/{id}/delete` → /home redirect |
| API | `DELETE /api/boards/{id}` |
| 인증 | 필수 (본인만) |

**Response**
```json
{
  "status": 200,
  "msg": "성공",
  "body": null
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 게시글 없음 | 404 | 게시글을 찾을 수 없습니다 |
| 권한 없음 | 403 | 본인이 작성한 게시글만 삭제할 수 있습니다 |

---

### 3.6 글 검색
| 항목 | 내용 |
|------|------|
| SSR | `GET /boards/search?keyword=검색어` → 목록 페이지 렌더링 |
| API | `GET /api/boards/search?keyword=검색어&page=0` |
| 인증 | 불필요 |

**Query Parameters**
| 파라미터 | 타입 | 기본값 | 설명 |
|----------|------|--------|------|
| keyword | String | (필수) | 검색 키워드 (제목 + 본문) |
| page | int | 0 | 페이지 번호 |

**Response**: 3.1 글 목록 조회와 동일한 형태

---

## 4. 댓글 API (Reply)

### 4.1 댓글 작성
| 항목 | 내용 |
|------|------|
| SSR | `POST /boards/{boardId}/replies` → form 제출 → 게시글 상세 redirect |
| API | `POST /api/boards/{boardId}/replies` |
| 인증 | 필수 |

**Request Body (ReplyRequest.Save)**
```json
{
  "comment": "댓글 내용입니다."
}
```

**Response (ReplyResponse.Max)**
```json
{
  "status": 200,
  "msg": "성공",
  "body": {
    "id": 4,
    "comment": "댓글 내용입니다.",
    "username": "ssar",
    "boardId": 1,
    "createdAt": "2026-03-16T12:00:00"
  }
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 게시글 없음 | 404 | 게시글을 찾을 수 없습니다 |

---

### 4.2 댓글 삭제
| 항목 | 내용 |
|------|------|
| SSR | `POST /replies/{id}/delete` → 게시글 상세 redirect |
| API | `DELETE /api/replies/{id}` |
| 인증 | 필수 (본인만) |

**Response**
```json
{
  "status": 200,
  "msg": "성공",
  "body": null
}
```

**에러 케이스**
| 상황 | 코드 | 메시지 |
|------|------|--------|
| 댓글 없음 | 404 | 댓글을 찾을 수 없습니다 |
| 권한 없음 | 403 | 본인이 작성한 댓글만 삭제할 수 있습니다 |

---

## 5. SSR 페이지 라우트 (Mustache)

| 메서드 | URL | 설명 | 인증 |
|--------|-----|------|------|
| GET | `/home` | 게시글 목록 (홈) | 불필요 |
| GET | `/join-form` | 회원가입 폼 | 불필요 |
| GET | `/login-form` | 로그인 폼 | 불필요 |
| GET | `/boards/{id}` | 게시글 상세 | 불필요 |
| GET | `/boards/save-form` | 게시글 작성 폼 | 필수 |
| GET | `/boards/{id}/update-form` | 게시글 수정 폼 | 필수 (본인) |
| GET | `/boards/search` | 게시글 검색 결과 | 불필요 |

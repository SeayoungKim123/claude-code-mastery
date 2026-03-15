# 투두리스트 웹 서비스 구현 계획

## Context

현재 `main.js`에 투두 함수 스텁만 존재하는 상태에서, Express 기반 REST API 서버와 바닐라 JS 프론트엔드를 갖춘 완전한 웹 서비스로 확장한다. 데이터는 `todos.json` 파일에 영구 저장된다.

---

## 파일 구조

```
claude-code-mastery/
├── package.json        (NEW - express 의존성)
├── server.js           (NEW - Express 서버 + API 라우트)
├── todos.json          (NEW - 첫 실행 시 자동 생성)
├── public/
│   └── index.html      (NEW - 바닐라 JS 프론트엔드)
├── CLAUDE.md           (UPDATE - 실행 명령어 추가)
└── main.js             (기존 유지 - 변경 없음)
```

---

## 구현 단계

### 1. package.json 생성

```json
{
  "name": "todo-service",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

`npm install` 실행 후 진행.

### 2. server.js 구현

**데이터 모델 (todos.json 내 각 항목):**
```json
{ "id": "uuid-v4", "text": "할 일", "completed": false, "createdAt": "ISO8601" }
```

**파일 I/O 헬퍼:**
- `readTodos()` — `todos.json` 읽기, 없거나 파싱 실패 시 `[]` 반환
- `writeTodos(todos)` — 동기 방식으로 파일 저장 (단일 프로세스 환경에서 race condition 방지)

**API 라우트:**

| 메서드 | 경로 | 동작 | 응답 |
|--------|------|------|------|
| GET | `/todos` | 전체 목록 조회 | 200 + 배열 |
| POST | `/todos` | 항목 추가 (body: `{ text }`) | 201 + 새 항목 |
| PUT | `/todos/:id` | 수정 (body: `{ text?, completed? }`) | 200 + 수정된 항목 |
| DELETE | `/todos/:id` | 삭제 | 204 |

**미들웨어:**
- `express.json()` — JSON body 파싱
- `express.static('public')` — 프론트엔드 서빙
- 에러 핸들러 — 500 응답

**UUID 생성:** `crypto.randomUUID()` (Node.js 내장, 외부 패키지 불필요)

**포트:** `process.env.PORT || 3000`

### 3. public/index.html 구현

단일 파일, 외부 의존성 없음. 인라인 CSS + 바닐라 JS.

**UI 구성:**
- 텍스트 입력 + 추가 버튼 (form)
- 투두 목록 (ul) — 체크박스, 텍스트, 삭제 버튼
- 완료된 항목은 취소선 표시

**JS 함수:**
- `loadTodos()` — GET 요청 후 렌더링
- `addTodo(text)` — POST 요청
- `toggleTodo(id, completed)` — PUT 요청
- `deleteTodo(id)` — DELETE 요청
- `renderTodos(todos)` — DOM 업데이트

### 4. CLAUDE.md 업데이트

실행 명령어 추가:
```
npm install   # 최초 1회
npm start     # 서버 시작 (http://localhost:3000)
npm run dev   # 개발 모드 (파일 변경 시 자동 재시작)
```

---

## 에러 처리

| 상황 | 처리 |
|------|------|
| POST 시 text 없음 | 400 `{ error: "Text is required" }` |
| 존재하지 않는 ID | 404 `{ error: "Todo not found" }` |
| todos.json 손상/없음 | `readTodos()`가 `[]` 반환 |
| 예상치 못한 서버 오류 | Express 에러 미들웨어 → 500 |
| 프론트엔드 fetch 실패 | 3초 후 사라지는 에러 메시지 표시 |

---

## 검증 방법

```bash
# 1. 설치 및 실행
npm install && npm start

# 2. 브라우저에서 http://localhost:3000 접속하여 UI 동작 확인

# 3. API 직접 테스트
curl http://localhost:3000/todos
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"text\":\"테스트\"}"

# 4. 영구 저장 확인: 서버 재시작 후 데이터 유지 여부 확인
```

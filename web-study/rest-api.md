# REST API 설계 원칙

## REST란?
- **REpresentational State Transfer**: 웹의 아키텍처 스타일
- **무상태**: 각 요청은 독립적이며 서버가 클라이언트 상태를 저장하지 않음
- **자원 중심**: URL로 자원을 식별하고 HTTP 메서드로 동작을 표현

## REST 6가지 제약조건

### 1. Client-Server (클라이언트-서버)
- 사용자 인터페이스와 데이터 저장 관심사 분리
- 독립적 진화 가능

### 2. Stateless (무상태)
- 서버는 클라이언트 상태 정보를 저장하지 않음
- 모든 요청에 필요한 정보를 포함해야 함

### 3. Cacheable (캐시 가능)
- 응답은 캐시 가능 여부를 명시해야 함
- 성능, 확장성, 사용자 경험 향상

### 4. Uniform Interface (균일한 인터페이스)
- 자원 식별, 메시지를 통한 자원 조작
- 자기 서술적 메시지, HATEOAS

### 5. Layered System (계층화)
- 클라이언트는 직접 연결된 서버와만 통신
- 중간 서버(프록시, 게이트웨이) 사용 가능

### 6. Code on Demand (선택적)
- 서버가 클라이언트에 실행 가능한 코드 전송
- JavaScript, 애플릿 등

## HTTP 메서드

### GET - 조회
```javascript
// 모든 사용자 조회
GET /api/users
Response: 200 OK
{
    "users": [
        { "id": 1, "name": "John", "email": "john@example.com" },
        { "id": 2, "name": "Jane", "email": "jane@example.com" }
    ]
}

// 특정 사용자 조회
GET /api/users/1
Response: 200 OK
{
    "id": 1,
    "name": "John",
    "email": "john@example.com",
    "createdAt": "2024-01-01T00:00:00Z"
}

// 사용자가 존재하지 않음
GET /api/users/999
Response: 404 Not Found
{
    "error": "User not found",
    "message": "User with id 999 does not exist"
}
```

### POST - 생성
```javascript
// 새 사용자 생성
POST /api/users
Content-Type: application/json
{
    "name": "Bob",
    "email": "bob@example.com",
    "password": "secure123"
}

Response: 201 Created
Location: /api/users/3
{
    "id": 3,
    "name": "Bob",
    "email": "bob@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
}

// 중복 이메일 에러
POST /api/users
Response: 409 Conflict
{
    "error": "Email already exists",
    "message": "bob@example.com is already registered"
}
```

### PUT - 전체 수정
```javascript
// 사용자 정보 전체 수정
PUT /api/users/1
Content-Type: application/json
{
    "name": "John Updated",
    "email": "john.new@example.com",
    "password": "newpassword123"
}

Response: 200 OK
{
    "id": 1,
    "name": "John Updated",
    "email": "john.new@example.com",
    "updatedAt": "2024-01-15T11:00:00Z"
}
```

### PATCH - 부분 수정
```javascript
// 사용자 이름만 수정
PATCH /api/users/1
Content-Type: application/json
{
    "name": "John Partial Update"
}

Response: 200 OK
{
    "id": 1,
    "name": "John Partial Update",
    "email": "john@example.com",
    "updatedAt": "2024-01-15T11:15:00Z"
}
```

### DELETE - 삭제
```javascript
// 사용자 삭제
DELETE /api/users/1
Response: 204 No Content

// 또는 삭제된 리소스 정보 반환
DELETE /api/users/1
Response: 200 OK
{
    "message": "User deleted successfully",
    "deletedUser": {
        "id": 1,
        "name": "John"
    }
}
```

## URL 설계 원칙

### 1. 명사 사용, 동사 금지
```javascript
// ✅ 좋은 예
GET /api/users          // 사용자 목록 조회
POST /api/users         // 사용자 생성
GET /api/users/1        // 특정 사용자 조회
DELETE /api/users/1     // 사용자 삭제

// ❌ 나쁜 예
GET /api/getUsers
POST /api/createUser
GET /api/getUserById/1
DELETE /api/deleteUser/1
```

### 2. 계층 관계 표현
```javascript
// 사용자의 게시글
GET /api/users/1/posts
POST /api/users/1/posts

// 게시글의 댓글
GET /api/posts/10/comments
POST /api/posts/10/comments

// 댓글의 답글
GET /api/comments/5/replies
```

### 3. 컬렉션과 문서
```javascript
// 컬렉션 (복수형)
GET /api/users          // 사용자들
GET /api/posts          // 게시글들

// 문서 (단수형 ID)
GET /api/users/1        // 특정 사용자
GET /api/posts/10       // 특정 게시글
```

## 상태 코드 활용

### 성공 응답 (2xx)
```javascript
200 OK              // 요청 성공
201 Created         // 리소스 생성 성공
204 No Content      // 성공했지만 반환할 내용 없음
```

### 클라이언트 에러 (4xx)
```javascript
400 Bad Request     // 잘못된 요청
401 Unauthorized    // 인증 필요
403 Forbidden       // 권한 없음
404 Not Found       // 리소스 없음
409 Conflict        // 충돌 (중복 등)
422 Unprocessable Entity  // 검증 실패
429 Too Many Requests     // 요청 제한 초과
```

### 서버 에러 (5xx)
```javascript
500 Internal Server Error    // 서버 내부 오류
502 Bad Gateway             // 게이트웨이 오류
503 Service Unavailable     // 서비스 사용 불가
```

## 페이지네이션

### Offset 기반
```javascript
GET /api/users?page=2&limit=10
// 또는
GET /api/users?offset=10&limit=10

Response: 200 OK
{
    "data": [...],
    "pagination": {
        "page": 2,
        "limit": 10,
        "total": 150,
        "totalPages": 15,
        "hasNext": true,
        "hasPrev": true
    }
}
```

### Cursor 기반
```javascript
GET /api/users?cursor=eyJpZCI6MTB9&limit=10

Response: 200 OK
{
    "data": [...],
    "pagination": {
        "nextCursor": "eyJpZCI6MjB9",
        "prevCursor": "eyJpZCI6MX0",
        "hasNext": true,
        "hasPrev": true
    }
}
```

## 필터링과 정렬

### 쿼리 파라미터 활용
```javascript
// 필터링
GET /api/users?status=active&role=admin&age_min=18

// 정렬
GET /api/users?sort=name&order=asc
GET /api/users?sort=-createdAt  // '-'는 desc

// 필드 선택
GET /api/users?fields=id,name,email

// 검색
GET /api/users?search=john&searchFields=name,email
```

## 버전 관리

### URL 경로 버전
```javascript
GET /api/v1/users
GET /api/v2/users

// 세부 버전
GET /api/v1.2/users
```

### 헤더 버전
```javascript
GET /api/users
Accept: application/vnd.api+json;version=1
// 또는
API-Version: 1
```

## 에러 응답 형식

### 표준화된 에러 구조
```javascript
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Request validation failed",
        "details": [
            {
                "field": "email",
                "message": "Email format is invalid"
            },
            {
                "field": "password",
                "message": "Password must be at least 8 characters"
            }
        ],
        "timestamp": "2024-01-15T10:30:00Z",
        "path": "/api/users"
    }
}
```

## 실제 구현 예시

### Express.js 구현
```javascript
const express = require('express');
const app = express();

// 사용자 CRUD
const users = [];
let nextId = 1;

// GET /api/users - 사용자 목록 조회
app.get('/api/users', (req, res) => {
    const { page = 1, limit = 10, status } = req.query;
    
    let filteredUsers = users;
    if (status) {
        filteredUsers = users.filter(user => user.status === status);
    }
    
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + parseInt(limit);
    const paginatedUsers = filteredUsers.slice(startIndex, endIndex);
    
    res.json({
        data: paginatedUsers,
        pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            total: filteredUsers.length,
            totalPages: Math.ceil(filteredUsers.length / limit)
        }
    });
});

// POST /api/users - 사용자 생성
app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    
    // 유효성 검사
    if (!name || !email) {
        return res.status(400).json({
            error: {
                code: 'VALIDATION_ERROR',
                message: 'Name and email are required'
            }
        });
    }
    
    // 중복 이메일 확인
    const existingUser = users.find(user => user.email === email);
    if (existingUser) {
        return res.status(409).json({
            error: {
                code: 'DUPLICATE_EMAIL',
                message: 'Email already exists'
            }
        });
    }
    
    const newUser = {
        id: nextId++,
        name,
        email,
        status: 'active',
        createdAt: new Date().toISOString()
    };
    
    users.push(newUser);
    
    res.status(201)
       .location(`/api/users/${newUser.id}`)
       .json(newUser);
});

// GET /api/users/:id - 특정 사용자 조회
app.get('/api/users/:id', (req, res) => {
    const user = users.find(u => u.id === parseInt(req.params.id));
    
    if (!user) {
        return res.status(404).json({
            error: {
                code: 'USER_NOT_FOUND',
                message: 'User not found'
            }
        });
    }
    
    res.json(user);
});

// PUT /api/users/:id - 사용자 전체 수정
app.put('/api/users/:id', (req, res) => {
    const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
    
    if (userIndex === -1) {
        return res.status(404).json({
            error: {
                code: 'USER_NOT_FOUND',
                message: 'User not found'
            }
        });
    }
    
    const { name, email, status } = req.body;
    
    users[userIndex] = {
        ...users[userIndex],
        name,
        email,
        status,
        updatedAt: new Date().toISOString()
    };
    
    res.json(users[userIndex]);
});

// DELETE /api/users/:id - 사용자 삭제
app.delete('/api/users/:id', (req, res) => {
    const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
    
    if (userIndex === -1) {
        return res.status(404).json({
            error: {
                code: 'USER_NOT_FOUND',
                message: 'User not found'
            }
        });
    }
    
    users.splice(userIndex, 1);
    res.status(204).send();
});
```

## 보안 고려사항

### 1. 인증과 인가
```javascript
// JWT 토큰 검증 미들웨어
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({
            error: {
                code: 'NO_TOKEN',
                message: 'Access token required'
            }
        });
    }
    
    jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({
                error: {
                    code: 'INVALID_TOKEN',
                    message: 'Invalid access token'
                }
            });
        }
        req.user = user;
        next();
    });
};

// 보호된 라우트
app.get('/api/users', authenticateToken, (req, res) => {
    // 인증된 사용자만 접근 가능
});
```

### 2. 입력 검증
```javascript
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
    body('email').isEmail().normalizeEmail(),
    body('name').trim().isLength({ min: 2, max: 50 }),
    body('password').isLength({ min: 8 })
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({
            error: {
                code: 'VALIDATION_ERROR',
                message: 'Request validation failed',
                details: errors.array()
            }
        });
    }
    
    // 사용자 생성 로직
});
```

## API 문서화

### OpenAPI/Swagger 예시
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /api/users:
    get:
      summary: Get users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

## 모범 사례
1. **일관성**: URL 패턴, 응답 형식 통일
2. **명확성**: 직관적이고 이해하기 쉬운 API 설계
3. **보안**: 인증, 인가, 입력 검증 필수
4. **성능**: 캐싱, 페이지네이션, 필드 선택
5. **문서화**: API 스펙 문서 작성 및 유지보수
6. **버전 관리**: 하위 호환성 고려한 버전 관리
7. **에러 처리**: 명확하고 도움이 되는 에러 메시지
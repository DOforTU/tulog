# HTTP 상태 코드

## 상태 코드 분류

HTTP 상태 코드는 첫 번째 숫자에 따라 5가지로 분류:
- **1xx**: 정보 응답 (Informational)
- **2xx**: 성공 응답 (Successful)
- **3xx**: 리다이렉션 메시지 (Redirection)
- **4xx**: 클라이언트 에러 응답 (Client Error)
- **5xx**: 서버 에러 응답 (Server Error)

## 1xx - 정보 응답

### 100 Continue
- 클라이언트가 서버에 요청 본문을 전송해도 되는지 확인
- 주로 큰 데이터 전송 전에 사용

### 101 Switching Protocols
- WebSocket 연결 시 프로토콜 전환
```javascript
// WebSocket 업그레이드 요청
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade

// 응답
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

## 2xx - 성공 응답

### 200 OK
- 요청이 성공적으로 처리됨
- GET, PUT, PATCH 요청의 일반적인 성공 응답
```javascript
GET /api/users/1
Response: 200 OK
{
    "id": 1,
    "name": "John",
    "email": "john@example.com"
}
```

### 201 Created
- 새로운 리소스가 생성됨
- POST 요청의 성공 응답
```javascript
POST /api/users
Response: 201 Created
Location: /api/users/123
{
    "id": 123,
    "name": "New User",
    "createdAt": "2024-01-15T10:30:00Z"
}
```

### 204 No Content
- 요청은 성공했지만 반환할 내용이 없음
- DELETE 요청이나 PUT 업데이트 시 사용
```javascript
DELETE /api/users/1
Response: 204 No Content
// 응답 본문 없음
```

### 206 Partial Content
- 범위 요청에 대한 부분 응답
- 파일 다운로드 중단/재시작 시 사용
```javascript
GET /api/files/large-file.zip
Range: bytes=1000-2000

Response: 206 Partial Content
Content-Range: bytes 1000-2000/5000
Content-Length: 1001
```

## 3xx - 리다이렉션

### 301 Moved Permanently
- 리소스가 영구적으로 새 위치로 이동
- SEO에서 도메인 변경 시 사용
```javascript
GET /old-page
Response: 301 Moved Permanently
Location: /new-page
```

### 302 Found (Temporary Redirect)
- 리소스가 일시적으로 다른 위치에 있음
```javascript
// 로그인 후 원래 페이지로 리다이렉트
Response: 302 Found
Location: /dashboard
```

### 304 Not Modified
- 캐시된 리소스가 여전히 유효함
- If-Modified-Since, If-None-Match 헤더와 함께 사용
```javascript
GET /api/users
If-None-Match: "abc123"

Response: 304 Not Modified
ETag: "abc123"
// 본문 없음, 캐시 사용
```

### 307 Temporary Redirect
- 302와 유사하지만 HTTP 메서드 변경 금지
```javascript
POST /api/submit
Response: 307 Temporary Redirect
Location: /api/submit-v2
// 클라이언트는 POST 메서드 유지해야 함
```

### 308 Permanent Redirect
- 301과 유사하지만 HTTP 메서드 변경 금지

## 4xx - 클라이언트 에러

### 400 Bad Request
- 잘못된 요청 구문
- 유효성 검사 실패
```javascript
POST /api/users
{
    "email": "invalid-email"
}

Response: 400 Bad Request
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid email format",
        "details": [
            {
                "field": "email",
                "message": "Must be a valid email address"
            }
        ]
    }
}
```

### 401 Unauthorized
- 인증이 필요함
- 인증 정보가 없거나 잘못됨
```javascript
GET /api/protected-resource

Response: 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
{
    "error": {
        "code": "AUTHENTICATION_REQUIRED",
        "message": "Access token is required"
    }
}
```

### 403 Forbidden
- 인증은 되었으나 권한이 없음
```javascript
DELETE /api/admin/users/1

Response: 403 Forbidden
{
    "error": {
        "code": "INSUFFICIENT_PERMISSIONS",
        "message": "Admin privileges required"
    }
}
```

### 404 Not Found
- 요청한 리소스가 존재하지 않음
```javascript
GET /api/users/999

Response: 404 Not Found
{
    "error": {
        "code": "RESOURCE_NOT_FOUND",
        "message": "User with id 999 not found"
    }
}
```

### 405 Method Not Allowed
- 허용되지 않은 HTTP 메서드
```javascript
DELETE /api/users
// DELETE가 지원되지 않는 경우

Response: 405 Method Not Allowed
Allow: GET, POST
{
    "error": {
        "code": "METHOD_NOT_ALLOWED",
        "message": "DELETE method is not allowed for this resource"
    }
}
```

### 409 Conflict
- 요청이 현재 서버 상태와 충돌
- 중복 데이터, 동시성 문제
```javascript
POST /api/users
{
    "email": "existing@example.com"
}

Response: 409 Conflict
{
    "error": {
        "code": "EMAIL_ALREADY_EXISTS",
        "message": "User with this email already exists"
    }
}
```

### 422 Unprocessable Entity
- 요청 구문은 올바르나 의미적으로 처리할 수 없음
```javascript
POST /api/users
{
    "name": "",
    "email": "valid@example.com",
    "age": -5
}

Response: 422 Unprocessable Entity
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Request validation failed",
        "details": [
            {
                "field": "name",
                "message": "Name cannot be empty"
            },
            {
                "field": "age",
                "message": "Age must be positive"
            }
        ]
    }
}
```

### 429 Too Many Requests
- 요청 제한 초과 (Rate Limiting)
```javascript
Response: 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1642694400

{
    "error": {
        "code": "RATE_LIMIT_EXCEEDED",
        "message": "Too many requests. Try again in 60 seconds"
    }
}
```

## 5xx - 서버 에러

### 500 Internal Server Error
- 서버 내부 오류
- 예상하지 못한 에러 발생
```javascript
GET /api/users

Response: 500 Internal Server Error
{
    "error": {
        "code": "INTERNAL_ERROR",
        "message": "An unexpected error occurred",
        "timestamp": "2024-01-15T10:30:00Z",
        "requestId": "req-123-abc"
    }
}
```

### 502 Bad Gateway
- 게이트웨이나 프록시 서버가 업스트림 서버로부터 잘못된 응답을 받음
```javascript
// 로드 밸런서 뒤의 서버가 응답하지 않을 때
Response: 502 Bad Gateway
{
    "error": {
        "code": "BAD_GATEWAY",
        "message": "Upstream server error"
    }
}
```

### 503 Service Unavailable
- 서버가 일시적으로 사용할 수 없음
- 유지보수, 과부하 상태
```javascript
Response: 503 Service Unavailable
Retry-After: 3600

{
    "error": {
        "code": "SERVICE_UNAVAILABLE",
        "message": "Service is temporarily unavailable. Please try again later.",
        "retryAfter": 3600
    }
}
```

### 504 Gateway Timeout
- 게이트웨이나 프록시가 업스트림 서버로부터 응답을 받지 못함
```javascript
Response: 504 Gateway Timeout
{
    "error": {
        "code": "GATEWAY_TIMEOUT",
        "message": "Upstream server did not respond in time"
    }
}
```

## 실제 구현 예시

### Express.js에서 상태 코드 사용
```javascript
const express = require('express');
const app = express();

// 성공 응답
app.get('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                error: {
                    code: 'USER_NOT_FOUND',
                    message: 'User not found'
                }
            });
        }
        
        res.status(200).json(user); // 또는 그냥 res.json(user)
    } catch (error) {
        res.status(500).json({
            error: {
                code: 'INTERNAL_ERROR',
                message: 'An unexpected error occurred'
            }
        });
    }
});

// 생성 응답
app.post('/api/users', async (req, res) => {
    try {
        const { email } = req.body;
        
        // 중복 검사
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json({
                error: {
                    code: 'EMAIL_EXISTS',
                    message: 'Email already registered'
                }
            });
        }
        
        const user = await User.create(req.body);
        
        res.status(201)
           .location(`/api/users/${user.id}`)
           .json(user);
    } catch (error) {
        if (error.name === 'ValidationError') {
            return res.status(400).json({
                error: {
                    code: 'VALIDATION_ERROR',
                    message: error.message
                }
            });
        }
        
        res.status(500).json({
            error: {
                code: 'INTERNAL_ERROR',
                message: 'Failed to create user'
            }
        });
    }
});
```

### 에러 처리 미들웨어
```javascript
// 전역 에러 처리 미들웨어
app.use((error, req, res, next) => {
    console.error(error);
    
    // 개발 환경에서만 스택 트레이스 포함
    const isDevelopment = process.env.NODE_ENV === 'development';
    
    const errorResponse = {
        error: {
            code: 'INTERNAL_ERROR',
            message: 'An unexpected error occurred',
            timestamp: new Date().toISOString(),
            path: req.path
        }
    };
    
    if (isDevelopment) {
        errorResponse.error.stack = error.stack;
    }
    
    res.status(500).json(errorResponse);
});

// 404 처리
app.use('*', (req, res) => {
    res.status(404).json({
        error: {
            code: 'ENDPOINT_NOT_FOUND',
            message: `Cannot ${req.method} ${req.originalUrl}`
        }
    });
});
```

### Rate Limiting 구현
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15분
    max: 100, // 최대 100개 요청
    message: {
        error: {
            code: 'RATE_LIMIT_EXCEEDED',
            message: 'Too many requests from this IP'
        }
    },
    standardHeaders: true, // X-RateLimit-* 헤더 포함
    legacyHeaders: false,
    handler: (req, res) => {
        res.status(429).json({
            error: {
                code: 'RATE_LIMIT_EXCEEDED',
                message: 'Too many requests. Please try again later.',
                retryAfter: Math.round(req.rateLimit.resetTime / 1000)
            }
        });
    }
});

app.use('/api', limiter);
```

## 클라이언트 측 처리

### Fetch API
```javascript
async function apiCall(url, options = {}) {
    try {
        const response = await fetch(url, options);
        
        if (!response.ok) {
            const errorData = await response.json();
            
            switch (response.status) {
                case 400:
                    throw new ValidationError(errorData.error.message);
                case 401:
                    // 토큰 갱신 또는 로그인 페이지로 이동
                    redirectToLogin();
                    return;
                case 403:
                    throw new PermissionError('접근 권한이 없습니다');
                case 404:
                    throw new NotFoundError('요청한 리소스를 찾을 수 없습니다');
                case 409:
                    throw new ConflictError(errorData.error.message);
                case 429:
                    const retryAfter = response.headers.get('Retry-After');
                    throw new RateLimitError(`요청 한도 초과. ${retryAfter}초 후 다시 시도하세요`);
                case 500:
                    throw new ServerError('서버 오류가 발생했습니다');
                default:
                    throw new Error(`HTTP ${response.status}: ${errorData.error.message}`);
            }
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        if (error instanceof TypeError) {
            throw new NetworkError('네트워크 연결을 확인해주세요');
        }
        throw error;
    }
}
```

### Axios 인터셉터
```javascript
import axios from 'axios';

// 응답 인터셉터
axios.interceptors.response.use(
    response => response,
    error => {
        const { response } = error;
        
        if (!response) {
            return Promise.reject(new Error('네트워크 오류'));
        }
        
        switch (response.status) {
            case 401:
                // 자동 토큰 갱신
                return refreshTokenAndRetry(error);
            case 429:
                const retryAfter = response.headers['retry-after'];
                return retryAfterDelay(error, retryAfter * 1000);
            default:
                return Promise.reject(error);
        }
    }
);

async function refreshTokenAndRetry(error) {
    try {
        const newToken = await refreshAccessToken();
        error.config.headers.Authorization = `Bearer ${newToken}`;
        return axios.request(error.config);
    } catch (refreshError) {
        redirectToLogin();
        return Promise.reject(refreshError);
    }
}
```

## 상태 코드 선택 가이드

### 성공 시나리오
- **조회 성공**: 200 OK
- **생성 성공**: 201 Created + Location 헤더
- **수정 성공**: 200 OK (수정된 데이터 반환) 또는 204 No Content
- **삭제 성공**: 204 No Content 또는 200 OK (삭제된 데이터 반환)

### 에러 시나리오
- **잘못된 입력**: 400 Bad Request
- **인증 실패**: 401 Unauthorized
- **권한 없음**: 403 Forbidden
- **리소스 없음**: 404 Not Found
- **중복 데이터**: 409 Conflict
- **유효성 검사 실패**: 422 Unprocessable Entity
- **서버 오류**: 500 Internal Server Error

## 모범 사례
1. **일관성**: 같은 상황에서 항상 같은 상태 코드 사용
2. **의미 있는 응답**: 상태 코드와 함께 상세한 에러 정보 제공
3. **적절한 헤더**: Location, Retry-After 등 관련 헤더 포함
4. **클라이언트 친화적**: 사용자가 이해할 수 있는 에러 메시지
5. **보안 고려**: 민감한 정보 노출 방지
6. **로깅**: 5xx 에러는 반드시 서버 로그 기록
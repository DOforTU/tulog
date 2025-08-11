# CORS (Cross-Origin Resource Sharing)

## CORS란?
- **교차 출처 리소스 공유**: 웹 페이지가 다른 도메인의 리소스에 접근할 수 있도록 허용하는 메커니즘
- **보안 정책**: 브라우저의 동일 출처 정책(Same-Origin Policy)을 우회하는 방법
- **서버 설정**: 서버에서 어떤 출처의 요청을 허용할지 결정

## 동일 출처 정책 (Same-Origin Policy)
같은 출처로 간주되려면 다음 3가지가 모두 같아야 함:
- **프로토콜** (http, https)
- **도메인** (example.com)
- **포트** (:3000, :8080)

```javascript
// 동일 출처 예시
https://example.com/api/users     // 기준
https://example.com/api/posts     // ✅ 같은 출처
https://example.com:443/api/data  // ✅ 같은 출처 (https 기본 포트)

// 다른 출처 예시
http://example.com/api/users      // ❌ 프로토콜 다름
https://api.example.com/users     // ❌ 서브도메인 다름
https://example.com:8080/api      // ❌ 포트 다름
```

## CORS 동작 과정

### 1. Simple Request (단순 요청)
조건을 만족하는 요청은 preflight 없이 바로 전송:
- **메서드**: GET, POST, HEAD
- **헤더**: Accept, Accept-Language, Content-Language, Content-Type (특정 타입만)
- **Content-Type**: text/plain, multipart/form-data, application/x-www-form-urlencoded

```javascript
// Simple Request 예시
fetch('https://api.example.com/users', {
    method: 'GET',
    headers: {
        'Accept': 'application/json'
    }
});
```

### 2. Preflight Request (사전 요청)
복잡한 요청은 실제 요청 전에 OPTIONS 요청으로 서버 확인:

```javascript
// Preflight가 필요한 요청
fetch('https://api.example.com/users', {
    method: 'DELETE',
    headers: {
        'Authorization': 'Bearer token123',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ id: 1 })
});

// 브라우저가 자동으로 보내는 Preflight 요청:
// OPTIONS /users HTTP/1.1
// Origin: https://myapp.com
// Access-Control-Request-Method: DELETE
// Access-Control-Request-Headers: Authorization, Content-Type
```

## CORS 헤더 설정

### 서버 응답 헤더
```javascript
// Express.js 예시
app.use((req, res, next) => {
    // 요청을 허용할 출처
    res.header('Access-Control-Allow-Origin', 'https://myapp.com');
    
    // 허용할 HTTP 메서드
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    
    // 허용할 헤더
    res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
    
    // 자격 증명 포함 허용
    res.header('Access-Control-Allow-Credentials', true);
    
    // Preflight 요청 캐시 시간 (초)
    res.header('Access-Control-Max-Age', 86400); // 24시간
    
    next();
});
```

## 실제 구현 예시

### Express.js + cors 미들웨어
```javascript
const cors = require('cors');

// 기본 설정 (모든 출처 허용)
app.use(cors());

// 상세 설정
const corsOptions = {
    origin: ['https://myapp.com', 'https://admin.myapp.com'],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization'],
    credentials: true, // 쿠키, Authorization 헤더 포함
    maxAge: 86400 // Preflight 캐시 24시간
};

app.use(cors(corsOptions));

// 동적 출처 허용
const dynamicCors = cors({
    origin: (origin, callback) => {
        // 허용할 도메인 목록
        const allowedOrigins = ['https://myapp.com', 'https://admin.myapp.com'];
        
        // origin이 없으면 (같은 출처) 또는 허용 목록에 있으면 허용
        if (!origin || allowedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    }
});

app.use('/api', dynamicCors);
```

### NestJS CORS 설정
```typescript
// main.ts
import { NestFactory } from '@nestjs/core';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    
    // CORS 활성화
    app.enableCors({
        origin: ['https://myapp.com'],
        methods: 'GET,HEAD,PUT,PATCH,POST,DELETE,OPTIONS',
        credentials: true,
    });
    
    await app.listen(3000);
}
```

### Next.js API 라우트
```javascript
// pages/api/users.js
export default function handler(req, res) {
    // CORS 헤더 설정
    res.setHeader('Access-Control-Allow-Origin', 'https://myapp.com');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    if (req.method === 'OPTIONS') {
        res.status(200).end();
        return;
    }
    
    // 실제 로직
    res.status(200).json({ users: [] });
}

// 또는 Next.js 설정 파일에서
// next.config.js
module.exports = {
    async headers() {
        return [
            {
                source: '/api/:path*',
                headers: [
                    { key: 'Access-Control-Allow-Origin', value: 'https://myapp.com' },
                    { key: 'Access-Control-Allow-Methods', value: 'GET, POST, PUT, DELETE' },
                    { key: 'Access-Control-Allow-Headers', value: 'Content-Type, Authorization' },
                ]
            }
        ];
    }
};
```

## 클라이언트 측 처리

### 자격 증명 포함 요청
```javascript
// 쿠키, Authorization 헤더 포함하여 요청
fetch('https://api.example.com/users', {
    method: 'POST',
    credentials: 'include', // 중요: 쿠키 포함
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    body: JSON.stringify({ name: 'John' })
});

// Axios
axios.defaults.withCredentials = true;
axios.post('https://api.example.com/users', data);
```

### CORS 에러 처리
```javascript
async function apiCall() {
    try {
        const response = await fetch('https://api.example.com/users');
        const data = await response.json();
        return data;
    } catch (error) {
        if (error.message.includes('CORS')) {
            console.error('CORS 에러: 서버 설정을 확인하세요');
            // 사용자에게 알림 표시
        }
        throw error;
    }
}
```

## 보안 고려사항

### 1. 와일드카드 사용 주의
```javascript
// 위험한 설정
res.header('Access-Control-Allow-Origin', '*');
res.header('Access-Control-Allow-Credentials', true);
// 자격 증명과 와일드카드는 함께 사용할 수 없음

// 안전한 설정
const allowedOrigins = ['https://myapp.com', 'https://admin.myapp.com'];
const origin = req.headers.origin;

if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
}
```

### 2. 민감한 헤더 노출 방지
```javascript
// 클라이언트에서 접근 가능한 헤더 명시
res.header('Access-Control-Expose-Headers', 'X-Total-Count, X-Page-Count');

// Authorization 헤더는 노출하지 않음
```

### 3. 환경별 설정
```javascript
const corsOptions = {
    origin: process.env.NODE_ENV === 'production' 
        ? ['https://myapp.com'] 
        : ['http://localhost:3000', 'http://127.0.0.1:3000'],
    credentials: true
};
```

## 개발 시 CORS 우회 방법

### 1. 프록시 설정 (개발 전용)
```javascript
// package.json (Create React App)
{
    "name": "my-app",
    "proxy": "http://localhost:8000"
}

// 또는 setupProxy.js
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
    app.use('/api', createProxyMiddleware({
        target: 'http://localhost:8000',
        changeOrigin: true
    }));
};
```

### 2. 브라우저 CORS 비활성화 (개발 전용)
```bash
# Chrome 실행 시 CORS 비활성화 (위험!)
chrome --disable-web-security --user-data-dir=/tmp/chrome_dev
```

## 트러블슈팅

### 자주 발생하는 문제들
1. **Preflight 요청 실패**: OPTIONS 메서드 처리 누락
2. **자격 증명 에러**: credentials: true와 와일드카드 동시 사용
3. **헤더 누락**: 필요한 헤더가 Allow-Headers에 없음
4. **개발/운영 환경 차이**: 도메인 설정 불일치

### 디버깅 팁
- **개발자 도구 Network 탭**: Preflight 요청 확인
- **Console 에러 메시지**: 구체적인 CORS 에러 내용 확인
- **서버 로그**: 실제 요청이 서버에 도달했는지 확인

## 모범 사례
1. **최소 권한 원칙**: 필요한 출처/메서드/헤더만 허용
2. **환경별 설정**: 개발/스테이징/운영 환경별 다른 설정
3. **동적 출처 검증**: 하드코딩보다 환경 변수나 설정 파일 사용
4. **보안 검토**: 정기적인 CORS 설정 점검
5. **에러 처리**: 클라이언트에서 CORS 에러 적절히 처리
# 미들웨어 vs 인터셉터 - 간단 비교

> NestJS에서 미들웨어(Middleware)와 인터셉터(Interceptor)의 핵심 차이점을 간단히 정리한 문서입니다.

## 실행 순서

```
Request → Middleware → Guards → Interceptor → Controller → Interceptor → Response
```

## 핵심 차이점

| 구분          | 미들웨어 (Middleware)   | 인터셉터 (Interceptor)            |
| ------------- | ----------------------- | --------------------------------- |
| **실행 시점** | 요청 처리 초기 단계     | 컨트롤러 실행 전후                |
| **주요 목적** | 요청/응답 전처리        | 데이터 변환 및 비즈니스 로직 확장 |
| **접근 범위** | Request, Response, Next | ExecutionContext, Observable      |
| **기반 기술** | Express 미들웨어        | RxJS Observable                   |

## 사용 목적

### 미들웨어 (Middleware)

-   **로깅**: HTTP 요청/응답 기록
-   **보안**: CORS, 헤더 설정, 쿠키 파싱
-   **인증**: 토큰 추출 및 기본 검증
-   **전처리**: 데이터 파싱, 압축 해제

```typescript
// 예시: 로깅 미들웨어
export class LoggingMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        console.log(`${req.method} ${req.url}`);
        next(); // 다음 단계로 진행
    }
}
```

### 인터셉터 (Interceptor)

-   **응답 변환**: API 응답 형식 표준화
-   **에러 처리**: 예외 상황 후처리
-   **성능 측정**: 실행 시간 추적
-   **캐싱**: 응답 데이터 캐시 처리

```typescript
// 예시: 응답 표준화 인터셉터
export class ResponseInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        return next.handle().pipe(
            map((data) => ({
                success: true,
                data,
                timestamp: new Date().toISOString(),
            }))
        );
    }
}
```

## 요약

### **미들웨어**

-   **언제**: 요청이 들어오자마자
-   **무엇을**: 요청/응답 기본 처리
-   **예시**: 로깅, 보안 헤더, 쿠키 파싱

### **인터셉터**

-   **언제**: 컨트롤러 실행 전후
-   **무엇을**: 데이터 변환 및 확장
-   **예시**: 응답 형식 통일, 성능 측정

## 언제 뭘 사용할까?

### 미들웨어 선택

-   HTTP 요청/응답을 가공해야 할 때
-   Express의 기능을 활용하고 싶을 때
-   모든 라우트에 공통으로 적용할 때

### 인터셉터 선택

-   비즈니스 로직의 결과를 변환할 때
-   Observable 스트림을 활용하고 싶을 때
-   특정 컨트롤러/메서드에만 적용할 때

## 실제 프로젝트 적용 예시

### 미들웨어 활용 사례
```typescript
// 1. 인증 토큰 검증
export class AuthMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        const token = req.headers.authorization?.split(' ')[1];
        if (!token) {
            throw new UnauthorizedException('Token not found');
        }
        // 토큰 검증 로직
        next();
    }
}

// 2. 요청 제한 (Rate Limiting)
export class RateLimitMiddleware implements NestMiddleware {
    private requests = new Map();
    
    use(req: Request, res: Response, next: NextFunction) {
        const ip = req.ip;
        const requests = this.requests.get(ip) || [];
        const now = Date.now();
        
        // 1분 내 요청 수 확인
        const recentRequests = requests.filter(time => now - time < 60000);
        
        if (recentRequests.length >= 100) {
            throw new HttpException('Too Many Requests', 429);
        }
        
        this.requests.set(ip, [...recentRequests, now]);
        next();
    }
}
```

### 인터셉터 활용 사례
```typescript
// 1. 캐시 처리
export class CacheInterceptor implements NestInterceptor {
    private cache = new Map();
    
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const request = context.switchToHttp().getRequest();
        const key = `${request.method}_${request.url}`;
        
        if (this.cache.has(key)) {
            return of(this.cache.get(key));
        }
        
        return next.handle().pipe(
            tap(response => this.cache.set(key, response))
        );
    }
}

// 2. 실행 시간 측정
export class TimeoutInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const startTime = Date.now();
        
        return next.handle().pipe(
            tap(() => {
                const endTime = Date.now();
                console.log(`Execution time: ${endTime - startTime}ms`);
            }),
            timeout(5000), // 5초 타임아웃
            catchError(err => {
                if (err instanceof TimeoutError) {
                    throw new RequestTimeoutException();
                }
                throw err;
            })
        );
    }
}
```

## Express.js에서의 차이점

### Express 미들웨어
```javascript
// 전역 미들웨어
app.use(express.json());
app.use(cors());
app.use(morgan('combined'));

// 라우트 특정 미들웨어
const authMiddleware = (req, res, next) => {
    // 인증 로직
    if (!req.headers.authorization) {
        return res.status(401).json({ error: 'Unauthorized' });
    }
    next();
};

app.get('/protected', authMiddleware, (req, res) => {
    res.json({ message: 'Protected data' });
});
```

### Express에서 인터셉터 패턴 구현
```javascript
// 응답 변환 함수 (인터셉터 역할)
const responseInterceptor = (req, res, next) => {
    const originalSend = res.send;
    
    res.send = function(data) {
        // 응답 데이터 변환
        const wrappedData = {
            success: true,
            data: typeof data === 'string' ? JSON.parse(data) : data,
            timestamp: new Date().toISOString()
        };
        
        originalSend.call(this, JSON.stringify(wrappedData));
    };
    
    next();
};

app.use(responseInterceptor);
```

---

**한 줄 정리**: 미들웨어는 "요청 전처리", 인터셉터는 "응답 후처리"의 역할!

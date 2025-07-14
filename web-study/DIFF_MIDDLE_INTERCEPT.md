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

---

**한 줄 정리**: 미들웨어는 "요청 전처리", 인터셉터는 "응답 후처리"의 역할!

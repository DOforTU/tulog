# TULOG Google OAuth 로그인 파이프라인

## 📋 개요

TULOG은 JWT + 리프레시 토큰 기반의 Google OAuth 인증 시스템을 사용합니다.
사용자가 Google 계정으로 로그인하면 자동으로 회원가입이 되며, 별도의 회원가입 과정이 없습니다.

# TULOG Google OAuth 로그인 파이프라인

## 📋 개요

TULOG은 JWT + 리프레시 토큰 기반의 Google OAuth 인증 시스템을 사용합니다.
사용자가 Google 계정으로 로그인하면 자동으로 회원가입이 되며, 별도의 회원가입 과정이 없습니다.

**기술 스택:**

-   **프론트엔드**: Vue 3 + Pinia + TypeScript
-   **백엔드**: NestJS + Passport.js + JWT
-   **데이터베이스**: PostgreSQL + TypeORM
-   **인증**: Google OAuth 2.0 + HttpOnly 쿠키

## 🔄 전체 로그인 프로세스 (간단 요약)

```
[사용자] → [Vue 프론트엔드] → [NestJS 백엔드] → [Google OAuth] → [토큰 생성] → [상태 관리] → [완료]
```

### 1. 로그인 시작

-   사용자가 "Google로 로그인" 버튼 클릭
-   `LoginView.vue`에서 `window.location.href = "/auth/google"` 실행
-   백엔드 OAuth 엔드포인트로 리디렉션

### 2. Google OAuth 처리

-   NestJS 백엔드에서 Google OAuth 플로우 시작
-   사용자가 Google에서 권한 승인
-   Google이 백엔드 콜백 URL로 인증 코드 전송

### 3. 토큰 생성 및 저장

-   백엔드에서 JWT 토큰 쌍 생성 (액세스 15분 + 리프레시 7일)
-   HttpOnly 쿠키로 토큰 저장
-   `/login?success=true`로 프론트엔드 리디렉션

### 4. 프론트엔드 상태 업데이트

-   `App.vue`에서 `success=true` 파라미터 감지
-   Pinia `authStore`에서 `/users/me/info` API 호출
-   사용자 정보를 전역 상태로 저장
-   인증 상태 변경 감지하여 홈페이지로 이동

### 5. 10분마다 accessToken 재발급

-   `AuthStore.ts`에서 `setupTokenRefresh()` 함수가 `setInterval`에 의해 10분 주기로 실행
-   `/auth/refresh` API를 호출하여 새로운 액세스 토큰 발급
-   리프레시 토큰이 유효하지 않으면 자동으로 로그아웃 처리
-   사용자는 별도의 로그인 과정 없이 지속적으로 서비스 이용 가능

### 6. 지속적인 인증 상태 유지

-   페이지 새로고침 시 `App.vue`에서 자동으로 인증 상태 복구
-   HttpOnly 쿠키의 토큰을 통해 서버에서 사용자 정보 검증
-   토큰이 만료되거나 유효하지 않으면 로그인 페이지로 리디렉션
-   모든 컴포넌트에서 실시간으로 인증 상태 변화 감지 및 UI 업데이트

---

## 🎯 프론트엔드 상세 프로세스

### **파일 구조 및 역할**

```
src/
├── App.vue                    # 앱 진입점, OAuth 성공 감지
├── stores/authStore.ts        # Pinia 인증 상태 관리
├── views/LoginView.vue        # 로그인 페이지
├── components/layout/
│   └── AppHeader.vue         # 헤더, 사용자 정보 표시
└── lib/api-client.ts         # API 클라이언트
```

> Pinia란?
> Pinia는 Vue 3의 공식 상태 관리 라이브러리로, Vuex보다 더 간단하고 타입 친화적이다.
> defineStore를 사용해 모듈별로 store를 만들고, 앱 전역에서 로그인 상태나 사용자 정보를 손쉽게 관리할 수 있다.
> 컴포넌트 간 상태 공유가 간편하고, Composition API와의 결합도 뛰어나고, Devtools 디버깅도 쉬워서 개발 생산성도 높다.
> Vue 앱에서 인증이나 테마 설정처럼 전역 상태가 필요한 경우 거의 필수적인 도구임.

### **1단계: 로그인 페이지 (`LoginView.vue`)**

```typescript
// 📁 src/views/LoginView.vue
const handleGoogleLogin = () => {
    // Google OAuth 시작
    window.location.href = `${API_BASE_URL}/auth/google`;
};

// 인증 상태 변화 감지
watch(
    isAuthenticated,
    (newValue) => {
        if (newValue && currentUser.value) {
            router.push("/"); // 홈페이지로 이동
        }
    },
    { immediate: true }
);
```

**역할**: Google OAuth 시작점  
**동작**: 사용자 클릭 시 백엔드 OAuth 엔드포인트로 리디렉션

### **2단계: 앱 진입점 (`App.vue`)**

```typescript
// 📁 src/App.vue
onMounted(async () => {
    const urlParams = new URLSearchParams(window.location.search);

    if (urlParams.get("success") === "true") {
        // OAuth 성공 시 URL 정리 및 처리
        window.history.replaceState({}, "", window.location.pathname);
        await authStore.handleLoginSuccess();
    } else {
        // 일반 앱 초기화
        await authStore.initAuth();
    }
});
```

**역할**: OAuth 콜백 성공 감지 및 중앙 초기화  
**동작**:

-   URL 파라미터 `success=true` 감지
-   성공 시 인증 상태 초기화
-   실패 시 일반 앱 초기화

### **3단계: 인증 상태 관리 (`authStore.ts`)**

```typescript
// 📁 src/stores/authStore.ts (Pinia Store)
export const useAuthStore = defineStore("auth", () => {
    const currentUser = ref<User | null>(null);
    const isAuthenticated = computed(() => currentUser.value !== null);

    // 사용자 정보 가져오기 (중복 방지)
    const fetchUserInfo = async (): Promise<User | null> => {
        if (isFetchingUser) return currentUser.value; // 중복 요청 방지
        isFetchingUser = true;

        try {
            const response = await api.auth.meInfo(); // /users/me/info 호출
            if (response.success && response.data) {
                currentUser.value = {
                    id: response.data.id.toString(),
                    email: response.data.email,
                    username: response.data.username,
                    nickname: response.data.nickname,
                    profilePicture: response.data.profilePicture,
                };
                return currentUser.value;
            }
            return null;
        } finally {
            isFetchingUser = false;
        }
    };

    // 인증 초기화 (Promise 캐싱으로 중복 방지)
    const initAuth = async () => {
        if (initPromise) return initPromise; // 이미 초기화 중이면 대기
        if (isInitialized) return Promise.resolve(); // 이미 완료됨

        initPromise = (async () => {
            try {
                isInitialized = true;
                const user = await fetchUserInfo();
                if (user) setupTokenRefresh(); // 토큰 자동 갱신 설정
            } finally {
                isLoading.value = false;
                initPromise = null;
            }
        })();

        return initPromise;
    };

    // OAuth 성공 후 처리
    const handleLoginSuccess = async () => {
        await initAuth();
        return currentUser.value;
    };

    // 토큰 자동 갱신 설정 (10분마다)
    const setupTokenRefresh = () => {
        refreshIntervalId = setInterval(async () => {
            try {
                const res = await api.auth.refresh(); // /auth/refresh 호출
                if (!res.success) await logout();
            } catch {
                await logout();
            }
        }, 10 * 60 * 1000);
    };
});
```

**역할**: 중앙화된 인증 상태 관리  
**핵심 기능**:

-   **중복 방지**: Promise 캐싱으로 동시 API 호출 방지
-   **상태 동기화**: 모든 컴포넌트에서 동일한 인증 상태 공유
-   **자동 갱신**: 10분마다 토큰 자동 갱신
-   **에러 처리**: 토큰 만료 시 자동 로그아웃

### **4단계: UI 반영 (`AppHeader.vue`)**

```typescript
// 📁 src/components/layout/AppHeader.vue
const authStore = useAuthStore()
const { isAuthenticated, currentUser } = storeToRefs(authStore)

// 템플릿에서 반응형 데이터 사용
<template>
  <div v-if="isAuthenticated" class="user-menu">
    <img :src="currentUser?.profilePicture" :alt="currentUser?.nickname" />
    <span>{{ currentUser?.nickname }}</span>
  </div>
  <router-link v-else to="/login">로그인</router-link>
</template>
```

**역할**: 인증 상태 UI 반영  
**동작**: Pinia store의 반응형 데이터 자동 동기화

### **5단계: API 클라이언트 (`api-client.ts`)**

```typescript
// 📁 src/lib/api-client.ts
export const api = {
    auth: {
        meInfo: () => fetch("/users/me/info", { credentials: "include" }),
        refresh: () =>
            fetch("/auth/refresh", {
                method: "POST",
                credentials: "include",
            }),
        logout: () =>
            fetch("/auth/logout", {
                method: "POST",
                credentials: "include",
            }),
    },
};
```

**역할**: HTTP 클라이언트  
**특징**: `credentials: 'include'`로 HttpOnly 쿠키 자동 전송

---

## 🔄 상태 관리 플로우

### **Pinia Store 아키텍처**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   App.vue       │───▶│   authStore.ts   │◀───│  AppHeader.vue  │
│ (OAuth 감지)    │    │ (중앙 상태 관리) │    │ (UI 반영)       │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌──────────────────┐
                       │  LoginView.vue   │
                       │ (인증 상태 감지) │
                       └──────────────────┘
```

### **중복 호출 방지 메커니즘**

1. **Promise 캐싱**: 동시 `initAuth()` 호출 시 같은 Promise 반환
2. **상태 플래그**: `isInitialized`, `isFetchingUser`로 중복 방지
3. **중앙 초기화**: `App.vue`에서만 초기화, 다른 컴포넌트는 상태 구독

### **반응형 데이터 흐름**

```
API 응답 → authStore.currentUser → storeToRefs → 컴포넌트 자동 업데이트
```

---

### 🟢 백엔드 함수들

#### `googleAuth()` (auth.controller.ts)

```typescript
@Get('google')
@UseGuards(AuthGuard('google'))
async googleAuth() {
    // Google OAuth 플로우를 시작하는 라우트
}
```

**역할**: Google OAuth 플로우 시작점  
**동작**:

1. Passport의 GoogleStrategy 활성화
2. Google OAuth 서버로 리다이렉트 URL 생성
3. 사용자를 Google 로그인 페이지로 리다이렉트

#### `googleAuthRedirect()` (auth.controller.ts)

```typescript
@Get('google/callback')
@UseGuards(AuthGuard('google'))
googleAuthRedirect(@Req() req: AuthenticatedRequest, @Res() res: Response) {
    const { user } = req.user; // Google Strategy에서 검증된 사용자

    // 토큰 생성 및 쿠키 설정
    const tokens = this.authService.generateTokenPair(user);
    this.authService.setAuthCookies(res, user, tokens);

    // 프론트엔드로 리다이렉트
    const frontendUrl = process.env.FRONTEND_URL || 'http://localhost:3000';
    res.redirect(`${frontendUrl}/login?success=true`);
}
```

**역할**: Google OAuth 콜백 처리 및 최종 리다이렉트  
**동작**:

1. Google에서 인증된 사용자 정보 수신
2. JWT 토큰 쌍 생성 (액세스 + 리프레시)
3. 보안 쿠키 설정 (HttpOnly)
4. 프론트엔드 성공 페이지로 리다이렉트

#### `validate()` (google.strategy.ts)

```typescript
async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
): Promise<AuthResult> {
    const googleUser: GoogleUser = {
        id: profile.id,
        email: profile.emails[0].value,
        firstName: profile.name.givenName,
        lastName: profile.name.familyName,
        picture: profile.photos[0].value,
    };

    return this.authService.validateGoogleUser(googleUser);
}
```

**역할**: Google 프로필을 우리 시스템 형식으로 변환  
**동작**:

1. Google에서 받은 프로필 정보 파싱
2. 우리 시스템의 GoogleUser 인터페이스로 변환
3. AuthService의 사용자 검증 로직 호출
4. 검증된 사용자 정보 반환

#### `validateGoogleUser()` (auth.service.ts)

```typescript
async validateGoogleUser(googleUser: GoogleUser): Promise<AuthResult> {
    const { id, email, firstName, lastName, picture } = googleUser;

    // 1. Google ID로 기존 사용자 찾기
    let user = await this.userService.findByGoogleId(id);

    if (!user) {
        // 2. 이메일로 기존 사용자 찾기 (다른 방법으로 가입했을 수 있음)
        const existingActiveUser = await this.userService.findByEmail(email);

        if (existingActiveUser) {
            // 3. 기존 사용자에 Google 계정 연동
            user = await this.userService.linkGoogleAccount(existingActiveUser.id, {
                googleId: id,
                profilePicture: picture,
            });
        } else {
            // 4. 완전히 새로운 사용자 생성
            user = await this.userService.createGoogleUser({
                googleId: id,
                email,
                username: `${firstName} ${lastName}`.trim(),
                nickname: email.split('@')[0],
                profilePicture: picture,
            });
        }
    }

    // 5. JWT 토큰 생성
    const payload = { sub: user.id, email: user.email };
    const accessToken = this.jwtService.sign(payload);

    return { accessToken, user };
}
```

**역할**: Google 사용자 검증 및 계정 생성/연동  
**동작**:

1. Google ID로 기존 사용자 확인
2. 없으면 이메일로 기존 계정 찾기
3. 기존 계정이 있으면 Google 연동
4. 완전히 새로운 경우 계정 생성
5. JWT 액세스 토큰 생성하여 반환

#### `generateTokenPair()` (auth.service.ts)

```typescript
generateTokenPair(user: User): TokenPair {
    // 액세스 토큰 생성 (15분)
    const accessToken = this.jwtService.sign(
        {
            sub: user.id,
            email: user.email,
            type: 'access',
        },
        {
            secret: process.env.JWT_SECRET || 'your-secret-key',
            expiresIn: '15m',
        },
    );

    // 리프레시 토큰 생성 (7일)
    const refreshToken = this.jwtService.sign(
        {
            sub: user.id,
            type: 'refresh',
        },
        {
            secret: process.env.JWT_REFRESH_SECRET || 'your-refresh-secret-key',
            expiresIn: '7d',
        },
    );

    return { accessToken, refreshToken };
}
```

**역할**: JWT 토큰 쌍 생성  
**동작**:

1. 액세스 토큰: 사용자 ID, 이메일 포함, 15분 만료
2. 리프레시 토큰: 사용자 ID만 포함, 7일 만료
3. 각각 다른 시크릿 키로 서명
4. 토큰 쌍을 객체로 반환

#### `setAuthCookies()` (auth.service.ts)

```typescript
setAuthCookies(res: Response, user: User, tokens: TokenPair): void {
    const { accessToken, refreshToken } = tokens;

    // HttpOnly 쿠키로 액세스 토큰 전달 (보안 강화)
    res.cookie('accessToken', accessToken, {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
        maxAge: 15 * 60 * 1000, // 15분
    });

    // HttpOnly 쿠키로 리프레시 토큰 전달 (보안 강화)
    res.cookie('refreshToken', refreshToken, {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
        maxAge: 7 * 24 * 60 * 60 * 1000, // 7일
    });

    // 사용자 정보만 쿠키로 전달 (토큰 없이)
    res.cookie('userInfo', JSON.stringify(user), {
        httpOnly: false, // 프론트엔드에서 읽을 수 있도록
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
        maxAge: 7 * 24 * 60 * 60 * 1000, // 7일
    });
}
```

**역할**: 보안 쿠키 설정  
**동작**:

1. 액세스/리프레시 토큰을 HttpOnly 쿠키로 설정 (XSS 방어)
2. 사용자 정보는 일반 쿠키로 설정 (프론트엔드에서 읽기 가능)
3. production 환경에서는 secure 플래그 활성화
4. sameSite='strict'로 CSRF 공격 방어

#### `refreshAccessToken()` (auth.service.ts)

```typescript
async refreshAccessToken(refreshToken: string): Promise<{
    success: boolean;
    accessToken?: string;
    user?: User;
    message?: string;
}> {
    try {
        // 리프레시 토큰 검증
        const decodedToken: unknown = this.jwtService.verify(refreshToken, {
            secret: process.env.JWT_REFRESH_SECRET || 'your-refresh-secret-key',
        });

        // 토큰 구조 검증
        if (!isValidJwtPayload(decodedToken) || decodedToken.type !== 'refresh') {
            return { success: false, message: '유효하지 않은 리프레시 토큰입니다.' };
        }

        // 사용자 정보 조회
        const user = await this.userService.findById(decodedToken.sub);
        if (!user) {
            return { success: false, message: '사용자를 찾을 수 없습니다.' };
        }

        // 새 액세스 토큰 생성
        const newAccessToken = this.generateAccessToken(user);

        return { success: true, accessToken: newAccessToken, user };
    } catch {
        return { success: false, message: '유효하지 않은 리프레시 토큰입니다.' };
    }
}
```

**역할**: 리프레시 토큰으로 새 액세스 토큰 발급  
**동작**:

1. 리프레시 토큰 서명 검증
2. 토큰 구조 및 타입 확인
3. 토큰에서 사용자 ID 추출하여 DB 조회
4. 사용자가 존재하면 새 액세스 토큰 생성
5. 성공/실패 결과 반환

---

## 🔐 보안 특징

### 1. HttpOnly 쿠키

-   **액세스 토큰**과 **리프레시 토큰**을 HttpOnly 쿠키로 저장
-   JavaScript에서 접근 불가능하여 XSS 공격 방어

### 2. 토큰 만료 관리

-   **액세스 토큰**: 15분 (짧은 만료 시간으로 보안 강화)
-   **리프레시 토큰**: 7일 (적절한 만료 시간으로 보안과 편의성 균형)

### 3. 자동 토큰 갱신

-   10분마다 자동으로 새 액세스 토큰 발급
-   사용자가 별도 로그인 없이 지속적 사용 가능

### 4. CSRF 방어

-   `sameSite='strict'` 설정으로 외부 사이트에서 쿠키 접근 차단

### 5. 로그 보안

-   URL이나 콘솔에 토큰 노출 없음
-   모든 민감 정보는 HttpOnly 쿠키로 처리

---

## 🚀 기술 스택

-   **프론트엔드**: Vue 3, Pinia, TypeScript, Vue Router
-   **백엔드**: NestJS, Passport.js, JWT
-   **데이터베이스**: PostgreSQL, TypeORM
-   **인증**: Google OAuth 2.0
-   **보안**: HttpOnly 쿠키, CSRF 방어, 자동 토큰 갱신

## 📊 성능 최적화

### **중복 API 호출 방지**

-   Promise 캐싱으로 동시 초기화 방지
-   상태 플래그로 중복 요청 차단
-   중앙집중식 상태 관리

### **효율적인 토큰 관리**

-   액세스 토큰 15분, 리프레시 토큰 7일
-   10분마다 자동 토큰 갱신
-   HttpOnly 쿠키로 보안 강화

### **반응형 상태 동기화**

-   Pinia store로 전역 상태 관리
-   storeToRefs로 반응형 데이터 제공
-   컴포넌트 간 자동 상태 동기화

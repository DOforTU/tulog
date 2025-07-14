# PATCH vs PUT - HTTP 메서드 차이점 완전 정리

> HTTP PATCH와 PUT 메서드의 차이점과 각각의 올바른 사용법에 대한 상세한 가이드입니다.

## 목차

-   [개요](#개요)
-   [PUT 메서드](#put-메서드)
-   [PATCH 메서드](#patch-메서드)
-   [핵심 차이점](#핵심-차이점)
-   [실제 예시](#실제-예시)
-   [RESTful API 설계 가이드](#restful-api-설계-가이드)
-   [보안 고려사항](#보안-고려사항)
-   [성능 비교](#성능-비교)

---

## 개요

HTTP 프로토콜에서 리소스를 수정하는 두 가지 주요 메서드인 PUT과 PATCH는 비슷해 보이지만 중요한 차이점들이 있습니다. 올바른 REST API 설계를 위해서는 이 둘의 차이를 명확히 이해하고 적절한 상황에 맞는 메서드를 선택해야 합니다.

### 간단 요약

| 구분       | PUT                | PATCH                   |
| ---------- | ------------------ | ----------------------- |
| **목적**   | 전체 리소스 교체   | 부분 리소스 수정        |
| **데이터** | 완전한 리소스 표현 | 변경할 필드만           |
| **멱등성** | 멱등 (Idempotent)  | 비멱등 (Non-idempotent) |
| **안전성** | 비안전             | 비안전                  |

---

## PUT 메서드

### 정의

PUT은 **전체 리소스를 완전히 교체(Replace)**하는 메서드입니다.

### 특징

1. **완전한 리소스 표현**: 요청 본문에 리소스의 모든 필드를 포함해야 함
2. **멱등성**: 동일한 요청을 여러 번 보내도 결과가 같음
3. **전체 교체**: 기존 리소스를 요청 데이터로 완전히 대체

### 동작 방식

```http
PUT /users/123
Content-Type: application/json

{
  "id": 123,
  "name": "홍길동",
  "email": "hong@example.com",
  "age": 30,
  "phone": "010-1234-5678",
  "address": "서울시 강남구"
}
```

**결과**: 사용자 123의 모든 정보가 요청 데이터로 완전히 교체됩니다.

### 멱등성 예시

```http
# 첫 번째 요청
PUT /users/123
{ "name": "김철수", "email": "kim@example.com", "age": 25 }

# 두 번째 동일한 요청 (결과 동일)
PUT /users/123
{ "name": "김철수", "email": "kim@example.com", "age": 25 }

# 서버 상태는 첫 번째 요청 후와 동일
```

### 주의사항

```http
# 위험한 PUT 사용 예시
PUT /users/123
{
  "name": "새로운이름"
}

# 결과: name만 남고 email, age, phone 등 모든 다른 필드가 삭제됨!
```

---

## PATCH 메서드

### 정의

PATCH는 **리소스의 일부분만 수정(Partial Update)**하는 메서드입니다.

### 특징

1. **부분 수정**: 변경하고 싶은 필드만 요청에 포함
2. **비멱등성**: 동일한 요청이라도 실행 시점에 따라 결과가 다를 수 있음
3. **유연한 업데이트**: 다양한 패치 형식 지원 (JSON Merge Patch, JSON Patch 등)

### 동작 방식

```http
PATCH /users/123
Content-Type: application/json

{
  "email": "newemail@example.com",
  "age": 31
}
```

**결과**: 사용자 123의 email과 age만 수정되고, 다른 필드들은 그대로 유지됩니다.

### 비멱등성 예시

```http
# 첫 번째 요청
PATCH /users/123
{ "age": "+1" }  # 나이를 1 증가

# 두 번째 동일한 요청 (결과 달라짐)
PATCH /users/123
{ "age": "+1" }  # 또 1 증가

# 서버 상태가 달라짐 (age가 2 증가)
```

### PATCH 형식들

#### 1. JSON Merge Patch (RFC 7396)

```http
PATCH /users/123
Content-Type: application/merge-patch+json

{
  "email": "updated@example.com",
  "phone": null  # null = 필드 삭제
}
```

#### 2. JSON Patch (RFC 6902)

```http
PATCH /users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/email", "value": "new@example.com" },
  { "op": "add", "path": "/nickname", "value": "새별명" },
  { "op": "remove", "path": "/phone" }
]
```

---

## 핵심 차이점

### 1. 데이터 전송량

#### PUT

```http
PUT /users/123
{
  "id": 123,
  "name": "홍길동",
  "email": "new@example.com",    # 변경됨
  "age": 30,
  "phone": "010-1234-5678",
  "address": "서울시 강남구",
  "createdAt": "2023-01-01T00:00:00Z",
  "updatedAt": "2024-01-01T00:00:00Z"
}
```

**전송량**: 전체 리소스 (큰 페이로드)

#### PATCH

```http
PATCH /users/123
{
  "email": "new@example.com"     # 변경된 필드만
}
```

**전송량**: 변경된 필드만 (작은 페이로드)

### 2. 안전성

#### PUT - 데이터 손실 위험

```javascript
// 클라이언트 A
const user = await getUserById(123);
// user = { id: 123, name: "홍길동", email: "old@example.com", age: 30 }

// 클라이언트 B가 동시에 나이를 업데이트
// PATCH /users/123 { "age": 31 }

// 클라이언트 A가 이메일만 변경하려고 PUT 사용
await fetch("/users/123", {
    method: "PUT",
    body: JSON.stringify({
        ...user,
        email: "new@example.com",
    }),
});

// 결과: 클라이언트 B가 변경한 age=31이 age=30으로 되돌려짐!
```

#### PATCH - 안전한 부분 업데이트

```javascript
// 클라이언트 A
await fetch("/users/123", {
    method: "PATCH",
    body: JSON.stringify({
        email: "new@example.com",
    }),
});

// 결과: 다른 필드들은 영향받지 않음
```

### 3. 의도의 명확성

#### PUT

```http
PUT /users/123
{
  "name": "홍길동",
  "email": "hong@example.com"
}
```

**의도**: "사용자 123을 이 데이터로 완전히 교체하라"

#### PATCH

```http
PATCH /users/123
{
  "email": "hong@example.com"
}
```

**의도**: "사용자 123의 이메일만 변경하라"

---

## 실제 예시

### 사용자 프로필 업데이트 시나리오

#### 상황: 사용자가 프로필 설정에서 이메일만 변경

##### ❌ 잘못된 PUT 사용

```javascript
// Frontend
const updateEmail = async (newEmail) => {
    // 전체 사용자 데이터를 먼저 가져와야 함
    const user = await fetch("/users/me").then((res) => res.json());

    // 전체 데이터와 함께 PUT 요청
    await fetch("/users/me", {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            ...user.data,
            email: newEmail,
        }),
    });
};
```

**문제점:**

-   불필요한 데이터 전송
-   동시성 문제 (다른 사용자가 동시에 수정하면 데이터 손실)
-   네트워크 오버헤드

##### ✅ 올바른 PATCH 사용

```javascript
// Frontend
const updateEmail = async (newEmail) => {
    await fetch("/users/me", {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            email: newEmail,
        }),
    });
};
```

**장점:**

-   최소한의 데이터 전송
-   동시성 문제 방지
-   의도가 명확함

### 블로그 포스트 수정 시나리오

#### 전체 포스트 다시 작성 (PUT 적합)

```http
PUT /posts/456
Content-Type: application/json

{
  "title": "새로운 제목",
  "content": "완전히 새로 작성된 내용...",
  "tags": ["새태그1", "새태그2"],
  "category": "새카테고리",
  "published": true
}
```

#### 제목만 수정 (PATCH 적합)

```http
PATCH /posts/456
Content-Type: application/json

{
  "title": "수정된 제목"
}
```

#### 태그 추가 (PATCH 적합)

```http
PATCH /posts/456
Content-Type: application/json-patch+json

[
  { "op": "add", "path": "/tags/-", "value": "새태그" }
]
```

---

## RESTful API 설계 가이드

### PUT 사용 권장 시나리오

1. **전체 리소스 교체가 의도인 경우**

    ```http
    PUT /users/123/profile
    # 프로필 전체를 새로 설정
    ```

2. **리소스 생성 또는 완전 교체**

    ```http
    PUT /configurations/app-settings
    # 앱 설정을 완전히 새로 정의
    ```

3. **버전 관리가 필요한 경우**
    ```http
    PUT /documents/123/versions/2
    # 특정 버전을 완전히 교체
    ```

### PATCH 사용 권장 시나리오

1. **부분 업데이트**

    ```http
    PATCH /users/123
    # 사용자 정보 일부만 수정
    ```

2. **상태 변경**

    ```http
    PATCH /orders/456
    { "status": "shipped" }
    ```

3. **필드별 개별 수정**
    ```http
    PATCH /settings/notifications
    { "emailEnabled": false }
    ```

### NestJS 구현 예시

#### PUT 컨트롤러

```typescript
@Controller("users")
export class UserController {
    @Put(":id")
    async replaceUser(
        @Param("id") id: string,
        @Body() userData: CompleteUserDto // 모든 필드 필수
    ): Promise<User> {
        // 전체 사용자 데이터를 완전히 교체
        return this.userService.replaceUser(id, userData);
    }
}

// DTO - 모든 필드가 필수
export class CompleteUserDto {
    @IsString()
    @IsNotEmpty()
    name: string;

    @IsEmail()
    email: string;

    @IsNumber()
    age: number;

    @IsString()
    phone: string;

    @IsString()
    address: string;
}
```

#### PATCH 컨트롤러

```typescript
@Controller("users")
export class UserController {
    @Patch(":id")
    async updateUser(
        @Param("id") id: string,
        @Body() userData: PartialUserDto // 선택적 필드
    ): Promise<User> {
        // 제공된 필드만 업데이트
        return this.userService.updateUser(id, userData);
    }
}

// DTO - 모든 필드가 선택적
export class PartialUserDto {
    @IsString()
    @IsOptional()
    name?: string;

    @IsEmail()
    @IsOptional()
    email?: string;

    @IsNumber()
    @IsOptional()
    age?: number;

    @IsString()
    @IsOptional()
    phone?: string;

    @IsString()
    @IsOptional()
    address?: string;
}
```

#### 서비스 구현

```typescript
@Injectable()
export class UserService {
    // PUT - 전체 교체
    async replaceUser(id: string, userData: CompleteUserDto): Promise<User> {
        const user = await this.userRepository.findById(id);
        if (!user) {
            throw new NotFoundException("User not found");
        }

        // 전체 데이터를 새 데이터로 교체
        const replacedUser = {
            ...userData,
            id: user.id,
            createdAt: user.createdAt,
            updatedAt: new Date(),
        };

        return this.userRepository.save(replacedUser);
    }

    // PATCH - 부분 업데이트
    async updateUser(id: string, userData: PartialUserDto): Promise<User> {
        const user = await this.userRepository.findById(id);
        if (!user) {
            throw new NotFoundException("User not found");
        }

        // 제공된 필드만 업데이트
        const updatedUser = {
            ...user,
            ...userData, // 제공된 필드만 덮어씀
            updatedAt: new Date(),
        };

        return this.userRepository.save(updatedUser);
    }
}
```

---

## 보안 고려사항

### 1. 권한 검증

#### PUT - 전체 접근 권한 필요

```typescript
@Put(':id')
@UseGuards(JwtAuthGuard, OwnershipGuard)
async replaceUser(@Param('id') id: string, @Body() userData: CompleteUserDto) {
  // 사용자가 해당 리소스의 완전한 소유권을 가져야 함
  return this.userService.replaceUser(id, userData);
}
```

#### PATCH - 필드별 권한 검증

```typescript
@Patch(':id')
@UseGuards(JwtAuthGuard)
async updateUser(@Param('id') id: string, @Body() userData: PartialUserDto) {
  // 필드별로 다른 권한 검증 필요
  if (userData.role && !this.authService.isAdmin(request.user)) {
    throw new ForbiddenException('Cannot modify role');
  }

  return this.userService.updateUser(id, userData);
}
```

### 2. 입력 검증

#### PUT - 완전성 검증

```typescript
export class CompleteUserDto {
    @IsString()
    @IsNotEmpty()
    name: string;

    @IsEmail()
    email: string;

    // 모든 필수 필드가 있어야 함
    @ValidateNested()
    @Type(() => AddressDto)
    address: AddressDto;
}
```

#### PATCH - 선택적 검증

```typescript
export class PartialUserDto {
    @IsString()
    @IsOptional()
    @IsNotEmpty() // 제공되면 비어있지 않아야 함
    name?: string;

    @IsEmail()
    @IsOptional()
    email?: string;

    // 제공된 필드만 검증
}
```

### 3. 민감한 필드 보호

```typescript
// 민감한 필드는 별도 엔드포인트로 분리
@Patch(':id/password')
@UseGuards(JwtAuthGuard, PasswordVerificationGuard)
async changePassword(
  @Param('id') id: string,
  @Body() passwordData: ChangePasswordDto
) {
  return this.userService.changePassword(id, passwordData);
}

@Patch(':id/profile')
@UseGuards(JwtAuthGuard)
async updateProfile(
  @Param('id') id: string,
  @Body() profileData: ProfileUpdateDto
) {
  // 비밀번호는 포함되지 않음
  return this.userService.updateProfile(id, profileData);
}
```

---

## 성능 비교

### 네트워크 사용량

#### 예시 시나리오: 1KB 사용자 데이터에서 이메일만 변경

```javascript
// PUT 방식
const putPayload = {
    id: 123,
    name: "홍길동",
    email: "new@example.com", // 변경된 필드
    bio: "매우 긴 자기소개...", // 1000자
    preferences: {
        /* 복잡한 설정 객체 */
    },
    // ... 기타 많은 필드들
};
// 페이로드 크기: ~1KB

// PATCH 방식
const patchPayload = {
    email: "new@example.com",
};
// 페이로드 크기: ~30B
```

**네트워크 절약**: PATCH가 PUT보다 97% 적은 데이터 전송

### 데이터베이스 성능

#### PUT - 전체 업데이트

```sql
UPDATE users SET
  name = ?, email = ?, bio = ?, preferences = ?,
  updated_at = NOW()
WHERE id = ?;
```

#### PATCH - 부분 업데이트

```sql
UPDATE users SET
  email = ?, updated_at = NOW()
WHERE id = ?;
```

**장점:**

-   인덱스 영향 최소화
-   로그 크기 감소
-   잠금 시간 단축

### 동시성 처리

#### PUT - 낙관적 잠금 필요

```typescript
async replaceUser(id: string, userData: CompleteUserDto) {
  const user = await this.userRepository.findById(id);

  // 버전 체크 (낙관적 잠금)
  if (user.version !== userData.version) {
    throw new ConflictException('User has been modified by another user');
  }

  return this.userRepository.save({
    ...userData,
    version: user.version + 1
  });
}
```

#### PATCH - 더 간단한 동시성 처리

```typescript
async updateUser(id: string, userData: PartialUserDto) {
  // 필드별 업데이트는 충돌 가능성이 낮음
  return this.userRepository.update(id, userData);
}
```

---

## 에러 처리

### PUT 에러 응답

```typescript
@Put(':id')
async replaceUser(@Param('id') id: string, @Body() userData: CompleteUserDto) {
  try {
    return await this.userService.replaceUser(id, userData);
  } catch (error) {
    if (error instanceof NotFoundException) {
      throw new NotFoundException('User not found');
    }
    if (error instanceof ConflictException) {
      throw new ConflictException('User has been modified by another user');
    }
    throw new InternalServerErrorException('Failed to replace user');
  }
}
```

### PATCH 에러 응답

```typescript
@Patch(':id')
async updateUser(@Param('id') id: string, @Body() userData: PartialUserDto) {
  try {
    return await this.userService.updateUser(id, userData);
  } catch (error) {
    if (error instanceof NotFoundException) {
      throw new NotFoundException('User not found');
    }
    if (error.code === 'UNIQUE_VIOLATION') {
      throw new ConflictException('Email already exists');
    }
    throw new InternalServerErrorException('Failed to update user');
  }
}
```

---

## 최종 권장사항

### ✅ PUT 사용 케이스

-   전체 리소스를 새로운 데이터로 완전히 교체할 때
-   클라이언트가 리소스의 전체 상태를 알고 있을 때
-   멱등성이 중요한 상황에서
-   설정 파일이나 구성 데이터 같은 완전한 교체가 필요한 경우

### ✅ PATCH 사용 케이스

-   리소스의 일부 필드만 수정할 때
-   대용량 리소스에서 소량의 데이터만 변경할 때
-   모바일 환경 등 네트워크 대역폭이 제한적일 때
-   동시 수정 가능성이 있는 환경에서

### 🚫 피해야 할 실수

1. **PUT으로 부분 업데이트 시도**

    ```http
    PUT /users/123
    { "email": "new@example.com" }
    # 다른 필드들이 삭제될 수 있음!
    ```

2. **PATCH로 전체 교체 시도**

    ```http
    PATCH /users/123
    { /* 모든 필드 포함 */ }
    # PUT을 사용하는 것이 의미상 올바름
    ```

3. **멱등성 혼동**
    ```http
    PATCH /users/123
    { "loginCount": "+1" }
    # 멱등하지 않음을 인지하고 사용해야 함
    ```

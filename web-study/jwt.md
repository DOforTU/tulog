# JWT (JSON Web Token)

## JWT란?
- JSON 형태의 정보를 안전하게 전송하기 위한 토큰 기반 인증 방식
- 헤더(Header), 페이로드(Payload), 서명(Signature) 세 부분으로 구성

## 구조
```
Header.Payload.Signature
```

### 1. Header
- 토큰의 타입과 해싱 알고리즘 정보
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2. Payload
- 실제 전달할 데이터 (Claims)
- 사용자 ID, 권한, 만료시간 등
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}
```

### 3. Signature
- 헤더와 페이로드를 비밀키로 서명한 값
- 데이터 무결성 보장

## 장점
- **무상태(Stateless)**: 서버에 세션 저장 불필요
- **확장성**: 여러 서버에서 토큰 검증 가능
- **자체 완결성**: 토큰 자체에 사용자 정보 포함

## 단점
- **토큰 크기**: 쿠키보다 상대적으로 큼
- **보안**: 토큰 탈취 시 만료 전까지 사용 가능
- **저장 위치**: XSS/CSRF 공격 고려 필요

## 보안 고려사항
1. **HTTPS 사용 필수**
2. **짧은 만료시간 설정**
3. **Refresh Token 활용**
4. **안전한 저장소 사용** (httpOnly 쿠키 또는 secure storage)

## 구현 예제

### Node.js + Express
```javascript
const jwt = require('jsonwebtoken');

// 토큰 생성
const generateToken = (user) => {
  return jwt.sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );
};

// 토큰 검증 미들웨어
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access denied' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(400).json({ error: 'Invalid token' });
  }
};
```

### React (프론트엔드)
```javascript
// API 요청 시 토큰 포함
const apiCall = async () => {
  const token = localStorage.getItem('token');
  
  const response = await fetch('/api/data', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
  return response.json();
};

// 토큰 만료 시 자동 갱신
axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response.status === 401) {
      // Refresh token으로 새 토큰 요청
      const newToken = await refreshToken();
      localStorage.setItem('token', newToken);
      
      // 원래 요청 재시도
      error.config.headers['Authorization'] = `Bearer ${newToken}`;
      return axios.request(error.config);
    }
    return Promise.reject(error);
  }
);
```

## vs 세션 기반 인증

| 구분 | JWT | 세션 |
|------|-----|------|
| 저장 위치 | 클라이언트 | 서버 |
| 확장성 | 우수 | 제한적 |
| 보안 | 토큰 탈취 위험 | 서버 관리로 안전 |
| 성능 | 검증 빠름 | DB 조회 필요 |
| 로그아웃 | 복잡 | 간단 |

## 모범 사례
1. **Access Token + Refresh Token 패턴 사용**
2. **민감한 정보는 페이로드에 포함하지 않기**
3. **적절한 만료시간 설정** (Access: 15분, Refresh: 7일)
4. **토큰 블랙리스트 관리**
5. **CSRF 보호를 위한 httpOnly 쿠키 활용**
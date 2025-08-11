# 데이터베이스 기본 개념

## 관계형 데이터베이스 (RDBMS)

### 기본 개념
- **테이블**: 데이터를 행(Row)과 열(Column)로 구성
- **스키마**: 데이터베이스 구조와 제약 조건 정의
- **정규화**: 데이터 중복을 최소화하는 설계 기법
- **ACID**: 트랜잭션의 4가지 속성

### 주요 SQL 연산

#### SELECT - 데이터 조회
```sql
-- 기본 조회
SELECT id, name, email FROM users;

-- 조건부 조회
SELECT * FROM users 
WHERE age >= 18 AND status = 'active';

-- 정렬
SELECT * FROM users 
ORDER BY created_at DESC, name ASC;

-- 제한
SELECT * FROM users 
LIMIT 10 OFFSET 20;

-- 집계 함수
SELECT 
    COUNT(*) as total_users,
    AVG(age) as average_age,
    MAX(created_at) as latest_signup
FROM users
WHERE status = 'active';
```

#### JOIN - 테이블 연결
```sql
-- INNER JOIN
SELECT u.name, p.title 
FROM users u
INNER JOIN posts p ON u.id = p.user_id;

-- LEFT JOIN
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- 복합 JOIN
SELECT 
    u.name,
    p.title,
    c.content as comment
FROM users u
INNER JOIN posts p ON u.id = p.user_id
LEFT JOIN comments c ON p.id = c.post_id
WHERE u.status = 'active';
```

#### INSERT, UPDATE, DELETE
```sql
-- INSERT
INSERT INTO users (name, email, age) 
VALUES ('John Doe', 'john@example.com', 25);

-- 여러 행 삽입
INSERT INTO users (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

-- UPDATE
UPDATE users 
SET email = 'newemail@example.com', updated_at = NOW()
WHERE id = 1;

-- DELETE
DELETE FROM users 
WHERE status = 'inactive' AND last_login < '2023-01-01';
```

### 인덱스 (Index)
```sql
-- 인덱스 생성
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- 복합 인덱스
CREATE INDEX idx_posts_status_created ON posts(status, created_at);

-- 유니크 인덱스
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

## NoSQL 데이터베이스

### MongoDB 예시

#### 문서 구조
```javascript
// 사용자 문서
{
    "_id": ObjectId("..."),
    "name": "John Doe",
    "email": "john@example.com",
    "profile": {
        "age": 25,
        "location": "Seoul",
        "interests": ["programming", "music"]
    },
    "posts": [
        {
            "title": "My First Post",
            "content": "Hello World",
            "createdAt": new Date("2024-01-15")
        }
    ],
    "createdAt": new Date("2024-01-01"),
    "updatedAt": new Date("2024-01-15")
}
```

#### CRUD 연산
```javascript
// CREATE
db.users.insertOne({
    name: "John Doe",
    email: "john@example.com",
    profile: { age: 25, location: "Seoul" }
});

// READ
db.users.find({ "profile.age": { $gte: 18 } });
db.users.findOne({ email: "john@example.com" });

// UPDATE
db.users.updateOne(
    { _id: ObjectId("...") },
    { 
        $set: { "profile.age": 26 },
        $push: { "interests": "travel" }
    }
);

// DELETE
db.users.deleteOne({ email: "john@example.com" });
```

## 데이터베이스 설계 원칙

### 정규화 (Normalization)

#### 제1정규형 (1NF)
- 각 컬럼은 원자값(Atomic Value)만 포함
```sql
-- 비정규화 (잘못된 예)
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    hobbies VARCHAR(255) -- "독서, 음악, 영화" (여러 값)
);

-- 1NF (올바른 예)
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
);

CREATE TABLE user_hobbies (
    user_id INT,
    hobby VARCHAR(100)
);
```

#### 제2정규형 (2NF)
- 1NF + 부분적 함수 종속 제거
```sql
-- 2NF 위반
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100), -- product_id에만 종속
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- 2NF 준수
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

#### 제3정규형 (3NF)
- 2NF + 이행적 함수 종속 제거
```sql
-- 3NF 위반
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100) -- department_id에 종속
);

-- 3NF 준수
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

### 역정규화 (Denormalization)
성능을 위해 의도적으로 중복 데이터 허용:
```sql
-- 조회 성능을 위한 역정규화
CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    user_id INT,
    user_name VARCHAR(100), -- 중복 데이터
    comment_count INT,      -- 계산된 값 저장
    created_at TIMESTAMP
);
```

## 트랜잭션 (Transaction)

### ACID 속성
- **Atomicity (원자성)**: 모두 성공 또는 모두 실패
- **Consistency (일관성)**: 데이터 무결성 유지
- **Isolation (격리성)**: 동시 실행 트랜잭션 간 독립성
- **Durability (지속성)**: 커밋된 데이터는 영구 저장

### SQL 트랜잭션 예시
```sql
BEGIN TRANSACTION;

-- 계좌 이체 예시
UPDATE accounts 
SET balance = balance - 1000 
WHERE id = 1; -- 송금자

UPDATE accounts 
SET balance = balance + 1000 
WHERE id = 2; -- 수신자

-- 잔액 확인
SELECT balance FROM accounts WHERE id = 1;

-- 문제없으면 커밋, 문제있으면 롤백
COMMIT; -- 또는 ROLLBACK;
```

### Node.js에서 트랜잭션
```javascript
// MySQL with Sequelize
const transaction = await sequelize.transaction();

try {
    // 계좌에서 출금
    await Account.update(
        { balance: sequelize.literal('balance - 1000') },
        { where: { id: fromAccountId }, transaction }
    );
    
    // 계좌로 입금
    await Account.update(
        { balance: sequelize.literal('balance + 1000') },
        { where: { id: toAccountId }, transaction }
    );
    
    // 거래 기록
    await Transaction.create({
        fromAccountId,
        toAccountId,
        amount: 1000,
        type: 'transfer'
    }, { transaction });
    
    await transaction.commit();
} catch (error) {
    await transaction.rollback();
    throw error;
}
```

## 성능 최적화

### 인덱스 최적화
```sql
-- 쿼리 실행 계획 확인
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- 복합 인덱스 순서 중요
CREATE INDEX idx_posts_status_created ON posts(status, created_at);

-- 이 쿼리는 인덱스를 효과적으로 사용
SELECT * FROM posts 
WHERE status = 'published' 
ORDER BY created_at DESC;

-- 이 쿼리는 인덱스를 부분적으로만 사용
SELECT * FROM posts 
WHERE created_at > '2024-01-01';
```

### 쿼리 최적화
```sql
-- N+1 문제 해결: JOIN 사용
-- 비효율적
SELECT * FROM posts;
-- 각 포스트마다 추가 쿼리
SELECT * FROM users WHERE id = ?;

-- 효율적
SELECT p.*, u.name as author_name
FROM posts p
INNER JOIN users u ON p.user_id = u.id;

-- 서브쿼리 최적화
-- 비효율적
SELECT * FROM users 
WHERE id IN (
    SELECT user_id FROM posts 
    WHERE created_at > '2024-01-01'
);

-- 더 효율적
SELECT DISTINCT u.*
FROM users u
INNER JOIN posts p ON u.id = p.user_id
WHERE p.created_at > '2024-01-01';
```

### 페이지네이션 최적화
```sql
-- OFFSET 방식 (큰 페이지에서 느림)
SELECT * FROM posts 
ORDER BY id 
LIMIT 10 OFFSET 10000;

-- Cursor 방식 (더 효율적)
SELECT * FROM posts 
WHERE id > 10000 
ORDER BY id 
LIMIT 10;
```

## 데이터베이스 연결 관리

### Connection Pool
```javascript
// MySQL Connection Pool
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'myapp',
    connectionLimit: 10,        // 최대 연결 수
    acquireTimeout: 60000,      // 연결 대기 시간
    timeout: 60000,             // 쿼리 타임아웃
    reconnect: true,            // 자동 재연결
    idleTimeout: 300000         // 유휴 연결 타임아웃
});

// 사용 예시
async function getUser(id) {
    const connection = await pool.getConnection();
    try {
        const [rows] = await connection.execute(
            'SELECT * FROM users WHERE id = ?',
            [id]
        );
        return rows[0];
    } finally {
        connection.release(); // 중요: 연결 반환
    }
}
```

### ORM 사용 예시 (Sequelize)
```javascript
const { Sequelize, DataTypes } = require('sequelize');

// 데이터베이스 연결
const sequelize = new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'mysql',
    pool: {
        max: 10,
        min: 0,
        acquire: 30000,
        idle: 10000
    },
    logging: process.env.NODE_ENV === 'development' ? console.log : false
});

// 모델 정의
const User = sequelize.define('User', {
    name: {
        type: DataTypes.STRING,
        allowNull: false,
        validate: {
            notEmpty: true,
            len: [2, 50]
        }
    },
    email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true
        }
    },
    age: {
        type: DataTypes.INTEGER,
        validate: {
            min: 0,
            max: 120
        }
    }
});

// 관계 설정
const Post = sequelize.define('Post', {
    title: DataTypes.STRING,
    content: DataTypes.TEXT
});

User.hasMany(Post);
Post.belongsTo(User);

// 사용 예시
async function createUserWithPost() {
    const user = await User.create({
        name: 'John Doe',
        email: 'john@example.com',
        age: 25,
        Posts: [
            { title: 'First Post', content: 'Hello World' }
        ]
    }, {
        include: [Post]
    });
    
    return user;
}
```

## 보안 고려사항

### SQL Injection 방지
```javascript
// 위험한 방법 (절대 사용 금지)
const query = `SELECT * FROM users WHERE email = '${email}'`;

// 안전한 방법 - Prepared Statements
const query = 'SELECT * FROM users WHERE email = ?';
const [rows] = await connection.execute(query, [email]);

// ORM 사용 (자동으로 이스케이프)
const user = await User.findOne({
    where: { email: email }
});
```

### 접근 권한 관리
```sql
-- 사용자별 권한 설정
CREATE USER 'app_user'@'%' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_user'@'%';

-- 읽기 전용 사용자
CREATE USER 'readonly_user'@'%' IDENTIFIED BY 'password';
GRANT SELECT ON myapp.* TO 'readonly_user'@'%';

-- 특정 테이블만 접근
GRANT SELECT ON myapp.users TO 'limited_user'@'%';
```

### 데이터 암호화
```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');

// 패스워드 해싱
async function hashPassword(password) {
    const saltRounds = 12;
    return await bcrypt.hash(password, saltRounds);
}

// 민감한 데이터 암호화
function encrypt(text, key) {
    const cipher = crypto.createCipher('aes-256-cbc', key);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return encrypted;
}
```

## 백업과 복구

### MySQL 백업
```bash
# 전체 데이터베이스 백업
mysqldump -u root -p myapp > backup.sql

# 특정 테이블만 백업
mysqldump -u root -p myapp users posts > tables_backup.sql

# 구조만 백업
mysqldump -u root -p --no-data myapp > schema_backup.sql

# 복구
mysql -u root -p myapp < backup.sql
```

### MongoDB 백업
```bash
# 백업
mongodump --host localhost --port 27017 --db myapp

# 복구
mongorestore --host localhost --port 27017 --db myapp dump/myapp/
```

## 모니터링과 로깅

### 성능 모니터링
```javascript
// 쿼리 실행 시간 측정
async function executeQueryWithTiming(query, params) {
    const startTime = Date.now();
    
    try {
        const result = await connection.execute(query, params);
        const executionTime = Date.now() - startTime;
        
        // 느린 쿼리 로깅
        if (executionTime > 1000) {
            console.warn(`Slow query detected: ${executionTime}ms`, {
                query,
                params,
                executionTime
            });
        }
        
        return result;
    } catch (error) {
        console.error('Query execution failed', {
            query,
            params,
            error: error.message
        });
        throw error;
    }
}
```

## 모범 사례
1. **정규화와 역정규화 균형**: 데이터 무결성과 성능 고려
2. **적절한 인덱스**: 과도한 인덱스는 INSERT/UPDATE 성능 저하
3. **연결 풀 관리**: 적절한 크기와 타임아웃 설정
4. **트랜잭션 최소화**: 필요한 경우에만 사용, 빠른 커밋/롤백
5. **보안 우선**: SQL Injection 방지, 접근 권한 관리
6. **정기 백업**: 자동화된 백업 시스템 구축
7. **모니터링**: 느린 쿼리, 연결 상태, 디스크 사용량 추적
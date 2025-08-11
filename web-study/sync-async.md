# 동기(Synchronous) vs 비동기(Asynchronous)

## 기본 개념

### 동기 (Synchronous)
- **순차적 실행**: 코드가 위에서 아래로 순서대로 실행
- **블로킹**: 이전 작업이 완료될 때까지 다음 작업 대기
- **예측 가능**: 실행 순서를 쉽게 파악 가능

### 비동기 (Asynchronous)  
- **병렬적 실행**: 여러 작업을 동시에 처리
- **논블로킹**: 작업 완료를 기다리지 않고 다음 코드 실행
- **효율적**: I/O 작업 시 성능 향상

## JavaScript에서의 차이점

### 동기 코드 예시
```javascript
console.log('1. 시작');

function syncTask() {
    // 시간이 오래 걸리는 작업 시뮬레이션
    const start = Date.now();
    while (Date.now() - start < 2000) {
        // 2초 동안 블로킹
    }
    return '동기 작업 완료';
}

console.log('2. ' + syncTask());
console.log('3. 끝');

// 출력 순서: 1 → 2 (2초 후) → 3
```

### 비동기 코드 예시
```javascript
console.log('1. 시작');

// Promise
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log('2. API 데이터:', data));

console.log('3. 끝');

// 출력 순서: 1 → 3 → 2 (API 응답 후)
```

## 비동기 처리 방법

### 1. Callback
```javascript
function fetchData(callback) {
    setTimeout(() => {
        const data = { id: 1, name: 'John' };
        callback(data);
    }, 1000);
}

fetchData((result) => {
    console.log('받은 데이터:', result);
});

// 콜백 지옥 (Callback Hell) 문제
getData(id, (data) => {
    getUser(data.userId, (user) => {
        getProfile(user.id, (profile) => {
            // 깊은 중첩...
        });
    });
});
```

### 2. Promise
```javascript
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true;
            if (success) {
                resolve({ id: 1, name: 'John' });
            } else {
                reject(new Error('데이터 가져오기 실패'));
            }
        }, 1000);
    });
}

// Promise 체이닝
fetchData()
    .then(data => {
        console.log('데이터:', data);
        return processData(data);
    })
    .then(processed => {
        console.log('처리된 데이터:', processed);
    })
    .catch(error => {
        console.error('에러:', error);
    });
```

### 3. Async/Await
```javascript
async function handleData() {
    try {
        console.log('시작');
        
        const data = await fetchData();
        console.log('받은 데이터:', data);
        
        const processed = await processData(data);
        console.log('처리된 데이터:', processed);
        
    } catch (error) {
        console.error('에러:', error);
    }
}

handleData();
```

## 실제 사용 사례

### API 호출
```javascript
// 동기적 접근 (권장하지 않음)
function syncApiCall() {
    // XMLHttpRequest를 동기 모드로 사용 (deprecated)
    const xhr = new XMLHttpRequest();
    xhr.open('GET', '/api/users', false); // false = 동기
    xhr.send();
    return JSON.parse(xhr.responseText);
}

// 비동기적 접근 (권장)
async function asyncApiCall() {
    try {
        const response = await fetch('/api/users');
        const users = await response.json();
        return users;
    } catch (error) {
        console.error('API 호출 실패:', error);
        throw error;
    }
}
```

### 파일 처리 (Node.js)
```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// 동기
function readFileSync() {
    try {
        const data = fs.readFileSync('file.txt', 'utf8');
        console.log(data);
    } catch (error) {
        console.error(error);
    }
}

// 비동기 - Callback
function readFileAsync() {
    fs.readFile('file.txt', 'utf8', (err, data) => {
        if (err) {
            console.error(err);
            return;
        }
        console.log(data);
    });
}

// 비동기 - Async/Await
async function readFileAsyncAwait() {
    try {
        const data = await fsPromises.readFile('file.txt', 'utf8');
        console.log(data);
    } catch (error) {
        console.error(error);
    }
}
```

## 병렬 처리

### Promise.all (모든 작업 완료 대기)
```javascript
async function parallelExecution() {
    const promises = [
        fetch('/api/users'),
        fetch('/api/posts'),
        fetch('/api/comments')
    ];
    
    try {
        const results = await Promise.all(promises);
        const [users, posts, comments] = await Promise.all(
            results.map(res => res.json())
        );
        
        console.log({ users, posts, comments });
    } catch (error) {
        // 하나라도 실패하면 전체 실패
        console.error('병렬 처리 실패:', error);
    }
}
```

### Promise.allSettled (모든 작업 결과 확인)
```javascript
async function parallelExecutionSettled() {
    const promises = [
        fetch('/api/users'),
        fetch('/api/posts'),
        fetch('/api/comments')
    ];
    
    const results = await Promise.allSettled(promises);
    
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`작업 ${index + 1} 성공:`, result.value);
        } else {
            console.log(`작업 ${index + 1} 실패:`, result.reason);
        }
    });
}
```

## Event Loop와 비동기 처리

```javascript
console.log('1'); // 동기

setTimeout(() => {
    console.log('2'); // 매크로 태스크
}, 0);

Promise.resolve().then(() => {
    console.log('3'); // 마이크로 태스크
});

console.log('4'); // 동기

// 출력 순서: 1 → 4 → 3 → 2
// 마이크로 태스크가 매크로 태스크보다 우선순위 높음
```

## 성능 고려사항

### 동기 처리가 적합한 경우
- **간단한 계산**: 복잡하지 않은 연산
- **순서가 중요**: 반드시 순차적으로 실행되어야 하는 로직
- **초기화 코드**: 앱 시작 시 필수 설정

### 비동기 처리가 적합한 경우
- **I/O 작업**: 파일 읽기/쓰기, 네트워크 요청
- **사용자 상호작용**: UI 반응성 유지
- **백그라운드 작업**: 데이터 처리, 로그 기록

## 에러 처리

### Promise 에러 처리
```javascript
// Unhandled Promise Rejection 방지
process.on('unhandledRejection', (reason, promise) => {
    console.error('처리되지 않은 Promise 거부:', reason);
});

// 전역 에러 핸들러
window.addEventListener('unhandledrejection', (event) => {
    console.error('처리되지 않은 Promise:', event.reason);
    event.preventDefault(); // 기본 에러 표시 방지
});
```

### Async/Await 에러 처리
```javascript
async function robustAsyncFunction() {
    try {
        const result = await riskyAsyncOperation();
        return result;
    } catch (error) {
        // 특정 에러 타입별 처리
        if (error.code === 'NETWORK_ERROR') {
            return await retryOperation();
        }
        
        // 로깅
        console.error('비동기 작업 실패:', error);
        
        // 사용자에게 친화적인 에러 메시지
        throw new Error('작업을 완료할 수 없습니다. 다시 시도해주세요.');
    }
}
```

## 모범 사례
1. **I/O 작업은 항상 비동기로 처리**
2. **에러 처리 필수**: try-catch 또는 .catch() 사용
3. **Promise 체이닝보다 async/await 선호**
4. **병렬 처리 가능한 작업은 Promise.all 활용**
5. **메모리 누수 방지**: 불필요한 Promise 체인 정리
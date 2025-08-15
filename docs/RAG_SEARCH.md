# RAG 기반 블로그 검색 시스템 개발 기획서

## 1. 프로젝트 개요

### 1.1 프로젝트명

블로그 포스트 의미 검색 시스템 (RAG 기반)

### 1.2 목적

기존 키워드 기반 검색의 한계를 극복하고, 사용자의 자연어 질문에 대해 맥락을 이해하여 관련 블로그 포스트를 찾아 AI가 생성한 답변을 제공하는 시스템 구축. 즉 사용자는 키워드 입력이 아닌 자연어 질문을 입력하고, 관련된 포스트와 AI 답변 및 포스트 요약을 확인 가능.

### 1.3 기대 효과

**서비스 부분**

-   사용자 경험 향상 (자연어 질문 가능)
-   블로그 콘텐츠 활용도 증대

**개발 경험 부분**

-   AI 활용 개발 경험 축적
-   포트폴리오 차별화

## 2. 기술 스택

### 2.1 Backend

-   **기존**: NestJS, TypeORM, PostgreSQL
-   **추가**: Python FastAPI (RAG 서버), LangChain, OpenAI API

> RAG 서버를 FastAPI를 활용할지, Django를 활용할지 아직 미정

### 2.2 Vector Database

-   **1순위**: Chroma (오픈소스, 로컬 개발 용이)
-   **2순위**: Pinecone (클라우드, 스케일링 용이)

### 2.3 AI/ML

-   **임베딩 모델**: OpenAI text-embedding-ada-002
-   **LLM**: OpenAI GPT-3.5-turbo 또는 GPT-4
-   **프레임워크**: LangChain

## 3. 시스템 아키텍처

```
[기존 NestJS App] ↔ [Python RAG Server] ↔ [Vector DB (Chroma)]
                                        ↔ [OpenAI API]
```

### 3.1 데이터 플로우

1. **인덱싱 단계**: 블로그 포스트 → 청크 분할 → 임베딩 생성 → Vector DB 저장
2. **검색 단계**: 사용자 질문 → 임베딩 생성 → 유사도 검색 → 관련 문서 추출 → LLM 답변 생성

## 4. 기능 명세

### 4.1 핵심 기능

#### 4.1.1 의미 기반 검색

-   자연어 질문 입력 지원
-   키워드 매칭이 아닌 의미 유사도 기반 검색
-   관련도 높은 상위 N개 문서 반환

#### 4.1.2 AI 답변 생성

-   검색된 문서를 바탕으로 한 종합적인 답변 제공
-   소스 문서 출처 표시
-   답변 신뢰도 표시

#### 4.1.3 하이브리드 검색

-   키워드 검색과 의미 검색 결과 통합
-   가중치 조절 가능

### 4.2 부가 기능

#### 4.2.1 필터링

-   카테고리별 검색
-   날짜 범위 지정
-   태그 기반 필터링

#### 4.2.2 검색 기록

-   사용자 검색 히스토리 저장
-   인기 검색어 분석

## 5. 구현 계획

### 5.1 Phase 1: 기본 RAG 시스템 구축 (2주)

-   [ ] Python FastAPI 서버 구축
-   [ ] LangChain 기본 파이프라인 구현
-   [ ] Chroma Vector DB 연동
-   [ ] 기존 블로그 포스트 인덱싱
-   [ ] 기본 검색 API 구현

### 5.2 Phase 2: NestJS 통합 (1주)

-   [ ] NestJS에서 Python RAG 서버 연동
-   [ ] 검색 컨트롤러 구현
-   [ ] 프론트엔드 검색 UI 추가

### 5.3 Phase 3: 고도화 (2주)

-   [ ] 하이브리드 검색 구현
-   [ ] 필터링 기능 추가
-   [ ] 검색 결과 캐싱
-   [ ] 성능 최적화

### 5.4 Phase 4: 배포 및 모니터링 (1주)

-   [ ] 운영 환경 배포
-   [ ] 모니터링 설정
-   [ ] 사용자 피드백 수집

## 6. 예상 비용

### 6.1 OpenAI API 비용

-   **임베딩**: $0.0001 / 1K tokens
-   **GPT-3.5**: $0.002 / 1K tokens
-   **월 예상**: $10-30 (개발 단계)

### 6.2 인프라 비용

-   **Vector DB**: Chroma (무료) 또는 Pinecone ($70/월)
-   **서버**: 기존 인프라 활용

## 7. 기술적 도전과제

### 7.1 성능 최적화

-   대용량 문서 처리 시 응답 속도
-   임베딩 벡터 저장 공간 최적화

### 7.2 정확도 향상

-   청킹 전략 최적화
-   프롬프트 엔지니어링

### 7.3 다국어 지원

-   한국어 임베딩 모델 고려
-   다국어 블로그 포스트 처리

## 8. 성공 지표

### 8.1 기술적 지표

-   검색 응답 시간 < 3초
-   검색 정확도 > 80%
-   시스템 가용성 > 99%

### 8.2 사용자 지표

-   검색 만족도 (별점)
-   검색 후 포스트 클릭률
-   재방문률

## 9. LangChain 이론 (실전 예제)

### 9.1 LangChain이란?

**간단히 말하면**: AI 레고 블록! 복잡한 AI 작업을 작은 조각들로 나누어 조립하는 도구입니다.

**비유**: 요리를 할 때 재료 준비 → 조리 → 플레이팅 단계가 있듯이, AI 작업도 여러 단계로 나누어 처리합니다.

### 9.2 핵심 컴포넌트 (실제 예제)

#### 9.2.1 Models - AI의 뇌

```python
from langchain.llms import OpenAI
from langchain.embeddings import OpenAIEmbeddings

# 텍스트 생성하는 뇌
llm = OpenAI(temperature=0.7)

# 텍스트를 숫자(벡터)로 바꾸는 뇌
embeddings = OpenAIEmbeddings()

# 예시
result = llm("NestJS에서 가장 중요한 개념 3가지만 알려줘")
# → "1. 모듈 시스템, 2. 의존성 주입, 3. 데코레이터"
```

#### 9.2.2 Prompts - AI에게 주는 지시서

```python
from langchain.prompts import PromptTemplate

# 나쁜 예: 지시가 애매함
bad_prompt = "이 코드에 대해 설명해줘"

# 좋은 예: 구체적인 지시
good_template = """
다음 {language} 코드를 분석하고:
1. 주요 기능 설명
2. 개선점 제안
3. 보안 이슈 체크

코드:
{code}

답변 형식: 초보자도 이해할 수 있게 설명
"""

prompt = PromptTemplate(
    template=good_template,
    input_variables=["language", "code"]
)

# 사용
final_prompt = prompt.format(
    language="TypeScript",
    code="async function deleteComment(id: number) { ... }"
)
```

#### 9.2.3 Memory - AI의 기억

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()

# 대화 1
memory.save_context(
    {"input": "NestJS 프로젝트 시작하는 방법"},
    {"output": "npm install -g @nestjs/cli 후 nest new project"}
)

# 대화 2 - 이전 대화를 기억함
memory.save_context(
    {"input": "그 프로젝트에 JWT 인증 추가하려면?"},
    {"output": "passport-jwt 패키지를 설치하고..."}
)

print(memory.load_memory_variables({}))
# → 이전 대화 내용들이 모두 출력됨
```

#### 9.2.4 Chains - 작업 흐름 연결

```python
from langchain.chains import LLMChain, SimpleSequentialChain

# 체인 1: 코드 분석
analyze_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        template="다음 코드의 문제점을 찾아줘: {code}",
        input_variables=["code"]
    )
)

# 체인 2: 개선 방안 제시
improve_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        template="다음 문제점들을 해결하는 코드를 작성해줘: {problems}",
        input_variables=["problems"]
    )
)

# 두 체인을 연결
overall_chain = SimpleSequentialChain(
    chains=[analyze_chain, improve_chain]
)

# 실행: 코드 분석 → 문제점 발견 → 개선 코드 생성
result = overall_chain.run("async function getData() { return fetch('/api') }")
```

### 9.3 실제 블로그 검색 예제

#### 9.3.1 Document Loaders - 문서 읽기

```python
from langchain.document_loaders import TextLoader
import os

# 블로그 포스트들 읽기
blog_posts = []
for filename in os.listdir("./blog_posts"):
    if filename.endswith(".md"):
        loader = TextLoader(f"./blog_posts/{filename}")
        documents = loader.load()
        blog_posts.extend(documents)

# 결과: 모든 .md 파일이 Document 객체로 변환됨
print(f"총 {len(blog_posts)}개의 블로그 포스트 로드")
```

#### 9.3.2 Text Splitters - 문서 자르기

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 긴 블로그 포스트를 작은 조각으로 나누기
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,      # 1000자씩 자르기
    chunk_overlap=200     # 앞뒤 200자는 겹치게 (맥락 보존)
)

# 예시: 3000자 블로그 포스트
long_post = "NestJS는 Node.js 기반의 프레임워크입니다... (3000자)"

chunks = text_splitter.split_text(long_post)
# → [chunk1(1000자), chunk2(1000자), chunk3(1000자)]
# 각 chunk는 앞뒤 200자씩 겹침
```

#### 9.3.3 Vector Stores - 검색 데이터베이스

```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()

# 문서들을 벡터로 변환해서 저장
vectorstore = Chroma.from_texts(
    texts=["NestJS 컨트롤러 만들기", "TypeORM 엔티티 설정", "JWT 인증 구현"],
    embeddings=embeddings
)

# 검색 테스트
results = vectorstore.similarity_search("로그인 기능 구현", k=2)
# → ["JWT 인증 구현", "NestJS 컨트롤러 만들기"] 반환
```

#### 9.3.4 완전한 RAG 시스템 예제

```python
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# 전체 시스템 조립
qa_system = RetrievalQA.from_chain_type(
    llm=OpenAI(temperature=0),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# 실제 질문
question = "NestJS에서 댓글 삭제 기능을 만들려면 어떻게 해야 해요?"

# 시스템 동작:
# 1. 질문을 벡터로 변환
# 2. 유사한 문서 3개 찾기
# 3. 찾은 문서 + 질문을 LLM에게 전달
# 4. LLM이 종합적인 답변 생성

answer = qa_system.run(question)
print(answer)
# → "댓글 삭제 기능을 구현하려면 다음과 같은 단계를 따르세요:
#    1. Comment 엔티티에 deletedAt 필드 추가
#    2. soft delete 메서드 구현
#    3. 컨트롤러에서 DELETE 엔드포인트 생성..."
```

### 9.4 고급 패턴 (쉬운 예제)

#### 9.4.1 Multi-Query - 다각도 검색

```python
from langchain.retrievers import MultiQueryRetriever

# 사용자 질문: "로그인이 안 돼요"
# AI가 자동으로 여러 각도의 질문 생성:
# - "JWT 토큰 인증 문제"
# - "로그인 API 오류 해결"
# - "인증 미들웨어 설정"

multi_retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=OpenAI(temperature=0)
)

# 더 정확한 검색 결과 제공
documents = multi_retriever.get_relevant_documents("로그인이 안 돼요")
```

#### 9.4.2 Self-Query - 스마트 필터링

```python
from langchain.retrievers.self_query.base import SelfQueryRetriever

# 사용자: "2024년에 작성된 NestJS 관련 글 찾아줘"
# AI가 자동으로 분석:
# - 내용 조건: "NestJS 관련"
# - 메타데이터 조건: "date >= 2024-01-01"

self_retriever = SelfQueryRetriever.from_llm(
    llm=OpenAI(temperature=0),
    vectorstore=vectorstore,
    document_contents="블로그 포스트 내용",
    metadata_field_info=[
        {"name": "date", "type": "string"},
        {"name": "category", "type": "string"}
    ]
)
```

### 9.5 실전 팁

#### 9.5.1 성능 최적화

```python
# 캐싱으로 비용 절약
from langchain.cache import SQLiteCache
import langchain

langchain.llm_cache = SQLiteCache(database_path=".langchain.db")

# 같은 질문을 또 하면 캐시에서 즉시 응답
llm = OpenAI()
result1 = llm("NestJS란?")  # API 호출 (비용 발생)
result2 = llm("NestJS란?")  # 캐시에서 응답 (비용 0)
```

#### 9.5.2 비용 관리

```python
# 토큰 사용량 추적
import tiktoken

def count_tokens(text, model="gpt-3.5-turbo"):
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# 질문 전에 토큰 수 확인
question = "매우 긴 질문..."
tokens = count_tokens(question)
print(f"이 질문은 {tokens}토큰, 예상 비용: ${tokens * 0.002 / 1000:.4f}")
```

### 9.6 왜 LangChain을 쓸까?

#### 9.6.1 LangChain 없이 (복잡함)

```python
# 직접 구현 - 복잡하고 길음
import openai

def manual_rag(question):
    # 1. 문서 검색
    docs = search_documents(question)

    # 2. 프롬프트 만들기
    context = "\n".join(docs)
    prompt = f"문서: {context}\n질문: {question}\n답변:"

    # 3. API 호출
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=500
    )

    return response.choices[0].text
```

#### 9.6.2 LangChain 사용 (간단함)

```python
# LangChain - 간단하고 직관적
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    retriever=vectorstore.as_retriever()
)

answer = qa_chain.run("질문")
```

**결론**: LangChain은 복잡한 AI 작업을 레고 블록처럼 쉽게 조립할 수 있게 해주는 도구입니다.

## 10. 결론

본 프로젝트는 기존 블로그 검색의 한계를 극복하고 AI를 활용한 차세대 검색 경험을 제공할 것입니다. LangChain을 활용한 RAG 시스템으로 의미 있는 포트폴리오를 구축하고, 실무에서 요구하는 AI 활용 역량을 습득할 수 있을 것으로 기대됩니다.

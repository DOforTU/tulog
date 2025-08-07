# TULOG

> Server Repository: [github.com/DOforTU/tulog-server](https://github.com/DOforTU/tulog-server)
> UI Repository: [github.com/DOforTU/tulog-ui](https://github.com/DOforTU/tulog-ui)

## 프로젝트 배경

이 프로젝트는 개인 및 팀 단위의 기술 블로그 운영을 직접 구현하고자 시작되었습니다. 다양한 기술 스택을 실무에 가깝게 적용하고, 배포 및 운영까지 고려한 시스템 구축을 목표로 합니다. 특히 팀 블로그 기능과 검색 기능을 통해 복잡한 예외 상황을 설계하고 처리하는 경험을 중점적으로 다루고자 하였습니다.

## 주요 기능

TULOG은 개인 및 팀 블로그를 운영할 수 있는 플랫폼으로, 다음과 같은 기능을 제공합니다:

### 사용자

-   Google OAuth 및 로컬 회원가입/로그인
-   프로필 수정 (닉네임, 이름, 이미지)
-   팔로우, 차단, 신고 기능

### 블로그

-   마크다운 기반 글 작성, 이미지 업로드
-   공개/비공개 설정, 임시 저장, 수정/삭제
-   태그, 즐겨찾기, 숨기기 기능

### 팀

-   팀 생성, 팀원 초대/참여 요청/탈퇴/강퇴
-   팀장 위임 및 팀 상태 관리
-   팀 즐겨찾기, 필터링 조회

### 댓글 & 좋아요

-   댓글 작성, 수정, 삭제
-   공개 글에 좋아요 및 필터링

### 검색

-   블로그/사용자 통합 검색
-   키워드 기반 검색 및 향후 의미 기반 검색 지원 예정

## 기술 스택

-   **Frontend**: Next.js (SSR/SSG 대응, SEO 최적화), TypeScript
-   **Backend**: NestJS (모듈 기반 구조, DI 지원), TypeORM, PostgreSQL
-   **인증/보안**: Google OAuth 2.0, JWT(access/refresh token), HttpOnly Cookie
-   **배포 인프라**: AWS EC2, RDS, S3, Docker, Nginx

Next.js의 SSR/SSG 기능을 활용해 초기 렌더링 속도와 SEO를 고려하였고, NestJS의 모듈 아키텍처로 유지보수성과 확장성을 확보하였습니다.

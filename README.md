# SCRD 프론트엔드

SCRD는 방탈출 테마 탐색, 예약, 리뷰를 지원하는 SPA입니다.
사용자 흐름은 이커머스 서비스와 유사하게 구성되어 있습니다.
테마 탐색 -> 검색/필터 -> 상세 확인 -> 예약 가능 시간 확인 -> 예약 -> 리뷰 확인/작성

[![SCRD 앱 홍보영상](https://img.youtube.com/vi/Qu4Drg5c4mA/0.jpg)](https://www.youtube.com/watch?v=Qu4Drg5c4mA)

- **기간**: 2024.10.23 ~ 2025.05.20 (총 7개월)<br>
- [📱 Google Play에서 SCRD 앱 설치하기](https://play.google.com/store/apps/details?id=com.scrd.scrd)
- [📽 베타 버전 데모 영상 (Google Drive)](https://drive.google.com/drive/folders/1C0baog9rQ4LC-XmpKbN3uXVEPXcWIz9O)
- [🌐 SCRD 웹페이지 접속하기](https://scrd.netlify.app/)

## 프로젝트 소개

- **형태**: React 기반 SPA (Single Page Application)
- **도메인**: 방탈출 예약/리뷰 서비스
- **핵심 방향**: 방탈출 테마를 "상품"처럼 탐색/비교/선택할 수 있도록 정보 구조 설계
- **주요 스택**: React, React Router, Recoil, React Query, Axios, styled-components

## 주요 기능

- **검색/필터**
  - 테마명 키워드 검색
  - 다중 조건 필터링: 지역, 난이도 범위, 공포도/활동성
  - 결과 기반 지역 드롭다운 재필터링
- **예약**
  - 테마 상세 페이지에서 날짜별 예약 가능 시간 조회
  - 7일 단위 탐색 UI로 일정 비교 지원
- **리뷰**
  - 테마별 리뷰 조회 및 렌더링
  - 최신순 정렬, 태그 표시, 펼치기/접기 인터랙션
- **인증 플로우**
  - 카카오 OAuth 로그인 콜백 처리
  - Access/Refresh 토큰 저장 및 앱 초기화 시 세션 복원
  - 401 응답 시 세션 정리 후 로그인 페이지로 이동

## 아키텍처

상태는 의도적으로 **클라이언트 상태**와 **서버 상태**로 분리해 관리합니다.

- **클라이언트 상태 (Recoil)**
  - 인증/세션 상태: `tokenState`, `refreshTokenState`, `userTokenState`
  - OAuth 중간 상태: `codeState`
- **서버 상태 (React Query)**
  - API 응답 캐싱, 재시도, stale 정책 관리
  - mutation 이후 query invalidation으로 UI-서버 데이터 동기화
- **구성 방식**
  - `App`에서 전역 Provider를 `ThemeProvider -> RecoilRoot -> QueryClientProvider` 순서로 구성
  - 토큰 초기화 로직으로 라우팅 이전 세션 복원 처리

## 기술적 결정

- **왜 Recoil + React Query 조합인가**
  - Recoil: 전역 UI/인증 상태를 가볍게 관리
  - React Query: 비동기 서버 데이터 생명주기(fetch/cache/stale/retry) 관리
  - 한 도구에 책임을 몰지 않고, 역할을 분리해 유지보수성을 높이기 위함
- **왜 커스텀 훅으로 설계했는가**
  - 반복되는 API/쿼리 로직을 한 곳에 모아 재사용성 확보
  - 페이지/컴포넌트는 렌더링과 인터랙션에 집중
  - 캐시 시간, 재시도, 헤더 정책 등 변경 포인트를 중앙화

## 핵심 구현

- **`useApi` 훅**
  - React Query 기반 공통 CRUD 래퍼(`useGet`, `usePost`, `usePut`, `useDelete`)
  - 쓰기 작업 이후 query invalidation 전략 적용
- **Axios 인터셉터 토큰 갱신**
  - 요청 인터셉터에서 토큰 만료를 확인하고 필요 시 refresh 토큰으로 재발급
  - 응답 인터셉터에서 인증 오류를 처리하고 세션 정리 경로 통합
- **필터 상태 관리**
  - `useFetchFilteredThemes`에서 필터 상태를 기반으로 endpoint/params 생성
  - query key에 필터 객체를 포함해 조건별 캐시 분리

## 프로젝트 구조

```text
scrd-front/
├─ escape-room/
│  ├─ src/
│  │  ├─ api/            # axios 인스턴스, 인터셉터
│  │  ├─ hooks/          # useApi, useFetchFilteredThemes
│  │  ├─ store/          # recoil atom 정의
│  │  ├─ components/     # 공통 UI (Header, OptionBar, CardSwiper, Reservation, Review)
│  │  ├─ pages/          # 라우트 페이지 (Login, ThemePage, Detail, MyPage, TierPage 등)
│  │  ├─ utils/          # token decode 등 유틸리티
│  │  ├─ App.js          # Provider 구성 및 앱 초기화
│  │  └─ Router.js       # lazy loading 기반 라우팅
│  ├─ package.json
│  └─ README.md
└─ README.md
```

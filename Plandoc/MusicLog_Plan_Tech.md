# MusicLog — 기술 문서
## 재생 전략, DB 설계, 기술 리스크, Phase 0 검증

> 기술 스택 및 아키텍처(Flutter MVVM, 폴더 구조, 상태관리)는 [MusicLog_Architecture.md](MusicLog_Architecture.md) 참조

---

## 1. 기술 스택

### Backend

```text
FastAPI 또는 Node.js
PostgreSQL
Redis (선택 사항)
```

### Auth / Account

```text
자체 로그인
Spotify OAuth
Google OAuth (YouTube Feasibility Prototype 전용)
```

### Music APIs / SDKs

| 항목 | 용도 | 비고 |
|------|------|------|
| Spotify Web API | 곡 검색, 트랙 정보, 현재 재생곡 | MVP 핵심 |
| Spotify App Remote SDK | 모바일 앱에서 Spotify 원격 제어, seekTo() | Premium 전용 |
| Spotify Web Playback SDK | 브라우저에서 Spotify 직접 재생 | **모바일 앱 사용 불가, MVP 범위 외** |
| YouTube IFrame Player API | WebView 안 YouTube 재생 실험 | Phase 0 Prototype 전용 |
| YouTube Data API | YouTube 곡 검색, 영상 정보 | Phase 2 이후 |
| iTunes Search API | 30초 Preview URL 제공 | Fallback 전략용 |

---

## 2. 재생 전략

### 2.1. Spotify 재생

#### 방식 1: Spotify App Remote SDK (MVP 기본)

```text
- MusicLog 화면을 유지하면서 Spotify 앱을 원격 제어
- Spotify 앱 설치 및 Premium 계정 필요
- seekTo(startSec)로 특정 구간 재생 가능
- [주의] seekTo() 정밀도는 네트워크 지연 + Spotify 내부 버퍼링에 따라 달라짐
         → Phase 0에서 반드시 정밀도 검증 필요
```

#### 방식 2: Spotify Web Playback SDK

```text
- 브라우저 전용 SDK
- Flutter 모바일 앱에서는 동작하지 않음
- 웹 버전 MusicLog 고려 시에만 해당
- MVP 범위 외
```

#### 방식 3: Spotify 외부 앱 열기 (Fallback)

```text
- Spotify URI 스킴을 통해 Spotify 앱으로 이동
- 가장 안정적인 fallback
- MusicLog 앱을 벗어나는 단점 존재
- Spotify Free 계정 사용자의 기본 경로
```

#### 재생 방식 선택 기준

```text
Premium 계정 + App Remote 연결 성공 → App Remote SDK seekTo() 사용
seekTo() 정밀도 검증 실패           → iTunes Preview Fallback으로 전환
Free 계정 또는 App Remote 연결 실패 → 외부 앱 열기
```

---

### 2.2. YouTube 재생 (Phase 0 검증 대상)

#### 방식 1: YouTube IFrame Player API

```text
- WebView 안에 embedded YouTube player 삽입
- startSeconds / endSeconds 파라미터로 구간 지정
- seekTo() 메서드로 특정 시간으로 이동
- controls=0으로 일부 UI 숨기기 가능
- [한계] YouTube 로고, 광고, 오버레이 완전 제거 불가
- [한계] 광고 노출 여부를 앱에서 보장할 수 없음
- [한계] WebView의 Google 로그인 상태가 앱과 다를 수 있음
```

#### 방식 2: YouTube 외부 앱 열기 (Fallback)

```text
- YouTube URI 스킴을 통해 YouTube 앱으로 이동
- 안정적이나 MusicLog 앱을 벗어남
- 15초 감상 UX 약화
```

---

### 2.3. iTunes Search API Fallback

#### 기본 방향

Spotify App Remote seekTo() 정밀도 검증이 실패할 경우 대안으로 사용한다.

#### 구현 방식

```text
- iTunes Search API로 곡 검색
- 응답에 포함된 previewUrl (m4a, 30초) 취득
- MusicLog 앱 내장 Native Audio Player로 직접 재생
- 외부 앱 / 구독 / 로그인 일절 불필요
```

#### 정확한 한계

```text
- previewUrl은 Apple이 선택한 30초 고정 클립이다.
- 사용자가 타임스탬프를 지정해도 그 구간으로 자르는 것이 아니다.
- Apple이 미리 잘라둔 30초를 재생하는 것이다.
- 이 방식 채택 시 "구간 공유" UX가 사라지고 "곡 추천" 앱이 된다.
- 채택 여부는 서비스 컨셉 변경을 인지한 후 결정한다.
```

---

## 3. YouTube 상세 제약

### 3.1. Premium 구독 여부

```text
- MusicLog는 사용자의 YouTube Premium 구독 여부를 공식적으로 확인할 수 없다.
- Google 계정 연동 여부와 Premium 구독은 별개의 문제다.
- YouTube Data API의 subscriptions는 채널 구독 정보이며 Premium 결제 정보가 아니다.
```

### 3.2. 광고 노출

```text
- YouTube IFrame/WebView에서 광고가 나올 수 있다.
- 광고 노출 여부는 MusicLog가 보장할 수 없다.
- 광고 여부는 계정 상태, 로그인 상태, 영상 설정, 지역, YouTube 정책, WebView 환경에 따라 결정된다.
- MusicLog는 광고 차단이나 우회를 하지 않는다.
```

### 3.3. UI 제어

```text
- controls=0 등으로 일부 UI를 줄일 수 있다.
- YouTube 로고, 광고, 오버레이, 플레이어 기본 동작을 완전히 제거할 수는 없다.
- YouTube 플레이어를 숨기거나 오디오만 재생하는 방식은 피한다.
```

### 3.4. 자동재생

```text
- autoplay 파라미터는 존재한다.
- 모바일 WebView에서 소리 있는 자동재생은 불안정하다.
- MusicLog는 사용자 터치 기반 재생을 기본으로 설계한다.
```

---

## 4. Phase 0: YouTube Feasibility + Spotify seekTo 검증

### 목적

YouTube IFrame/WebView 재생과 Spotify App Remote seekTo() 양쪽이  
MusicLog의 핵심 UX를 만족할 수 있는지 **본개발 전에** 확인한다.

### YouTube 검증 기능

```text
- YouTube videoId 입력
- startSec / endSec 입력
- WebView 안에서 YouTube IFrame Player 로드
- controls=0 적용 테스트
- 앱 자체 재생 버튼
- 앱 자체 구간 슬라이더
- seekTo(startSec)
- endSec 도달 시 pause
- 다시 듣기 버튼
- 다음 Moment 카드로 넘김
```

### Spotify seekTo 검증 기능

```text
- Spotify App Remote SDK 연결
- seekTo(startSec) 후 실제 재생 시작 위치 측정
- endSec 도달 시 pause 동작 확인
- Premium 계정 전용 동작 확인
- Wi-Fi / LTE 환경별 정밀도 비교
- seekTo 오차 허용 범위 기준 수립 (예: ±1.5초 이내)
```

### 테스트 환경

```text
- Android 실제 기기
- iPhone 실제 기기
- YouTube Premium 계정 / YouTube 무료 계정
- Spotify Premium 계정
- Google 로그인 상태 / 비로그인 상태
- Wi-Fi / LTE
- Official Audio / Topic 영상 / Music Video
```

### 확인해야 할 질문

```text
[YouTube]
1. WebView에서 YouTube IFrame이 안정적으로 재생되는가?
2. 특정 구간 재생이 자연스러운가?
3. endSec 도달 시 pause가 자연스럽게 동작하는가?
4. 광고가 얼마나 자주 체감되는가?
5. YouTube Premium 사용자도 WebView에서 광고 없는 경험을 얻는가?
6. WebView 안에서 Google 로그인 상태가 안정적으로 유지되는가?
7. YouTube 기본 UI/로고/오버레이가 서비스 감성을 크게 해치지 않는가?
8. 자동재생이 가능한가, 혹은 터치 기반 재생만 현실적인가?
9. 스토리형 피드에서 "짧게 듣고 넘기기" 경험이 살아나는가?

[Spotify seekTo]
10. seekTo() 후 실제 재생 시작 위치의 오차가 허용 범위 이내인가?
11. 네트워크 환경에 따라 오차가 얼마나 달라지는가?
12. endSec 도달 시 pause가 안정적으로 동작하는가?
```

### 성공 기준

```text
[YouTube]
- 15~30초 구간 재생이 대부분 자연스럽다.
- 사용자가 한 번 탭하면 재생이 안정적으로 시작된다.
- 광고가 뜨더라도 빈도나 체감이 서비스 핵심 UX를 크게 망치지 않는다.
- YouTube UI가 앱 감성을 심하게 해치지 않는다.
- 다음 Moment로 넘어가는 흐름이 어색하지 않다.
- 최소한 YouTube Premium 사용자에게는 쓸 만한 경험이 나온다.

[Spotify seekTo]
- seekTo() 오차가 ±1.5초 이내로 안정적이다.
- Wi-Fi / LTE 양쪽에서 일관된 정밀도를 보인다.
- endSec 도달 시 pause가 대부분 자연스럽게 동작한다.
```

### 실패 기준

```text
[YouTube]
- 광고가 자주 떠서 15초 감상이 깨진다.
- YouTube Premium 사용자도 WebView에서 광고/로그인 문제가 잦다.
- YouTube UI가 너무 강하게 드러나 앱 정체성이 약해진다.
- 자동/연속 재생이 거의 불가능하다.
- 앱 안 재생보다 그냥 YouTube 링크를 여는 것이 낫게 느껴진다.

[Spotify seekTo]
- seekTo() 오차가 ±3초 이상 빈번하게 발생한다.
- 네트워크 환경에 따라 오차가 너무 커서 예측 불가능하다.
- endSec 도달 시 pause가 자주 실패한다.
```

### 실패 시 의사결정

```text
[YouTube 실패]
- YouTube 본개발 보류
- Spotify 중심 서비스 또는 다른 기획으로 전환

[Spotify seekTo 실패]
- iTunes Preview Fallback을 MVP 메인으로 전환
- 단, 이 경우 "구간 공유" → "곡 추천"으로 핵심 UX가 변경됨을 인지

[양쪽 모두 실패]
- MusicLog 기획 전면 재검토
  1. Spotify 중심 포트폴리오 프로젝트로 축소
  2. 외부 앱 열기 기반 음악 기록 앱으로 전환
  3. 음악 로그/감상 기록/그룹 일기 중심으로 전환
  4. 완전히 다른 프로젝트 아이디어로 전환
```

### 산출물

```text
- YouTube 재생 UX 검증 결과 리포트
- Spotify seekTo 정밀도 측정 데이터
- 광고 체감 빈도 기록
- Google 로그인 상태 유지 가능성 기록
- YouTube UI/오버레이 문제 기록
- 재생 전략 최종 결정 (App Remote / iTunes Preview / 외부 앱 열기)
- YouTube 통합 진행/전환 여부 판단
```

---

## 5. 데이터베이스 설계

### Users

```sql
id               UUID PRIMARY KEY
nickname         TEXT NOT NULL
profile_image_url TEXT
created_at       TIMESTAMP
```

### ConnectedAccounts

```sql
id               UUID PRIMARY KEY
user_id          UUID REFERENCES Users(id)
provider         TEXT          -- 'spotify' | 'google'
provider_user_id TEXT
access_token     TEXT
refresh_token    TEXT
expires_at       TIMESTAMP
created_at       TIMESTAMP
```

### Tracks

```sql
id               UUID PRIMARY KEY
title            TEXT NOT NULL
artist           TEXT NOT NULL
album            TEXT
album_art_url    TEXT
duration_ms      INTEGER
isrc             TEXT          -- 곡 동일성 확인용 표준 코드
created_at       TIMESTAMP
```

### TrackSources

```sql
id               UUID PRIMARY KEY
track_id         UUID REFERENCES Tracks(id)
source_type      TEXT          -- 'spotify' | 'youtube' | 'itunes'
source_id        TEXT          -- spotify_track_id / youtube_video_id 등
source_url       TEXT
duration_sec     INTEGER
confidence_score FLOAT         -- 동일 곡 매핑 신뢰도
created_at       TIMESTAMP
last_verified_at TIMESTAMP
```

### Moments

```sql
id               UUID PRIMARY KEY
user_id          UUID REFERENCES Users(id)
track_id         UUID REFERENCES Tracks(id)
caption          TEXT
visibility       TEXT          -- 'group' | 'private'
group_id         UUID REFERENCES Groups(id)
                 -- ※ MVP는 단일 그룹 공유만 지원
                 -- ※ Phase 3에서 다중 그룹 공유 필요 시
                 --    MomentGroups(moment_id, group_id) N:N 테이블로 마이그레이션
expires_at       TIMESTAMP
is_deleted       BOOLEAN DEFAULT FALSE  -- Soft Delete 플래그
created_at       TIMESTAMP
```

### MomentSources

```sql
id               UUID PRIMARY KEY
moment_id        UUID REFERENCES Moments(id)
source_type      TEXT          -- 'spotify' | 'youtube' | 'itunes'
source_id        TEXT
start_sec        FLOAT
end_sec          FLOAT
source_url       TEXT
created_at       TIMESTAMP
```

### Groups

```sql
id               UUID PRIMARY KEY
name             TEXT NOT NULL
owner_id         UUID REFERENCES Users(id)
description      TEXT
join_policy      TEXT          -- 'invite_only' | 'link' | 'approval'
created_at       TIMESTAMP
```

### GroupMembers

```sql
id               UUID PRIMARY KEY
group_id         UUID REFERENCES Groups(id)
user_id          UUID REFERENCES Users(id)
role             TEXT          -- 'owner' | 'member'
status           TEXT          -- 'active' | 'pending'
joined_at        TIMESTAMP
```

### Reactions

```sql
id               UUID PRIMARY KEY
moment_id        UUID REFERENCES Moments(id)
user_id          UUID REFERENCES Users(id)
reaction_type    TEXT          -- 이모지 코드 또는 타입
created_at       TIMESTAMP
```

### Replies

```sql
id               UUID PRIMARY KEY
moment_id        UUID REFERENCES Moments(id)
user_id          UUID REFERENCES Users(id)
body             TEXT NOT NULL
created_at       TIMESTAMP
```

---

## 6. 기술 리스크

### 6.1. Spotify 리스크 (1순위)

#### App Remote SDK seekTo() 정밀도

```text
[리스크]
- 네트워크 지연 + Spotify 내부 버퍼링으로
  지정 타임스탬프와 실제 재생 시작점이 다를 수 있다.
- 15~30초 구간 경험의 완성도에 직접 영향을 미친다.
- 이 문제는 YouTube 리스크와 동급의 핵심 기술 리스크다.

[대응]
- Phase 0에서 반드시 정밀도 측정 및 오차 허용 기준 수립
- 실패 시 iTunes Preview Fallback으로 전환
  단, 이 경우 "구간 공유" → "곡 추천"으로 컨셉이 변경됨
```

#### Spotify Free 계정 제한

```text
[리스크]
- Spotify Free 계정은 seekTo() 자체가 제한된다.
- MusicLog의 구간 재생 UX는 사실상 Spotify Premium 전용이다.

[대응]
- Free 계정 사용자에게는 외부 앱 열기 fallback만 제공한다.
- 온보딩 화면에서 Premium 계정 권장 안내를 표시한다.
```

#### Web Playback SDK 모바일 사용 불가

```text
[리스크]
- Spotify Web Playback SDK는 브라우저 전용이다.
- Flutter 모바일 앱에서는 동작하지 않는다.

[대응]
- MVP는 App Remote SDK만 사용한다.
- Web Playback SDK는 웹 버전 고려 시에만 재검토한다.
```

### 6.2. YouTube 리스크

```text
[리스크]
- Premium 여부 확인 불가
- 광고 노출 보장 불가
- WebView 로그인 상태 불확실
- YouTube UI 완전 제어 불가
- 자동재생 불안정
- 백그라운드 / 오디오-only 재생 불가

[대응]
- YouTube 본개발 전 Phase 0 Feasibility Prototype에서 검증
- 광고 차단/우회 금지
- YouTube 플레이어 표시 유지
- 사용자 터치 기반 재생 기본 설계
- 실패 시 YouTube 통합 보류 및 기획 방향 재검토
```

### 6.3. iTunes Preview Fallback 한계

```text
[리스크]
- previewUrl은 Apple이 선택한 30초 고정 클립이며 사용자가 구간을 지정할 수 없다.
- 이 방식 채택 시 "구간 공유" UX가 사라진다.

[대응]
- iTunes Preview는 Spotify seekTo 실패 시의 최후 수단으로만 사용한다.
- 채택 결정 전 서비스 컨셉 변경 여부를 팀 전체가 인지해야 한다.
```

---

## 7. 개발 순서 권장

> Flutter 학습 순서 및 단계별 구현 순서는 [MusicLog_Architecture.md](MusicLog_Architecture.md) 섹션 9 참조

```text
Phase 0 (검증)
  ├── Spotify App Remote SDK 연결 및 seekTo() 정밀도 측정
  └── YouTube IFrame WebView 재생 UX 검증
      ↓ 결과에 따라 재생 전략 확정 (섹션 2 참조)

Phase 1 (MVP)
  ├── 자체 로그인 + Spotify OAuth
  ├── 곡 검색 + 구간 선택 + Moment Card 생성
  ├── 그룹 생성 + 초대 링크
  └── 그룹 피드 + 리액션 + 답장

Phase 2 (YouTube 통합 — 검증 성공 시만)
Phase 3 (소셜 강화 — 24시간 Story, 다중 그룹 공유)
Phase 4 (아카이브 — Deep Archive / Music Magazine)
```

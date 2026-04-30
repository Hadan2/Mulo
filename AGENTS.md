# AGENTS.md

이 문서는 AI 코딩 에이전트가 MusicLog 프로젝트를 수정할 때 반드시 참고해야 하는 작업 지침이다.

목표는 빠르게 코드를 많이 만드는 것이 아니라, MusicLog의 제품 의도와 기술 제약을 지키면서 작고 검증 가능한 단위로 구현하는 것이다.

---

## 1. 프로젝트 한 줄 정의

MusicLog는 사용자가 좋아하는 음악의 특정 15~30초 순간을 가까운 친구들과 공유하는 폐쇄형 음악 소셜 앱이다.

MusicLog는 음원을 저장하거나 편집하는 앱이 아니다.  
곡 정보, 플랫폼 source, 시작/종료 타임스탬프, 감상 텍스트, 그룹 정보를 저장하는 앱이다.

---

## 2. 절대 지켜야 할 제품 원칙

- 음원 파일을 저장하지 않는다.
- 음원을 다운로드하지 않는다.
- 음원을 자르거나 재가공하지 않는다.
- YouTube 또는 Spotify 오디오를 추출하지 않는다.
- 광고 차단, 우회, 숨김 처리를 구현하지 않는다.
- YouTube 플레이어를 숨기고 오디오만 재생하는 방식을 구현하지 않는다.
- Spotify Free 사용자가 Premium 기능을 쓸 수 있다고 가정하지 않는다.
- YouTube Premium 여부를 API로 확인할 수 있다고 가정하지 않는다.
- MVP에서 연락처 동기화를 구현하지 않는다.
- MVP에서 공개 SNS 피드나 추천 알고리즘을 구현하지 않는다.

---

## 3. 현재 제품 방향

MVP는 Spotify-first로 구현한다.

우선순위는 다음과 같다.

1. Spotify App Remote SDK 연결
2. Spotify `seekTo()` 정밀도 검증
3. Moment 생성과 그룹 피드
4. 리액션과 짧은 답장
5. YouTube는 Phase 0 검증 성공 후에만 본격 통합

YouTube 기능은 한국 시장 확장을 위한 중요한 후보지만, 아직 확정 기능이 아니다.  
광고, WebView 로그인, 자동재생, UI 제어 문제가 있으므로 반드시 실험 기능으로 다룬다.

---

## 4. AI 작업 원칙

### 4.1. 구현 전에 생각하기

작업이 애매하면 바로 구현하지 말고 먼저 다음을 정리한다.

- 내가 이해한 요구사항
- 필요한 가정
- 모호한 부분
- 가능한 선택지
- 추천하는 선택

단순 오타 수정이나 명확한 한 줄 변경은 바로 처리해도 된다.

### 4.2. 단순하게 만들기

- 요청받지 않은 기능을 추가하지 않는다.
- 한 번만 쓰는 코드에 과한 추상화를 만들지 않는다.
- 미래 확장성을 이유로 현재 필요 없는 구조를 만들지 않는다.
- 200줄로 만든 코드가 50줄로 가능하면 줄인다.
- 기존 구조와 스타일을 우선 따른다.

### 4.3. 필요한 부분만 고치기

- 요청과 직접 관련된 파일만 수정한다.
- 주변 코드를 임의로 리팩터링하지 않는다.
- 기존 주석, 포맷, 네이밍을 이유 없이 바꾸지 않는다.
- 관련 없는 죽은 코드를 발견하면 삭제하지 말고 보고만 한다.
- 내가 만든 변경으로 생긴 unused import, unused variable은 정리한다.

### 4.4. 성공 기준을 정하고 검증하기

작업을 시작할 때 가능하면 성공 기준을 정한다.

예시:

- "Moment 생성 validation 추가"
  - 시작 시간이 종료 시간보다 작아야 한다.
  - 구간 길이는 15~30초여야 한다.
  - 잘못된 값에 대한 테스트가 있어야 한다.

- "Spotify 재생 구현"
  - Premium + App Remote 연결 시 `seekTo()`가 호출된다.
  - 연결 실패 시 외부 앱 열기 fallback이 동작한다.
  - 실패 상태가 UI에 표시된다.

구현 후 가능한 검증을 실행한다.

- 정적 분석
- 단위 테스트
- 위젯 테스트
- 수동 실행 확인
- 관련 문서 업데이트 여부 확인

---

## 5. Flutter 아키텍처 규칙

MusicLog는 Flutter + Riverpod + MVVM 구조를 사용한다.

기본 흐름:

```text
Widget
→ ViewModel / Notifier
→ Repository
→ Service / API Client / SDK
```

규칙:

- Widget은 Repository나 SDK를 직접 호출하지 않는다.
- Widget은 ViewModel의 상태를 구독하고 사용자 이벤트를 전달한다.
- ViewModel은 화면 상태와 사용자 액션을 관리한다.
- Repository는 API 호출과 데이터 변환을 담당한다.
- Service는 Spotify, YouTube, iTunes 같은 외부 SDK/API를 감싼다.
- 두 화면 이상에서 공유하는 상태만 전역 Provider로 둔다.
- 한 화면에서만 쓰는 상태는 해당 ViewModel에 둔다.

---

## 6. 권장 폴더 구조

```text
lib/
├── core/
│   ├── router.dart
│   ├── theme.dart
│   ├── constants.dart
│   └── errors/
├── data/
│   ├── models/
│   ├── repositories/
│   └── services/
├── presentation/
│   ├── auth/
│   ├── feed/
│   ├── moment/
│   ├── group/
│   └── player/
└── shared/
    ├── widgets/
    └── providers/
```

이 구조와 다르게 구현해야 할 이유가 있으면 먼저 이유를 설명한다.

---

## 7. 재생 전략

Spotify 우선순위:

1. Spotify Premium + App Remote 연결 성공
   - App Remote SDK로 `seekTo()` 사용
2. App Remote 연결 실패 또는 Free 사용자
   - Spotify 외부 앱 열기
3. `seekTo()` 정밀도 검증 실패
   - iTunes Preview fallback 검토

iTunes Preview 주의:

- iTunes `previewUrl`은 Apple이 정한 30초 미리듣기다.
- 사용자가 지정한 구간을 재생할 수 없다.
- 이 fallback을 메인으로 쓰면 MusicLog는 "구간 공유 앱"이 아니라 "곡 추천 앱"에 가까워진다.
- 이 변경은 반드시 제품 문서에 반영해야 한다.

YouTube 주의:

- Phase 0 검증 전에는 실험 기능으로만 구현한다.
- WebView 안에 보이는 YouTube IFrame Player를 사용한다.
- 사용자 터치 기반 재생을 기본으로 한다.
- 광고 차단이나 UI 숨김 우회는 구현하지 않는다.

---

## 8. MVP 범위

MVP에 포함한다.

- 회원가입 / 로그인
- Spotify 계정 연동
- Spotify 곡 검색
- 현재 재생곡 가져오기
- 구간 선택
- Moment Card 생성
- 그룹 생성
- 초대 링크 공유
- 그룹 피드
- 리액션
- 짧은 답장
- Spotify 재생 또는 외부 앱 fallback

MVP에서 제외한다.

- YouTube 정식 통합
- Kakao SDK 커스텀 메시지
- 연락처 동기화
- 공개 피드
- 추천 알고리즘
- 24시간 Music Story
- Deep Archive / Music Magazine
- 다중 그룹 공유

---

## 9. 데이터 모델 기준

기본 도메인 모델은 기존 기획서를 따른다.

- Users
- ConnectedAccounts
- Tracks
- TrackSources
- Moments
- MomentSources
- Groups
- GroupMembers
- Reactions
- Replies

중요한 해석:

- Moment는 사용자가 올린 소셜 게시물이다.
- MomentSource는 재생 플랫폼과 타임스탬프 정보를 담는다.
- MVP에서는 Moment 하나가 하나의 그룹에만 공유된다.
- 다중 그룹 공유는 Phase 3 이후 `MomentGroups` 같은 N:N 테이블로 확장한다.

---

## 10. 문서 동기화 규칙

구현 중 제품 방향이나 기술 결정이 바뀌면 관련 문서를 함께 수정한다.

- `Plandoc/MusicLog_Plan_Product.md`
- `Plandoc/MusicLog_Plan_Tech.md`
- `Plandoc/MusicLog_Architecture.md`
- `AGENTS.md`

코드가 문서와 조용히 어긋나게 두지 않는다.

---

## 11. AI가 특히 조심할 것

다음 행동은 하지 않는다.

- "편의를 위해" 오디오 클립 저장 기능 추가
- "UX 개선을 위해" YouTube 플레이어 숨김
- "나중에 필요할 수 있으니" 대형 추상화 추가
- 요청받지 않은 인증 방식 추가
- 요청받지 않은 소셜 기능 추가
- 기존 문서와 반대되는 구현을 설명 없이 진행
- 테스트 없이 핵심 재생 로직 완료 처리

---

## 12. 작업 완료 전 체크리스트

작업을 마치기 전에 확인한다.

- 요청한 문제를 실제로 해결했는가?
- 바꾼 파일이 요청 범위를 벗어나지 않는가?
- 기존 아키텍처 흐름을 지켰는가?
- MusicLog의 제품 원칙을 어기지 않았는가?
- 가능한 테스트나 검증을 실행했는가?
- 실행하지 못한 검증이 있다면 이유를 설명했는가?
- 문서 업데이트가 필요한 변경인지 확인했는가?

---

## 13. 참고 문서

상세 제품/기술/아키텍처 결정은 아래 문서를 우선한다.

- `Plandoc/MusicLog_Plan_Product.md`
- `Plandoc/MusicLog_Plan_Tech.md`
- `Plandoc/MusicLog_Architecture.md`

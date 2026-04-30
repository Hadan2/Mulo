# MusicLog — 아키텍처 설계
## Flutter + Riverpod + MVVM

---

## 1. 기술 스택 확정

### Frontend
```text
Flutter (Dart)
- iOS / Android 크로스플랫폼
- iOS 우선 개발, Android 순차 지원
```

### 상태관리
```text
Riverpod
- Provider의 개선판, 현재 Flutter 생태계 표준
- MVVM 패턴과 자연스럽게 결합
```

### 주요 패키지
```text
flutter_riverpod    상태관리 (ViewModel 역할)
go_router           화면 라우팅
dio                 HTTP 클라이언트
freezed             불변 Model 클래스 자동 생성
json_serializable   JSON 직렬화 자동 생성
spotify_sdk         Spotify App Remote SDK 연동
flutter_webview     YouTube IFrame Prototype 전용
flutter_secure_storage  토큰 안전 저장
```

---

## 2. 아키텍처 패턴: MVVM

### WPF MVVM과의 개념 대응

C# WPF MVVM 경험이 있으면 아래 매핑으로 바로 이해할 수 있다.

| WPF MVVM | Flutter Riverpod |
|----------|-----------------|
| View (XAML) | Widget |
| ViewModel | Notifier (AsyncNotifier) |
| INotifyPropertyChanged | state + ref.watch() |
| Command | 메서드 (void / Future) |
| Binding | ref.watch(provider) |
| Model | Model 클래스 (freezed) |
| Service / Repository | Repository 클래스 |
| DI Container | Riverpod Provider |

### 데이터 흐름

```
Widget (View)
  │
  │  ref.watch(feedViewModelProvider)   ← WPF의 Binding
  ▼
FeedViewModel (AsyncNotifier)
  │
  │  momentRepository.getFeed()         ← WPF의 Command + Service 호출
  ▼
MomentRepository
  │
  │  apiClient.get('/moments')          ← WPF의 DataService / HTTP
  ▼
Backend API / Spotify SDK / YouTube API
```

### 단방향 데이터 흐름 원칙

```text
Widget → ViewModel 메서드 호출 (이벤트)
ViewModel → state 변경
state 변경 → ref.watch()로 Widget 자동 리빌드

Widget이 직접 Repository를 호출하지 않는다.
Widget이 직접 state를 변경하지 않는다.
```

---

## 3. 폴더 구조

```
lib/
├── main.dart
│
├── core/                            # 앱 전역 설정
│   ├── router.dart                  # 화면 라우팅 (go_router)
│   ├── theme.dart                   # 디자인 테마, 컬러, 타이포
│   ├── constants.dart               # API URL 등 상수
│   └── errors/
│       └── app_exception.dart       # 공통 에러 타입 정의
│
├── data/                            # 데이터 계층
│   ├── models/                      # 순수 데이터 클래스 (freezed)
│   │   ├── moment.dart
│   │   ├── moment.freezed.dart      # 자동 생성
│   │   ├── track.dart
│   │   ├── track.freezed.dart
│   │   ├── group.dart
│   │   └── user.dart
│   │
│   ├── repositories/                # API 호출 + 데이터 가공
│   │   ├── moment_repository.dart
│   │   ├── track_repository.dart
│   │   ├── group_repository.dart
│   │   └── auth_repository.dart
│   │
│   └── services/                    # 외부 SDK / API 래핑
│       ├── spotify_service.dart     # Spotify App Remote SDK
│       ├── itunes_service.dart      # iTunes Search API (Fallback)
│       └── youtube_service.dart    # YouTube IFrame (Phase 0)
│
├── presentation/                    # UI 계층
│   ├── auth/
│   │   ├── login_screen.dart
│   │   └── auth_viewmodel.dart
│   │
│   ├── feed/
│   │   ├── feed_screen.dart
│   │   └── feed_viewmodel.dart
│   │
│   ├── moment/
│   │   ├── moment_create_screen.dart
│   │   ├── moment_detail_screen.dart
│   │   └── moment_viewmodel.dart
│   │
│   ├── group/
│   │   ├── group_screen.dart
│   │   ├── group_create_screen.dart
│   │   └── group_viewmodel.dart
│   │
│   └── player/
│       ├── player_screen.dart       # 구간 재생 UI
│       └── player_viewmodel.dart
│
└── shared/                          # 공통 위젯 / 전역 Provider
    ├── widgets/
    │   ├── moment_card.dart
    │   ├── track_thumbnail.dart
    │   └── reaction_bar.dart
    └── providers/
        ├── auth_provider.dart       # 로그인 상태 전역 관리
        └── spotify_provider.dart    # Spotify 연결 상태 전역 관리
```

---

## 4. 계층별 역할 상세

### 4.1. Model (data/models/)

```dart
// freezed로 불변 클래스 생성 — C#의 record와 유사
@freezed
class Moment with _$Moment {
  const factory Moment({
    required String id,
    required String userId,
    required Track track,
    required String caption,
    required double startSec,
    required double endSec,
    required DateTime createdAt,
    DateTime? expiresAt,
  }) = _Moment;

  factory Moment.fromJson(Map<String, dynamic> json) => _$MomentFromJson(json);
}
```

- 순수 데이터 구조만 담는다.
- 비즈니스 로직이 없다.
- JSON 직렬화 포함.

---

### 4.2. Repository (data/repositories/)

```dart
// API 호출과 데이터 변환 담당 — WPF의 DataService
class MomentRepository {
  final Dio _dio;

  MomentRepository(this._dio);

  Future<List<Moment>> getFeed(String groupId) async {
    final response = await _dio.get('/groups/$groupId/moments');
    return (response.data as List)
        .map((e) => Moment.fromJson(e))
        .toList();
  }

  Future<Moment> createMoment(CreateMomentRequest request) async {
    final response = await _dio.post('/moments', data: request.toJson());
    return Moment.fromJson(response.data);
  }
}

// Riverpod Provider로 등록
final momentRepositoryProvider = Provider<MomentRepository>((ref) {
  return MomentRepository(ref.watch(dioProvider));
});
```

- HTTP 통신만 담당한다.
- Widget도 ViewModel도 직접 Dio를 쓰지 않는다.

---

### 4.3. Service (data/services/)

```dart
// 외부 SDK 래핑 — Spotify, iTunes, YouTube
class SpotifyService {
  Future<void> connect() async { ... }

  Future<void> seekTo(double positionMs) async {
    await SpotifySdk.seekToPosition(positionedMilliseconds: positionMs.toInt());
  }

  Future<void> pause() async {
    await SpotifySdk.pause();
  }
}

final spotifyServiceProvider = Provider<SpotifyService>((ref) {
  return SpotifyService();
});
```

- SDK의 복잡한 인터페이스를 단순하게 래핑한다.
- ViewModel은 SDK를 직접 호출하지 않고 Service를 통한다.

---

### 4.4. ViewModel (presentation/*/viewmodel.dart)

```dart
// WPF의 ViewModel과 동일한 역할
// AsyncNotifier = 비동기 상태를 가진 ViewModel
class FeedViewModel extends AsyncNotifier<List<Moment>> {
  @override
  Future<List<Moment>> build() async {
    // 초기 데이터 로드 — WPF의 생성자 초기화와 동일
    return _loadFeed();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _loadFeed());
  }

  Future<List<Moment>> _loadFeed() {
    final groupId = ref.read(currentGroupProvider)!.id;
    return ref.read(momentRepositoryProvider).getFeed(groupId);
  }
}

final feedViewModelProvider =
    AsyncNotifierProvider<FeedViewModel, List<Moment>>(FeedViewModel.new);
```

- Repository / Service만 호출한다.
- UI 관련 코드가 없다.
- state 변경만 담당한다.

---

### 4.5. View / Widget (presentation/*/screen.dart)

```dart
// ConsumerWidget = ref.watch() 사용 가능한 Widget
// WPF의 View(XAML) + Code-behind 역할
class FeedScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref.watch = WPF의 Binding
    final feedState = ref.watch(feedViewModelProvider);

    return feedState.when(
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('에러: $e'),
      data: (moments) => ListView.builder(
        itemCount: moments.length,
        itemBuilder: (_, i) => MomentCard(moment: moments[i]),
      ),
    );
  }
}
```

- ViewModel의 state를 watch해서 화면을 그린다.
- 사용자 액션은 ViewModel 메서드를 호출한다.
- 비즈니스 로직이 없다.

---

## 5. 화면 구조 및 라우팅

```text
/ (splash)
├── /login
├── /home
│   ├── /feed                     그룹 피드 (메인)
│   ├── /moment/create            Moment 생성
│   ├── /moment/:id               Moment 상세 / 재생
│   ├── /group/:id                그룹 페이지
│   └── /group/create             그룹 생성
└── /settings
    └── /settings/spotify         Spotify 연동 관리
```

### go_router 설정 예시

```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (_, __) => const SplashScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(
      path: '/home',
      builder: (_, __) => const HomeScreen(),
      routes: [
        GoRoute(path: 'feed', builder: (_, __) => const FeedScreen()),
        GoRoute(
          path: 'moment/:id',
          builder: (_, state) => MomentDetailScreen(
            momentId: state.pathParameters['id']!,
          ),
        ),
      ],
    ),
  ],
);
```

---

## 6. 상태 관리 범위

### 전역 상태 (shared/providers/)

```text
authProvider          로그인한 사용자 정보, 토큰
spotifyProvider       Spotify 연결 상태, 현재 재생 정보
currentGroupProvider  현재 선택된 그룹
```

### 화면별 로컬 상태 (presentation/**/viewmodel.dart)

```text
feedViewModelProvider         피드 목록, 로딩/에러 상태
momentViewModelProvider       Moment 생성 폼 상태
playerViewModelProvider       재생 위치, 재생/정지 상태
groupViewModelProvider        그룹 멤버 목록, 초대 링크
```

### 규칙

```text
- 두 화면 이상에서 공유하는 상태 → 전역 Provider
- 한 화면에서만 쓰는 상태 → 해당 화면 ViewModel
- Widget 내부의 단순 UI 상태 (애니메이션 등) → StatefulWidget
```

---

## 7. 에러 처리 전략

```dart
// 공통 에러 타입
sealed class AppException {
  const AppException();
}

class NetworkException extends AppException {
  final int? statusCode;
  const NetworkException({this.statusCode});
}

class SpotifyException extends AppException {
  final String message;
  const SpotifyException(this.message);
}

class UnauthorizedException extends AppException {}
```

```text
- Repository에서 예외를 AppException으로 변환
- ViewModel에서 AsyncValue.error로 state에 반영
- View에서 feedState.when(error: ...) 로 처리
- 401 Unauthorized → authProvider에서 토큰 갱신 후 재시도
```

---

## 8. Spotify 연동 구조

```text
SpotifyService
  ├── connect()           App Remote SDK 연결
  ├── seekTo(ms)          특정 위치로 이동
  ├── pause()             일시정지
  ├── resume()            재개
  └── getCurrentTrack()   현재 재생 중인 트랙 정보

PlayerViewModel
  ├── state: PlayerState  (playing / paused / loading / error)
  ├── startPlayback()     seekTo(startSec) 호출
  ├── stopAtEnd()         endSec 도달 감지 후 pause()
  └── retry()             재생 실패 시 재시도
```

> 재생 Fallback 순서 및 seekTo() 정밀도 리스크는 [MusicLog_Plan_Tech.md](MusicLog_Plan_Tech.md) 섹션 2, 6 참조

---

## 9. 개발 시작 순서

처음 Flutter를 시작할 때 구조를 한꺼번에 다 잡으려 하지 않는다.  
아래 순서대로 점진적으로 구조를 확장한다.

```text
Step 1. Flutter 기본 익히기 (3~4일)
  - Widget, StatefulWidget, StatelessWidget
  - Column / Row / ListView / Stack
  - Navigator로 화면 전환

Step 2. Riverpod 기본 익히기 (1~2일)
  - Provider, StateNotifier
  - ref.watch / ref.read 차이
  - 로그인 상태 하나를 Riverpod으로 관리해보기

Step 3. 로그인 화면부터 MVVM 구조 적용
  - auth/ 폴더 구조 잡기
  - AuthViewModel 만들기
  - LoginScreen에서 ref.watch 연결

Step 4. 이후 화면마다 같은 패턴 반복
  - 새 화면 = Screen + ViewModel + Repository 세트

Step 5. freezed / json_serializable 도입
  - Model 클래스 자동 생성으로 보일러플레이트 제거

Step 6. go_router로 라우팅 정리
  - Navigator 직접 호출 → go_router로 교체
```

> Phase별 개발 순서는 [MusicLog_Plan_Techㄴ.md](MusicLog_Plan_Tech.md) 섹션 7 참조

---

## 10. 참고 자료

```text
공식 문서
- Flutter: https://docs.flutter.dev
- Riverpod: https://riverpod.dev
- go_router: https://pub.dev/packages/go_router
- freezed: https://pub.dev/packages/freezed

Spotify SDK
- spotify_sdk (pub.dev): Spotify App Remote Flutter 래퍼
- Spotify App Remote SDK 공식 문서

패키지 검색
- pub.dev: Flutter/Dart 패키지 공식 저장소
```

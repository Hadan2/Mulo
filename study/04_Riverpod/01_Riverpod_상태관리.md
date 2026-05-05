# Riverpod 상태관리

Riverpod은 Mulo의 ViewModel 역할을 한다.
WPF의 INotifyPropertyChanged + DI Container를 합친 것이라고 보면 된다.

---

## 왜 Riverpod인가?

Flutter에서 상태관리 없이 코딩하면 이런 문제가 생긴다:

```
피드 화면 → 새 Moment 생성 → 피드 화면이 자동으로 갱신되어야 함
```

Widget끼리 직접 데이터를 주고받으면 코드가 복잡해진다.
Riverpod은 **전역 상태 저장소**를 제공해서 어느 Widget에서든 같은 데이터를 읽고 쓸 수 있게 한다.

---

## Provider — 데이터 제공자

Provider는 데이터나 객체를 만들고 앱 전체에 제공하는 "공장"이다.

```dart
// Repository를 앱 전체에 제공
final momentRepositoryProvider = Provider<MomentRepository>((ref) {
  return MomentRepository(ref.watch(dioProvider));
});
```

WPF의 DI Container에 서비스를 등록하는 것과 같다.

---

## ref.watch vs ref.read

| | 용도 |
|--|------|
| `ref.watch` | 값이 바뀌면 Widget을 다시 그림 (WPF Binding) |
| `ref.read` | 값을 한 번만 읽음, 변경 감지 없음 (이벤트 핸들러에서 사용) |

```dart
// Widget 안에서 — 값 바뀌면 자동 리빌드
final moments = ref.watch(feedViewModelProvider);

// 버튼 클릭 같은 이벤트 안에서 — 한 번만 읽기
onPressed: () => ref.read(feedViewModelProvider.notifier).refresh(),
```

---

## AsyncNotifier — 비동기 ViewModel

API 호출처럼 비동기 작업이 있는 ViewModel에 사용한다.

```dart
class FeedViewModel extends AsyncNotifier<List<Moment>> {
  @override
  Future<List<Moment>> build() async {
    // 화면 처음 열릴 때 자동 실행 — WPF 생성자와 같음
    return _loadFeed();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();  // 로딩 상태로 변경
    state = await AsyncValue.guard(() => _loadFeed());
  }

  Future<List<Moment>> _loadFeed() {
    final groupId = ref.read(currentGroupProvider)!.id;
    return ref.read(momentRepositoryProvider).getFeed(groupId);
  }
}

// Provider 등록
final feedViewModelProvider =
    AsyncNotifierProvider<FeedViewModel, List<Moment>>(FeedViewModel.new);
```

---

## AsyncValue — 로딩/에러/데이터 3가지 상태

API 호출 결과는 항상 3가지 상태가 있다.

```dart
// Widget에서 3가지 상태 처리
final feedState = ref.watch(feedViewModelProvider);

feedState.when(
  loading: () => CircularProgressIndicator(),   // 로딩 중
  error: (e, _) => Text('에러: $e'),             // 에러
  data: (moments) => ListView.builder(...),      // 데이터 있음
)
```

WPF에서 IsLoading, HasError, Data를 각각 바인딩하던 것을 when() 하나로 처리한다.

---

## ConsumerWidget — ref를 쓸 수 있는 Widget

Riverpod을 사용하려면 Widget이 ConsumerWidget이어야 한다.

```dart
// 일반 StatelessWidget 대신 ConsumerWidget 사용
class FeedScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    //                                         ^^^
    //                               ref가 추가됨 — 여기서 watch/read 가능

    final feedState = ref.watch(feedViewModelProvider);
    return feedState.when(...);
  }
}
```

---

## ProviderScope — 앱 루트에 감싸기

Riverpod을 쓰려면 앱 전체를 ProviderScope로 감싸야 한다.
한 번만 설정하면 된다.

```dart
void main() {
  runApp(
    ProviderScope(        // 이거 하나로 Riverpod 활성화
      child: MyApp(),
    ),
  );
}
```

---

## 전체 흐름 요약

```
1. ProviderScope로 앱 감쌈 (main.dart에서 한 번)

2. Provider로 Repository/Service 등록

3. AsyncNotifier로 ViewModel 작성
   - build() : 초기 데이터 로드
   - 메서드  : 사용자 액션 처리

4. ConsumerWidget에서 ref.watch로 상태 구독
   - AsyncValue.when()으로 로딩/에러/데이터 처리

5. 버튼 클릭 → ref.read(provider.notifier).메서드() 호출
```

---

## WPF MVVM vs Riverpod 대응표

| WPF | Riverpod |
|-----|----------|
| INotifyPropertyChanged | state 변경 |
| Binding | ref.watch() |
| Command | ViewModel 메서드 |
| DI Container 등록 | Provider 정의 |
| ViewModel | AsyncNotifier |
| View | ConsumerWidget |
| IsLoading / HasError / Data | AsyncValue.when() |

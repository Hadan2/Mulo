# 화면 라우팅 — go_router

화면 간 이동을 관리하는 패키지.
WPF의 NavigationService나 Frame 네비게이션과 같은 역할.

---

## 기본 개념

URL 경로 방식으로 화면을 정의한다.

```
/           → SplashScreen
/login      → LoginScreen
/home/feed  → FeedScreen
/home/moment/abc123 → MomentDetailScreen
```

---

## 라우터 설정 (core/router.dart)

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const SplashScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/home',
      builder: (context, state) => const HomeScreen(),
      routes: [
        // 중첩 라우트
        GoRoute(
          path: 'feed',
          builder: (context, state) => const FeedScreen(),
        ),
        GoRoute(
          path: 'moment/:id',    // :id = 동적 파라미터
          builder: (context, state) => MomentDetailScreen(
            momentId: state.pathParameters['id']!,
          ),
        ),
      ],
    ),
  ],
);
```

---

## main.dart에 적용

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router,  // GoRouter 연결
    );
  }
}
```

---

## 화면 이동

```dart
// 이동
context.go('/home/feed');

// 뒤로가기 (스택에 쌓음)
context.push('/home/moment/abc123');

// 뒤로가기
context.pop();

// 파라미터 전달
context.go('/home/moment/${moment.id}');
```

---

## WPF NavigationService vs go_router

| WPF | go_router |
|-----|-----------|
| NavigationService.Navigate(typeof(FeedPage)) | context.go('/home/feed') |
| NavigationService.GoBack() | context.pop() |
| NavigationService.Navigate(typeof(DetailPage), id) | context.go('/detail/$id') |
| Frame 중첩 | 중첩 라우트 (routes: [...]) |

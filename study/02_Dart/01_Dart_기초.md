# Dart 기초

Flutter는 Dart 언어로 작성한다. C#과 비슷한 부분이 많다.

---

## 변수

```dart
// 타입 명시
String name = '홍길동';
int age = 25;
double height = 175.5;
bool isLoggedIn = true;

// 타입 추론 (C#의 var와 동일)
var title = '안녕하세요';

// 절대 변경 안 되는 상수 (C#의 const와 동일)
const String appName = 'Mulo';

// 런타임 상수 (C#의 readonly와 비슷)
final String userId = fetchUserId();
```

---

## null 처리

```dart
// null 가능 타입은 ? 붙임 (C#과 동일)
String? nickname;       // null 가능
String username = ''; // null 불가

// null 체크
if (nickname != null) {
  print(nickname);
}

// null이면 기본값 (C#의 ?? 연산자와 동일)
String display = nickname ?? '익명';

// null이면 호출 안 함 (C#의 ?. 연산자와 동일)
int? length = nickname?.length;
```

---

## 함수

```dart
// 기본 함수
String greet(String name) {
  return 'Hello, $name';
}

// 한 줄 함수 (=> 화살표)
String greet(String name) => 'Hello, $name';

// named parameter (C# 명명 인수와 비슷, 하지만 더 자주 씀)
void createMoment({required String trackId, String? caption}) {
  // ...
}

// 호출 시
createMoment(trackId: 'abc123', caption: '이 부분이 좋아');
```

---

## 비동기 (async / await)

C#과 문법이 거의 동일하다.

```dart
// Future = C#의 Task
Future<String> fetchUserName() async {
  final response = await dio.get('/user');  // 기다림
  return response.data['name'];
}

// 호출
final name = await fetchUserName();
```

---

## 클래스

```dart
class User {
  final String id;
  final String name;

  // 생성자
  User({required this.id, required this.name});

  // 메서드
  String get displayName => '@$name';
}

// 사용
final user = User(id: '1', name: '홍길동');
print(user.displayName); // @홍길동
```

---

## List / Map

```dart
// List (C#의 List<T>)
List<String> tags = ['팝', '재즈', '힙합'];
tags.add('클래식');
tags.map((t) => t.toUpperCase()).toList();

// Map (C#의 Dictionary)
Map<String, dynamic> data = {
  'title': 'Blinding Lights',
  'artist': 'The Weeknd',
};
print(data['title']);
```

---

## String interpolation

```dart
String name = 'Mulo';
print('앱 이름은 $name 입니다');
print('길이는 ${name.length}자 입니다');  // 표현식은 ${}
```

---

## C# vs Dart 대응표

| C# | Dart |
|----|------|
| `string` | `String` |
| `int`, `double` | `int`, `double` |
| `bool` | `bool` |
| `List<T>` | `List<T>` |
| `Dictionary<K,V>` | `Map<K,V>` |
| `Task<T>` | `Future<T>` |
| `async/await` | `async/await` |
| `var` | `var` |
| `const` | `const` |
| `readonly` | `final` |
| `string?` | `String?` |
| `??` | `??` |
| `?.` | `?.` |
| `=>` (람다) | `=>` (화살표 함수) |

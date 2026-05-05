# Flutter Widget 기초

Flutter에서 화면의 모든 것은 Widget이다.
버튼, 텍스트, 레이아웃, 여백까지 전부 Widget.

---

## Widget이란?

```
WPF     → UserControl, Grid, TextBlock, Button
Flutter → 전부 Widget
```

Widget을 트리(나무) 구조로 쌓아서 화면을 만든다.

```dart
// 이게 화면 하나
Scaffold(
  appBar: AppBar(title: Text('Mulo')),
  body: Center(
    child: Text('안녕하세요'),
  ),
)
```

---

## StatelessWidget vs StatefulWidget

### StatelessWidget — 상태 없는 위젯

화면에 표시만 하고, 데이터가 바뀌어도 스스로 다시 그리지 않는다.

```dart
class MyLabel extends StatelessWidget {
  final String text;

  const MyLabel({required this.text});

  @override
  Widget build(BuildContext context) {
    return Text(text);
  }
}
```

### StatefulWidget — 상태 있는 위젯

내부 데이터가 바뀌면 스스로 다시 그린다.
애니메이션, 체크박스, 탭 같은 단순 UI 상태에만 쓴다.

```dart
class Counter extends StatefulWidget {
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () => setState(() => count++),  // setState = 다시 그려
      child: Text('$count'),
    );
  }
}
```

> Mulo에서는 비즈니스 데이터는 Riverpod ViewModel이 관리하고,
> StatefulWidget은 단순 UI 상태(애니메이션 등)에만 쓴다.

---

## 자주 쓰는 레이아웃 Widget

### Column — 세로 배치

```dart
Column(
  children: [
    Text('첫 번째'),
    Text('두 번째'),
    Text('세 번째'),
  ],
)
```

### Row — 가로 배치

```dart
Row(
  children: [
    Icon(Icons.music_note),
    SizedBox(width: 8),   // 간격
    Text('Blinding Lights'),
  ],
)
```

### Stack — 겹쳐 쌓기

```dart
Stack(
  children: [
    Image.network('앨범아트 URL'),
    Positioned(
      bottom: 8,
      left: 8,
      child: Text('재생 중'),
    ),
  ],
)
```

### Container — 크기/패딩/색상 지정

```dart
Container(
  width: 200,
  height: 100,
  padding: EdgeInsets.all(16),
  color: Colors.black,
  child: Text('Moment Card'),
)
```

### SizedBox — 크기 고정 / 간격

```dart
SizedBox(height: 16)  // 세로 간격
SizedBox(width: 8)    // 가로 간격
SizedBox(height: 200, child: SomeWidget())  // 크기 제한
```

---

## 자주 쓰는 UI Widget

```dart
Text('텍스트')

TextButton(onPressed: () {}, child: Text('버튼'))

ElevatedButton(onPressed: () {}, child: Text('강조 버튼'))

Icon(Icons.play_arrow)

Image.network('https://...')

CircularProgressIndicator()  // 로딩 스피너

Scaffold(
  appBar: AppBar(title: Text('타이틀')),
  body: ...,
  bottomNavigationBar: ...,
)
```

---

## ListView — 목록

피드 화면처럼 항목이 많을 때 사용.

```dart
// 항목 수가 정해진 경우
ListView(
  children: [
    MomentCard(),
    MomentCard(),
  ],
)

// 항목 수가 동적인 경우 (성능 좋음, 이걸 주로 씀)
ListView.builder(
  itemCount: moments.length,
  itemBuilder: (context, index) => MomentCard(moment: moments[index]),
)
```

---

## build() 메서드

Widget의 `build()`는 WPF의 XAML 렌더링과 같다.
상태가 바뀌면 Flutter가 자동으로 `build()`를 다시 호출해서 화면을 다시 그린다.

```dart
@override
Widget build(BuildContext context) {
  // 여기서 화면 구조를 반환
  return Scaffold(...);
}
```

---

## WPF vs Flutter 대응표

| WPF | Flutter |
|-----|---------|
| Window | Scaffold |
| Grid / StackPanel (세로) | Column |
| StackPanel (가로) | Row |
| Canvas | Stack |
| Border | Container |
| TextBlock | Text |
| Button | ElevatedButton / TextButton |
| ListBox / ItemsControl | ListView |
| Image | Image.network / Image.asset |
| UserControl | StatelessWidget |
| UserControl + 상태 | StatefulWidget |

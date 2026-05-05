# Flutter 설치 및 환경 설정

---

## 설치 완료 항목

| 항목 | 버전 | 용도 |
|------|------|------|
| Flutter | 3.41.9 | 앱 개발 프레임워크 |
| Dart | 3.11.5 | Flutter가 사용하는 언어 |
| Android Studio | Meerkat | Android SDK + 에뮬레이터 관리 |
| Android SDK | 36.1.0 | Android 앱 빌드 도구 |
| VS Code | - | 코드 에디터 (메인 개발 도구) |

---

## flutter doctor — 환경 상태 확인

```powershell
flutter doctor
```

모든 항목이 ✓ 이면 정상.

```
[√] Flutter
[√] Android toolchain
[√] Chrome
[√] Visual Studio
[√] Connected device
[√] Network resources
```

---

## 프로젝트 위치

```
C:\Projects\Mulo\
├── mulo\          ← Flutter 프로젝트 루트 (여기서 flutter 명령 실행)
├── study\         ← 학습 문서
└── Plandoc\       ← 기획/아키텍처 문서
```

**모든 flutter 명령은 `C:\Projects\Mulo\mulo\` 에서 실행해야 한다.**

```powershell
cd C:\Projects\Mulo\mulo
flutter run
```

---

## VS Code 필수 확장

| 확장 이름 | 용도 |
|-----------|------|
| Flutter (공식) | 자동완성, 디버깅, 핫 리로드 |
| Dart (공식) | Dart 언어 지원 |
| Riverpod Snippets | Riverpod 코드 스니펫 |

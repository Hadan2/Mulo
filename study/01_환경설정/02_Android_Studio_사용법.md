# Android Studio 사용법

Mulo 개발에서 Android Studio는 **에뮬레이터 관리 전용**으로만 사용한다.
코드 작성은 VS Code에서 한다.

---

## Mulo에서 Android Studio의 역할

| 작업 | 도구 |
|------|------|
| 코드 작성 | VS Code |
| 앱 실행 / 빌드 | VS Code 터미널 (`flutter run`) |
| 에뮬레이터 생성 / 관리 | Android Studio Device Manager |

---

## Device Manager 여는 방법

**상단 메뉴 → View → Tool Windows → Device Manager**

---

## 에뮬레이터(AVD) 생성 순서

1. Device Manager 패널 → **+** 버튼 → **Create Virtual Device**
2. 기기 선택 (Pixel 8 추천) → **Next**
3. System Image 선택
   - **API 35** 권장 (안정적)
   - **Google Play / Intel x86_64** 선택
   - 처음이면 옆에 Download 버튼 클릭 후 설치
4. **Next → Finish**

### System Image란?
에뮬레이터 안에 설치될 Android OS.
실제 폰에 Android가 설치되어 있듯이 가상 기기에도 OS가 필요하다.

| 종류 | 설명 |
|------|------|
| Google Play | Play 스토어 + Google 서비스 포함 ← 이걸 선택 |
| Google APIs | Google 서비스만 포함 |
| AOSP | 순정 Android, Google 서비스 없음 |

---

## 에뮬레이터 크기 조절

### 키보드 단축키 (에뮬레이터 창 선택 후)
- `Ctrl + Down` — 작게
- `Ctrl + Up` — 크게

### Tool Window로 도킹
**File → Settings → Tools → Emulator → "Launch in a tool window" 체크**
Android Studio 안에 도킹된 창으로 실행되어 크기가 자동으로 줄어든다.

---

## 에뮬레이터 실행 방법 3가지

### 방법 1 — VS Code 하단 상태바 (가장 편함)
VS Code 우측 하단 기기 이름 클릭 → Pixel 8 선택
→ 에뮬레이터 자동 실행 + 앱 자동 빌드

### 방법 2 — flutter run 실행 시 자동 질문
```powershell
flutter run
# No devices found. Would you like to launch an emulator? (Y/n)
# y 입력
```

### 방법 3 — 터미널에서 직접 실행
```powershell
# 에뮬레이터 목록 확인
flutter emulators

# 에뮬레이터 실행
flutter emulators --launch Pixel_8
```

---

## 결론

Android Studio는 **처음 AVD 만들 때 한 번만** 쓴다.
이후 개발은 전부 VS Code + 터미널에서 진행한다.

# Stainless

실시간 얼룩 탐지 Android 애플리케이션. YOLOv5 + TensorFlow Lite 기반으로 카메라 프리뷰에서 얼룩을 실시간 감지하고, 등록된 연락처로 사진을 공유할 수 있습니다.

---

## 주요 기능

- **실시간 얼룩 탐지**: Camera2 API + YOLOv5로 카메라 프리뷰에서 얼룩 실시간 감지
- **바운딩박스 시각화**: 탐지된 얼룩의 위치와 신뢰도 점수 오버레이 표시
- **TTS 음성 안내**: 탐지 결과를 한국어 음성으로 알림
- **진동 안내**: 탐지 시 진동 피드백 (설정에서 토글)
- **사진 공유**: 탐지된 화면을 등록된 연락처로 MMS 전송
- **다중 추론 가속**: CPU / GPU / NNAPI 선택 가능

---

## 기술 스택

| 분류 | 내용 |
|------|------|
| 언어 | Java |
| 플랫폼 | Android (minSdk 21, targetSdk 33) |
| ML 프레임워크 | TensorFlow Lite 2.4.0 |
| 모델 | YOLOv5 (FP16, 640×640 입력) |
| 카메라 | Camera2 API |
| 빌드 | Gradle 8.1.3 |

---

## 프로젝트 구조

```
app/src/main/
├── java/stainless/tensorflow/lite/examples/detection/
│   ├── DetectorActivity.java       # 메인 카메라 탐지 화면
│   ├── CameraActivity.java         # Camera2 기본 추상 클래스
│   ├── SettingActivity.java        # 설정 (진동, 연락처 관리)
│   ├── QnaActivity.java            # 문의 화면
│   ├── MainActivity.java           # 이미지 탐지 테스트 화면
│   ├── ContactModel.java           # 연락처 데이터 모델
│   ├── ContactAdapter.java         # 연락처 RecyclerView 어댑터
│   ├── customview/
│   │   ├── OverlayView.java        # 탐지 결과 캔버스 오버레이
│   │   ├── AutoFitTextureView.java # 카메라 프리뷰 TextureView
│   │   ├── RecognitionScoreView.java
│   │   └── ResultsView.java
│   ├── env/
│   │   ├── Utils.java              # 이미지 변환, 모델 로드 유틸
│   │   ├── ImageUtils.java         # YUV→RGB 변환, 비트맵 저장
│   │   ├── Logger.java             # 로깅 유틸
│   │   ├── BorderedText.java       # 경계선 텍스트 렌더링
│   │   └── Size.java
│   └── tracking/
│       └── MultiBoxTracker.java    # 객체 추적 및 바운딩박스 렌더링
│   ├── Classifier.java             # 분류기 인터페이스
│   ├── YoloV5Classifier.java       # YOLOv5 추론 엔진
│   ├── YoloV5ClassifierDetect.java
│   └── DetectorFactory.java        # 모델 생성 팩토리
├── res/
│   ├── layout/                     # XML 레이아웃 (10개)
│   ├── drawable/                   # 아이콘 리소스
│   └── values/                     # 색상, 문자열 정의
└── assets/
    ├── best-fp16.tflite            # 메인 YOLOv5 모델 (39.9MB)
    ├── customclasses2.txt          # 탐지 클래스 목록
    ├── test.jpg                    # 테스트 이미지
    └── test2.jpg
```

---

## 빌드 및 실행

### 요구 사항

- Android Studio Hedgehog 이상
- JDK 1.8
- Android 기기 또는 에뮬레이터 (API 21+, OpenGL ES 3.1 필수)

### 빌드

```bash
git clone https://github.com/your-repo/Stainless.git
cd Stainless
./gradlew assembleDebug
```

또는 Android Studio에서 프로젝트를 열고 **Run > Run 'app'** 실행.

### 권한

앱 실행 시 다음 권한이 필요합니다.

- `CAMERA` — 카메라 프리뷰 및 탐지
- `VIBRATE` — 진동 안내
- `READ_CONTACTS` — 연락처 등록
- `READ_EXTERNAL_STORAGE` — 이미지 저장/공유
- `INTERNET` — (선택) 네트워크 연결

---

## 모델 정보

| 항목 | 내용 |
|------|------|
| 모델 파일 | `best-fp16.tflite` |
| 입력 크기 | 640 × 640 |
| 정밀도 | FP16 |
| 파일 크기 | 39.9 MB |
| 신뢰도 임계값 | 0.3 (30%) |

### 지원 추론 가속

- **CPU**: 기본, 멀티스레드 지원
- **GPU**: GpuDelegate (지속 속도 모드)
- **NNAPI**: Android P 이상 기기에서 사용 가능

`DetectorActivity`의 하단 시트에서 런타임 중 가속 방식 및 스레드 수를 변경할 수 있습니다.

### 커스텀 모델 교체

`DetectorFactory.java`에서 모델 파일명과 입력 크기를 수정하고, `assets/` 에 새 `.tflite` 파일을 추가하면 됩니다.

```java
// DetectorFactory.java
detector = YoloV5Classifier.create(
    assetManager,
    "your-model.tflite",   // 모델 파일명
    "your-classes.txt",    // 클래스 파일명
    isQuantized,
    inputSize              // 640 등
);
```

---

## 탐지 흐름

```
카메라 프레임 캡처 (Camera2)
        ↓
YUV → RGB 변환 (ImageUtils)
        ↓
이미지 640×640 리사이즈 (Utils)
        ↓
YOLOv5 모델 추론 (YoloV5Classifier)
        ↓
신뢰도 필터링 (> 0.3)
        ↓
객체 추적 (MultiBoxTracker)
        ↓
오버레이 렌더링 (OverlayView)
        ↓
TTS 음성 안내 / 진동 피드백
```

---

## 개인정보처리방침

### 1. 개인정보의 처리 목적

본 개발자가 작성한 앱은 다음의 목적을 위하여 개인정보를 처리하며, 다음의 목적 이외의 용도로는 이용하지 않습니다.

- 앱 사용자의 사용정보를 수집 및 보유하지 않습니다.

### 2. 개인정보처리 위탁 여부

- 본 개발자의 앱은 타 업체에 개인정보처리를 위탁하지 않습니다.

### 3. 정보주체의 권리, 의무 및 그 행사방법

- 이용자는 개인정보주체로서 언제든지 개인정보 보호 관련 권리를 행사할 수 있습니다.
- 다만, 본 앱은 앱 사용자의 사용정보를 수집 및 보유하지 않습니다.

### 4. 처리하는 개인정보의 항목

- 앱 사용자의 사용정보를 수집 및 보유하지 않습니다.

### 5. 개인정보의 파기

- 앱 사용자의 사용정보를 수집 및 보유하지 않으므로, 파기해야 할 개인정보가 없습니다.

### 6. 개인정보의 안전성 확보 조치

- 앱 사용자의 사용정보를 수집 및 보유하지 않습니다.

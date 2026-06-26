<div align="center">

# HEAR:O

### 집 안의 소리를 듣고, 가전을 돌보다

**생활가전 소음에 민감한 사용자**를 위한 소음 반응형 스마트홈 사운드케어 솔루션입니다.
스마트폰이 가전 소음을 **온디바이스 AI로 분류**하고, 그 결과를 알림·리포트·루틴 추천으로 연결합니다.

<br/>

`On-Device AI` · `YAMNet / TFLite` · `Spring Boot` · `FastAPI` · `Flutter` · `Tauri` · `ESP32-S3`

</div>

---

## 🎧 왜 HEAR:O 인가요?

세탁기, 식기세척기, 로봇청소기 같은 생활가전은 끊임없이 소리를 냅니다.
소음에 민감한 사용자에게 이 소리는 **언제 시작되고 끝나는지, 지금 얼마나 시끄러운지**가 중요한 정보입니다.

HEAR:O는 이 문제를 **"듣고 → 이해하고 → 알려주는"** 세 단계로 풉니다.

| 단계 | 동작 | 구현 |
| :--: | --- | --- |
| 🎙️ **듣는다** | 스마트폰 마이크 / ESP32-S3 센서가 가전 주변 소리를 수집 | Flutter(Kotlin Native) · ESP32-S3 INMP441 |
| 🧠 **이해한다** | YAMNet 기반 모델이 소음의 **종류**를 분류하고 dB로 **크기**를 측정 | TFLite 온디바이스 추론 (16 kHz mono) |
| 🔔 **알려준다** | 소음 이벤트를 알림·대시보드·자연어 리포트·루틴 추천으로 전달 | Spring Boot · FastAPI(GPT) · Tauri |

> **개인정보 우선 설계** — 원본 오디오는 서버로 전송하지 않습니다. 스마트폰에서 로컬로 분류한 뒤,
> 라벨·confidence·dB 통계 같은 **요약 이벤트만** 업로드합니다.

---

## 🧩 분류하는 소음 클래스

스마트폰 마이크에서 받은 **16 kHz mono waveform**을 입력으로, YAMNet 기반 분류기가 아래 라벨 중 하나를 예측합니다.

| 모델 라벨 | 서비스 라벨 | 설명 |
| --- | --- | --- |
| `vacuum_cleaner` | `robot_vacuum` | 청소기 계열 소음 → 로봇청소기 경로로 매핑 |
| `washing_machine` | `washing_machine` | 세탁기 소음 |
| `dishwasher` | `dishwasher` | 식기세척기 소음 |
| `background` | `background` | 민감 가전으로 판단하지 않는 배경 소음 |

분류는 **Flutter 앱에서 로컬 TFLite로** 실행되며, 서버는 추론을 수행하지 않습니다.

---

## 🧠 AI — 소음 분류 모델

HEAR:O의 핵심은 **YAMNet 전이학습(Transfer Learning)** 기반 소음 분류기입니다.
Google의 YAMNet에서 추출한 1024차원 임베딩 위에 가벼운 분류 헤드를 학습시켜,
**스마트폰에서 실시간으로 동작할 만큼 가벼우면서도** 생활가전 소음을 구분하도록 만들었습니다.

### 학습 파이프라인

```text
오디오 데이터셋(16 kHz mono WAV)
  → YAMNet 임베딩 추출
  → 분류 헤드 학습 (washing / dishwasher / vacuum / background)
  → 평가 (Accuracy · Macro F1 · Confusion Matrix · 모바일 지연)
  → TFLite export
  → Flutter 앱(assets/models/soundcare_yamnet.tflite)에 탑재
```

| 단계 | 스크립트 / 산출물 | 설명 |
| --- | --- | --- |
| **학습** | `yamnet/training/train_yamnet_classifier.py` | `data_config.yaml`의 manifest로 분류기 학습 → `saved_model` |
| **평가** | `yamnet/training/evaluate_model.py` | `metrics.json` · `classification_report.txt` · `confusion_matrix.csv` 생성 |
| **export** | `yamnet/export/export_to_tflite.py` | `saved_model` → `soundcare_yamnet.tflite` 변환 |
| **탑재** | `soundcare-flutter-iot-app/assets/models/` | Kotlin Native(NNAPI → CPU fallback)로 온디바이스 추론 |

학습 manifest는 `filepath, label, split` 컬럼을 가지며, 모든 오디오는 **16 kHz · mono · WAV**로 준비합니다.

```csv
filepath,label,split
washing_machine/room_a_001.wav,washing_machine,train
dishwasher/room_b_001.wav,dishwasher,val
background/night_001.wav,background,test
```

### 데이터 수집 & 증강(Augmentation)

- **공개 데이터셋** — ReaLISED · ESC-50 · FSD50K 등에서 가전 소음을 수집하고, Freesound API로 추가 확보
- **하드 네거티브** — 물소리·스피커·휴대폰·TV·말소리 등 혼동되기 쉬운 소리를 `background`로 학습
- **증강** — 게인 조절(예: -15 dB), 배경음 믹싱으로 **SNR 0~20 dB** 구간 모사 → 실제 소음 환경 재현
- **5초 입력 정합** — Flutter 앱의 5초(80,000 샘플) 입력 길이에 맞춰 학습/추론 입력을 일치시켜 정확도 손실 방지

### 평가 지표

| 지표 | 목적 |
| --- | --- |
| Accuracy / Macro F1 | 전체 성능과 클래스 불균형 영향을 함께 확인 |
| Class별 Precision/Recall/F1 | 특정 가전 소음의 과탐지·미탐지 분석 |
| Confusion Matrix | `vacuum_cleaner` ↔ `background` 등 혼동 구간 파악 |
| 환경별 성능 | 방 크기·거리·배경음·시간대에 따른 성능 차이 |
| 모바일 지연 시간 | Kotlin/TFLite/NNAPI 실제 추론 시간 측정 |

### TFLite & 온디바이스

- float32 export를 기본으로 하며, 크기가 문제일 때만 **dynamic-range 양자화**를 선택적으로 적용
- 앱은 NNAPI delegate를 우선 사용하고, 실패 시 **CPU fallback**으로 안정적으로 동작
- 실제 `.tflite` 가중치, 대용량 오디오 데이터셋, WAV/MP3 원본은 **저장소에 커밋하지 않습니다**(용량·개인정보 정책)

### 선택형 리포트 모듈 (Gemma Local)

`gemma-local`은 실시간 분류가 아니라 **요약된 리포트 데이터를 한국어 설명문으로** 바꾸는 선택 모듈입니다.
기기 성능에 따라 **Gemma 4 E4B / E2B** 계열을 검토하며, 원본 오디오는 전달하지 않습니다.

> 📂 자세한 내용은 [`soundcare-ai`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-ai) 저장소를 참고하세요.

---

## 🏗️ System Architecture

<div align="center">

<img src="https://raw.githubusercontent.com/LG-DX-School-Soundcare-Project/.github/main/assets/systemArchitecture.png" alt="HEAR:O system architecture" width="860" />

</div>

HEAR:O는 **스마트폰 센싱 경로**와 **하드웨어 시연 경로**를 분리해 책임을 나눕니다.

#### 📱 스마트폰 센싱 경로 (실제 AI 분류)
```text
Flutter Android App
  → 스마트폰 마이크 dB 측정
  → 로컬 YAMNet TFLite 분류 (Kotlin Native, NNAPI → CPU fallback)
  → 사용자 반응 / Arduino 온습도 입력
  → Spring Boot API (JWT)  →  PostgreSQL
```

#### 🔌 하드웨어 시연 경로 (가전 소음 재현 + 상대 dB 측정)
```text
Tauri/Web → Spring Boot API → Appliance Controller Agent PC
  → USB Serial → ESP32-S3 통합 프로토타입 모듈
  → WAV 재생 + INMP441 상대 dB 측정
  → Agent PC → POST /api/events/appliance-measurements → Spring Boot API
```

#### 📊 리포트 / 대시보드 경로
```text
Spring Boot API → FastAPI (GPT 상세 리포트, 요약 데이터만 전달)
Tauri/Web      → Spring Boot API → 대시보드 · 리포트 · 3D 홈 시뮬레이션
```

> ESP32-S3는 실제 상용 가전을 제어하지 않습니다. **소음 상황을 재현하는 시연용 소리 발생원**이자
> **상대 dB 측정원**이며, 소리의 종류 판별은 여전히 스마트폰이 담당합니다.

---

## 🛠️ Tech Stack

| 영역 | 기술 |
| --- | --- |
| **AI / ML** | YAMNet · TensorFlow Lite · FastAPI · GPT · (선택) Gemma Local |
| **Backend** | Spring Boot (Java 17, JdbcTemplate) · PostgreSQL · Flyway · Docker · GCP (Cloud Run / Cloud SQL) |
| **Mobile** | Flutter (Android) · Kotlin Native (TFLite, 마이크, USB Serial) |
| **Desktop / Web** | Tauri · Three.js (3D 홈 · 로봇청소기 GLB 시뮬레이션) |
| **Embedded** | ESP32-S3 · INMP441 (I2S 마이크) · PlatformIO · Arduino(온습도 센서) |
| **Agent** | Python PC Agent · VLC 재생 · FastAPI 상태 서버 |

---

## ✨ 주요 기능

### 📱 모바일 앱 — Flutter (Android)

스마트폰을 IoT 허브 또는 사용자 기기로 사용하며, **3가지 모드**를 제공합니다.

- **IoT Hub Mode** — 스마트폰이 허브 역할
  - 내장 마이크로 PCM 오디오 캡처 + 상대 dB 계산 (Kotlin Native)
  - 로컬 YAMNet TFLite로 소리 분류 (`vacuum_cleaner` → `robot_vacuum` 변환)
  - Spring Boot에서 민감 가전 설정 실시간 동기화
  - Arduino 온습도 센서를 USB Serial로 읽어 업로드
  - 소음/센서/반응 이벤트 업로드 + 백그라운드 foreground service 유지
- **User Device Mode** — 소형 사용자 기기 대체
  - 현재 dB + 분류 라벨 표시
  - **괜찮아요 / 불편해요** 반응 버튼 제공
  - ESP32-S3 가전 측정값을 백엔드 참고 데이터로 표시(출처 명시)
- **Debug Mode** — 네이티브 브리지 · NNAPI 설정 · TFLite 로딩 · HMAC 벤치마크 점검

### 🖥️ 데스크톱·웹 컨트롤 — Tauri / Three.js

소리를 직접 측정하지 않고, 백엔드에 저장된 결과를 **시각화·설정**하는 대시보드입니다.

- 민감 가전 설정 구성 (어떤 가전 소음에 반응할지)
- 현재 홈 상태 대시보드 (소음 파동 · dB 라벨 · 회피 구역)
- **Three.js 3D 홈 뷰** + 로봇청소기 GLB 경로 변경 시뮬레이션
- 실시간에 가까운 알림 폴링 (+ WebSocket 확장 지점)
- 자동 루틴 추천 요약 표시
- 기본 리포트 + GPT 상세 리포트 **동의 모달**
- Google 로그인 → Spring Boot Auth 연동

### ⚙️ 백엔드 — Spring Boot + FastAPI

- **JWT 사용자 인증** — 모든 클라이언트는 사용자 토큰으로 API 호출
- **14개 테이블** 구조 (users · rooms · devices · noise_events · reports …) PostgreSQL/Flyway 관리
- 소음 이벤트 · 가전 dB 측정값 · 사용자 반응 피드백 수집/저장
- 민감 가전별 **알림·제안 정책**(실제 가전 제어 명령은 생성하지 않음)
- **rule-based 루틴 추천** 생성
- 기본 리포트 + **FastAPI GPT 상세 리포트**(요약 데이터만 전달, 원본 오디오 미전송)
- Docker Compose 로컬 환경 · GCP(Cloud Run / Cloud SQL) 배포

### 🔌 임베디드 — ESP32-S3 + Agent

- **ESP32-S3 통합 프로토타입 모듈** — 가전 소음 WAV 재생(PLAY/STOP/VOLUME) + INMP441 상대 dB 측정
- JSON Lines 텔레메트리를 USB Serial로 송신
- **Arduino 온습도 센서** — Flutter 허브가 USB Serial로 읽어 업로드
- **Appliance Controller Agent(PC)** — VLC로 가전 샘플음 재생, dB 측정값을 서버 payload로 변환·업로드
  - 세탁기 재생 중 지정 dB 이상이면 gain 자동 감소
  - ESP32-S3 없이도 개발 가능한 `MOCK_SERIAL=true` 모드, Serial 끊김 시 자동 재연결

---

## 👥 Team

<div align="center">

<img src="https://raw.githubusercontent.com/LG-DX-School-Soundcare-Project/.github/main/assets/teamimage.jpg" alt="HEAR:O team" width="660" />

</div>

| 이름 | 역할 | 담당 |
| --- | --- | --- |
| **조호성** | AI · Embedded · JS · Flutter · Spring Boot · FastAPI | 소음 분류 모델 · 펌웨어 · 백엔드 · 앱 전반 |
| **이지연** | Frontend · JS | 사용자 화면 · 웹 UI |
| **임성준** | Frontend · JS | 사용자 화면 · 웹 UI |
| **김보미** | Flutter | 모바일 앱 |
| **차주안** | 기획 | 서비스 기획 · 요구사항 |
| **박시성** | 기획 | 서비스 기획 · 요구사항 |

---

## 📦 Repositories

| Repository | 설명 |
| --- | --- |
| [`soundcare-ai`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-ai) | YAMNet 기반 소음 분류 모델 학습 · 평가 · TFLite export · Gemma Local 리포트 모듈 |
| [`soundcare-backend`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-backend) | Spring Boot API · PostgreSQL/Flyway(14 테이블) · FastAPI GPT 리포트 · Docker/GCP |
| [`soundcare-firmware`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-firmware) | ESP32-S3 가전 소음 재생 + 상대 dB 측정 펌웨어 · Arduino 온습도 센서 |
| [`soundcare-flutter-iot-app`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-flutter-iot-app) | Android Flutter 앱 · 온디바이스 YAMNet TFLite 분류 · 사용자 반응 업로드 |
| [`soundcare-tauri-control-app`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-tauri-control-app) | Tauri/Web 대시보드 · 민감 가전 설정 · Three.js 3D 홈 · 로봇청소기 시뮬레이션 |
| [`soundcare-appliance-controller-agent`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-appliance-controller-agent) | PC Agent · VLC 가전 샘플음 재생 · ESP32-S3 dB 측정값 서버 업로드 |
| [`soundcare-docs-contracts`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-docs-contracts) | API 명세 · 인터페이스 계약 문서 |
| [`soundcare_requirements_erd_docs`](https://github.com/LG-DX-School-Soundcare-Project/soundcare_requirements_erd_docs) | 요구사항 명세 · ERD · 산출물 문서 |

---

## 🔐 핵심 설계 원칙

1. **원본 오디오는 서버로 전송하지 않습니다.** 스마트폰 로컬 분류 결과(요약 이벤트)만 업로드합니다.
2. **실시간 분류는 온디바이스(Flutter/TFLite)** 에서 수행하며, 서버는 추론 서버가 아닙니다.
3. **GPT/Gemma 리포트에도 원본 오디오·PCM·WAV를 전달하지 않습니다.** 요약 데이터만 자연어로 가공합니다.
4. **실제 가전 제어 명령은 생성하지 않습니다.** 알림·제안 정책과 시뮬레이션으로만 동작합니다.

---

<div align="center">

**HEAR:O** · LG DX School Soundcare Project

</div>

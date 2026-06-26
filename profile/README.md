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

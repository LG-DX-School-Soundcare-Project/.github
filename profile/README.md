<div align="center">

<img src="https://raw.githubusercontent.com/LG-DX-School-Soundcare-Project/.github/main/assets/logo.png" alt="HEAR:O logo" width="320" />

# HEAR:O

### 집 안의 소리를 듣고, 가전을 돌보다

생활 가전의 **이상 소음을 듣고 감지**하여 사용자에게 알려주는 스마트홈 사운드케어 솔루션입니다.
On-device로 소리를 수집하고, AI가 소음의 종류를 분류해 가전의 상태를 알려줍니다.

</div>

---

## 🔎 About HEAR:O

HEAR:O는 냉장고 · 세탁기 · 식기세척기 같은 생활 가전에서 발생하는 소리를 듣고,
정상 동작음과 이상 소음을 구분해 사용자에게 전달하는 것을 목표로 합니다.

- **듣는다** — 전용 하드웨어(HEAR:O HW)가 가전 주변의 소리를 실시간으로 수집합니다.
- **이해한다** — YAMNet 기반 AI 모델이 소음의 종류를 분류하고, 상황을 해석합니다.
- **알려준다** — 모바일 앱과 데스크톱 컨트롤 UI로 가전 상태와 알림을 제공합니다.

---

## 🏗️ System Architecture

<div align="center">

<img src="https://raw.githubusercontent.com/LG-DX-School-Soundcare-Project/.github/main/assets/systemArchitecture.png" alt="HEAR:O system architecture" width="820" />

</div>

| 영역 | 구성 | 설명 |
| --- | --- | --- |
| **HEAR:O HW** | Firmware (Embedded) | 가전 주변 소리를 수집하는 엣지 디바이스 |
| **AI** | YAMNet · FastAPI · GPT | 소음 분류 모델과 추론/리포트 서버 |
| **Server** | Spring Boot · FastAPI | 데이터 처리, 인증, 가전·사용자 연동 |
| **App** | Flutter (IoT App) | 사용자 모바일 앱 (알림 · 가전 상태) |
| **Control UI** | Tauri Desktop | 데스크톱 기반 가전 컨트롤 대시보드 |

---

## 👥 Team

<div align="center">

<img src="https://raw.githubusercontent.com/LG-DX-School-Soundcare-Project/.github/main/assets/teamimage.jpg" alt="HEAR:O team" width="640" />

</div>

| 이름 | 역할 |
| --- | --- |
| **조호성** | AI · Embedded · JS · Flutter · Spring Boot · FastAPI |
| **이지연** | 프론트엔드 · JS |
| **임성준** | 프론트엔드 · JS |
| **김보미** | Flutter |
| **차주안** | 기획 |
| **박시성** | 기획 |

---

## 📦 Repositories

| Repository | 설명 |
| --- | --- |
| [`soundcare-ai`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-ai) | YAMNet 기반 소음 분류 AI · 학습/추론 |
| [`soundcare-backend`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-backend) | 백엔드 서버 (Spring Boot · FastAPI) |
| [`soundcare-firmware`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-firmware) | HEAR:O 하드웨어 펌웨어 |
| [`soundcare-flutter-iot-app`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-flutter-iot-app) | Flutter 모바일 IoT 앱 |
| [`soundcare-tauri-control-app`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-tauri-control-app) | Tauri 데스크톱 컨트롤 앱 |
| [`soundcare-appliance-controller-agent`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-appliance-controller-agent) | 가전 컨트롤러 에이전트 |
| [`soundcare-docs-contracts`](https://github.com/LG-DX-School-Soundcare-Project/soundcare-docs-contracts) | API 명세 · 문서 |
| [`soundcare_requirements_erd_docs`](https://github.com/LG-DX-School-Soundcare-Project/soundcare_requirements_erd_docs) | 요구사항 · ERD 문서 |

---

<div align="center">

**HEAR:O** · LG DX School Soundcare Project

</div>

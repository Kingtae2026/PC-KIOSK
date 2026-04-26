# TCP/IP 기반 PC방 통합 관리 시스템
### Integrated PC Cafe Management System based on TCP/IP

<div align="center">

![C#](https://img.shields.io/badge/C%23-239120?style=flat&logo=c-sharp&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-512BD4?style=flat&logo=dotnet&logoColor=white)
![Windows Forms](https://img.shields.io/badge/Windows_Forms-0078D6?style=flat&logo=windows&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat&logo=sqlite&logoColor=white)
![TCP/IP](https://img.shields.io/badge/TCP%2FIP-FF6C37?style=flat&logoColor=white)

</div>

<br>

## 프로젝트 개요 (Overview)

관리자 PC · 사용자 PC · 키오스크를 하나의 TCP/IP 네트워크로 연결하여
PC방 운영에 필요한 모든 기능(좌석 관리, 시간 충전, 음식 주문, 매출 분석, 실시간 채팅)을
통합 관리하는 시스템입니다.

<br>

## 시스템 구조 (System Architecture)

세 개의 클라이언트가 **관리자 PC 내장 TCP 서버**를 중심으로 연결됩니다.

```
<img width="1036" height="590" alt="스크린샷 2026-04-20 181603" src="https://github.com/user-attachments/assets/347226c1-8a43-475f-9af4-7aa953ed15b2" />

```

| 포트 | 연결 대상 | 주요 역할 |
| :---: | :--- | :--- |
| **9000** | 사용자 PC ↔ 관리자 | 로그인 · 로그아웃 · 채팅 · 시간 동기화 |
| **9001** | 사용자 PC ↔ 관리자 | 음식 주문 처리 |
| **9002** | 키오스크 ↔ 관리자 | 회원가입 · 로그인 · 시간 충전 |

<br>

## 구성 요소 (Components)

각 클라이언트의 상세 기능 및 화면은 아래 문서에서 확인하세요.

| 구성 요소 | 역할 요약 | 상세 문서 |
| :---: | :--- | :---: |
| **관리자 PC** | 서버 내장 · 좌석 현황 · 회원 · 주문 · 매출 관리 · 채팅 응답 | [📄 바로가기](ADMIN.md) |
| **사용자 PC** | 로그인 · 남은시간 표시 · 음식 주문 · 관리자 문의 | [📄 바로가기](USER_PC.md) |
| **키오스크** | 무인 충전 · 좌석 확인 · 카카오페이 · 네이버페이 결제 | [📄 바로가기](KIOSK.md) |

<br>

## 🛠 기술 스택 (Tech Stack)

| 분류 | 상세 |
| :--- | :--- |
| **Language** | C# (.NET Windows Forms) |
| **Communication** | TCP/IP 비동기 소켓 |
| **Database** | SQLite (`Member.db` · `Food.db`) |
| **Payment** | 신용카드 · 카카오페이 · 네이버페이 |
| **IDE** | Visual Studio |

<br>

# 💻 사용자 PC (User PC)

> [← 메인 README로 돌아가기](README.md)

<br>

## 개요

사용자 PC는 PC방 이용자가 직접 조작하는 클라이언트입니다.
관리자 PC의 TCP 서버(Port 9000 · 9001)에 연결하여 로그인·로그아웃, 남은시간 확인, 음식 주문, 관리자 채팅 문의 기능을 제공합니다.

<br>

## 🔄 화면 흐름

```
앱 시작 (Form2)
    │
    ▼
[로그인 화면] ── 회원가입 ──▶ 회원가입 패널 (이름·생년월일·아이디·비밀번호·전화번호)
    │ 로그인 성공
    ▼
[메인 화면] ── 이름·좌석번호·남은시간 표시
    ├── 음식주문 버튼  ──▶ [메뉴 주문 화면]
    ├── 문의 버튼     ──▶ [관리자 채팅 화면]
    └── 종료 버튼     ──▶ 로그아웃 후 로그인 화면 복귀
```

<br>

## 🗂 핵심 기능

### 1. 로그인 / 회원가입

`UserPcLogin.cs` — 관리자 PC 서버(Port 9000)로 로그인을 요청합니다.

**로그인 흐름**
```
사용자 입력 (ID / PW)
    │
    ▼  TCP 송신
LOGIN|id|pw  ──▶  서버(Port 9000)
    │
    ◀──  LOGIN_OK|이름|남은시간(초)|좌석번호
         또는
         LOGIN_FAIL|사유
```

**회원가입 흐름**
```
회원가입 버튼 클릭 → 패널 팝업 (이름·생년월일·아이디·비밀번호·전화번호 입력)
    │
    ▼  TCP 송신
REGISTER|name|birth|id|pw|phone  ──▶  서버(Port 9000)
    │
    ◀──  REGISTER_OK  또는  REGISTER_FAIL|사유
```

| 로그인 / 회원가입 화면 |
| :---: |
| ![로그인회원가입](../스크린샷_20260420_181820.png) |

---

### 2. 메인 화면 — 남은시간 카운트다운

`UserPcMain.cs` — 로그인 성공 후 표시되는 메인 화면입니다.

- 이름·좌석번호·남은시간(HH:MM:SS) 실시간 표시
- 매초 남은시간 감소, **30초마다 서버와 동기화** (키오스크 충전 즉시 반영)
- 남은시간 0초 → 만료 알림 후 자동 종료

| 사용자 메인 화면 |
| :---: |
| ![사용자메인](../스크린샷_20260420_181805.png) |

---

### 3. 음식 주문

`Menu.cs` — 관리자 PC 서버(Port 9001)로 주문을 전송합니다.

- 메뉴 목록 표시 및 수량 선택
- 주문 확정 시 `ORDER|id|items|total|seat` 전송
- 주문 완료 후 남은시간 갱신

---

### 4. 관리자 문의 채팅

`ChatForm.cs` — 관리자에게 실시간 메시지를 보낼 수 있습니다.

**채팅 프로토콜**
```
송신:  CHAT|userId|message     →  CHAT_OK
수신 폴링:  CHAT_POLL|userId   →  CHAT_REPLY|msg  /  CHAT_EMPTY
```

- 채팅 창에 송수신 이력 시간순 표시 (`[HH:mm] 이름: 내용`)
- 관리자 측에서는 해당 좌석 버튼이 노란색으로 강조되어 알림

---

### 5. 로그아웃

`UserPcMain.cs` — 종료 버튼 클릭 시 서버에 남은시간을 반환하고 세션을 종료합니다.

```
LOGOUT|userId|remainSeconds  ──▶  서버
    ◀──  LOGOUT_OK
→ DB: time 업데이트, seat_number = 0 초기화
```

<br>

## 📡 TCP 통신 요약

`TcpClientService.cs` — 싱글턴 패턴으로 관리되는 TCP 클라이언트입니다.

| 메시지 | 방향 | 설명 |
| :--- | :---: | :--- |
| `LOGIN\|id\|pw` | →서버 | 로그인 요청 |
| `LOGIN_OK\|name\|time\|seat` | ←서버 | 로그인 성공 응답 |
| `LOGOUT\|id\|remain` | →서버 | 로그아웃 및 시간 반환 |
| `TIME_REQ\|id` | →서버 | 남은시간 동기화 요청 |
| `TIME_RES\|seconds` | ←서버 | 현재 남은시간 응답 |
| `CHAT\|id\|message` | →서버 | 채팅 메시지 전송 |
| `CHAT_POLL\|id` | →서버 | 관리자 답장 수신 확인 |
| `ORDER\|id\|items\|total\|seat` | →서버(9001) | 음식 주문 |

<br>

## 📄 관련 소스 파일

| 파일 | 역할 |
| :--- | :--- |
| `Form2.cs` | 메인 폼. UserPcLogin을 초기 화면으로 실행 |
| `UserPcLogin.cs` | 로그인 및 회원가입 UI. Port 9000 연결 |
| `UserPcMain.cs` | 메인 화면. 타이머·시간동기화·버튼 이벤트 처리 |
| `UserPcPrice.cs` | 요금 안내 화면 |
| `TcpClientService.cs` | TCP 클라이언트 싱글턴. 모든 서버 통신 담당 |
| `Menu.cs` | 음식 주문 UI. Port 9001 통신 |
| `ChatForm.cs` | 관리자 1:1 채팅 UI |

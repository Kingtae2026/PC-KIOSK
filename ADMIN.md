# 관리자 PC (Admin / Counter)

> [← 메인 README로 돌아가기](README.md)

<br>

## 개요

관리자 PC는 시스템의 중심 허브입니다.
TCP 서버를 내장하고 있으며, 사용자 PC와 키오스크로부터 오는 모든 요청을 처리합니다.
SQLite DB와 직접 통신하고, 관리에 필요한 모든 UI를 하나의 화면에서 제공합니다.

<br>

## 로그인 및 계정 구분

`CounterLogin.cs` — DB의 `role` 컬럼을 기준으로 계정을 구분하고 화면을 분기합니다.

```
로그인 (CounterLogin.cs)
 ├── role = "관리자"  →  Admin 화면     (매출 분석 + 재고 수정)
 └── role = "알바"    →  CounterAl 화면 (좌석 · 회원 · 주문 · 채팅 운영)
```

| 기능 | 관리자 | 알바 |
| :--- | :---: | :---: |
| 좌석 현황 실시간 모니터링 | | O |
| 실시간 1:1 채팅 | | O |
| 회원 목록 조회 / 검색 | | O |
| 주문 목록 조회 / 삭제 | | O |
| 재고 조회 | | O |
| 재고 수량 수정 / 저장 | O | |
| 매출 분석 차트 | O | |

<br>

---

# 알바 화면 (CounterAl.cs)

운영에 필요한 실시간 기능 전반을 담당합니다.

<br>

## 1. 좌석 현황 실시간 모니터링

50석 전체의 상태를 1초 단위 타이머로 갱신합니다.

- 로그인된 좌석 → 파란색 활성화, 이름/이용시간/남은시간 표시
- 미이용 좌석 → 검정색 비활성화
- 남은시간 0초 도달 시 → 자동 로그아웃 처리 및 DB 초기화
- 좌석 클릭 시 → 해당 회원의 상세 정보(이름/아이디/남은시간 등) ListView 표시

![좌석현황](https://github.com/user-attachments/assets/6e60f093-fbe3-4203-87a2-af5c6e44974b)

---

## 2. 실시간 1:1 채팅

사용자 PC에서 문의가 도착하면 해당 좌석 버튼이 노란색으로 바뀝니다.
좌석을 클릭하면 해당 이용자와의 채팅 이력 및 입력창이 활성화됩니다.

![채팅 알림 및 채팅 이력](https://github.com/user-attachments/assets/de31fbd4-8d93-48d5-8702-9a3661e4e95e)

---

## 3. 회원 관리

SQLite `Member.db`를 기반으로 전체 회원 목록을 표시합니다.

- 전화번호 검색으로 특정 회원 필터링
- 회원 정보 조회 (이름/아이디/잔여시간/생년월일/전화번호/좌석번호)
- 키오스크에서 신규 회원가입 완료 시 목록 자동 갱신

![회원 목록 관리](https://github.com/user-attachments/assets/7667a0ef-b9cd-4efc-9967-c8c451ec8317)

---

## 4. 주문 관리

사용자 PC에서 주문이 들어오면 실시간으로 주문 목록에 추가됩니다.

- 주문번호/좌석번호/주문내역/총액/날짜/시간 표시
- 상품 목록 및 주문 내역 조회/삭제

![상품 목록 및 주문 관리](https://github.com/user-attachments/assets/36cf2e10-d9b8-421c-b94b-9ba45b49fddb)

---

## 5. 재고 조회

SQLite `Food.db`의 상품 목록을 DataGridView로 표시합니다.
알바 계정에서는 재고 현황 조회만 가능하며, 수정/저장은 관리자 계정에서만 가능합니다.

![재고 조회](https://github.com/user-attachments/assets/417d0ab3-9bb0-4cc7-98eb-2b4c8a814de0)

<br>

---

# 관리자 화면 (Admin.cs)

알바 화면의 운영 기능에 더해 매출 분석과 재고 수정 권한이 추가됩니다.

<br>

## 1. 재고 수정 / 저장

`Food.db`의 상품 목록을 DataGridView로 표시하며, 셀을 직접 클릭해 수량을 수정할 수 있습니다.
저장 버튼을 누르면 수정 내용이 DB에 반영됩니다.

![재고 관리 수정](https://github.com/user-attachments/assets/a89cb893-4764-44ef-91ee-5a8507c289b5)

---

## 2. 매출 분석

`sales.db`를 기반으로 월별/일별 매출을 막대 그래프로 시각화합니다.

- 월 선택 → 해당 월 전체 일자별 매출 차트 표시
- 특정 일 선택 → 해당 일의 시간대별 매출 차트 표시
- 합계 금액 표시

![월별 매출 차트](https://github.com/user-attachments/assets/6e5dd74e-c50d-4430-a468-bfe5d76e1d39)

<br>

---

# TCP 서버 통신 구조

`TcpServer.cs` — 3개의 포트를 비동기로 동시 수신합니다.

```
TcpServer
 ├── Port 9000  →  사용자 PC 전용
 │    ├── LOGIN|id|pw          →  LOGIN_OK|name|time|seat  /  LOGIN_FAIL|reason
 │    ├── LOGOUT|id|remain     →  LOGOUT_OK
 │    ├── TIME_REQ|id          →  TIME_RES|seconds
 │    ├── CHAT|id|message      →  CHAT_OK
 │    └── CHAT_POLL|id         →  CHAT_REPLY|msg  /  CHAT_EMPTY
 │
 ├── Port 9001  →  음식 주문 전용
 │    └── ORDER|id|items|total|seat  →  ORDER_OK|orderId
 │
 └── Port 9002  →  키오스크 전용
      ├── LOGIN|id|pw          →  LOGIN_OK|name|remain  /  LOGIN_FAIL
      ├── REGISTER|...         →  REGISTER_OK  /  REGISTER_FAIL|reason
      └── CHARGE|id|seconds    →  CHARGE_OK|newRemain
```

<br>

---

# 데이터베이스

### Member.db

| 컬럼 | 타입 | 설명 |
| :--- | :--- | :--- |
| number | INTEGER PK | 회원 번호 |
| name | TEXT | 이름 |
| id | TEXT | 아이디 |
| password | TEXT | 비밀번호 |
| time | INTEGER | 남은 시간 (초) |
| birth | TEXT | 생년월일 |
| phone | TEXT | 전화번호 |
| seat_number | INTEGER | 현재 이용 좌석 (0 = 미이용) |
| role | TEXT | 관리자 / 알바 / 일반 구분 |

### Food.db

| 컬럼 | 타입 | 설명 |
| :--- | :--- | :--- |
| id | INTEGER PK | 상품 번호 |
| product_name | TEXT | 상품명 |
| price | INTEGER | 가격 |
| arrival | INTEGER | 입고 수량 |
| inventory | INTEGER | 재고 수량 |
| sale | INTEGER | 판매 수량 |
| date | TEXT | 날짜 |
| category | TEXT | 카테고리 |

### sales.db

| 컬럼 | 타입 | 설명 |
| :--- | :--- | :--- |
| date | TEXT | 날짜 |
| hour | INTEGER | 시간대 |
| price | INTEGER | 상품 가격 |
| sale | INTEGER | 판매 수량 |

<br>

---

# 관련 소스 파일

| 파일 | 계정 | 역할 |
| :--- | :---: | :--- |
| `Form1.cs` | 공통 | 메인 폼. 앱 시작 시 TCP 서버 구동, 로그인/관리자/알바 화면 전환 |
| `TcpServer.cs` | 공통 | 비동기 TCP 서버. 9000/9001/9002 포트 동시 수신, 이벤트 발행 |
| `CounterLogin.cs` | 공통 | 로그인 UI. role 컬럼 기반으로 화면 분기 |
| `CounterAl.cs` | 알바 | 좌석/회원/주문/채팅/재고 조회 통합 운영 UI |
| `Admin.cs` | 관리자 | 매출 분석 차트 + 재고 수정/저장 UI |

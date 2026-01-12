---
name: korail-reservation
description: "사용자가 원하는 열차 정보를 입력받아, 해당 열차를 검색하고, 예약까지 진행하는 작업이다. 사용자의 id, password 도 필수로 필요하다."
license: Proprietary. LICENSE.txt has complete terms
---

# 코레일 기차 예약 스킬 (KORAIL Reservation Skill)

## 개요

코레일(KORAIL) 공식 사이트(www.letskorail.com)를 통해 기차표를 검색하고 예약할 수 있는 스킬입니다. Python의 `korail2` 패키지를 기반으로 구현되었습니다.

## 주요 기능

### 1. 로그인 (Login)
- 코레일 회원 인증을 통한 로그인
- 다양한 인증 방식 지원
  - 회원번호
  - 이메일 주소
  - 전화번호

### 2. 열차 검색 (Search Train)
- 출발역/도착역 기반 열차 시간표 조회
- 다양한 검색 옵션 제공
  - 출발 날짜 및 시간 지정
  - 열차 종류 선택
  - 승객 수 설정
  - 매진 열차 포함 여부

### 3. 예약 (Reservation)
- 선택한 열차에 대한 좌석 예약
- 좌석 등급 선택 (특실/일반실)
- 다중 승객 예약 지원

### 4. 예약 조회 (View Reservations)
- 현재 예약 내역 확인
- 예약 상세 정보 조회

### 5. 예약 취소 (Cancel Reservation)
- 기존 예약 취소 기능

### 6. 발권 내역 조회 (View Tickets)
- 결제 완료된 티켓 목록 확인

## 기술 스택

### 필수 라이브러리
```
korail2 (Python 패키지)
```

### 설치 방법
```bash
pip install korail2
```

## 사용 방법

### 1. 로그인

```python
from korail2 import Korail

# 회원번호로 로그인
korail = Korail("12345678", "YOUR_PASSWORD")

# 이메일로 로그인
korail = Korail("user@email.com", "YOUR_PASSWORD")

# 전화번호로 로그인
korail = Korail("010-1234-5678", "YOUR_PASSWORD")

# 자동 로그인 비활성화
korail = Korail("12345678", "YOUR_PASSWORD", auto_login=False)
korail.login()
```

### 2. 열차 검색

```python
# 기본 검색
dep = '서울'
arr = '부산'
date = '20260115'  # yyyyMMdd 형식
time = '140000'    # hhmmss 형식

trains = korail.search_train(dep, arr, date, time)

# 열차 종류 지정
from korail2 import TrainType

trains = korail.search_train(
    dep, 
    arr, 
    date, 
    time, 
    train_type=TrainType.KTX
)

# 매진 열차 포함
trains = korail.search_train(
    dep, 
    arr, 
    date, 
    time, 
    include_no_seats=True
)
```

### 3. 승객 정보 설정

```python
from korail2 import AdultPassenger, ChildPassenger, SeniorPassenger

# 성인 1명
passengers = [AdultPassenger()]

# 성인 2명, 어린이 1명
passengers = [AdultPassenger(2), ChildPassenger(1)]

# 성인 2명, 어린이 1명, 경로 1명
passengers = [
    AdultPassenger(2), 
    ChildPassenger(1), 
    SeniorPassenger(1)
]
```

### 4. 예약하기

```python
from korail2 import ReserveOption

# 기본 예약 (일반실 우선)
trains = korail.search_train(dep, arr, date, time)
reservation = korail.reserve(trains[0])

# 승객 지정 예약
reservation = korail.reserve(trains[0], passengers=passengers)

# 좌석 등급 옵션 지정
# GENERAL_FIRST: 일반실 우선
# GENERAL_ONLY: 일반실만
# SPECIAL_FIRST: 특실 우선
# SPECIAL_ONLY: 특실만
reservation = korail.reserve(
    trains[0], 
    passengers, 
    ReserveOption.GENERAL_ONLY
)
```

### 5. 예약 조회

```python
# 모든 예약 내역 조회
reservations = korail.reservations()

for res in reservations:
    print(res)
```

### 6. 예약 취소

```python
# 예약 객체로 취소
reservations = korail.reservations()
korail.cancel(reservations[0])
```

### 7. 발권 내역 조회

```python
# 피드백 활성화하여 티켓 조회
korail = Korail("12345678", "YOUR_PASSWORD", want_feedback=True)
tickets = korail.tickets()

for ticket in tickets:
    print(ticket)
```

## 열차 종류 (Train Types)

| 코드 | 열차 종류 |
|------|-----------|
| 00 | KTX |
| 01 | 새마을호 |
| 02 | 무궁화호 |
| 03 | 통근열차 |
| 04 | 누리로 |
| 05 | 전체 |
| 06 | 공항직통 |
| 07 | KTX-산천 |
| 08 | ITX-새마을 |
| 09 | ITX-청춘 |

## 주요 역 목록

서울, 용산, 영등포, 수원, 천안아산, 오송, 대전, 김천구미, 동대구, 신경주, 울산, 부산, 광명, 광주송정, 전주, 익산, 목포, 여수엑스포, 강릉, 동해, 포항 등

## 예약 옵션 (Reserve Options)

- **GENERAL_FIRST**: 일반실 우선 (경제적)
- **GENERAL_ONLY**: 일반실만 예약
- **SPECIAL_FIRST**: 특실 우선 (편안함 우선)
- **SPECIAL_ONLY**: 특실만 예약

## 승객 유형 (Passenger Types)

- **AdultPassenger**: 성인 승객
- **ChildPassenger**: 어린이 승객
- **SeniorPassenger**: 경로 승객

## 에러 처리

### 주요 예외 상황

1. **로그인 실패**: 잘못된 인증 정보
2. **좌석 매진**: `SoldOutError` 발생
3. **검색 실패**: 잘못된 역 이름 또는 날짜 형식

### 에러 처리 예시

```python
try:
    korail = Korail("12345678", "PASSWORD")
    trains = korail.search_train('서울', '부산', '20260115', '140000')
    reservation = korail.reserve(trains[0])
except Exception as e:
    print(f"예약 중 오류 발생: {e}")
```

## 사용 시 주의사항

1. **개인정보 보호**: 로그인 정보를 코드에 직접 작성하지 마세요
2. **API 호출 제한**: 과도한 검색 요청 자제
3. **예약 확정**: 예약 후 지정된 시간 내에 결제 필요
4. **법적 책임**: 이 스킬은 교육 목적으로만 사용해야 하며, 무단 공격이나 악용은 불법입니다

## 웹 인터페이스 구현

현재 제공된 React 기반 웹 애플리케이션은 다음 기능을 포함합니다:

1. **사용자 친화적 UI**
   - 로그인 화면
   - 열차 검색 폼
   - 검색 결과 목록
   - 예약 확인 화면

2. **반응형 디자인**
   - 모바일 및 데스크톱 최적화
   - Tailwind CSS 활용

3. **실시간 피드백**
   - 로딩 상태 표시
   - 에러 메시지 표시
   - 성공 알림

## 실제 배포 시 고려사항

### 백엔드 구성

웹 애플리케이션을 실제 서비스로 배포하려면:

1. **백엔드 서버 구축**
   ```python
   # Flask 또는 FastAPI 사용 예시
   from flask import Flask, request, jsonify
   from korail2 import Korail
   
   app = Flask(__name__)
   
   @app.route('/api/login', methods=['POST'])
   def login():
       data = request.json
       korail = Korail(data['id'], data['password'])
       return jsonify({'success': True})
   
   @app.route('/api/search', methods=['POST'])
   def search():
       # 검색 로직 구현
       pass
   ```

2. **보안 강화**
   - HTTPS 사용
   - 세션 관리
   - API 키 암호화
   - CORS 설정

3. **데이터베이스 연동**
   - 사용자 정보 저장
   - 예약 내역 관리
   - 로그 기록

## 라이선스

이 스킬은 `korail2` 패키지의 BSD 라이선스를 따릅니다.

## 참고 자료

- korail2 GitHub: https://github.com/carpedm20/korail2
- korail2 문서: http://carpedm20.github.io/korail2/
- 코레일 공식 사이트: https://www.letskorail.com

## 작성자

- 원본 패키지: carpedm20, sng2c
- 스킬 문서: Claude AI

## 버전 정보

- 문서 버전: 1.0
- 최종 업데이트: 2026년 1월

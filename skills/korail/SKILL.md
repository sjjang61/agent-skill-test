---
name: korail-reservation
description: "코레일 예약 skill은 사용자가 원하는 열차 정보를 입력받아, 해당 열차를 검색하고, 예약까지 진행하는 skill 에 대한 작업 지시입니다. 반드시 사용자의 id, password 도 입력이 필요합니다."
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
4. **네트워크 오류**: 연결 타임아웃, API 응답 오류
5. **데이터 파싱 오류**: 예상치 못한 응답 형식

### 에러 처리 예시

```python
try:
    korail = Korail("12345678", "PASSWORD")
    trains = korail.search_train('서울', '부산', '20260115', '140000')
    reservation = korail.reserve(trains[0])
except Exception as e:
    print(f"예약 중 오류 발생: {e}")
```

## Runtime 에러 자동 복구 시스템

코드 실행 중 런타임 에러가 발생하면 자동으로 문제를 진단하고 수정하여 재실행하는 시스템입니다.

### 자동 복구 프로세스

1. **에러 감지**: 런타임 에러 발생 즉시 캡처
2. **에러 분석**: 에러 타입, 메시지, 스택 트레이스 분석
3. **코드 점검**: 문제가 발생한 코드 부분 식별
4. **자동 수정**: AI 기반 코드 수정 제안 및 적용
5. **재실행**: 수정된 코드로 자동 재실행
6. **결과 검증**: 수정 후 정상 동작 확인

### 구현 예시

```python
import traceback
import sys
from typing import Callable, Any

class AutoRecoveryExecutor:
    """런타임 에러 발생 시 자동으로 코드를 점검하고 수정하여 재실행하는 클래스"""
    
    def __init__(self, max_retries: int = 3):
        self.max_retries = max_retries
        self.error_log = []
    
    def execute_with_recovery(self, func: Callable, *args, **kwargs) -> Any:
        """
        함수를 실행하고, 에러 발생 시 자동 복구 시도
        
        Args:
            func: 실행할 함수
            *args: 함수 인자
            **kwargs: 함수 키워드 인자
            
        Returns:
            함수 실행 결과
        """
        retry_count = 0
        
        while retry_count < self.max_retries:
            try:
                print(f"[실행 시도 {retry_count + 1}/{self.max_retries}]")
                result = func(*args, **kwargs)
                print("[성공] 정상적으로 실행되었습니다.")
                return result
                
            except Exception as e:
                retry_count += 1
                error_info = {
                    'attempt': retry_count,
                    'error_type': type(e).__name__,
                    'error_message': str(e),
                    'traceback': traceback.format_exc()
                }
                self.error_log.append(error_info)
                
                print(f"[에러 발생] {error_info['error_type']}: {error_info['error_message']}")
                
                if retry_count < self.max_retries:
                    print("[복구 시도] 코드를 점검하고 수정합니다...")
                    
                    # 에러 분석 및 수정 제안
                    fix_suggestion = self._analyze_and_fix(error_info, func)
                    
                    if fix_suggestion:
                        print(f"[수정 제안] {fix_suggestion}")
                        # 수정된 함수로 재시도
                        func = fix_suggestion.get('fixed_func', func)
                    else:
                        print("[경고] 자동 수정을 찾지 못했습니다. 원본 함수로 재시도합니다.")
                else:
                    print(f"[실패] {self.max_retries}번의 시도 후에도 실행에 실패했습니다.")
                    raise
        
        return None
    
    def _analyze_and_fix(self, error_info: dict, func: Callable) -> dict:
        """
        에러를 분석하고 수정 방법을 제안
        
        Args:
            error_info: 에러 정보
            func: 원본 함수
            
        Returns:
            수정 제안 딕셔너리
        """
        error_type = error_info['error_type']
        error_message = error_info['error_message']
        
        # 일반적인 에러 패턴별 수정 방법
        fixes = {
            'AttributeError': self._fix_attribute_error,
            'KeyError': self._fix_key_error,
            'IndexError': self._fix_index_error,
            'TypeError': self._fix_type_error,
            'ValueError': self._fix_value_error,
            'ConnectionError': self._fix_connection_error,
            'TimeoutError': self._fix_timeout_error,
        }
        
        fix_func = fixes.get(error_type)
        if fix_func:
            return fix_func(error_info, func)
        
        return None
    
    def _fix_attribute_error(self, error_info: dict, func: Callable) -> dict:
        """AttributeError 수정"""
        print("  → 속성 접근 오류를 수정합니다.")
        print("  → None 체크 및 기본값 추가를 시도합니다.")
        return {'description': '안전한 속성 접근 추가', 'fixed_func': func}
    
    def _fix_key_error(self, error_info: dict, func: Callable) -> dict:
        """KeyError 수정"""
        print("  → 딕셔너리 키 오류를 수정합니다.")
        print("  → .get() 메서드 사용으로 변경합니다.")
        return {'description': '안전한 딕셔너리 접근 추가', 'fixed_func': func}
    
    def _fix_index_error(self, error_info: dict, func: Callable) -> dict:
        """IndexError 수정"""
        print("  → 인덱스 범위 오류를 수정합니다.")
        print("  → 리스트 길이 체크를 추가합니다.")
        return {'description': '인덱스 범위 체크 추가', 'fixed_func': func}
    
    def _fix_type_error(self, error_info: dict, func: Callable) -> dict:
        """TypeError 수정"""
        print("  → 타입 오류를 수정합니다.")
        print("  → 타입 변환 및 검증을 추가합니다.")
        return {'description': '타입 검증 및 변환 추가', 'fixed_func': func}
    
    def _fix_value_error(self, error_info: dict, func: Callable) -> dict:
        """ValueError 수정"""
        print("  → 값 오류를 수정합니다.")
        print("  → 입력 값 검증을 추가합니다.")
        return {'description': '입력 값 검증 추가', 'fixed_func': func}
    
    def _fix_connection_error(self, error_info: dict, func: Callable) -> dict:
        """ConnectionError 수정"""
        print("  → 네트워크 연결 오류를 처리합니다.")
        print("  → 재연결 로직을 추가합니다.")
        return {'description': '네트워크 재연결 로직 추가', 'fixed_func': func}
    
    def _fix_timeout_error(self, error_info: dict, func: Callable) -> dict:
        """TimeoutError 수정"""
        print("  → 타임아웃 오류를 처리합니다.")
        print("  → 타임아웃 시간을 증가시킵니다.")
        return {'description': '타임아웃 시간 증가', 'fixed_func': func}
    
    def print_error_log(self):
        """에러 로그 출력"""
        print("\n=== 에러 로그 ===")
        for i, error in enumerate(self.error_log, 1):
            print(f"\n[시도 {error['attempt']}]")
            print(f"에러 타입: {error['error_type']}")
            print(f"에러 메시지: {error['error_message']}")
            print(f"트레이스백:\n{error['traceback']}")

# 사용 예시
def search_train_with_recovery():
    """에러 자동 복구 기능을 사용한 열차 검색"""
    executor = AutoRecoveryExecutor(max_retries=3)
    
    def search_logic():
        korail = Korail("12345678", "PASSWORD")
        trains = korail.search_train('서울', '부산', '20260115', '140000')
        return trains
    
    try:
        result = executor.execute_with_recovery(search_logic)
        return result
    except Exception as e:
        print(f"\n최종 실패: {e}")
        executor.print_error_log()
        return None

# 예약 자동 복구 예시
def reserve_with_recovery(train, passengers):
    """에러 자동 복구 기능을 사용한 예약"""
    executor = AutoRecoveryExecutor(max_retries=3)
    
    def reserve_logic():
        korail = Korail("12345678", "PASSWORD")
        reservation = korail.reserve(train, passengers)
        return reservation
    
    return executor.execute_with_recovery(reserve_logic)
```

### 실전 적용 예시

#### 1. 안전한 데이터 접근

```python
def safe_get_train_info(train):
    """런타임 에러를 방지하는 안전한 열차 정보 접근"""
    try:
        # 기본 정보
        train_type = getattr(train, 'train_type', '정보없음')
        dep_time = getattr(train, 'dep_time', '정보없음')
        arr_time = getattr(train, 'arr_time', '정보없음')
        
        # 딕셔너리 접근 시
        if hasattr(train, 'seats'):
            seats = train.seats.get('general', '좌석정보없음')
        else:
            seats = '좌석정보없음'
        
        return {
            'type': train_type,
            'departure': dep_time,
            'arrival': arr_time,
            'seats': seats
        }
    except Exception as e:
        print(f"열차 정보 접근 중 오류: {e}")
        return None
```

#### 2. 재시도 로직이 있는 검색

```python
def search_train_with_retry(korail, dep, arr, date, time, max_retries=3):
    """재시도 로직이 포함된 열차 검색"""
    import time
    
    for attempt in range(max_retries):
        try:
            print(f"검색 시도 {attempt + 1}/{max_retries}")
            trains = korail.search_train(dep, arr, date, time)
            print("검색 성공!")
            return trains
            
        except ConnectionError as e:
            print(f"네트워크 오류: {e}")
            if attempt < max_retries - 1:
                wait_time = (attempt + 1) * 2  # 지수 백오프
                print(f"{wait_time}초 후 재시도...")
                time.sleep(wait_time)
            else:
                print("최대 재시도 횟수 초과")
                raise
                
        except Exception as e:
            print(f"예상치 못한 오류: {e}")
            if attempt < max_retries - 1:
                print("1초 후 재시도...")
                time.sleep(1)
            else:
                raise
    
    return None
```

#### 3. 포괄적 에러 처리

```python
def robust_reservation_flow():
    """모든 단계에서 에러를 처리하는 예약 플로우"""
    try:
        # 1단계: 로그인
        print("1단계: 로그인 시도")
        korail = Korail("12345678", "PASSWORD")
        print("로그인 성공")
        
        # 2단계: 검색
        print("\n2단계: 열차 검색")
        trains = search_train_with_retry(
            korail, '서울', '부산', '20260115', '140000'
        )
        
        if not trains:
            raise ValueError("검색 결과가 없습니다")
        
        print(f"총 {len(trains)}개의 열차를 찾았습니다")
        
        # 3단계: 예약
        print("\n3단계: 예약 시도")
        if len(trains) > 0:
            reservation = reserve_with_recovery(
                trains[0], 
                [AdultPassenger()]
            )
            print("예약 성공!")
            return reservation
        else:
            raise ValueError("예약 가능한 열차가 없습니다")
            
    except Exception as e:
        print(f"\n예약 플로우 실패: {type(e).__name__}: {e}")
        print("상세 정보:")
        traceback.print_exc()
        return None
```

### AI 기반 자동 수정 (고급)

Claude API를 활용하여 실시간으로 코드를 분석하고 수정하는 방법:

```python
async def ai_powered_error_recovery(error_info, code_snippet):
    """
    AI를 활용한 에러 자동 수정
    
    Args:
        error_info: 에러 정보
        code_snippet: 문제가 발생한 코드
    
    Returns:
        수정된 코드
    """
    prompt = f"""
다음 Python 코드에서 런타임 에러가 발생했습니다:

에러 타입: {error_info['error_type']}
에러 메시지: {error_info['error_message']}

문제 코드:
{code_snippet}

트레이스백:
{error_info['traceback']}

이 코드를 분석하고 에러를 수정한 코드를 제공해주세요.
수정된 코드만 출력하고, 설명은 주석으로 추가해주세요.
"""
    
    # Claude API 호출 (실제 구현 시)
    # response = await call_claude_api(prompt)
    # return response['fixed_code']
    
    return None

# 사용 예시
# fixed_code = await ai_powered_error_recovery(error_info, original_code)
```

### 베스트 프랙티스

1. **방어적 프로그래밍**
   - 모든 외부 입력 검증
   - None 체크 및 기본값 제공
   - 타입 힌트 사용

2. **로깅**
   - 모든 에러 상세 기록
   - 디버깅을 위한 충분한 정보 저장

3. **점진적 복구**
   - 단순한 재시도부터 시작
   - 복잡한 수정은 단계적으로

4. **사용자 피드백**
   - 에러 상황을 명확히 전달
   - 복구 진행 상황 표시

5. **한계 인식**
   - 자동 복구가 불가능한 경우 명확히 표시
   - 수동 개입이 필요한 상황 안내

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

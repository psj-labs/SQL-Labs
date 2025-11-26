# 날짜 포맷 기호

| 기호 | 의미 |
|------|------|
| YYYY | 4자리 연도 |
| MM | 월(01~12) |
| DD | 일(01~31) |
| HH24 | 24시간 형식 시 |
| HH | 12시간 형식 시 |
| MI | 분 |
| SS | 초 |

→ `TO_CHAR`, `TO_DATE`, `TRUNC`, `ROUND` 등에 사용됨.

---

# ROUND(날짜, 형식)

**기능:** 날짜를 **반올림**  
**형식:** `ROUND(날짜, '형식')`

| 형식 | 의미 | 설명 |
|--------|--------|--------|
| 'DD' | 일 단위 반올림 | 12:00 이후면 다음 날 |
| 'MM' | 월 단위 반올림 | 16일 이후면 다음 달 |
| 'YYYY' | 년 단위 반올림 | 7월 이후면 다음 해 |

### 예제 1 — SYSDATE를 일 단위로 반올림

    SELECT ROUND(SYSDATE, 'DD') AS 반올림_일
    FROM dual;

→ 실행 시간이 정오 기준으로 날짜가 달라질 수 있음.

---

# TRUNC(날짜, 형식)

**기능:** 날짜를 **절삭(버림)**  
**형식:** `TRUNC(날짜, '형식')`

| 형식 | 의미 | 결과 예 |
|--------|--------|-----------|
| 'DD' | 오늘 00:00:00 | TRUNC(SYSDATE,'DD') |
| 'MM' | 해당 달 1일 00:00 | TRUNC(SYSDATE,'MM') |
| 'YYYY' | 해당 해 1월1일 00:00 | TRUNC(SYSDATE,'YYYY') |

**왜 TRUNC를 많이 쓰나?**  
날짜에 시간이 들어있으면 `sysdate - hdate` 는 **0.9342** 같은 소수가 나옴.  
→ 둘 다 `TRUNC` 하면 정확한 "일수"만 계산 가능.

### 예제 2 — '문시헌' 근무 일수 계산

    SELECT ename,
           TRUNC(SYSDATE) - TRUNC(hdate) + 1 AS 근무일수
    FROM emp
    WHERE ename = '문시헌';

- TRUNC(SYSDATE) = 오늘 00:00  
- TRUNC(hdate) = 입사일 00:00  
- +1 = 입사 첫날 포함

---

# MONTHS_BETWEEN(d1, d2)

**기능:** 두 날짜의 **개월 수 차이(실수)**  
**형식:** `MONTHS_BETWEEN(d1, d2)`  
**주의:** d1 > d2 → 양수 / d1 < d2 → 음수

### 예제 3

    SELECT MONTHS_BETWEEN(DATE '2013-09-01', DATE '2013-01-01') AS 개월수
    FROM dual;

→ 결과: 8

### 4.1 부서 20 직원들의 근무 개월수

    SELECT eno,
           ename,
           TRUNC(MONTHS_BETWEEN(SYSDATE, hdate)) AS 근무_개월
    FROM emp
    WHERE dno = '20';

→ 소수점 제거하려고 TRUNC 사용.

---

# LAST_DAY(날짜)

**기능:** 해당 날짜가 속한 **달의 마지막 날**

### 예제 4

    SELECT LAST_DAY(DATE '2013-09-24') AS 마지막날
    FROM dual;

### 5.1 입사한 달의 마지막 날 + 그 달 근무일수

    SELECT eno,
           ename,
           hdate,
           LAST_DAY(hdate) AS 마지막날,
           LAST_DAY(TRUNC(hdate)) - TRUNC(hdate) + 1 AS 마지막달_근무일수
    FROM emp
    WHERE dno = '20';

- TRUNC(hdate) : 입사일 00:00  
- LAST_DAY(TRUNC(hdate)) : 해당 달 마지막  
- 마지막 - 처음 + 1 → 입사한 날 포함

---

# ADD_MONTHS(날짜, n)

**기능:** 날짜에 **n개월을 더함**  
예: `ADD_MONTHS('2011/07/01', 23)` → 2013/06/01

### 예제 5 — 입사 100일 + 입사 10년

    SELECT eno,
           ename,
           hdate AS 입사일,
           hdate + 99 AS "100일",
           ADD_MONTHS(hdate, 120) AS "10년"
    FROM emp
    WHERE dno = '20';

- 날짜 + 숫자 = "일"단위 증가  
- 120개월 = 10년

---

# NEXT_DAY(날짜, 요일)

**기능:** 기준 날짜 이후 처음 나오는 요일  
**형식:** `NEXT_DAY(날짜, '요일')`

한국어 세션: `'일요일'`  
영문 세션: `'SUNDAY'`

### 예제 6

    SELECT eno,
           ename,
           hdate,
           NEXT_DAY(hdate, '일요일') AS sunday
    FROM emp
    WHERE dno = '20';

---

# 날짜 연산 기본 정리

- 날짜 + 숫자 → 숫자만큼 **일(day)** 증가  
- 날짜 - 숫자 → 숫자만큼 **일(day)** 감소  
- 날짜1 - 날짜2 → **일수** (실수 가능)  
- 시간 때문에 오차 발생 시 → **TRUNC 양쪽 적용**

왜 TRUNC를 반드시 쓰나?  
→ 실제 데이터는 "09:13:54" 같은 시간 포함.  
→ 시험·과제는 "날짜만" 계산.  
→ TRUNC로 정규화하면 실수 0%.

---

# 날짜 함수 요약 표

| 함수명 | 기능 | 예시 | 결과 |
|--------|--------|--------|--------|
| ROUND | 날짜 반올림 | ROUND(SYSDATE,'DD') | 가까운 날짜 |
| TRUNC | 날짜 절삭 | TRUNC(SYSDATE,'YYYY') | 해당 해 1월 1일 |
| MONTHS_BETWEEN | 개월 수 차이 | MONTHS_BETWEEN(d1,d2) | 8 등 |
| LAST_DAY | 달의 마지막 날 | LAST_DAY('2013-09-24') | 2013-09-30 |
| ADD_MONTHS | n개월 후 날짜 | ADD_MONTHS(hdate,120) | 입사 10년 |
| NEXT_DAY | 다음 요일 날짜 | NEXT_DAY(hdate,'일요일') | 첫 일요일 |

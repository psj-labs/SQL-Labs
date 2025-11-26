## 단일 행 함수란?

- 한 행(row)을 입력하면 한 행을 그대로 반환하는 함수  
- `SELECT`, `WHERE`, `HAVING` 등 대부분의 절에서 사용 가능  
- 인수로 컬럼, 상수, 표현식, 다른 함수의 결과 사용 가능  
- 종류: **숫자 함수 / 날짜 함수 / 문자 함수 / 문자 치환 함수**

---

# 숫자 함수

숫자를 반올림/절삭하거나 나머지/거듭제곱을 구할 때 사용.

| 함수명 | 형식 | 설명 | 예시 |
|-------|------|------|------|
| ROUND | ROUND(m, n) | 소수점 이하 **반올림** | ROUND(123.4567, 3) → **123.457** |
| TRUNC | TRUNC(m, n) | 소수점 이하 **절삭** | TRUNC(123.4567, 3) → **123.456** |
| MOD | MOD(m, n) | 나머지 구함 | MOD(10, 4) → **2** |
| POWER | POWER(m, n) | mⁿ 계산 | POWER(2, 4) → **16** |
| CEIL | CEIL(m) | m보다 크거나 같은 가장 작은 정수 | CEIL(2.34) → **3** |
| FLOOR | FLOOR(m) | m보다 작거나 같은 가장 큰 정수 | FLOOR(2.34) → **2** |
| SQRT | SQRT(m) | 제곱근 | SQRT(9) → **3** |
| SIGN | SIGN(m) | 부호 반환 | SIGN(-3) → **-1** |

### 숫자 함수 예제

    -- (1) 급여를 둘째 자리에서 반올림
    SELECT eno, ename, ROUND(sal, 2) AS sal_round
    FROM emp;

    -- (2) 급여를 천원 단위로 절삭
    SELECT eno, ename, TRUNC(sal, -3) AS sal_trunc
    FROM emp;

    -- (3) 나머지 예제
    SELECT MOD(10,4) AS mod_example
    FROM dual;

---

# 날짜 연산 & 날짜 함수

Oracle의 날짜는 **날짜 + 시/분/초**까지 저장됨.  
따라서 날짜끼리 연산하면 “일수 차이”가 나온다.

### 기본 개념

- 날짜 + 숫자 → 숫자만큼 **일수 증가**
- 날짜 - 숫자 → 숫자만큼 **일수 감소**
- 날짜 - 날짜 → **두 날짜 사이의 일수**
- 비교/연산 시 시간 제거하려면 → `TRUNC(날짜)`

### 2.1 날짜 연산표

| 연산 | 결과 | 의미 |
|------|------|------|
| 날짜 + m | 날짜 | m일 후 |
| 날짜 - m | 날짜 | m일 전 |
| 날짜 - 날짜 | 숫자 | 두 날짜 간 일수 |

### 2.2 TRUNC를 이용한 날짜 비교 (수업 예)

    SELECT eno,
           ename,
           sysdate AS 오늘,
           hdate   AS 입사일,
           TRUNC(sysdate) - TRUNC(hdate) + 1 AS 근무일수,
           hdate + 99 AS "100일"
    FROM emp;

- `TRUNC(sysdate)` → 오늘 00:00:00  
- `TRUNC(hdate)` → 입사일 00:00:00  
- 근무일수 계산 정확도를 위해 시간 제거  
- `hdate + 99` → 입사 후 100일째 날짜

### TRUNC를 ROUND보다 더 많이 쓰는 이유

ROUND는 **12:00 기준 반올림**이라 날짜가 하루 밀릴 수 있음.  
단순히 "00시로 맞추는 것"이 목적이면 **TRUNC가 정석**.

---

# 문자 함수

문자열을 대문자/소문자로 변환하거나 일부 추출하거나 길이를 측정하는 기능.

| 함수명   |  형식  |   설명  |   예시  |
|--------|-------|--------|--------|
| LOWER | LOWER(str) | 소문자 변환 | LOWER('ORACLE') → **oracle** |
| UPPER | UPPER(str) | 대문자 변환 | UPPER('oracle') → **ORACLE** |
| INITCAP | INITCAP(str) | 첫 글자만 대문자 | INITCAP('oracle') → **Oracle** |
| SUBSTR | SUBSTR(str, pos, len) | 문자열 일부 추출 | SUBSTR('oracle',1,2) → **or** |
| INSTR | INSTR(str, substring) | 특정 문자 위치 | INSTR('ORACLE','A') → **3** |
| TRIM | TRIM(char FROM str) | 앞뒤 특정 문자 제거 | TRIM('o' FROM 'oracle') → **racle** |
| LENGTH | LENGTH(str) | 글자 수 | LENGTH('디비') → **2** |
| LENGTHB | LENGTHB(str) | 바이트 수 | LENGTHB('디비') → **6** (환경마다 다름) |
| LPAD/RPAD | LPAD(str, n, fill) | 왼/오른쪽 채우기 | LPAD('20000',10,'*') → *****20000 |

### 3.1 문자 검색 예제

    SELECT dname,
           SUBSTR(dname, 1, LENGTH(dname) - 1) AS dname_removed_last
    FROM dept;

→ 마지막 글자를 제외한 문자열 추출.

### 3.2 이름이 네 글자인 사원 검색

    SELECT eno, ename
    FROM emp
    WHERE LENGTH(ename) = 4;

---

# 문자 치환 함수 (TRANSLATE vs REPLACE)

두 함수 모두 문자 교체 기능이지만 **동작 방식이 다름**.

---

## 4.1 TRANSLATE

- 형식: `TRANSLATE(str, search, replace)`
- **한 글자씩 1:1 대응**  
- search의 첫 번째 글자는 replace의 첫 글자로 치환

예:

    SELECT TRANSLATE('oracle', 'a', '#')
    FROM dual;

→ **or#cle**

---

## 4.2 REPLACE

- 형식: `REPLACE(str, search, replace)`
- **문자열 단위로 교체**

예:

    SELECT REPLACE('oracle', 'or', '##')
    FROM dual;

→ **##acle**

---

## 4.3 두 함수 비교 예제

    SELECT TRANSLATE('World of Warcraft', 'Wo', '--') AS tr_res,
           REPLACE  ('World of Warcraft', 'Wo', '--') AS rp_res
    FROM dual;

### 차이 정리

- TRANSLATE → **W→- , o→-** 이런 식의 "1글자씩" 전부 치환  
- REPLACE → **"Wo"라는 문자열**이 보일 때만 "--"로 교체

---

# 핵심 요약

- 반올림 → **ROUND**, 절삭 → **TRUNC**  
- 날짜–날짜 = **일수 차이**  
- 시간 필요 없으면 항상 **TRUNC(날짜)**  
- 문자열 일부 추출할 때 **SUBSTR + LENGTH** 조합 필수  
- 치환 시  
  - 글자 단위 → **TRANSLATE**  
  - 문자열 단위 → **REPLACE**

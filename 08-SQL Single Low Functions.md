# Single Low Functions

## 개요

- 단일 행 함수는 한 행(row)을 입력하면 한 행을 그대로 반환하는 함수입니다.
- SELECT, WHERE, HAVING 등 대부분의 절에서 사용 가능합니다.
- 인수로 컬럼, 상수, 표현식, 다른 함수의 결과를 사용할 수 있습니다.
- 종류는 숫자 함수, 날짜 함수, 문자 함수, 문자 치환 함수로 구분됩니다.

---

## 숫자 함수

- 숫자를 반올림하거나 절삭하고, 나머지 및 거듭제곱을 계산할 때 사용합니다.

| 함수명 | 형식 | 설명 | 예시 |
|-------|------|------|------|
| ROUND | ROUND(m, n) | 소수점 이하 반올림 | ROUND(123.4567, 3) → 123.457 |
| TRUNC | TRUNC(m, n) | 소수점 이하 절삭 | TRUNC(123.4567, 3) → 123.456 |
| MOD | MOD(m, n) | 나머지 계산 | MOD(10, 4) → 2 |
| POWER | POWER(m, n) | m의 n제곱 | POWER(2, 4) → 16 |
| CEIL | CEIL(m) | m보다 크거나 같은 최소 정수 | CEIL(2.34) → 3 |
| FLOOR | FLOOR(m) | m보다 작거나 같은 최대 정수 | FLOOR(2.34) → 2 |
| SQRT | SQRT(m) | 제곱근 | SQRT(9) → 3 |
| SIGN | SIGN(m) | 부호 반환 | SIGN(-3) → -1 |

### 숫자 함수 예제
```sql
SELECT eno, ename, ROUND(sal, 2) AS sal_round  
FROM emp;
```
```sql
SELECT eno, ename, TRUNC(sal, -3) AS sal_trunc  
FROM emp;
```
```sql
SELECT MOD(10,4) AS mod_example  
FROM dual;
```
---

## 날짜 연산 및 날짜 함수

- oracle의 날짜는 날짜와 시, 분, 초까지 함께 저장됩니다.
- 날짜 간 연산 결과는 일수 단위로 계산됩니다.

### 기본 개념

- 날짜 + 숫자 -> 해당 숫자만큼 일수 증가합니다.
- 날짜 - 숫자 -> 해당 숫자만큼 일수 감소합니다.
- 날짜 - 날짜 -> 두 날짜 사이의 일수 차이를 반환합니다.
- 시간 제거가 필요할 경우 `TRUNC` 함수를 사용합니다.

### 날짜 연산표

| 연산 | 결과 | 의미 |
|------|------|------|
| 날짜 + m | 날짜 | m일 후 |
| 날짜 - m | 날짜 | m일 전 |
| 날짜 - 날짜 | 숫자 | 두 날짜 간 일수 |

### TRUNC를 이용한 날짜 비교 예제
```sql
SELECT eno,  
       ename,  
       sysdate AS 오늘,  
       hdate   AS 입사일,  
       TRUNC(sysdate) - TRUNC(hdate) + 1 AS 근무일수,  
       hdate + 99 AS 100일  
FROM emp;
```
- TRUNC(sysdate)는 오늘 날짜를 00시 기준으로 변환합니다.
- TRUNC(hdate)는 입사일을 00시 기준으로 변환합니다.
- 근무일수 계산 시 시간 제거로 정확도를 확보합니다.
- hdate + 99는 입사 후 100일째 날짜를 의미합니다.

### TRUNC를 사용하는 이유

- ROUND는 12시 기준 반올림으로 날짜가 하루 밀릴 수 있습니다.
- 날짜를 단순히 00시로 맞출 목적이라면 TRUNC 사용이 적합합니다.

---

## 문자 함수

- 문자열을 변환하거나 일부를 추출하고 길이를 확인할 때 사용합니다.

| 함수명 | 형식 | 설명 | 예시 |
|--------|------|------|------|
| LOWER | LOWER(str) | 소문자 변환 | LOWER('ORACLE') → oracle |
| UPPER | UPPER(str) | 대문자 변환 | UPPER('oracle') → ORACLE |
| INITCAP | INITCAP(str) | 첫 글자만 대문자 | INITCAP('oracle') → Oracle |
| SUBSTR | SUBSTR(str, pos, len) | 문자열 일부 추출 | SUBSTR('oracle',1,2) → or |
| INSTR | INSTR(str, substring) | 특정 문자 위치 | INSTR('ORACLE','A') → 3 |
| TRIM | TRIM(char FROM str) | 앞뒤 문자 제거 | TRIM('o' FROM 'oracle') → racle |
| LENGTH | LENGTH(str) | 글자 수 | LENGTH('디비') → 2 |
| LENGTHB | LENGTHB(str) | 바이트 수 | LENGTHB('디비') → 6 |
| LPAD/RPAD | LPAD(str, n, fill) | 좌우 문자 채우기 | LPAD('20000',10,'*') → *****20000 |

### 문자 검색 예제
```sql
SELECT dname,  
       SUBSTR(dname, 1, LENGTH(dname) - 1) AS dname_removed_last  
FROM dept;
```
- 문자열의 마지막 글자를 제외하고 출력합니다.

### 이름이 네 글자인 사원 검색
```sql
SELECT eno, ename  
FROM emp  
WHERE LENGTH(ename) = 4;
```
---

## 문자 치환 함수

- 문자 치환 함수에는 TRANSLATE와 REPLACE가 있습니다.
- 두 함수는 동작 방식이 다릅니다.

### TRANSLATE

- 형식: TRANSLATE(str, search, replace)
- 문자 단위로 1:1 치환됩니다.
```sql
SELECT TRANSLATE('oracle', 'a', '#')  
FROM dual;
```
- 결과는 or#cle 입니다.

### REPLACE

- 형식: REPLACE(str, search, replace)
- 문자열 단위로 치환됩니다.
```sql
SELECT REPLACE('oracle', 'or', '##')  
FROM dual;
```
- 결과는 ##acle 입니다.

### 두 함수 비교 예제
```sql
SELECT TRANSLATE('World of Warcraft', 'Wo', '--') AS tr_res,  
       REPLACE('World of Warcraft', 'Wo', '--') AS rp_res  
FROM dual;
```
- TRANSLATE는 W와 o를 각각 치환합니다.
- REPLACE는 Wo 문자열이 존재할 때만 치환합니다.

---

## 정리

- 반올림은 ROUND를 사용합니다.
- 절삭은 TRUNC를 사용합니다.
- 날짜 간 차이는 일수로 계산됩니다.
- 시간 제거가 필요할 경우 TRUNC를 사용합니다.
- 문자열 추출은 SUBSTR와 LENGTH를 조합합니다.
- 문자 치환 시 글자 단위는 TRANSLATE를 사용합니다.
- 문자열 단위 치환은 REPLACE를 사용합니다.

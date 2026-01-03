# Data Format

## 개요

- 본 문서는 Oracle SQL에서 날짜/시간과 숫자를 문자열로 변환하거나, 문자열을 날짜/숫자로 변환할 때 사용하는 형식 모델(Format Model)을 정리한 문서입니다.
- TO_CHAR, TO_DATE, TO_NUMBER 사용법을 설명합니다.

---

## 날짜·시간 형식 (Format Model)

- Oracle은 날짜를 문자로 변환할 때(TO_CHAR) 또는 문자를 날짜로 변환할 때(TO_DATE) 형식 문자열을 사용합니다.

### 연도(Year)

| 형식 | 의미 |
|------|------|
| YYYY | 4자리 연도 |
| YY | 2자리 연도 |
| RR | 2자리 연도 (1950~2049 자동 보정) |

### 월(Month)

| 형식 | 의미 |
|------|------|
| MM | 월 숫자 (01~12) |
| MONTH | 월 이름 전체 |
| MON | 월 이름 축약 |

### 일 / 요일

| 형식 | 의미 |
|------|------|
| DD | 일 |
| DAY | 요일 전체 |
| DY | 요일 축약 |

### 시 / 분 / 초

| 형식 | 의미 |
|------|------|
| HH24 | 24시간 |
| HH | 12시간 |
| MI | 분 |
| SS | 초 |
| SSSSS | 하루를 초로 표현 |
| AM / PM | 오전 / 오후 |

---

## TO_CHAR 날짜 출력

- 날짜를 화면에 출력할 때는 반드시 TO_CHAR로 변환합니다.

### 예제: 오늘 날짜 출력
```sql
SELECT TO_CHAR(SYSDATE, 'YYYY/MM/DD') AS 날짜1,
       TO_CHAR(SYSDATE, 'YYYY/MM/DD:HH24:MI:SS') AS 날짜2,
       TO_CHAR(SYSDATE, 'YY/MM/DD:HH:MI:SS AM') AS 날짜3
FROM dual;
```
---

### 예제: 요일까지 출력
```sql
SELECT TO_CHAR(SYSDATE, 'DAY Mon YY') AS 오늘
FROM dual;
```
---

### 예제: 문장 형태 출력
```sql
SELECT ename || ' 사원의 입사일은 '
       || TO_CHAR(hdate, 'YYYY"년" MM"월" DD"일" 입니다."') AS 입사일문장
FROM emp;
```
---

## TO_DATE 날짜 변환

- 문자열 날짜는 반드시 TO_DATE로 변환해야 비교 가능
```sql
SELECT eno,
       ename,
       hdate
FROM emp
WHERE hdate < TO_DATE('19920101', 'YYYYMMDD');
```
---

### 시분초 포함 비교
```sql
SELECT eno,
       ename,
       hdate
FROM emp
WHERE hdate <= TO_DATE('19911231:235959', 'YYYYMMDD:HH24MISS');
```
---

## 숫자 형식 (Mask)

| 형식 | 의미 |
|------|------|
| 9 | 숫자 자리 |
| 0 | 빈 자리 0 |
| $ | 달러 기호 |
| L | 로컬 화폐 |
| , | 천 단위 |
| . | 소수점 |
| MI | 음수 표시 |
| EEEE | 지수 표기 |

---

## TO_CHAR 숫자 출력
```sql
SELECT TO_CHAR(12345.678, '999,999.99999') AS num
FROM dual;
```
```sql
SELECT TO_CHAR(12345.678, '099,999.999') AS num
FROM dual;
```
```sql
SELECT TO_CHAR(1234, '$999,999') AS num
FROM dual;
```
```sql
SELECT TO_CHAR(1234, 'L999,999') AS num
FROM dual;
```
```sql
SELECT TO_CHAR(123456789, '9.999EEEE') AS num
FROM dual;
```
```sql
SELECT TO_CHAR(-1234, '999,999MI') AS num
FROM dual;
```
---

### 예시
```sql
SELECT eno,
       ename,
       TO_CHAR(NVL(comm,0)/(sal*12)*100, '90.99') || '%' AS 급여_비율
FROM emp
WHERE dno = '10';
```
```sql
SELECT eno,
       ename,
       ROUND(NVL(comm,0)/(sal*12)*100, 2) || '%' AS 급여_비율
FROM emp
WHERE dno = '10';
```
---

## TO_NUMBER
```sql
SELECT TO_NUMBER('1,234', '9,999') + 100 AS result
FROM dual;
```
---

## 정리

- 문자열 날짜는 반드시 TO_DATE 사용
- 출력용 날짜/숫자는 TO_CHAR 사용
- 날짜 비교 시 시분초 주의 -> TRUNC 활용
- 자리수 부족 시 ##### 출력됨

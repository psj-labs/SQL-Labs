# SQL 날짜 함수 정리

## 개요

- 본 문서는 Oracle SQL에서 날짜 포맷 기호와 날짜 함수 사용법을 정리한 문서입니다.
- 날짜 반올림, 절삭, 개월 계산, 월의 마지막 날 계산, 특정 요일 계산에 대해 설명합니다.

---

## 날짜 포맷 기호

- 날짜 포맷 기호는 TO_CHAR, TO_DATE, TRUNC, ROUND 함수에서 사용됩니다.

| 기호 | 의미 |
|------|------|
| YYYY | 4자리 연도 |
| MM | 월(01~12) |
| DD | 일(01~31) |
| HH24 | 24시간 형식 시 |
| HH | 12시간 형식 시 |
| MI | 분 |
| SS | 초 |

---

## ROUND(날짜, 형식)

- ROUND 함수는 날짜를 지정한 기준으로 반올림합니다.
- 기준 시점에 따라 날짜가 다음 단위로 넘어갈 수 있습니다.

### `형식`
```sql
ROUND(날짜, 형식)
```
| 형식 | 의미 | 설명 |
|------|------|------|
| DD | 일 단위 반올림 | 12시 이후면 다음 날 |
| MM | 월 단위 반올림 | 16일 이후면 다음 달 |
| YYYY | 년 단위 반올림 | 7월 이후면 다음 해 |

`예시`  
```sql
SELECT ROUND(SYSDATE, 'DD') AS 반올림_일
FROM dual;
```
- 실행 시각이 정오 이전인지 이후인지에 따라 결과가 달라질 수 있습니다.

---

## TRUNC(날짜, 형식)

- TRUNC 함수는 날짜를 절삭하여 기준 단위의 시작 시점으로 만듭니다.
- 시간 정보를 제거할 때 가장 많이 사용됩니다.

### `형식`
```sql
TRUNC(날짜, 형식)
```
| 형식 | 의미 | 결과 |
|------|------|------|
| DD | 해당 날짜 00:00:00 | 오늘 시작 |
| MM | 해당 달 1일 00:00 | 월 시작 |
| YYYY | 해당 해 1월 1일 00:00 | 연 시작 |

### `예시 ` 
```sql
SELECT eno,
       ename,
       SYSDATE AS 오늘,
       hdate AS 입사일,
       TRUNC(SYSDATE) - TRUNC(hdate) + 1 AS 근무일수,
       hdate + 99 AS 백일
FROM emp;
```
- TRUNC(SYSDATE)는 오늘 00시 기준입니다.
- TRUNC(hdate)는 입사일 00시 기준입니다.
- 시간으로 인한 오차를 제거하기 위해 TRUNC를 사용합니다.

### ROUND보다 TRUNC를 많이 사용하는 이유  
- ROUND는 12시 기준으로 반올림되어 날짜가 바뀔 수 있습니다.
- 날짜 비교 목적일 경우 TRUNC가 가장 안전합니다.

---

## MONTHS_BETWEEN(d1, d2)

- 두 날짜 사이의 개월 수 차이를 실수 형태로 반환합니다.
- 기준 날짜의 크기에 따라 양수 또는 음수가 반환됩니다.

### `형식`
```sql 
MONTHS_BETWEEN(d1, d2)
```
### `예시`  
```sql
SELECT MONTHS_BETWEEN(DATE '2013-09-01', DATE '2013-01-01') AS 개월수
FROM dual;
```
### `예시 결과`  
```text
8
```
### 부서 20 직원의 근무 개월 수 계산  
```sql
SELECT eno,
       ename,
       TRUNC(MONTHS_BETWEEN(SYSDATE, hdate)) AS 근무_개월
FROM emp
WHERE dno = '20';
```
- 소수점 제거를 위해 `TRUNC`를 사용합니다.

---

## LAST_DAY(날짜)

- 지정한 날짜가 속한 달의 마지막 날짜를 반환합니다.

### `예시`  
```sql
SELECT LAST_DAY(DATE '2013-09-24') AS 마지막날
FROM dual;
```
### 입사한 달의 마지막 날과 근무 일수 계산  
```sql
SELECT eno,
       ename,
       hdate,
       LAST_DAY(hdate) AS 마지막날,
       LAST_DAY(TRUNC(hdate)) - TRUNC(hdate) + 1 AS 마지막달_근무일수
FROM emp
WHERE dno = '20';
```
- 입사일을 포함한 근무 일수를 계산합니다.

---

## ADD_MONTHS(날짜, n)

- 지정한 날짜에 n개월을 더한 날짜를 반환합니다.
- 날짜에 숫자를 더하면 일 단위 증가가 됩니다.

### `예시`  
```sql
SELECT eno,
       ename,
       hdate AS 입사일,
       hdate + 99 AS 백일,
       ADD_MONTHS(hdate, 120) AS 십년
FROM emp
WHERE dno = '20';
```
- 120개월은 10년입니다.

---

## NEXT_DAY(날짜, 요일)

- 기준 날짜 이후 처음 도래하는 지정 요일의 날짜를 반환합니다.
- 세션 언어 설정에 따라 요일 문자열이 달라집니다.

### `형식`
```text
NEXT_DAY(날짜, 요일)
```
### `예시`  
```sql
SELECT eno,
       ename,
       hdate,
       NEXT_DAY(hdate, '일요일') AS 다음_일요일
FROM emp
WHERE dno = '20';
```
---

## 날짜 연산 기본 정리

- 날짜 + 숫자 = 숫자만큼 일 증가합니다.
- 날짜 - 숫자 = 숫자만큼 일 감소합니다.
- 날짜 - 날짜 = 일수 차이입니다.
- 시간으로 인한 오차가 발생하면 양쪽 모두 TRUNC를 적용합니다.

TRUNC를 반드시 사용하는 이유  
- 실제 날짜 데이터에는 시분초가 포함됩니다.
- 날짜만 비교해야 하는 경우 `TRUNC`로 정규화해야 합니다.

---

## 정리

- 날짜 반올림은 `ROUND`를 사용합니다.
- 날짜 절삭과 비교는 `TRUNC`가 기준입니다.
- 개월 수 계산은 `MONTHS_BETWEEN`을 사용합니다.
- 월의 마지막 날짜는 `LAST_DAY`로 구합니다.
- n개월 이후 날짜는 `ADD_MONTHS`로 계산합니다.
- 특정 요일 계산은 `NEXT_DAY`를 사용합니다.

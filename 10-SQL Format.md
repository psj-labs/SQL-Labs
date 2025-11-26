## 날짜·시간 형식 (Format Model)

오라클은 날짜를 문자로 바꿀 때(`TO_CHAR`) 또는 문자를 날짜로 바꿀 때(`TO_DATE`) **형식 문자열(Format Model)** 을 사용한다.

### 연도(Year)

| 형식 | 의미 |
|------|------|
| `YYYY` | 4자리 연도 (2021, 1999) |
| `YY` | 2자리 연도 (21 → 2021) |
| `RR` | 2자리 연도. 1950~2049 기준으로 자동 보정. 구형 시스템 호환용 |

### 월(Month)

| 형식 | 의미 |
|------|------|
| `MM` | 월 숫자 (01~12) |
| `MONTH` | 월 이름 전체 (예: MARCH, 3월) |
| `MON` | 월 이름 축약 (MAR, 3월) |

### 일/요일

| 형식 | 의미 |
|------|------|
| `DD` | 날짜 (01~31) |
| `DAY` | 요일 전체 (SUNDAY, 일요일) |
| `DY` | 요일 축약 (SUN, 월) |

### 시/분/초

| 형식 | 의미 |
|------|------|
| `HH24` | 24시간제 시 |
| `HH` | 12시간제 시 |
| `MI` | 분 |
| `SS` | 초 |
| `SSSSS` | 하루를 초로 표현 (0~86399) |
| `AM / PM` | 오전/오후 |

**중요 포인트:**  
날짜를 화면에 예쁘게 출력하고 싶다면 **반드시 `TO_CHAR` 로 변환해서 출력**해야 한다.

---

## TO_CHAR 날짜 출력 예제

### 예제 — 오늘 날짜 여러 형식으로 출력

```sql
SELECT TO_CHAR(SYSDATE, 'YYYY/MM/DD')               AS 날짜1,
       TO_CHAR(SYSDATE, 'YYYY/MM/DD:HH24:MI:SS')    AS 날짜2,
       TO_CHAR(SYSDATE, 'YY/MM/DD:HH:MI:SS AM')     AS 날짜3
FROM   dual;
```

예시 결과:

| 날짜1 | 날짜2 | 날짜3 |
|-------|--------|--------|
| 2025/10/31 | 2025/10/31:15:52:10 | 25/10/31:03:52:10 오후 |

---

### 예제 — 요일까지 출력

```sql
SELECT TO_CHAR(SYSDATE, 'DAY Mon YY') AS 오늘
FROM   dual;
```

예시:

| 오늘 |
|------|
| FRIDAY Oct 25 |

---

### 예제 — 문장 형태로 날짜 출력

```sql
SELECT ename || ' 사원의 입사일은 '
       || TO_CHAR(hdate, 'YYYY"년" MM"월" DD"일" 입니다."') AS 입사일문장
FROM   emp;
```

예시:

| 입사일문장 |
|-------------|
| SCOTT 사원의 입사일은 2021년 09월 25일 입니다. |

포인트:  
`" "` 로 감싸면 그대로 출력됨 → `'YYYY"년" MM"월" DD"일"'`

---

## TO_DATE를 이용한 날짜 검색

문자열로 전달된 날짜는 반드시:

```
TO_DATE('문자열', '형식')
```

### 예제 — 1992년 이전 입사자 검색

```sql
SELECT eno AS 사번,
       ename AS 이름,
       hdate AS 입사일
FROM   emp
WHERE  hdate < TO_DATE('19920101', 'YYYYMMDD');
```

예시:

| 사번 | 이름 | 입사일 |
|------|------|---------|
| 7369 | SMITH | 1991-12-17 |
| 7566 | JONES | 1990-04-02 |

---

### 예제 — 1991년 12월 31일 23:59:59까지 포함 검색

```sql
SELECT eno AS 사번,
       ename AS 이름,
       hdate AS 입사일
FROM   emp
WHERE  hdate <= TO_DATE('19911231:235959', 'YYYYMMDD:HH24MISS');
```

---

**핵심 요약**

- 날짜 비교는 양쪽 모두 DATE 타입이어야 함  
- 문자열 날짜는 반드시 `TO_DATE()`  
- 범위 검색은 BETWEEN 또는 TRUNC 사용

---

## 숫자 형식(Mask)

`TO_CHAR(숫자, '형식')` 에서 사용하는 형식 기호들

| 형식 | 의미 |
|------|------|
| `9` | 숫자 자리수 지정. 자리가 부족하면 ##### |
| `0` | 빈 자리를 0으로 채움 |
| `$` | 달러 기호 |
| `L` | 로컬 화폐 기호 (₩ 등) |
| `,` | 천 단위 구분 |
| `.` | 소수점 |
| `MI` | 음수일 때 기호를 오른쪽에 표시 |
| `EEEE` | 지수 표기 |

---

## TO_CHAR 숫자 출력 예제

### 예제

```sql
SELECT TO_CHAR(12345.678, '999,999.99999') AS num
FROM   dual;
```

결과: `12,345.67800`

---

### 예제 — 선행 0 채우기

```sql
SELECT TO_CHAR(12345.678, '099,999.999') AS num
FROM   dual;
```

→ `012,345.678`

---

### 예제 — 화폐 기호

```sql
SELECT TO_CHAR(1234, '$999,999') AS num
FROM   dual;
```

→ `$ 1,234`

---

### 예제 — 로컬 화폐

```sql
SELECT TO_CHAR(1234, 'L999,999') AS num
FROM   dual;
```

예시 → `₩1,234`

---

### 예제 — 지수 표기

```sql
SELECT TO_CHAR(123456789, '9.999EEEE') AS num
FROM   dual;
```

→ `1.235E+08`

---

### 예제 — 음수 표시

```sql
SELECT TO_CHAR(-1234, '999,999MI') AS num
FROM   dual;
```

→ `1,234-`

---

## 실전 예제 — 부서 10번 사원의 보너스 비율

### 예제 (TO_CHAR로 퍼센트 출력)

```sql
SELECT eno AS 사번,
       ename AS 이름,
       TO_CHAR( NVL(comm, 0) / (sal * 12) * 100, '90.99') || '%' AS 급여_비율
FROM   emp
WHERE  dno = '10';
```

---

### 예제 ROUND로 소수 둘째 자리)

```sql
SELECT eno AS 사번,
       ename AS 이름,
       ROUND( NVL(comm,0)/(sal*12) * 100, 2 ) || '%' AS 급여_비율
FROM   emp
WHERE  dno = '10';
```

---

## TO_NUMBER

사용빈도는 낮지만, 문자열을 숫자로 강제 변환할 때 사용한다.

### 예제 

```sql
SELECT TO_NUMBER('1,234', '9,999') + 100 AS result
FROM   dual;
```

→ `1334`

---

## 자주 하는 실수 정리

- 문자열 날짜를 WHERE 절에 직접 쓰기  
  → 반드시 `TO_DATE()`  
- 날짜 비교 시 시·분·초 때문에 검색 누락되는 경우  
  → `TRUNC(hdate)`, `BETWEEN`  
- 출력 폭 부족  
  → 자리수 작으면 `#####` 출력됨
※ 실습 결과는 오라클 버전 및 NLS 설정에 따라 조금씩 다를 수 있음.

| 함수 | 설명 |
|------|------|
| MAX(컬럼) | 최대값 |
| MIN(컬럼) | 최소값 |
| AVG(컬럼) | 평균 |
| SUM(컬럼) | 합계 |
| COUNT(컬럼 \| *) | NULL 제외 건수 / `*`는 전체 행 수 |
| STDDEV(컬럼) | 표준편차 |
| VARIANCE(컬럼) | 분산 |

**특징**
- 그룹 함수는 **NULL 값을 자동으로 제외**한다.

---

## 그룹 함수와 NULL

```sql
SELECT
    SUM(comm) AS 총액,
    TO_CHAR(AVG(comm),'999,999') AS 평균
FROM emp;
```

```sql
SELECT
    SUM(NVL(comm,0)) AS 총액,
    COUNT(*) AS 전체사원수,
    TO_CHAR(AVG(NVL(comm,0)),'999,999') AS 평균
FROM emp;
```

예시(comm)

| 사원 | comm |
|------|------|
| SCOTT | 300 |
| KING | NULL |
| JONES | 500 |

결과

| 계산식 | 값 |
|--------|------|
| SUM(comm) | 800 |
| AVG(comm) | 400 |
| AVG(NVL(comm,0)) | 266.6 |

---

## 그룹 함수 + 다중행 서브쿼리

```sql
SELECT eno AS 사번, ename AS 이름, dno AS 부서번호
FROM emp
WHERE sal > (
    SELECT MAX(sal)
    FROM emp
    WHERE dno = 10
);
```

또는

```sql
SELECT eno, ename, dno
FROM emp
WHERE sal > ALL (
    SELECT sal
    FROM emp
    WHERE dno = 10
);
```

---

## GROUP BY

동일한 값을 가진 행을 묶어서 요약 계산할 때 사용한다.

```sql
SELECT job, AVG(sal)
FROM emp
GROUP BY job
ORDER BY job;
```

### 규칙
- **SELECT에 일반 컬럼과 그룹 함수가 같이 나오면**
- 그 일반 컬럼은 반드시 **GROUP BY에 포함**해야 한다.
- 위반 시 오류:

```sql
ORA-00937: not a single-group group function
```

---

## 다중 컬럼 GROUP BY

```sql
SELECT d.dno AS 부서번호,
       d.dname AS 부서명,
       TO_CHAR(AVG(sal),'999,999') AS 평균급여,
       TO_CHAR(AVG(sal*12 + NVL(comm,0)),'999,999') AS 평균연봉
FROM dept d, emp e
WHERE d.dno = e.dno
GROUP BY d.dno, d.dname
ORDER BY d.dno;
```

---

## Cardinality(카디널리티)

**카디널리티 = 한 컬럼의 고유값 개수(DISTINCT 수)**

SELECT에 포함된 모든 컬럼의 카디널리티는  
- > 결과 행 개수가 서로 일치해야 한다.

### 잘못된 예
```sql
SELECT dname, AVG(sal)
FROM emp;
```

### 올바른 예
```sql
SELECT dname, AVG(sal)
FROM emp
GROUP BY dname;
```

**정렬**
- GROUP BY는 정렬 보장 X
- 정렬 필요 시 반드시 `ORDER BY` 사용

---

---

#HAVING 절

## HAVING이란

**GROUP BY로 집계된 결과에 조건을 거는 절**

### 실행 구조

```sql
SELECT 컬럼, 그룹함수
FROM 테이블
WHERE 행조건
GROUP BY 그룹기준
HAVING 그룹조건
ORDER BY 정렬;
```

- WHERE → 개별 행 필터
- HAVING → **집계 결과 필터**

---

## WHERE vs HAVING

| 구분 | WHERE | HAVING |
|------|--------|---------|
| 적용 시점 | GROUP BY 이전 | GROUP BY 이후 |
| 대상 | 개별 행 | 집계 그룹 |
| 그룹 함수 | 사용 불가 | 사용 가능 |

---

## HAVING 예제 ①

```sql
SELECT dno AS 부서번호,
       TO_CHAR(AVG(sal),'$999,999') AS 평균급여
FROM emp
GROUP BY dno
HAVING AVG(sal) < 3000;
```

---

## HAVING 예제 ②

```sql
SELECT dno AS 부서번호,
       MAX(sal) AS 최대급여
FROM emp
GROUP BY dno
HAVING AVG(sal) < 3000;
```

---

## WHERE + HAVING 함께 사용

```sql
SELECT dno AS 부서번호,
       TO_CHAR(AVG(sal),'999,999') AS 평균급여
FROM emp
WHERE dno != 20
GROUP BY dno
HAVING AVG(sal) >= 3000
ORDER BY dno;
```

- WHERE → 개별 행 필터
- HAVING → 집계 결과 필터

---

## 최종 요약

- 그룹 함수 → 여러 행 → 하나로 요약
- NULL 자동 제외 → 필요 시 NVL 사용
- GROUP BY → 동일값 묶음
- 일반 컬럼은 반드시 GROUP BY 포함
- 조건 처리:
  - 행 조건: WHERE
  - 집계 조건: HAVING
- 실행 순서:

FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY

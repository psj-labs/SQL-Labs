## 1\. CASE 표현식 표현식(Expression)

**IF / ELSE IF** 로 작성하던 조건문을 SQL 안에서 표현하는 방법입니다.

### 1-1. 단순 CASE(Simple CASE)

특정 컬럼의 값이 무엇이냐에 따라 결과를 나누는 형태입니다.

```
CASE 컬럼
    WHEN 값1 THEN 결과1
    WHEN 값2 THEN 결과2
    ELSE 결과기본
END
```

**예시)** 학번에 따라 등급 표시:

```
SELECT sno,
       CASE sno
           WHEN 1 THEN 'A'
           WHEN 2 THEN 'B'
           ELSE 'C'
       END AS grade
FROM student;
```

### 1-2. 검색 CASE(Searched CASE)

비교, 범위 등 **조건식 자체**를 사용할 수 있습니다.

```
CASE
    WHEN 조건식1 THEN 결과1
    WHEN 조건식2 THEN 결과2
    ELSE 결과기본
END
```

**예시)** 급여 구간에 따라 공제금액 계산:

```
SELECT eno, ename, sal,
       CASE
           WHEN sal <= 1000 THEN 0
           WHEN sal BETWEEN 1001 AND 2000 THEN ROUND(sal * 0.05)
           WHEN sal BETWEEN 2001 AND 3000 THEN ROUND(sal * 0.07)
           WHEN sal BETWEEN 3001 AND 4000 THEN ROUND(sal * 0.10)
           ELSE ROUND(sal * 0.15)
       END AS 공제금액
FROM emp;
```

특징 정리  
\- ELSE를 생략하면 조건에 안 걸리는 경우 **NULL** 반환  
\- 비교, 범위, 함수 호출 등 자유로운 조건식 가능  
\- SELECT, WHERE, ORDER BY 어디서든 사용 가능 (“표현식”이기 때문)

## 2\. DECODE 함수 함수(Function)

DECODE는 **Oracle 전용** 함수로, **\= (동일 비교)** 에 특화된 간단한 조건 분기입니다.

```
DECODE(표현식,
       값1, 결과1,
       값2, 결과2,
       ...,
       기본값)
```

**예시)** 업무별 인상 급여:

```
SELECT job, eno, ename, sal,
       DECODE(job,
              '모델링', sal * 1.2,
              '분석',   sal * 1.2,
                        sal * 1.1) AS 인상급여
FROM emp;
```

특징 정리  
\- 오직 **값 일치 (=)** 비교만 가능 (복잡한 조건 어렵다)  
\- 가독성이 떨어질 만큼 복잡해지면 **CASE 사용 권장**  
\- Oracle에서만 사용 가능 → 이식성 측면에서도 CASE가 더 좋음

## 3\. CASE vs DECODE 비교

| 구분 | CASE | DECODE |
| --- | --- | --- |
| 형태 | 표현식(Expression) | 함수(Function) |
| 조건 | 여러 조건식 사용 가능 (<, >, BETWEEN, IN 등) | \= 비교만 가능 |
| 가독성 | 복잡한 로직에 유리 | 간단한 매핑에 유리 |
| 이식성 | 표준 SQL → 타 DB에서도 사용 | Oracle 전용 |

시험/실무 팁: **"복잡한 조건 → CASE", "간단한 값 치환 → DECODE(Oracle 한정)"** 로 기억하면 된다.

## 4\. ROLLUP: 소계 + 합계를 자동으로

**ROLLUP**은 GROUP BY의 확장 기능으로, **그룹별 합계 + 상위 그룹 합계 + 전체 합계**를 한 번에 구해줍니다. (Oracle, 일부 DB에서 지원)

### 4-1. 기본 문법

```
SELECT 컬럼1, 컬럼2, 집계함수
FROM 테이블
GROUP BY ROLLUP(컬럼1, 컬럼2);
```

### 4-2. 동작 원리

`GROUP BY ROLLUP(a, b)` 는 아래 순서의 그룹을 생성합니다.

1.  (a, b) 기준 그룹
2.  (a) 기준 소계
3.  () 전체 합계

### 4-3. 예시

```
SELECT region, product, SUM(sales) AS total_sales
FROM sales
GROUP BY ROLLUP(region, product)
ORDER BY region, product;
```

결과 예시:

| region | product | total\_sales |
| --- | --- | --- |
| 부산 | A | 150 |
| 부산 | B | 250 |
| 부산 | **NULL** | **400** (부산 소계) |
| 서울 | A | 100 |
| 서울 | B | 200 |
| 서울 | **NULL** | **300** (서울 소계) |
| **NULL** | **NULL** | **700** (전체 합계) |

NULL 대신 “소계”, “총합”처럼 표시하고 싶다면:

```
SELECT NVL(region, '총합')     AS region,
       NVL(product, '소계')    AS product,
       SUM(sales)             AS total_sales
FROM sales
GROUP BY ROLLUP(region, product);
```

\- ROLLUP은 **컬럼 순서**에 따라 소계가 어디까지 계산되는지가 달라진다.  
\- ROLLUP: 계층 구조 요약 / CUBE: 모든 조합 요약 / GROUPING SETS: 원하는 조합만 직접 지정.

## 5\. 연관 서브쿼리(Correlated Subquery)

### 5-1. 일반 서브쿼리 vs 연관 서브쿼리

-   **일반 서브쿼리**: 서브쿼리가 먼저 한 번 실행 → 결과를 메인쿼리가 사용.
-   **연관 서브쿼리**: 메인쿼리의 각 행마다 서브쿼리가 실행됨.  
    메인 테이블의 컬럼을 서브쿼리에서 사용하기 때문에 행마다 결과가 달라질 수 있음.

### 5-2. 예시 1) 부서 평균 급여보다 많이 받는 사원

```
SELECT eno, ename, sal, dno
FROM emp e
WHERE sal > (
    SELECT AVG(sal)
    FROM emp
    WHERE dno = e.dno
);
```

\- 서브쿼리 안에서 `dno = e.dno` 처럼 **외부(메인)의 컬럼(e.dno)**을 사용 → 연관 서브쿼리.  
\- 부서별 평균 급여를 각 행 기준으로 다시 계산해 비교.

### 5-3. 예시 2) SELECT 절에서 사용하는 연관 서브쿼리

```
SELECT eno, ename, sal, dno,
       (SELECT ROUND(AVG(sal))
        FROM emp
        WHERE dno = e.dno) AS 부서별_평균급여
FROM emp e
ORDER BY dno;
```

## 6\. EXISTS / NOT EXISTS

### 6-1. EXISTS

서브쿼리 결과가 “행이 하나라도 존재하냐?”만 체크하는 연산자.

```
SELECT eno, ename, job, dno
FROM emp e
WHERE EXISTS (
    SELECT 1
    FROM emp
    WHERE mgr = e.eno
);
```

→ 부하직원이 존재하는 사원만 조회.

### 6-2. NOT EXISTS

서브쿼리 결과가 전혀 없을 때만 참.

```
SELECT dno, dname
FROM dept d
WHERE NOT EXISTS (
    SELECT 1
    FROM emp
    WHERE dno = d.dno
);
```

→ 아직 사원이 배치되지 않은 부서만 조회.


※ 일부 예제와 개념은 수업 교재 내용을 기반으로 재구성하였습니다. \[oai\_citation:0‡\[21차시\] 함수\_ DECODE, CASE, 연관 서브 쿼리.pdf\](sediment://file\_000000005884720b9f744e52298b8333)

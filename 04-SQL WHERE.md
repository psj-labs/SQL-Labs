# WHERE 절과 조건 검색 정리

## 개요

- 본 문서는 Oracle SQL에서 WHERE 절을 중심으로 조건 검색, 날짜 비교, NULL 처리, 범위 조건, 목록 조건을 정리한 문서입니다.

---

## SELECT 실행 논리 순서 (중요)

- SELECT 문은 다음 순서로 논리적으로 실행됩니다.
```text
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```
- 이 실행 순서 때문에 다음과 같은 제약이 발생합니다.
  - WHERE 절에서는 SELECT 별칭(alias)을 사용할 수 없습니다.
  - ORDER BY 절에서는 컬럼명, 별칭, 출력 순번 모두 사용할 수 있습니다.

---

## WHERE 기본 형태와 핵심 개념

- WHERE 절은 행(Row)을 제한하는 필터링 기능을 수행합니다.
- 비교 연산자와 논리 연산자를 조합하여 조건을 작성합니다.

### 사용 가능한 연산자
- 비교 연산자: =, >, >=, <, <=, <>
- 논리 연산자: AND, OR, NOT

- `예시: 급여가 3000 이상인 사원 조회`
```sql
    SELECT eno, ename, sal
    FROM emp
    WHERE sal >= 3000
    ORDER BY sal DESC;
```
- ORDER BY 팁
  - ORDER BY 1, 3 DESC 처럼 출력 컬럼 순번도 사용할 수 있습니다.
  - 가독성과 유지보수를 위해 컬럼명 또는 별칭 사용을 권장합니다.

---

## 날짜 출력 형식 (Oracle)

- 세션 단위로 날짜 출력 형식을 고정하면 결과가 일관됩니다.
```sql
    ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY/MM/DD';
```
- TO_DATE 변환 예시
```sql
    INSERT INTO emp_log (eno, hdate)
    VALUES ('E001', TO_DATE('2025/10/26', 'YYYY/MM/DD'));
```
- SYSDATE는 시분초까지 포함되어 저장됩니다.

### 주의 사항
- DATE 타입은 날짜 + 시분초를 포함합니다.
- 날짜만 비교할 경우 다음 방식 중 하나를 사용해야 합니다.
  - TRUNC 함수 사용
  - 반개구간 방식 (권장)

- TRUNC 예시
```sql
    SELECT *
    FROM orders
    WHERE TRUNC(order_date)
          BETWEEN DATE '2025-10-01' AND DATE '2025-10-31';
```
- 반개구간 예시
```sql
    SELECT *
    FROM orders
    WHERE order_date >= DATE '2025-10-01'
      AND order_date <  DATE '2025-11-01';
```
- 날짜는 반드시 0시로 저장돼야 한다는 규칙은 없습니다.
- 저장 방식보다 비교 로직이 더 중요합니다.

---

## NULL 비교

- NULL은 값이 없음을 의미합니다.
- 일반 비교 연산자로는 NULL을 비교할 수 없습니다.
- 반드시 IS NULL 또는 IS NOT NULL을 사용해야 합니다.

- `예시`
```sql
    SELECT *
    FROM emp
    WHERE comm IS NULL;
```
- `잘못된 예시`
```sql
    SELECT *
    FROM emp
    WHERE comm = NULL;
```
- 위 쿼리는 항상 결과가 없습니다.

COUNT 관련 주의
- COUNT(컬럼): NULL 제외
- COUNT(`*`): NULL 포함

---

## BETWEEN … AND (양끝 범위 포함)

- BETWEEN은 양쪽 값을 모두 포함하는 범위 조건입니다.
- 다음 조건과 동일합니다.
```text
    값1 <= 컬럼 <= 값2
```
- `숫자 예시`
```sql
    SELECT eno, ename, sal
    FROM emp
    WHERE sal BETWEEN 2000 AND 3000
    ORDER BY sal;
```
- `날짜 예시 (반개구간 권장)`
```sql
    SELECT *
    FROM orders
    WHERE order_date >= DATE '2025-10-01'
      AND order_date <  DATE '2025-11-01';
```
---

## IN 연산자

- IN은 목록 중 하나라도 일치하면 참이 됩니다.
- 여러 OR 조건을 나열하는 것보다 가독성과 유지보수성이 좋습니다.
- 서브쿼리와 함께 자주 사용됩니다.

- `예시: 부서가 10, 20, 30인 사원`
```sql
    SELECT eno, ename, dno
    FROM emp
    WHERE dno IN (10, 20, 30)
    ORDER BY dno, ename;
```
- `서브쿼리 예시`
```sql
    SELECT eno, ename
    FROM emp
    WHERE dno IN (
        SELECT dno
        FROM dept
        WHERE loc = 'SEOUL'
    );
```
### 주의 사항
- NOT IN과 NULL이 함께 사용되면 결과가 사라질 수 있습니다.
- 이 경우 NOT EXISTS 패턴을 권장합니다.

- `NOT EXISTS 예시`
```sql
    SELECT e.*
    FROM emp e
    WHERE NOT EXISTS (
        SELECT 1
        FROM blacklist b
        WHERE b.eno = e.eno
    );
```
---

## 자주 쓰는 조건식 패턴 모음

- 부분 문자열 검색
```sql
    WHERE ename LIKE '%KIM%'
```
- NULL 대체 후 비교
```sql
    WHERE NVL(comm, 0) >= 500
```
- 하루 구간 조회
```sql
    WHERE order_date >= DATE '2025-10-26'
      AND order_date <  DATE '2025-10-27'
```
- 별칭을 이용한 정렬
```sql
    SELECT sal * 12 AS annual
    FROM emp
    ORDER BY annual DESC
```
- 다중 OR 조건
```sql
    WHERE dno IN (10, 20, 30)
```
---

## 정리

- WHERE 절에는 SELECT 별칭을 사용할 수 없습니다.
- DATE 타입은 시분초를 포함하므로 비교 로직을 신중하게 작성해야 합니다.
- BETWEEN은 포함 범위입니다.
- NOT IN + NULL 조합은 위험하며 `NOT EXISTS`가 더 안전합니다.

# SQL 문자열 처리 및 정렬 정리

## 개요

- 본 문서는 Oracle SQL에서 문자열 처리, 중복 제거, 정렬, 묶음 검색 개념을 정리한 문서입니다.
- 자주 사용되는 연산자와 정렬 기준을 중심으로 설명합니다.

---

## 연결 연산자 (`||`) - 문자열 합치기

- 연결 연산자는 여러 문자열이나 컬럼을 이어붙여 하나의 문자열로 만드는 연산자입니다.
- 컬럼 값과 문자열 리터럴을 조합하여 가독성 있는 결과를 만들 수 있습니다.

- `예시`
```sql
    SELECT ename || '의 직업은 ' || job || ' 입니다.' AS 설명
    FROM emp;
```
- `예시 결과`
```sql
    SMITH의 직업은 CLERK 입니다.
    ALLEN의 직업은 SALESMAN 입니다.
    WARD의 직업은 SALESMAN 입니다.
```
### 주의 사항

- 문자열 리터럴은 반드시 작은따옴표를 사용합니다.
- 수식이 복잡한 경우 가독성을 위해 괄호로 묶어 사용하는 것이 좋습니다.

---

## 중복 제거 (DISTINCT)

- DISTINCT는 조회 결과에서 중복된 값을 제거할 때 사용합니다.

- `예시`
```sql
    SELECT DISTINCT job
    FROM emp;
```
- `예시 결과`
```sql
    CLERK
    SALESMAN
    MANAGER
    ANALYST
    PRESIDENT
```
### 정리

- `DISTINC`T : 중복 제거
- `ALL` : 중복 포함 (기본값)
- DISTINCT 사용 시 내부적으로 정렬이 발생하므로 성능 부담이 생길 수 있습니다.

---

## 정렬 (ORDER BY)

- ORDER BY 절은 조회 결과를 특정 기준에 따라 정렬할 때 사용합니다.

- `사용 형태`
```sql
    SELECT 컬럼명
    FROM 테이블명
    ORDER BY 컬럼명 ASC 또는 DESC;
```
### 특징

- `ASC` : 오름차순 정렬
- `DESC` : 내림차순 정렬
- 여러 개의 컬럼을 기준으로 정렬할 수 있습니다.
- Oracle DB에서 NULL 값은 가장 큰 값으로 취급됩니다.

- `예시`
```sql
    SELECT ename, sal
    FROM emp
    ORDER BY sal DESC;
```
- `예시 결과`
```sql
    KING   5000
    SCOTT  3000
    FORD   3000
    ALLEN  1600
```
---

## 묶음 검색 (업무별 / 부서별 정렬)

- `업무별 정렬 예제`
```sql
    SELECT job AS 업무,
           empno AS 사번,
           ename AS 이름,
           sal AS 급여
    FROM emp
    ORDER BY 업무;
```
- `부서별 급여 내림차순 정렬 예제`
```sql
    SELECT deptno AS 부서번호,
           empno AS 사번,
           ename AS 이름,
           sal AS 급여
    FROM emp
    ORDER BY 부서번호, sal DESC;
```
- 첫 번째 컬럼으로 그룹처럼 정렬한 뒤, 두 번째 컬럼으로 세부 정렬이 수행됩니다.

---

## 카디널리티 (Cardinality)

- 카디널리티는 컬럼이 가질 수 있는 값의 다양성 정도를 의미합니다.

- `예시`

- 카디널리티가 낮은 경우  
  성별 (M / F)

- 카디널리티가 높은 경우  
  이름, 주민번호, 사번

- 카디널리티가 높은 컬럼은 인덱스 효율이 높은 편입니다.

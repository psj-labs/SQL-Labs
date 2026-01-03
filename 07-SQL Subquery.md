# 서브쿼리 및 집합 연산 정리

## 개요

- 본 문서는 Oracle SQL에서 서브쿼리(Subquery)와 집합 연산자를 정리한 문서입니다.
- 단일행, 다중행, 다중열 서브쿼리와 UNION 계열 연산자를 중심으로 설명합니다.

---

## 서브쿼리(Subquery)란?

- 서브쿼리는 하나의 SQL문 안에 포함된 SELECT 문입니다.
- 바깥쪽 SQL문은 메인쿼리입니다.
- 안쪽에 포함된 SELECT 문은 서브쿼리입니다.
```sql
SELECT 컬럼명
FROM 테이블
WHERE 컬럼명 = (SELECT 컬럼명 FROM 테이블 WHERE 조건);
```
---

## 서브쿼리의 종류

| 종류 | 설명 | 비교 연산자 |
|------|------|-------------|
| 단일행 서브쿼리 | 결과가 1행 1열인 경우입니다 | =, <, >, <=, >=, <> |
| 다중행 서브쿼리 | 결과가 여러 행인 경우입니다 | IN, ANY, ALL |
| 다중열 서브쿼리 | 결과가 여러 열인 경우입니다 | IN |

---

## 단일행 서브쿼리

- 서브쿼리의 실행 결과가 하나의 값일 때 사용하는 방식입니다.
```sql
SELECT eno 사번, ename 이름
FROM emp
WHERE sal > (SELECT sal FROM emp WHERE ename = '남궁연호');
```
- 남궁연호보다 급여가 높은 사원을 조회합니다.
- 서브쿼리가 여러 행을 반환하면 ORA-01427 오류가 발생합니다.
- 단일 비교 연산자는 반드시 하나의 값만 반환해야 합니다.

---

## 다중행 서브쿼리

- 서브쿼리 결과가 여러 행일 때 사용하는 방식입니다.

### IN 연산자
```sql
SELECT eno, ename
FROM emp
WHERE dno IN (SELECT dno FROM dept WHERE loc = 'DALLAS');
```
- DALLAS 지역 부서에 속한 사원을 조회합니다.

### ANY 연산자
```sql
SELECT eno, ename, sal
FROM emp
WHERE sal < ANY (SELECT sal FROM emp WHERE dno = 10);
```
- 10번 부서 사원의 급여 중 하나라도 큰 값이 존재하면 조건을 만족합니다.

### ALL 연산자
```sql
SELECT eno, ename, sal
FROM emp
WHERE sal < ALL (SELECT sal FROM emp WHERE dno = 10);
```
- 10번 부서 사원의 최저 급여보다 작은 급여를 가진 사원을 조회합니다.

---

## 다중열 서브쿼리

- 여러 컬럼을 하나의 묶음으로 비교할 때 사용하는 방식입니다.
```sql
SELECT sname 이름, avr 평점
FROM student
WHERE (syear, avr) IN (
SELECT syear, avr
FROM student
WHERE major = '화학'
)
AND major != '화학';
```
- 화학과 학생과 동일한 학년과 평점을 가진 타 학과 학생을 조회합니다.

---

## 집합 연산자

| 연산자 | 의미 |
|--------|------|
| UNION | 합집합이며 중복을 제거합니다 |
| UNION ALL | 합집합이며 중복을 허용합니다 |
| INTERSECT | 교집합입니다 |
| MINUS | 차집합입니다 |
```sql
SELECT sno, sname, major, avr
FROM student
WHERE major IN ('화학','물리')
INTERSECT
SELECT sno, sname, major, avr
FROM student
WHERE avr >= 3;
```
- 화학 및 물리 학과이면서 평점이 3 이상인 학생을 조회합니다.

---

## NVL 함수
```sql
SELECT sal*12 + NVL(comm, 0) AS 연봉
FROM emp;
```
- 수당(comm)이 NULL인 경우 0으로 처리하여 연봉을 계산합니다.

---

## 정리

- 단일행 서브쿼리는 하나의 값을 반환합니다.
- 다중행 서브쿼리는 여러 값을 반환합니다.
- 다중열 서브쿼리는 튜플 단위로 비교합니다.
- 집합 연산자는 SELECT 결과 집합을 기준으로 연산합니다.
- NVL 함수는 NULL 값을 처리할 때 필수적으로 사용됩니다.

## Natural Join (자연 조인)

**Natural Join**은 _두 테이블에 같은 이름과 데이터 타입을 가진 컬럼_이 있을 때 자동으로 조인을 수행합니다.

### 특징

-   같은 이름·같은 타입의 컬럼이 1개 있어야 함
-   조인 컬럼은 자동 인식 (명시 X)
-   조인 컬럼명이 다르면 사용 불가
-   등가(=) 조인만 가능

### 기본 구문

```
SELECT 컬럼, ...
FROM 테이블1
NATURAL JOIN 테이블2
[NATURAL JOIN 테이블3 ...]
WHERE 조건;
```

### 예시 — 부서와 사원 정보

```
SELECT dno, dname, eno, ename, job, sal
FROM dept
NATURAL JOIN emp;
```

\---

## USING 절을 이용한 JOIN

**USING** 절은 **조인에 사용할 컬럼을 명시**적으로 지정하는 방식입니다. `NATURAL`과 달리, 동일 이름 컬럼이 여러 개여도 선택적으로 지정 가능합니다.

### 특징

-   조인 컬럼을 `USING(컬럼명)`으로 지정
-   조인 컬럼 외 동일한 이름은 `테이블명.컬럼명`으로 구분
-   `NATURAL`과 함께 사용할 수 없음

### 기본 구문

```
SELECT 컬럼, ...
FROM 테이블1
JOIN 테이블2 USING (조인컬럼)
WHERE 조건;
```

### 예시 — 부서별 사원

```
SELECT dno, dname, eno, ename, job, sal
FROM dept
JOIN emp USING (dno);
```

\---

## NATURAL vs USING 비교

| 항목 | NATURAL JOIN | USING JOIN |
| --- | --- | --- |
| 조인 컬럼 지정 | 자동 인식 (동일 이름) | 직접 지정 |
| 조건 기술 | 불필요 | USING(컬럼명) |
| 등가/비등가 | 등가만 가능 | 등가만 가능 |
| 컬럼명 다를 때 | 불가능 | 불가능 (ON 사용 필요) |

### 예시 — 화학과 학생의 일반화학 점수

```
SELECT major, syear, sno, sname, cname, result
FROM student
NATURAL JOIN score
NATURAL JOIN course
WHERE major='화학' AND cname='일반화학';

-- 동일 결과 (USING 이용)
SELECT major, syear, sno, sname, cname, result
FROM student
JOIN score USING (sno)
JOIN course USING (cno)
WHERE major='화학' AND cname='일반화학';
```

\---

## ON 절을 이용한 JOIN

**ON** 절은 조인 조건을 직접 지정하는 방식으로, `NATURAL`·`USING`보다 훨씬 유연합니다.

### 특징

-   등가/비등가 조인 모두 가능
-   컬럼명이 달라도 조인 가능
-   `ON`과 `USING` 혼용 가능

### 기본 구문

```
SELECT t1.컬럼, t2.컬럼
FROM 테이블1 t1
JOIN 테이블2 t2 ON t1.컬럼 = t2.컬럼
WHERE 조건;
```

### 예시 — 사원과 부서

```
SELECT e.eno, e.ename, d.dno, d.dname
FROM emp e
JOIN dept d ON e.dno = d.dno;
```

### 비등가 조인 예시 — 급여 등급

```
SELECT eno, ename, job, grade
FROM emp
JOIN salgrade ON sal BETWEEN losal AND hisal
WHERE job = '지원';
```

**ON절 장점:**

-   조인 조건을 자유롭게 제어
-   비등가 조인 가능
-   컬럼명이 달라도 조인 가능

\---

## Outer Join (외부 조인)

**Outer Join**은 조인 조건에 맞지 않아도 특정 테이블의 모든 행을 결과에 포함시킵니다.

### 종류

-   **LEFT OUTER JOIN** — 왼쪽 테이블 모든 행 포함
-   **RIGHT OUTER JOIN** — 오른쪽 테이블 모든 행 포함
-   **FULL OUTER JOIN** — 양쪽 테이블 모든 행 포함

### 기본 구문

```
SELECT t1.컬럼, t2.컬럼
FROM 테이블1 t1
[LEFT | RIGHT | FULL] JOIN 테이블2 t2
ON t1.조인컬럼 = t2.조인컬럼;
```

### 예시 — RIGHT JOIN

```
SELECT eno, ename, dno, dname
FROM emp
RIGHT JOIN dept USING (dno);
```

**주의:** NATURAL JOIN과 OUTER JOIN은 함께 사용할 수 없습니다.

### 기존 Oracle(+) 방식 비교

| 기존 방식 | 표준 SQL 방식 |
| --- | --- |
| `WHERE e.dno(+) = d.dno;` | `RIGHT JOIN dept ON emp.dno = dept.dno;` |

\---

## Cross Join (교차 조인)

Cross Join은 조인 조건이 없는 모든 행의 조합을 생성합니다. 즉, 두 테이블의 **행 수의 곱**만큼 결과가 만들어집니다.

### 구문

```
SELECT t1.컬럼, t2.컬럼
FROM 테이블1
CROSS JOIN 테이블2;
```

### 예시

```
SELECT eno, ename, dname
FROM emp
CROSS JOIN dept;
```

\---

## 실습 문제 예시

1.  각 과목명과 담당 교수명 검색
2.  화학과 학생의 기말고사 성적 전체 검색
3.  유기화학 수강생의 기말고사 점수 검색
4.  화학과 학생이 수강하는 과목 담당 교수 검색
5.  모든 교수와 과목 명단 검색 (Outer Join 사용)
6.  직원 중 사수보다 급여가 높은 사람과 사수 이름/급여 검색
7.  학생 중 동명이인 검색

\---

## 조인 핵심 요약

| 유형 | 조인 조건 | 특징 |
| --- | --- | --- |
| **NATURAL JOIN** | 자동(동일 컬럼명) | 등가만 가능, 컬럼명 달라지면 불가 |
| **USING JOIN** | USING(컬럼명) | 명시적 지정, 등가만 가능 |
| **ON JOIN** | ON 조건 | 비등가/다른 컬럼명 가능 |
| **OUTER JOIN** | ON 조건 | 조인 불일치 행도 포함 |
| **CROSS JOIN** | 없음 | 모든 행 조합 (카티션 곱) |

---

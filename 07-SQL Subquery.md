## 서브쿼리(Subquery)란?

**서브쿼리**는 하나의 SQL문 안에 포함된 **SELECT 문**이다.  
- 바깥쪽 SQL → **메인쿼리(Main Query)**  
- 안쪽 SELECT → **서브쿼리(Sub Query)**

    SELECT 컬럼명
    FROM 테이블
    WHERE 컬럼명 = (SELECT 컬럼명 FROM 테이블 WHERE 조건);

---

## 서브쿼리의 종류

| 종류 | 설명 | 비교 연산자 |
|------|------|--------------|
| **단일행 서브쿼리** | 결과가 1행 1열(값 1개) | =, <, >, <=, >=, <> |
| **다중행 서브쿼리** | 결과가 여러 행 | IN, ANY, ALL |
| **다중열 서브쿼리** | 결과가 여러 열(튜플) | IN |

---

## 단일행 서브쿼리

서브쿼리 결과가 **값 1개**일 때 사용.

    SELECT eno 사번, ename 이름
    FROM emp
    WHERE sal > (SELECT sal
                 FROM emp
                 WHERE ename = '남궁연호');

**의미:**  
‘남궁연호’보다 급여가 높은 직원 출력.

**주의:**  
- 여러 행 반환되면 ORA-01427 오류  
- 단일 비교 연산자는 반드시 1행만 필요함

---

## 다중행 서브쿼리

결과가 **여러 행**일 때 사용.  
→ **IN / ANY / ALL**

### IN (여러 값 중 하나라도 일치하면 TRUE)

    SELECT eno, ename
    FROM emp
    WHERE dno IN (SELECT dno
                  FROM dept
                  WHERE loc = 'DALLAS');

→ DALLAS에 위치한 부서에서 근무하는 사원 출력

---

### ANY (여러 값 중 ‘하나’와 비교)

    SELECT eno, ename, sal
    FROM emp
    WHERE sal < ANY (SELECT sal FROM emp WHERE dno = '10');

→ 10번 부서 사원의 **최대 급여보다 작은 사람**

---

### ALL (여러 값 ‘모두’와 비교)

    SELECT eno, ename, sal
    FROM emp
    WHERE sal < ALL (SELECT sal FROM emp WHERE dno = '10');

→ 10번 부서의 **최저 급여보다 작은 사람**

---

## 다중열 서브쿼리 (튜플 비교)

여러 열을 함께 비교할 때 사용.

    SELECT sname 이름, avr 평점
    FROM student
    WHERE (syear, avr) IN (
            SELECT syear, avr
            FROM student
            WHERE major = '화학'
          )
      AND major != '화학';

**의미:**  
화학과 학생과 **같은 학년 + 같은 평점**을 가진 타 학과 학생 조회.

---

## 집합 연산자 (UNION / INTERSECT / MINUS)

| 연산자 | 의미 | 특징 |
|--------|------|------|
| **UNION** | 합집합 | 중복 제거 |
| **UNION ALL** | 합집합 | 중복 허용 |
| **INTERSECT** | 교집합 | 공통 결과 |
| **MINUS** | 차집합 | 첫 SELECT - 두 번째 SELECT |

    SELECT sno, sname, major, avr
    FROM student
    WHERE major IN ('화학', '물리')
    INTERSECT
    SELECT sno, sname, major, avr
    FROM student
    WHERE avr >= 3;

→ 화학·물리 학과 + 평점 3 이상 학생

---

## NVL 함수 (NULL 처리)

    NVL(comm, 0)

comm(수당)이 NULL이면 0으로 변환.

    SELECT sal*12 + NVL(comm, 0) AS 연봉
    FROM emp;

---

## 실행 순서 핵심 정리

- 서브쿼리는 항상 괄호()로 감싼다  
- **서브쿼리 먼저 실행 → 메인쿼리가 그 결과 사용**  
- 단일행 → =, >, <  
- 다중행 → IN, ANY, ALL  
- 다중열 → (a, b) IN (SELECT ...)

---

## 예제 요약 표

| 종류 | 반환 | 연산자 | 예시 |
|------|-------|---------|-------|
| 단일행 단일열 | 1행 1열 | =, >, < | 특정 사람보다 급여 높은 직원 |
| 다중행 단일열 | 여러 행 | IN, ANY, ALL | 10번 부서 급여 비교 |
| 다중행 다중열 | 여러 행/열 | IN | (syear, avr) 튜플 비교 |

---

## 마무리 요약

- **단일행 = 값 1개**  
- **다중행 = 여러 개 값**  
- **단일열 = 1개 컬럼**  
- **다중열 = 여러 컬럼(튜플)**  
- **IN / ANY / ALL → 다중행 서브쿼리**  
- **UNION / INTERSECT / MINUS → SELECT 결과 집합 연산**  
- **NVL → NULL 처리 필수 함수**

SQL 서브쿼리와 집합연산자 개념을 이 구조로 잡으면 실무에서도 바로 적용할 수 있다.

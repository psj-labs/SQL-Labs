### 1️⃣ 연결 연산자 (||) - 문자열 합치기  
연결 연산자는 여러 문자열이나 컬럼을 이어붙여 하나의 문자열로 만드는 연산자입니다.

예제:
SELECT ename || '의 직업은 ' || job || ' 입니다.' AS 설명
FROM emp;

예시 결과:
SMITH의 직업은 CLERK 입니다.  
ALLEN의 직업은 SALESMAN 입니다.  
WARD의 직업은 SALESMAN 입니다.

주의:
- 리터럴은 작은따옴표(' ')  
- 복잡하면 괄호로 묶기  

---

### 2️⃣ 중복 제거 (DISTINCT)

예제:
SELECT DISTINCT job
FROM emp;

예시 결과:
CLERK  
SALESMAN  
MANAGER  
ANALYST  
PRESIDENT  

정리:
- DISTINCT: 중복 제거  
- ALL: 중복 포함  
- 정렬은 비용이 큼(성능 부담)  

---

### 3️⃣ 정렬 (ORDER BY)

문법:
SELECT 컬럼명  
FROM 테이블명  
ORDER BY 컬럼명 ASC | DESC;

특징:
- ASC: 오름차순  
- DESC: 내림차순  
- 여러 정렬 기준 가능  
- 오라클에서 NULL은 가장 큰 값  

예제:
SELECT ename, sal  
FROM emp  
ORDER BY sal DESC;

예시:
KING 5000  
SCOTT 3000  
FORD 3000  
ALLEN 1600  

---

### 4️⃣ 묶음 검색 (업무별 / 부서별)

업무별 정렬:
SELECT job AS 업무, eno AS 사번, ename AS 이름, sal AS 급여  
FROM emp  
ORDER BY 업무;

부서별 급여 내림차순:
SELECT dno AS 부서번호, eno AS 사번, ename AS 이름, sal AS 급여  
FROM emp  
ORDER BY 부서번호, sal DESC;

---

### 카디널리티(Cardinality)
값의 다양성 정도  
- 낮음: 성별(M/F)  
- 높음: 이름  

---

### ✅ 핵심 요약  
- || : 문자열 연결  
- DISTINCT : 중복 제거  
- ORDER BY : 정렬  
- NULL(오라클) = 가장 큰 값  
- 카디널리티 = 값의 다양성  

## 1️⃣ 수식 검색

    SELECT 10 + 20 AS "결과"
    FROM dual;

Oracle DB에서 dual은 실제 테이블이 아닌 더미(dummy) 테이블이다. 단순 연산이나 시스템 함수 테스트에 사용한다.

- SELECT 절에 컬럼이 없으면 FROM 절에는 반드시 dual 사용
- 즉, SELECT문에 FROM절을 생략할 수 없다

실무 예시:

    -- 오늘 날짜와 10일 뒤 날짜 계산
    SELECT SYSDATE AS 오늘, SYSDATE + 10 AS "10일 후"
    FROM dual;

---

## 2️⃣ 별명을 이용한 검색

    SELECT ename AS "이름", sal AS "급여", deptno AS "부서번호"
    FROM emp;

- 별명(alias)은 출력 헤더를 바꿀 때 사용한다.
- AS 생략 가능
- 아래 경우 이중 인용부호("") 필요  
  1. 공백 포함  
  2. 특수문자 포함  
  3. 대소문자 유지 필요  

실무 예시:

    SELECT empno AS "사번",
           ename AS "이름",
           sal * 12 AS "연봉(12개월)"
    FROM emp;

💡 별명은 개발 시 변수명 또는 JSON 키로 사용되므로 가독성 있게 작성하는 것이 좋다.

---

## 3️⃣ 수식을 이용한 검색

    SELECT empno AS "사번",
           ename AS "이름",
           sal * 12 AS "연봉",
           sal * 1.1 AS "인상_급여"
    FROM emp;

- 수식 결과는 헤더에 그대로 출력되므로 별명을 지정하면 가독성이 크게 향상된다.

실무 예시:

    -- 부서별 연간 인건비 예측
    SELECT deptno AS "부서번호",
           SUM(sal * 12) AS "연간_인건비"
    FROM emp
    GROUP BY deptno;

---

## 4️⃣ NULL 값이 포함된 연산

    SELECT empno, ename, sal, comm, sal * 12 + comm AS "총급여"
    FROM emp;

NULL은 값이 없음이 아니라 값이 결정되지 않음을 의미한다.  
NULL이 포함된 모든 연산 결과는 항상 NULL이다.

예시:

    사번   이름     급여   커미션   총급여
    ----------------------------------
    7369  SMITH    800   (NULL)    (NULL)
    7499  ALLEN   1600     300     19500

NVL 사용:

    SELECT empno, ename, sal, NVL(comm, 0) AS "보너스",
           sal * 12 + NVL(comm, 0) AS "총급여"
    FROM emp;

실무 응용:

    -- NVL을 활용한 인센티브 포함 연봉 계산
    SELECT ename AS "이름",
           sal * 12 + NVL(comm, 0) AS "실제 연봉"
    FROM emp
    ORDER BY "실제 연봉" DESC;

---

## 5️⃣ 출력 화면 제어 (SET 명령)

    SET LINESIZE 150
    -- 한 줄 출력 너비

    SET PAGESIZE 30
    -- 한 페이지 출력 행 수

💡 보고서 형태 출력하거나 데이터가 잘릴 때 유용하다.

---

## 6️⃣ 컬럼 형식 지정 (COLUMN FORMAT)

    COLUMN ename FORMAT A15
    COLUMN sal FORMAT 999,999

- 문자 컬럼: A##  
- 숫자 컬럼: 0, 9 패턴으로 형식 지정

예시:

    COLUMN ename FORMAT A10
    COLUMN sal FORMAT 999,999.00
    SELECT ename, sal FROM emp;

초기화:

    COLUMN sal CLEAR

---

## 실무 응용 요약

- dual : 임시 연산용 더미 테이블  
- alias : 출력 헤더 변경 및 가독성 향상  
- NVL() : NULL 보정 필수  
- SET / COLUMN FORMAT : 출력 제어  

---

## 예제 종합

    SET LINESIZE 120
    SET PAGESIZE 30
    COLUMN ename FORMAT A15
    COLUMN "실제 연봉" FORMAT 999,999.00

    SELECT ename AS "이름",
           sal * 12 + NVL(comm, 0) AS "실제 연봉",
           deptno AS "부서번호"
    FROM emp
    ORDER BY "실제 연봉" DESC;

직원 실질 연봉 리스트를 출력할 때 활용한다.

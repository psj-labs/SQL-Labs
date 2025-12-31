# SQL 연산 및 출력 제어 정리

## 개요

- 본 문서는 Oracle SQL에서 수식 연산, 별명(alias), NULL 처리, 출력 제어 기능을 정리한 문서입니다.
- 단순 조회가 아닌 실무에서 자주 사용되는 SQL 작성 패턴을 중심으로 설명합니다.
- 모든 예시는 Oracle DB 기준으로 작성되었습니다.

---

## 수식 검색

- SELECT 절에서 수식을 사용하여 계산 결과를 바로 조회할 수 있습니다.
- Oracle DB에서 dual은 실제 테이블이 아닌 `더미(dummy) 테이블`입니다.
- 단순 연산이나 시스템 함수 테스트에 사용됩니다.
```sql
    SELECT 10 + 20 AS "결과"
    FROM dual;
```
- SELECT 절에 컬럼이 없는 경우 FROM 절에는 반드시 dual을 사용해야 합니다.
- Oracle에서는 SELECT 문에서 FROM 절을 생략할 수 없습니다.

- `예시`
```sql
    -- 오늘 날짜와 10일 뒤 날짜 계산
    SELECT SYSDATE AS 오늘,
           SYSDATE + 10 AS "10일 후"
    FROM dual;
```
---

## 별명을 이용한 검색

- 별명(alias)은 출력되는 컬럼의 헤더명을 변경할 때 사용합니다.
- AS 키워드는 생략할 수 있습니다.
- 다음과 같은 경우 이중 인용부호를 사용해야 합니다.
  - 공백이 포함된 경우
  - 특수문자가 포함된 경우
  - 대소문자를 유지해야 하는 경우
```sql
    SELECT ename AS "이름",
           sal AS "급여",
           deptno AS "부서번호"
    FROM emp;
```
- `예시`
```sql
    SELECT empno AS "사번",
           ename AS "이름",
           sal * 12 AS "연봉(12개월)"
    FROM emp;
```
- 별명은 개발 시 변수명 또는 JSON 키로 사용되는 경우가 많으므로 가독성 있게 작성하는 것이 좋습니다.

---

## 수식을 이용한 검색

- 컬럼에 산술 연산을 적용하여 계산된 값을 조회할 수 있습니다.
- 수식 결과는 그대로 출력되므로 별명을 지정하면 가독성이 크게 향상됩니다.
```sql
    SELECT empno AS "사번",
           ename AS "이름",
           sal * 12 AS "연봉",
           sal * 1.1 AS "인상_급여"
    FROM emp;
```
- `예시`
```sql
    -- 부서별 연간 인건비 예측
    SELECT deptno AS "부서번호",
           SUM(sal * 12) AS "연간_인건비"
    FROM emp
    GROUP BY deptno;
```
---

## NULL 값이 포함된 연산

- NULL은 값이 없다는 의미가 아니라 값이 결정되지 않았음을 의미합니다.
- NULL이 포함된 모든 연산 결과는 항상 NULL이 됩니다.
```sql
    SELECT empno, ename, sal, comm, sal * 12 + comm AS "총급여"
    FROM emp;
```
- `예시`
```sql
    사번   이름     급여   커미션   총급여
    ----------------------------------
    7369  SMITH   800   (NULL)    (NULL)
    7499  ALLEN   1600     300     19500
```
- NULL 값을 보정하기 위해 NVL 함수를 사용합니다.
```sql
    SELECT empno,
           ename,
           sal,
           NVL(comm, 0) AS "보너스",
           sal * 12 + NVL(comm, 0) AS "총급여"
    FROM emp;
```
- `예시`
```sql
    -- NVL을 활용한 인센티브 포함 연봉 계산
    SELECT ename AS "이름",
           sal * 12 + NVL(comm, 0) AS "실제 연봉"
    FROM emp
    ORDER BY "실제 연봉" DESC;
```
---

## 출력 화면 제어 (SET 명령)

- SQL*Plus 환경에서 출력 형식을 제어할 수 있습니다.
```sql
    SET LINESIZE 150
    -- 한 줄 출력 너비 설정

    SET PAGESIZE 30
    -- 한 페이지당 출력 행 수 설정
```
- 보고서 형태 출력이나 데이터 잘림 방지 시 유용합니다.

---

## 컬럼 형식 지정 (COLUMN FORMAT)

- 출력되는 컬럼의 형식을 지정할 수 있습니다.
```sql
    COLUMN ename FORMAT A15
    COLUMN sal FORMAT 999,999
```
- 문자 컬럼은 A숫자 형식으로 지정합니다.
- 숫자 컬럼은 9 또는 0 패턴으로 형식을 지정합니다.

- `예시`
```sql
    COLUMN ename FORMAT A10
    COLUMN sal FORMAT 999,999.00
    SELECT ename, sal FROM emp;
```
- 형식 초기화
```sql
    COLUMN sal CLEAR
```

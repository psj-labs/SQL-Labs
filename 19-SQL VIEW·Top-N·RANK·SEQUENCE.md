## VIEW (뷰)
​
뷰(View)는 **하나 이상의 테이블에서 파생된 가상 테이블(Virtual Table)**입니다. 실제 데이터를 저장하지 않고, `SELECT` 문 결과를 이름으로 저장하여 재사용할 수 있습니다.
​
### 뷰의 종류
​
-   **단순 뷰(Simple View)** — 하나의 테이블에서 생성, `DML` 가능
-   **복합 뷰(Complex View)** — 여러 테이블, 그룹함수, 조인 포함 → `DML` 불가
​
### 뷰 생성 구문
​
```
CREATE [OR REPLACE] [FORCE | NOFORCE] VIEW 뷰이름 (컬럼명, ...)
AS (SELECT 문장)
[WITH CHECK OPTION [CONSTRAINT 제약명]]
[WITH READ ONLY [CONSTRAINT 제약명]];
```
​
-   `OR REPLACE` : 기존 뷰를 대체 (ALTER VIEW 역할)
-   `FORCE` : 기반 테이블이 없어도 생성
-   `WITH CHECK OPTION` : 뷰 조건에 맞는 행만 수정/삽입 가능
-   `WITH READ ONLY` : SELECT만 가능
​
### 뷰 삭제 및 확인
​
```
DROP VIEW 뷰이름;
​
SELECT view_name, text FROM user_views;
```
​
**권한 주의:** 일반 사용자 계정은 기본적으로 `CREATE VIEW` 권한이 없습니다.  
관리자 권한으로 다음 명령을 실행해야 합니다.  
`GRANT CREATE VIEW TO resource;`
​
### 실습 예시 — 일반화학 기말고사 평균 뷰
​
```
CREATE VIEW ma_result (과목번호, 과목명, 학과, 기말고사평균)
AS SELECT c.cno, cname, major, ROUND(AVG(result))
   FROM student s, course c, score r
   WHERE s.sno = r.sno AND r.cno = c.cno
     AND cname = '일반화학'
   GROUP BY c.cno, cname, major;
​
SELECT * FROM ma_result;
```
​
### WITH CHECK OPTION 예시
​
```
CREATE VIEW st_ch AS
SELECT sno, sname, syear, avr
FROM student
WHERE syear = 1;
​
INSERT INTO st_ch VALUES ('000001','시현',2,4.0);  -- 삽입됨
SELECT * FROM st_ch WHERE sname='시현'; -- 검색되지 않음
​
-- CHECK 옵션 추가 후 재생성
CREATE OR REPLACE VIEW st_ch AS
SELECT sno, sname, syear, avr
FROM student
WHERE syear = 1
WITH CHECK OPTION CONSTRAINT view_st_ch_ck;
​
INSERT INTO st_ch VALUES ('000001','시현',2,4.0);
-- ORA-01402 에러 발생 (조건 위배)
```
​
\---
​
## 인라인 뷰 (Inline View)
​
인라인 뷰는 **FROM절 내부에 작성된 서브쿼리**입니다. 일시적으로 존재하며, SQL이 실행될 때만 유지됩니다.
​
### 형식
​
```
SELECT ...
FROM (SELECT ... FROM ...) [별칭];
```
​
### 예시 — 부서별 최소 급여자
​
```
SELECT eno, ename, e.dno, sal, msal
FROM emp e,
     (SELECT dno, MIN(sal) msal FROM emp GROUP BY dno) d
WHERE e.dno = d.dno
AND sal = msal;
```
​
\---
​
## Top-N 분석과 RANK 함수
​
### Top-N 분석 개념
​
**Top-N 분석**은 정렬된 결과 중 상위 N개 혹은 하위 N개 행만을 추출하는 분석 기법입니다. SQL에서 정렬은 마지막 단계이므로 **인라인 뷰**를 이용해야 올바른 결과를 얻습니다.
​
### 기본 구문
​
```
SELECT ROWNUM, 컬럼들
FROM (SELECT ... ORDER BY 정렬기준)
WHERE ROWNUM <= N;
```
​
### 예시 — 급여 상위 3인
​
```
SELECT ROWNUM, eno, ename, sal
FROM (SELECT eno, ename, sal FROM emp ORDER BY sal DESC)
WHERE ROWNUM <= 3;
```
​
**주의:** `ORDER BY`를 바깥에 두면 ROWNUM이 먼저 평가되어 잘못된 순위가 나옵니다.
​
### RANK 관련 함수 비교
​
| 함수 | 특징 |
| --- | --- |
| `RANK()` | 동점 순위를 부여하고, 다음 순위는 건너뜀 |
| `DENSE_RANK()` | 동점 순위를 하나로 취급, 다음 순위를 연속 부여 |
| `ROW_NUMBER()` | 모든 행에 고유 순번 부여 (동점 無) |
​
### 예시 — 급여 순위 출력
​
```
SELECT eno, ename, sal,
       RANK()        OVER (ORDER BY sal DESC) RANK,
       DENSE_RANK()  OVER (ORDER BY sal DESC) DRANK,
       ROW_NUMBER()  OVER (ORDER BY sal DESC) RRANK
FROM emp;
```
​
### 부서별 급여 순위
​
```
SELECT eno, ename, dno, sal,
       RANK() OVER (PARTITION BY dno ORDER BY sal DESC) RANK
FROM emp;
```
​
### 연봉 기준 순위
​
```
SELECT RANK() OVER (ORDER BY sal*12+NVL(comm,0) DESC) RRANK,
       eno, ename, sal*12+NVL(comm,0) 연봉
FROM emp;
```
​
\---
​
## SEQUENCE (시퀀스)
​
시퀀스는 **유일한 숫자값을 자동으로 생성하는 객체**입니다. 보통 `PK`값을 자동 생성하거나, 고유 일련번호를 부여할 때 사용합니다.
​
### 시퀀스 생성 구문
​
```
CREATE SEQUENCE 시퀀스명
START WITH 시작값
INCREMENT BY 증가값
MAXVALUE [상한값 | NOMAXVALUE]
MINVALUE [하한값 | NOMINVALUE]
[CYCLE | NOCYCLE]
CACHE [캐시개수 | NOCACHE];
```
​
### 조회 및 삭제
​
```
SELECT sequence_name, max_value, min_value, increment_by,
       cache_size, last_number, cycle_flag
FROM user_sequences;
​
DROP SEQUENCE 시퀀스명;
```
​
### 예시 — emp\_eno\_seq 시퀀스 생성
​
```
CREATE SEQUENCE emp_eno_seq
START WITH 1
INCREMENT BY 1
NOMAXVALUE
NOMINVALUE
NOCYCLE
CACHE 20;
```
​
### 시퀀스 값 사용
​
```
INSERT INTO emp (eno, ename)
VALUES (emp_eno_seq.NEXTVAL, '첫 번째');
​
SELECT emp_eno_seq.CURRVAL FROM dual;
```
​
**용어 정리:**  
`NEXTVAL` : 다음 시퀀스 번호 생성  
`CURRVAL` : 마지막 생성된 번호 반환
​
### 게시판 예시 — board / content 테이블
​
```
CREATE SEQUENCE bo_bno_seq;
​
INSERT INTO board (bno, name, sub)
VALUES (bo_bno_seq.NEXTVAL, '이름', '제목은');
​
INSERT INTO content (bno, text)
VALUES (bo_bno_seq.CURRVAL, '본문');
```
​
\---
​
## 요약 정리
​
-   **VIEW**: 가상의 테이블. SELECT 결과를 재사용. `WITH CHECK`, `READ ONLY` 옵션 존재.
-   **INLINE VIEW**: FROM절 서브쿼리. Top-N, RANK 분석의 핵심.
-   **RANK 함수**: 순위 부여 함수 (RANK, DENSE\_RANK, ROW\_NUMBER)
-   **SEQUENCE**: 자동 일련번호 생성. `NEXTVAL`, `CURRVAL` 사용.

## 1️⃣ Database와 DBMS
### Database(데이터베이스)
정형화된 형태로 구조적으로 저장된 데이터의 집합.  
예: 학생 관리 시스템 → 학생 테이블, 성적 테이블, 과목 테이블 등

### DBMS(Database Management System)
데이터베이스를 생성·관리·제어하는 소프트웨어.

- 계층형(Hierarchical)
- 망형(Network)
- 관계형(Relational)  ✅ 가장 많이 사용됨

### RDBMS 예시
Oracle, MySQL, MariaDB, SQL Server, DB2

---

## 2️⃣ RDB의 기본 구조
관계형 데이터베이스는 데이터를 **테이블(Table)** 형태로 저장한다.

| 요소 | 설명 |
|------|------|
| 행(Row) | 실제 데이터 한 단위(레코드) |
| 열(Column) | 데이터 속성 |
| 테이블(Table) | 동일 구조의 행 집합 |

💡 모든 테이블은 '공통 컬럼'을 통해 관계를 맺으며 JOIN이 가능하다.

학생(student)    과목(subject)  
────────────────────────────────────────────────────────  
학번 | 이름 | 학과   과목코드 | 과목명  
101 | 김철수 | 컴퓨터  C101 | 데이터베이스  
102 | 이영희 | 전자  C102 | 네트워크  
→ 관계(Relation): 학번/과목코드 등으로 연결

---

## 3️⃣ SQL(Structured Query Language)
SQL은 관계형 데이터베이스에서 데이터를 조작하고 관리하는 표준 언어이다.

### SQL 언어 분류

| 종류 | 의미 | 예 |
|------|------|------|
| DML | 데이터 조작 | SELECT, INSERT, UPDATE, DELETE |
| DDL | 데이터 구조 정의 | CREATE, ALTER, DROP, RENAME |
| DCL | 권한 제어 | GRANT, REVOKE |
| TCL | 트랜잭션 제어 | COMMIT, ROLLBACK |

---

## 4️⃣ SELECT문 구조와 사용법
데이터 조회의 핵심.

SELECT [DISTINCT] 컬럼명1, 컬럼명2 [AS 별칭]  
FROM 테이블명  
WHERE 조건  
ORDER BY 정렬기준;

* = 전체 컬럼  
AS = 컬럼 별칭

예:
SELECT empno, ename AS 이름, sal AS 급여  
FROM emp  
WHERE deptno = 10  
ORDER BY sal DESC;

### Projection / Selection / Join
- Projection: 특정 컬럼만 선택  
- Selection: 특정 행 선택  
- Join: 여러 테이블 결합

JOIN 예:
SELECT emp.ename, dept.dname  
FROM emp  
JOIN dept ON emp.deptno = dept.deptno;

---

## 5️⃣ 테이블 구조 확인
SELECT * FROM tab;  
DESC emp;

💡 오라클에서 스키마(schema) = 사용자(user)

---

## 6️⃣ SQL 문법 규칙
- SQL 문장은 반드시 세미콜론(;)으로 종료  
- 대소문자 구분 없음 (문자열 제외)  
- 여러 줄 작성 가능  
- 실행하면 항시 결과 반환

---

## 7️⃣ 데이터 타입

| 타입 | 설명 | 예 |
|------|------|------|
| VARCHAR2(n) | 가변 길이 문자 | '홍길동' |
| NUMBER(n) | 숫자형 | 100, 3.14 |
| DATE | 날짜 | 2025-10-23 |

숫자형은 우측 정렬, 문자형은 좌측 정렬.

---

## 8️⃣ 무결성(Integrity)
데이터에 오류나 중복이 없도록 보장하는 개념.

- 각 테이블은 반드시 PK(Primary Key)를 가져야 함  
- Foreign Key로 테이블 간 관계 유지

예시:
CREATE TABLE student (  
  student_id NUMBER PRIMARY KEY,  
  name VARCHAR2(50),  
  dept_id NUMBER REFERENCES department(dept_id)  
);

---

## 9️⃣ 실무 팁
- 대규모 조회 시 WHERE + LIMIT/ROWNUM 사용  
- INDEX 작성 시 SELECT 성능 향상  
- COMMIT/ROLLBACK으로 트랜잭션 안전성 유지  
- 실행계획(EXPLAIN PLAN)으로 병목 분석

---

##  요약
- DBMS는 데이터 저장·관리 시스템  
- RDBMS는 관계형 DB를 관리  
- SQL은 데이터 조작·정의·제어 언어  
- SELECT는 조회의 핵심  
- 무결성·키 설정은 데이터 품질 핵심

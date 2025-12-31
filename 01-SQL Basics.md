# Database & SQL 기초 정리

## 개요

- 데이터베이스와 SQL은 서버, 웹, 보안 시스템 전반의 기반이 되는 핵심 개념입니다.
- 본 문서는 Database, DBMS, 관계형 데이터베이스 구조와 SQL 기본 문법을 이해하는 것을 목표로 합니다.
---

## Database와 DBMS

### Database(데이터베이스)

- 데이터베이스는 정형화된 형태로 구조적으로 저장된 데이터의 집합입니다.
- 데이터는 일정한 규칙과 구조를 가지고 저장되며 효율적인 검색과 관리가 가능합니다.

- `예시: 학생 관리 시스템`
```text
- 학생 테이블  
- 성적 테이블  
- 과목 테이블  
```
---

### DBMS(Database Management System)

- DBMS는 데이터베이스를 생성,관리,제어하는 소프트웨어입니다.
- 사용자는 DBMS를 통해 데이터를 저장, 조회, 수정, 삭제할 수 있습니다.

- 계층형(Hierarchical)
- 망형(Network)
- 관계형(Relational)  (가장 많이 사용됨)

---

- ### RDBMS 예시
```text
- Oracle
- MySQL
- MariaDB
- SQL Server
- DB2
```
---

## 관계형 데이터베이스의 기본 구조

- 관계형 데이터베이스는 데이터를 테이블(Table) 형태로 저장합니다.

| 요소 | 설명 |
|------|------|
| 행(Row) | 실제 데이터 한 단위(레코드) |
| 열(Column) | 데이터 속성 |
| 테이블(Table) | 동일 구조의 행 집합 |

- 모든 테이블은 공통 컬럼을 통해 관계를 맺으며 JOIN이 가능합니다.

```text
학생(student)        과목(subject)
────────────────────────────────────────
학번 | 이름   | 학과      과목코드  | 과목명
101 | 김철수 | 컴퓨터      C101   | 데이터베이스
102 | 이영희 | 전자       C102   | 네트워크
``` 

- 관계(Relation)는 학번, 과목코드 등을 기준으로 연결됩니다.

---

## SQL(Structured Query Language)

- SQL은 관계형 데이터베이스에서 데이터를 조작하고 관리하는 표준 언어입니다.

### SQL 언어 분류

| 종류 | 의미 | 예 |
|------|------|------|
| DML | 데이터 조작 | SELECT, INSERT, UPDATE, DELETE |
| DDL | 데이터 구조 정의 | CREATE, ALTER, DROP, RENAME |
| DCL | 권한 제어 | GRANT, REVOKE |
| TCL | 트랜잭션 제어 | COMMIT, ROLLBACK |

---

## SELECT문 구조와 사용법

- SELECT 문은 데이터 조회의 핵심이 되는 구문입니다.
```sql
SELECT DISTINCT 컬럼명1, 컬럼명2 AS 별칭  
FROM 테이블명  
WHERE 조건  
ORDER BY 정렬기준;
```
- `*` 은 전체 컬럼을 의미합니다.
- AS 는 컬럼에 별칭을 부여할 때 사용합니다.

- `예시`
```sql
SELECT empno, ename AS 이름, sal AS 급여  
FROM emp  
WHERE deptno = 10  
ORDER BY sal DESC;
```
---

## Projection / Selection / Join

- Projection  
  특정 컬럼만 선택합니다.

- Selection  
  조건에 맞는 행을 선택합니다.

- Join  
  여러 테이블을 하나의 결과로 결합합니다.

- `JOIN 예시`
```sql
SELECT emp.ename, dept.dname  
FROM emp  
JOIN dept ON emp.deptno = dept.deptno;
```
---

## 테이블 구조 확인

- 테이블 목록 확인
```sql
SELECT * FROM tab;
```
- 테이블 구조 확인
```sql
DESC emp;
```
- 오라클 기준에서 스키마(schema)는 사용자(user)를 의미합니다.

---

## SQL 문법 규칙

- SQL 문장은 반드시 세미콜론으로 종료합니다.
- 대소문자를 구분하지 않습니다. 단, 문자열 데이터는 예외입니다.
- 여러 줄로 작성이 가능합니다.
- 실행 시 항상 결과가 반환됩니다.

---

## 데이터 타입

| 타입 | 설명 | 예 |
|------|------|------|
| VARCHAR2(n) | 가변 길이 문자 | 홍길동 |
| NUMBER(n) | 숫자형 | 100, 3.14 |
| DATE | 날짜 | 2025-10-23 |

- 숫자형 데이터는 우측 정렬됩니다.
- 문자형 데이터는 좌측 정렬됩니다.

---

## 무결성(Integrity)

- 무결성은 데이터에 오류나 중복이 없도록 보장하는 개념입니다.

- 각 테이블은 반드시 Primary Key를 가져야 합니다.
- Foreign Key를 통해 테이블 간 관계를 유지합니다.

- `예시`
```sql
CREATE TABLE student (  
  student_id NUMBER PRIMARY KEY,  
  name VARCHAR2(50),  
  dept_id NUMBER REFERENCES department(dept_id)  
);
```

---

## 정리

- DBMS는 데이터를 저장하고 관리하는 시스템입니다.
- RDBMS는 관계형 데이터베이스를 관리합니다.
- SQL은 데이터 조작·정의·제어를 위한 표준 언어입니다.
- SELECT 문은 데이터 조회의 핵심입니다.
- 무결성과 키 설정은 데이터 품질을 결정하는 핵심 요소입니다.

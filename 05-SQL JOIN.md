# WHERE 절과 조건 검색

## 개요

- 본 문서는 SQL에서 전통 JOIN을 정리한 문서입니다.

---

## JOIN (전통 표기 방식)

- JOIN은 서로 다른 테이블의 관련 데이터를 연결하여 조회하는 방식입니다.
- 전통 방식 JOIN은 FROM 절에 여러 테이블을 나열하고 WHERE 절에서 조인 조건을 지정합니다.

### 사원 테이블과 부서 테이블을 연결하는 예시는 다음과 같습니다.
```sql
SELECT e.eno, e.ename, d.dname, d.loc
FROM emp e, dept d
WHERE e.dno = d.dno;
```
---

## 전통 JOIN 기본 구조

### 전통 JOIN의 기본 형태는 다음과 같습니다.
```sql
SELECT 컬럼들  
FROM 테이블A a, 테이블B b  
WHERE a.공통키 = b.공통키  
AND 추가 조건 …
```
- 조인 조건은 반드시 WHERE 절에 작성해야 합니다.
- 동일한 컬럼명이 존재할 경우 테이블 별칭을 사용해야 합니다.
- 추가 조건은 AND로 연결합니다.

---

## 전통 JOIN 예제

### 화학 학과 교수가 강의하는 과목을 조회하는 예시는 다음과 같습니다.
```sql
SELECT cname
FROM course, professor
WHERE course.pno = professor.pno
AND professor.section = '화학';
```
### 2학점 과목을 강의하는 교수 이름을 조회하는 예시는 다음과 같습니다.
```sql
SELECT p.pname
FROM course c, professor p
WHERE c.pno = p.pno
AND c.st_num = 2
ORDER BY p.pname;
```
### 과목명, 학점, 교수명을 함께 조회하는 예시는 다음과 같습니다.
```sql
SELECT c.cname, c.st_num, p.pname
FROM course c, professor p
WHERE c.pno = p.pno
AND c.st_num = 2
ORDER BY c.cname, p.pname;
```
---

## 카테시안 곱 주의

- 조인 조건을 누락하면 두 테이블의 모든 행이 곱해진 결과가 생성됩니다.
- 이는 의도하지 않은 대량의 결과를 발생시킵니다.

### 잘못된 예시는 다음과 같습니다.
```sql
SELECT *
FROM course c, professor p;
```
- 항상 WHERE 절에 조인 조건을 포함해야 합니다.

---

## 정리

- 전통 JOIN은 WHERE 절에 조인 조건을 반드시 포함해야 합니다.

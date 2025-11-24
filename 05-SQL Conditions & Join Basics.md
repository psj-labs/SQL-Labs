## 1) SELECT 기본 구조
SELECT 컬럼명 또는 *  
FROM 테이블명  
WHERE 조건  
ORDER BY 컬럼 ASC|DESC;

- WHERE : 조건 검색(행 단위 필터링)  
- ORDER BY : 정렬 (ASC 오름차순 / DESC 내림차순)  
- WHERE 절에는 별명(alias) 사용 불가

예제:  
SELECT *  
FROM emp  
WHERE sal >= 3000  
ORDER BY sal DESC;

---

## 2) NULL 과 비교
NULL은 "값이 없음" 의미이며 0이나 빈 문자열과 다르다.  
비교연산자(=, !=)로 비교 불가.  
IS NULL / IS NOT NULL 사용.

SELECT * FROM emp WHERE comm IS NULL;  
SELECT * FROM emp WHERE comm IS NOT NULL;

---

## 3) BETWEEN 연산자
컬럼 BETWEEN 값1 AND 값2 는 값1 ≤ 컬럼 ≤ 값2 와 동일.

SELECT eno, ename, sal  
FROM emp  
WHERE sal BETWEEN 2000 AND 4000;

- 가독성 좋음  
- 값1 < 값2 이어야 정상 동작  
- 날짜 비교 시 시분초 문제 주의(0시 기준 저장이 가장 안전함)

---

## 4) IN 연산자
다중 OR 조건을 단순하게 표현하는 방식.

SELECT ename, job  
FROM emp  
WHERE job IN ('CLERK', 'MANAGER', 'SALESMAN');

아래와 동일:  
job = 'CLERK' OR job = 'MANAGER' OR job = 'SALESMAN'

- 가독성 높음  
- 서브쿼리와 함께 자주 사용됨

---

## 5) JOIN (전통 표기: 쉼표 + WHERE)
서로 다른 테이블의 관련 데이터를 연결하여 조회하는 것.

SELECT e.eno, e.ename, d.dname, d.loc  
FROM emp e, dept d  
WHERE e.dno = d.dno;

---

# 🔗 조인(JOIN) — 전통 표기(쉼표 + WHERE) 정리

## 1) 기본 형태
SELECT 컬럼들  
FROM 테이블A a, 테이블B b  
WHERE a.공통키 = b.공통키  
-- AND 추가 조건 …

- 조인 조건은 WHERE에 반드시 작성  
- 동일 컬럼명 충돌 시 a.컬럼, b.컬럼 사용  
- 추가 조건은 AND로 이어 붙임

---

## 2) 예제 ① — 학과가 ‘화학’인 교수가 강의하는 과목
SELECT cname  
FROM course, professor  
WHERE course.pno = professor.pno  
AND professor.section = '화학';

해석  
1) course와 professor를 pno로 연결  
2) section이 '화학'인 교수만 필터  
3) cname 출력

---

## 3) 예제 ② — 2학점 과목을 강의하는 교수 이름
SELECT p.pname  
FROM course c, professor p  
WHERE c.pno = p.pno  
AND c.st_num = 2  
ORDER BY p.pname;

중복 제거 필요 시: SELECT DISTINCT p.pname …

---

## 4) 예제 ③ — 과목명 + 학점 + 교수명
SELECT c.cname, c.st_num, p.pname  
FROM course c, professor p  
WHERE c.pno = p.pno  
AND c.st_num = 2  
ORDER BY c.cname, p.pname;

---

## 5) 카테시안 곱(주의)
조인 조건을 누락하면 두 테이블 행 수가 곱해짐.

SELECT * FROM course c, professor p;

항상 조인 조건을 WHERE에 포함해야 한다.

---

## 6) LIKE / BETWEEN / IN 과 함께 쓰기

### 이름에 ‘민’ 포함된 교수의 과목
SELECT c.cname, p.pname  
FROM course c, professor p  
WHERE c.pno = p.pno  
AND p.pname LIKE '%민%';

### 학점이 2~3 사이인 과목
SELECT c.cname, c.st_num, p.pname  
FROM course c, professor p  
WHERE c.pno = p.pno  
AND c.st_num BETWEEN 2 AND 3;

### 화학/물리 학과 교수
SELECT p.pname, p.section, c.cname  
FROM course c, professor p  
WHERE c.pno = p.pno  
AND p.section IN ('화학', '물리');

---

## 7) 실수 방지 체크리스트
- WHERE 절에 조인 조건을 넣었는가?  
- AND/OR 빠뜨리지 않았는가?  
- 숫자 비교에 문자열 리터럴('2') 사용하지 않았는가?  
- 동일 컬럼명일 때 테이블 별칭을 썼는가?  

---

※ 참고: 전통 표기는 ANSI JOIN … ON … 과 동일 의미.  
현대 SQL은 JOIN 문법을 권장하나, 본 문서는 수업 방식에 맞춰 쉼표+WHERE 형식 사용.

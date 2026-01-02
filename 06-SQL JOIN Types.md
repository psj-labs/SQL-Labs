# SQL 조인(Join) 정리

## 개요

- 본 문서는 SQL에서 내부 조인, 셀프 조인, 외부 조인의 개념과 사용 방법을 정리한 문서입니다.

---

## 내부 조인

- 내부 조인은 두 테이블에서 조인 조건을 만족하는 행만 조회하는 방식입니다.
- 가장 기본이 되는 조인 방식이며, 조건에 맞지 않는 행은 결과에서 제외됩니다.

### 다음은 표준 JOIN 문법을 사용한 내부 조인 예시입니다.
```sql
SELECT A.col, B.col  
FROM 테이블A A  
JOIN 테이블B B  
ON A.공통컬럼 = B.공통컬럼;
```
- 과거에는 JOIN 키워드를 사용하지 않고 다음과 같은 방식으로 내부 조인을 수행했습니다. 이를 암시적 Inner Join이라고 합니다.
```sql
SELECT ...  
FROM 테이블A A, 테이블B B  
WHERE A.공통컬럼 = B.공통컬럼;
```
---

## 셀프 조인 (Self Join)

- 셀프 조인은 하나의 테이블을 두 번 불러와 내부 데이터를 서로 비교하는 조인 방식입니다.
- 동일 테이블 내에서 관계를 찾아야 할 때 사용합니다.

### 동명이인을 찾는 예시는 다음과 같습니다.
```sql
SELECT DISTINCT s1.sno, s1.sname  
FROM student s1, student s2  
WHERE s1.sname = s2.sname  
AND s1.sno != s2.sno;
```
### DISTINCT를 사용하는 이유는 다음과 같습니다.

- 이름은 같지만 학번이 다른 행이 여러 번 출력되는 것을 방지하기 위함입니다.
- sno와 sname 조합이 완전히 동일한 행만 중복 제거 대상이 됩니다.

### 다음은 표준 JOIN 문법을 사용한 셀프 조인 예시입니다.
```sql
SELECT DISTINCT s1.sno, s1.sname  
FROM student s1  
JOIN student s2  
ON s1.sname = s2.sname  
AND s1.sno != s2.sno;
```
- 하나의 테이블을 두 번 사용하므로 Self Inner Join에 해당합니다.

---

## 외부 조인 (Outer Join)

- 외부 조인은 조인 조건을 만족하지 않는 행도 결과에 포함시키는 방식입니다.
- Oracle 고전 문법에서는 (+) 기호를 사용합니다.

### 교수는 존재하지만 담당 과목이 없는 경우를 조회하는 예시는 다음과 같습니다.
```sql
SELECT p.pname, c.cname  
FROM professor p, course c  
WHERE p.pno = c.pno(+)  
ORDER BY p.section;
```
- (+)가 course 테이블 쪽에 있으므로 과목이 없어도 교수 정보는 출력됩니다.  
- 이는 LEFT OUTER JOIN과 동일한 동작입니다.

### 다음은 표준 JOIN 문법을 사용한 외부 조인 예시입니다.
```sql
SELECT p.pname, c.cname  
FROM professor p  
LEFT JOIN course c  
ON p.pno = c.pno  
ORDER BY p.section;
```
---

## 조인 방식 요약

- 내부 조인은 조건이 일치하는 행만 출력합니다.
- 셀프 조인은 하나의 테이블을 두 번 사용하여 내부 데이터를 비교합니다.
- 외부 조인은 매칭되지 않은 행도 함께 출력합니다.
- Oracle의 (+) 문법은 표준 LEFT JOIN으로 이해해야 합니다.

---

## 정리

- 조인은 어떤 행을 남길지 결정하는 과정입니다.
- 기준 테이블과 조인 방향을 명확히 이해해야 합니다.

## SELECT 실행 논리 순서(중요)
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY  
이 순서 때문에 WHERE 절에는 SELECT 별칭(alias)을 사용할 수 없고, ORDER BY에서는 별칭이나 정렬 순번을 사용할 수 있습니다.

---

## 1) WHERE 기본 형태와 핵심 개념
WHERE는 행을 제한하는 필터링 기능입니다.  
비교 연산자(=, >, >=, <, <=, <>), 논리 연산자(AND, OR, NOT)를 사용합니다.  
정확한 조건을 작성하려면 테이블 컬럼명과 타입을 정확히 알아야 합니다.

예시: 급여가 3000 이상인 사원만 조회  
SELECT eno, ename, sal  
FROM emp  
WHERE sal >= 3000  
ORDER BY sal DESC;

ORDER BY 팁: ORDER BY 1, 3 DESC 처럼 출력 컬럼 순번도 가능하지만 가독성 때문에 컬럼명/별칭 권장.

---

## 2) 날짜 출력 형식(Oracle)
세션 날짜 포맷을 고정하면 결과가 일관됩니다.  
ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY/MM/DD';

TO_DATE 변환 예시:  
INSERT INTO emp_log (eno, hdate) VALUES ('E001', TO_DATE('2025/10/26', 'YYYY/MM/DD'));  
SYSDATE 저장 시 시분초 포함됨.

주의: DATE 타입은 시분초 포함. 날짜만 비교하려면 TRUNC 사용 또는 반개구간(>= 시작일 AND < 종료일+1) 방식 권장.

TRUNC 예시:  
SELECT * FROM orders  
WHERE TRUNC(order_date) BETWEEN DATE '2025-10-01' AND DATE '2025-10-31';

반개구간 예시:  
SELECT * FROM orders  
WHERE order_date >= DATE '2025-10-01'  
AND   order_date <  DATE '2025-11-01';

---

## 3) NULL 비교
NULL은 “값 없음”.  
일반 비교연산자(=, <>, > …)로 비교 불가.  
IS NULL, IS NOT NULL 사용해야 한다.

예시:  
SELECT * FROM emp WHERE comm IS NULL;  
SELECT * FROM emp WHERE comm = NULL;  -- 결과 없음(잘못된 비교)

COUNT 컬럼은 NULL 제외, COUNT(*)는 NULL 포함.

---

## 4) BETWEEN … AND (양끝 범위 포함)
BETWEEN은 포함 범위이며 값1 <= 컬럼 <= 값2 와 동일.  
값1은 값2보다 작거나 같아야 한다.

숫자 예시:  
SELECT eno, ename, sal  
FROM emp  
WHERE sal BETWEEN 2000 AND 3000  
ORDER BY sal;

날짜 예시는 시분초 때문에 반개구간 추천:  
SELECT * FROM orders  
WHERE order_date >= DATE '2025-10-01'  
AND   order_date <  DATE '2025-11-01';

잘못된 오해: 날짜는 반드시 0시로 저장돼야 한다는 규칙은 없음. 저장 방식보다 비교 로직이 더 중요하다.

---

## 5) IN 연산자
목록 중 하나라도 일치하면 참.  
= OR OR 여러 개 나열한 것보다 가독성·유지보수성 좋음.  
서브쿼리와 함께 자주 사용.

예시: 부서가 10,20,30인 사원  
SELECT eno, ename, dno  
FROM emp  
WHERE dno IN (10,20,30)  
ORDER BY dno, ename;

서브쿼리 예시:  
SELECT eno, ename  
FROM emp  
WHERE dno IN (SELECT dno FROM dept WHERE loc = 'SEOUL');

주의: NOT IN + NULL → UNKNOWN 발생 → 결과 사라짐.  
안전하게 NOT EXISTS 패턴 권장.

NOT EXISTS 예시:  
SELECT e.*  
FROM emp e  
WHERE NOT EXISTS (SELECT 1 FROM blacklist b WHERE b.eno = e.eno);

---

## 6) 자주 쓰는 조건식 패턴 모음
부분 문자열 검색: WHERE ename LIKE '%KIM%'  
NULL 대체 후 비교: WHERE NVL(comm, 0) >= 500  
하루 구간: WHERE order_date >= DATE '2025-10-26' AND order_date < DATE '2025-10-27'  
정렬(별칭): SELECT sal*12 AS annual FROM emp ORDER BY annual DESC  
다중 OR: WHERE dno IN (10,20,30)

---

## 7) 미니 실습 문제

1) 급여 2000~3500(포함) + 커미션 NULL인 사원  
SELECT eno, ename, sal  
FROM emp  
WHERE sal BETWEEN 2000 AND 3500  
AND comm IS NULL  
ORDER BY sal DESC;

2) 서울 또는 부산 부서의 사원  
SELECT ename, dno  
FROM emp  
WHERE dno IN (SELECT dno FROM dept WHERE loc IN ('SEOUL','BUSAN'))  
ORDER BY ename;

3) 2025년 10월 한 달 주문(시분초 섞여 있음 → 반개구간 방식)  
SELECT *  
FROM orders  
WHERE order_date >= DATE '2025-10-01'  
AND   order_date <  DATE '2025-11-01';

---

## 8) 메모 수정 요약
- WHERE 절에는 별칭 사용 불가(실행 순서 때문).  
- DATE는 시분초 포함 → 비교 로직을 안전하게.  
- BETWEEN은 포함 범위.  
- NOT IN + NULL은 위험 → NOT EXISTS 권장.

예시 테이블: emp(eno, ename, sal, comm, dno…), dept(dno, dname, loc…), orders(order_id, order_date…)

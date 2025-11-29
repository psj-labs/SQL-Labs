## 서브쿼리를 이용한 테이블 생성

```sql
CREATE TABLE 테이블명 [(컬럼 [DEFAULT 값], ...)]
AS (SELECT ... );
```

- 서브쿼리 결과를 그대로 새 테이블로 생성
- 컬럼 목록 생략 시 조회된 컬럼명이 그대로 사용됨
- `DEFAULT` 값 설정 가능
- 테이블 복제 또는 컬럼 일부 추출 시 사용

### 예시

```sql
CREATE TABLE emp_copy
AS SELECT empno, ename, sal
FROM emp
WHERE deptno = 10;
```

---

## 테이블 관리 – ALTER TABLE

```sql
ALTER TABLE 테이블명
  ADD (컬럼 데이터타입 [DEFAULT 값]);

ALTER TABLE 테이블명
  MODIFY (컬럼 데이터타입 [DEFAULT 값]);

ALTER TABLE 테이블명
  DROP COLUMN 컬럼명;
```

ALTER TABLE로 가능한 작업

- 컬럼 추가 (ADD)
- 컬럼 변경 (MODIFY)
- 컬럼 삭제 (DROP COLUMN)

---

## CREATE / ALTER / DROP 구분

- **CREATE** → 생성
- **ALTER** → 구조 수정
- **DROP** → 객체 삭제

> 유의사항
- 데이터 존재 시 **데이터 타입 변경 불가**
- SYS/SYSTEM 계정 소유 테이블은 `DROP COLUMN` 불가
- `NOT NULL` 제약 → **ADD 불가 / MODIFY로 변경**

---

## 컬럼 변경 (MODIFY)

```sql
ALTER TABLE emp2
  MODIFY (hdate DEFAULT SYSDATE);

DESC emp2;
SELECT * FROM emp2;
```

**동작 특성**

- `SYSDATE` : 입력 시 날짜 자동 등록
- 기존 데이터 수용 가능 범위면 타입·길이 수정 가능
- 행이 없으면 제한 없이 수정 가능
- 변경 결과는 **이후 INSERT부터 적용**
- 실무에서 컬럼 수정은 빈번하지 않음

---

## 컬럼 삭제 (DROP COLUMN)

```sql
ALTER TABLE emp2
  DROP COLUMN 부서번호;

DESC emp2;
SELECT * FROM emp2;
```

**주의 사항**

- 데이터 존재 여부와 관계없이 삭제 가능
- 컬럼은 **1회에 1개만 삭제**
- 테이블은 항상 **최소 1컬럼 유지**
- 삭제된 컬럼 **복구 불가**
- SYS 계정 테이블 → 삭제 불가
- 운영 환경에서 **DROP 계열 사용은 극히 제한**

---

## 테이블 내용 완전 삭제 – TRUNCATE

```sql
TRUNCATE TABLE 테이블명;
```

**특징**

- 모든 행 + 공간 완전 삭제
- **DDL 작업 → ROLLBACK 불가**
- `DELETE FROM` 대비 속도 매우 빠름
- 공간까지 해제
- `DELETE` → COMMIT 필요
- `TRUNCATE` → **자동 COMMIT**

### 예시

```sql
TRUNCATE TABLE emp2;
```

---

## 테이블 이름 변경 – RENAME

```sql
RENAME OLD_NAME TO NEW_NAME;
```

테이블, 뷰, 시퀀스 등 객체 이름 변경 가능.

```sql
RENAME emp2 TO eemp;
SELECT * FROM tab;
```

---

## DROP 사용 주의

**실제 운영 환경에서 DROP 사용은 거의 금기**  
DROP은 **테이블 구조 자체를 통째로 삭제**하므로  
복구가 사실상 불가능하다.

---

## 요약

- **CREATE AS** → 테이블 복사
- **ALTER** → 구조 수정
- **TRUNCATE** → 데이터 완전 초기화
- **RENAME** → 객체 이름 변경
- **DDL은 자동 COMMIT → ROLLBACK 불가**

## 한눈에 보기

| 분류 | 명령 | 핵심 기능 | 비고 |
|------|------|-----------|------|
| **DML** | `INSERT` | 행 삽입 | 트랜잭션 안에서 동작 |
| | `UPDATE` | 행 수정 | **WHERE 없으면 전체 수정** |
| | `DELETE` | 행 삭제 | **WHERE 없으면 전체 삭제** |
| **TCL** | `COMMIT` | 작업 확정(영구 저장) | 커밋 전까지 되돌리기 가능 |
| | `ROLLBACK` | 직전 커밋 시점으로 복구 | 미커밋 변경만 취소 |

---

## INSERT 기본 문법

```sql
INSERT INTO 테이블명 [(컬럼1, 컬럼2, ...)]
VALUES (값1, 값2, ...);
```

### 날짜 입력 (Oracle)

세션 날짜 포맷 의존 X → **`TO_DATE` 사용 권장**

```sql
INSERT INTO emp (eno, ename, hdate)
VALUES ('0201', '안영숙',
        TO_DATE('2021/09/25:03:07:15', 'YYYY/MM/DD:HH24:MI:SS'));
```

- `YYYY` 연
- `MM` 월
- `DD` 일
- `HH24` 시(0~23)
- `MI` 분
- `SS` 초

대안:

```sql
DATE '2021-09-25'
```

---

## 3) UPDATE / DELETE 기본 문법

### UPDATE

```sql
UPDATE 테이블명
   SET 컬럼1 = 값1, 컬럼2 = 값2, ...
 WHERE 조건;   -- 중요
```

### DELETE

```sql
DELETE FROM 테이블명
 WHERE 조건;   -- 없으면 전체 행 삭제
```

---

### 예시: 특정 사원 부서 이동 및 급여 인상

```sql
UPDATE emp
   SET dno = '10',
       sal = sal * 1.10
 WHERE ename = '김주란';

COMMIT;
```

---

### 예시: 부서 10 전원 삭제

```sql
DELETE FROM emp
 WHERE dno = '10';

COMMIT;
```

---

## 트랜잭션 (TCL)

**트랜잭션 = 데이터 변경 작업의 논리적 단위**

- 커밋 전 → 임시 상태
- `COMMIT` → 영구 저장
- `ROLLBACK` → 마지막 커밋 시점으로 복구

### 처리 흐름

1. INSERT / UPDATE / DELETE 수행
2. 결과 검증
3. 문제 없으면 `COMMIT;`
4. 취소할 경우 `ROLLBACK;`

---

## 실무에서 자주 터지는 사고

- **WHERE 빠트림 → 전체 수정/삭제**
    - 항상 먼저 `SELECT ... WHERE ...`로 대상 검증
- 커밋 안 하고 종료 → 변경 데이터 소실
- **DDL 실행(CREATE / ALTER / DROP) → 자동 COMMIT**
    - 이후 ROLLBACK 불가
- 대량 작업 전:

```sql
SAVEPOINT p1;
ROLLBACK TO p1;
```

---

## 실무 체크리스트

- 변경 전 → SELECT로 대상 검증
- 변경 후 → 영향 행 수 확인 (`SQL%ROWCOUNT`)
- 날짜/숫자 → `TO_DATE`, `TO_NUMBER` 등 **명시 변환**
- 대량 작업 → `SAVEPOINT`
- DDL & DML 혼합 실행 주의

---

## 요약 문법 카드

### INSERT

```sql
INSERT INTO T (c1, c2)
VALUES (v1, v2);
```

---

### UPDATE

```sql
UPDATE T
   SET c1 = v1
 WHERE 조건;
```

---

### DELETE

```sql
DELETE FROM T
 WHERE 조건;
```

---

### TCL

```sql
COMMIT;
ROLLBACK;
SAVEPOINT p1;
ROLLBACK TO p1;
```

---

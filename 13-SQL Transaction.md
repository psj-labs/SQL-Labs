## ACID 특성

| 특성 | 설명 | 한 줄 예시 |
|------|------|-------------|
| **Atomicity (원자성)** | 트랜잭션은 전부 수행되거나 전혀 수행되지 않아야 한다 | 출금 성공 + 입금 실패 → `ROLLBACK` |
| **Consistency (일관성)** | 실행 전후 데이터는 항상 제약/규칙을 만족 | 잔액 음수 금지 |
| **Isolation (고립성)** | 동시에 실행돼도 중간 결과를 서로 보지 않음 | A의 미커밋 결과는 B가 못 봄 |
| **Durability (영속성)** | `COMMIT`된 결과는 영구 보존 | 재시작 후에도 이체 기록 유지 |

---

## 트랜잭션 시작 & 종료

### 시작 시점

- 이전 트랜잭션 종료 후 **DML 수행 시**
  - `INSERT`, `UPDATE`, `DELETE`
- 일부 DB/툴은
  - `AUTOCOMMIT OFF` 상태에서 첫 SQL 실행과 함께 시작

### 종료 시점

- `COMMIT` 실행
- `ROLLBACK` 실행
- **DDL 실행**
  - `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
  - → 보통 **자동 커밋**
- **DCL 실행**
  - `GRANT`, `REVOKE`
- 세션 종료 (환경에 따라 자동 커밋 또는 롤백)
- 교착상태(Deadlock) 감지 시 한 쪽이 강제 `ROLLBACK`

> 기억하기  
> **DML → 명시적 COMMIT / ROLLBACK 필요**  
> **DDL → 자동 커밋 발생**

---

## UNDO Segment & Lock

### UNDO Segment

- 변경 전 데이터의 **이전 이미지 저장**
- `ROLLBACK` 기반 데이터 복구 가능
- **읽기 일관성(Read Consistency)** 제공
  - 다른 세션은
  - 변경 중이더라도 **커밋 전 상태를 조회**

### 잠금(Lock)

#### Exclusive Lock (배타 잠금)

- DML 수행 시 해당 행(Row)에 적용
- 다른 세션의 수정·삭제 차단

#### Share Lock (공유 잠금)

- 테이블 구조 변경 충돌 방지
- 일반 SELECT는 차단하지 않음

> 커밋 가시성  
> **다른 세션은 오직 “커밋된 데이터”만 조회 가능**  
> 미커밋 변경 내용은 절대 노출되지 않음

---

## Deadlock (교착상태)

서로 잠금을 잡은 채 상대 자원을 기다리며  
**영원히 대기하는 상태**

### 사례

```sql
-- 세션 A
UPDATE emp SET sal = sal * 1.2 WHERE ename = '문시현';

-- 세션 B
UPDATE emp SET sal = sal * 1.2 WHERE ename = '안영희';

-- 이후 서로 잠긴 행을 교차 접근 → 교착

-- DB는 한쪽을 강제 ROLLBACK하여 교착 해소
```

### 예방 팁

- 갱신 대상 접근 순서 **일관성 유지**
- 트랜잭션은 **짧게 유지**
- 불필요한 대기/입력은 트랜잭션 밖에서 처리
- 락 힌트, 격리수준 남용 금지

---

## 실습 시나리오

### 미커밋 가시성

```sql
-- Session A
UPDATE emp SET sal = sal + 100 WHERE eno = 1001;

-- Session B
SELECT sal FROM emp WHERE eno = 1001;
-- 변경 전 값 조회(UNDO 기반)

-- Session A
ROLLBACK;

-- Session B
SELECT sal FROM emp WHERE eno = 1001;
```

---

### 커밋 가시성

```sql
-- Session A
UPDATE emp SET sal = sal + 100 WHERE eno = 1001;
COMMIT;

-- Session B
SELECT sal FROM emp WHERE eno = 1001;
-- 변경값 조회 가능
```

---

### DDL 자동 커밋

```sql
-- Session A
INSERT INTO dept VALUES (99, 'TEST', 'SEOUL');

CREATE INDEX ix_emp_sal ON emp(sal);
-- DDL 수행 → 자동 커밋
-- INSERT도 되돌릴 수 없음
```

---

## 6) 자주 하는 오해

- ❌ SELECT도 잠금을 건다  
  ✅ 일반 SELECT는 잠금 없이 읽기 일관성만 유지

- ❌ DDL은 ROLLBACK 가능  
  ✅ 대부분 자동 커밋 → 취소 불가

- ❌ COMMIT 안 해도 다른 세션이 본다  
  ✅ 미커밋 데이터는 절대 외부 세션에 보이지 않음

---

## 실전 체크리스트

- 트랜잭션 단위 구분 명확한가?
- 사용자 입력 대기 등을 트랜잭션 안에 넣지 않았는가?
- 다중 자원 접근 순서 통일했는가?
- DDL 자동 커밋 특성을 고려한 배포/테스트 설계가 되어 있는가?

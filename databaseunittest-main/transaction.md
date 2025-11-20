

# 🚀 SQL 트랜잭션(Transaction)

---

# 1️⃣ 트랜잭션(Transaction)이란?

**트랜잭션은 여러 SQL 작업을 하나의 논리적 단위로 묶어
모두 성공하거나(All) 모두 실패(Nothing)하도록 보장하는 기술이다.**

✔ 성공 → COMMIT
✔ 실패 → ROLLBACK

---

# 2️⃣ 트랜잭션의 핵심: ACID

ACID 는 트랜잭션을 정의하는 4대 원칙이다.

| 원칙                        | 설명                        |
| ------------------------- | ------------------------- |
| **A — Atomicity (원자성)**   | 모든 작업이 전부 수행되거나 전부 취소됨    |
| **C — Consistency (일관성)** | 트랜잭션 전후 데이터의 논리적 일관성 유지   |
| **I — Isolation (격리성)**   | 동시에 실행되는 트랜잭션이 서로 간섭하지 않음 |
| **D — Durability (지속성)**  | COMMIT된 데이터는 영구 저장됨       |

---

# 3️⃣ 트랜잭션 흐름

1. `START TRANSACTION`
2. SQL 작업들 수행
3. `COMMIT` 또는 `ROLLBACK`
4. 종료

---

# 4️⃣ 격리 수준 (Isolation Level)

동시성 문제(Dirty Read, Phantom Read 등)를 막기 위한 설정.

| Level                          | 설명                            |
| ------------------------------ | ----------------------------- |
| **READ UNCOMMITTED**           | 커밋되지 않은 데이터 읽기(더티 Read 발생 가능) |
| **READ COMMITTED**             | 커밋된 데이터만 읽음                   |
| **REPEATABLE READ** (MySQL 기본) | 같은 SELECT 동일 결과 보장            |
| **SERIALIZABLE**               | 완전 직렬 처리(가장 안전하지만 느림)         |

---

# 5️⃣ 실습용 테이블 생성

트랜잭션 실습을 위해 은행 계좌(Account) 구조를 사용합니다.

```sql
CREATE TABLE Account (
    id        INT NOT NULL AUTO_INCREMENT,
    user_name VARCHAR(50),
    balance   INT,
    PRIMARY KEY (id)
);
```

---

# 6️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Account (user_name, balance) VALUES
('김철수', 100000),
('이영희', 50000),
('박민수', 80000),
('최가영', 120000);
```

---

# 7️⃣ 트랜잭션 실습 예제


---

## ✔ 예제 1) 기본 트랜잭션 시작 & 커밋

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 10000 WHERE user_name = '김철수';
UPDATE Account SET balance = balance + 10000 WHERE user_name = '이영희';

COMMIT;
```

### 결과

| user_name | balance |
| --------- | ------- |
| 김철수       | 90000   |
| 이영희       | 60000   |

---

## ✔ 예제 2) 트랜잭션 롤백

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 50000 WHERE user_name = '김철수';
UPDATE Account SET balance = balance + 50000 WHERE user_name = '박민수';

ROLLBACK;
```

### 결과 (변경 없음)

| user_name | balance |
| --------- | ------- |
| 김철수       | 90000   |
| 박민수       | 80000   |

---

## ✔ 예제 3) 오류 발생 시 전체 롤백

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 20000 WHERE user_name = '김철수';
UPDATE Account SET balance_typo = balance + 20000 WHERE user_name = '이영희';  -- 오류!

ROLLBACK;
```

### 결과

→ 모든 변경사항 취소됨

---

## ✔ 예제 4) CHECK 제약조건 위반 시 롤백

(잔액이 음수면 롤백되는 시나리오)

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 200000 WHERE user_name = '김철수';

ROLLBACK;
```

### 결과

잔액 변경 없음

---

## ✔ 예제 5) 두 계좌 간 송금 처리 (원자성)

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 30000 WHERE user_name = '이영희';
UPDATE Account SET balance = balance + 30000 WHERE user_name = '최가영';

COMMIT;
```

---

## ✔ 예제 6) WHERE 조건을 잘못 걸어서 롤백

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 10000 WHERE user_name LIKE '%영%';  -- 여러 명!

ROLLBACK;
```

---

## ✔ 예제 7) 하나의 트랜잭션 안에서 여러 UPDATE

```sql
START TRANSACTION;

UPDATE Account SET balance = balance + 1000 WHERE user_name = '김철수';
UPDATE Account SET balance = balance + 2000 WHERE user_name = '박민수';
UPDATE Account SET balance = balance + 3000 WHERE user_name = '최가영';

COMMIT;
```

---

## ✔ 예제 8) SAVEPOINT 사용

```sql
START TRANSACTION;

UPDATE Account SET balance = balance - 10000 WHERE user_name = '김철수';
SAVEPOINT sp1;

UPDATE Account SET balance = balance - 50000 WHERE user_name = '김철수';

ROLLBACK TO sp1;   -- 마지막 작업만 취소

COMMIT;
```

---

## ✔ 예제 9) READ UNCOMMITTED로 Dirty Read 실습 (이론 시나리오)

세션 A:

```sql
START TRANSACTION;
UPDATE Account SET balance = 0 WHERE user_name = '이영희';
```

세션 B:

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM Account;  -- Dirty Read 발생 가능
```

세션 A:

```sql
ROLLBACK;
```

---

## ✔ 예제 10) READ COMMITTED – Dirty Read 방지

세션 B:

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM Account;  
```

결과 → COMMIT 전 데이터 보이지 않음.

---

## ✔ 예제 11) REPEATABLE READ — Phantom Read 실습

세션 A:

```sql
START TRANSACTION;
SELECT * FROM Account WHERE balance >= 80000;
```

세션 B:

```sql
INSERT INTO Account(user_name, balance) VALUES('새사용자', 90000);
COMMIT;
```

세션 A 다시 조회:

```sql
SELECT * FROM Account WHERE balance >= 80000;
```

MySQL에서는 공격적인 MVCC로 **Phantom Read 방지됨**

---

## ✔ 예제 12) SERIALIZABLE – 완전 직렬화 시나리오

세션 A:

```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
SELECT * FROM Account;
```

세션 B:

```sql
INSERT INTO Account VALUES (NULL, '지연사용자', 50000);
-- 세션 A가 종료될 때까지 BLOCK
```

---

## ✔ 예제 13) SELECT … FOR UPDATE

(락 걸고 업데이트)

```sql
START TRANSACTION;

SELECT * FROM Account
WHERE user_name = '김철수'
FOR UPDATE;

UPDATE Account SET balance = balance - 1000 WHERE user_name = '김철수';

COMMIT;
```

---

## ✔ 예제 14) SELECT … LOCK IN SHARE MODE

(읽기 락)

```sql
START TRANSACTION;

SELECT * FROM Account
WHERE user_name = '박민수'
LOCK IN SHARE MODE;

COMMIT;
```

---

## ✔ 예제 15) 트랜잭션 안의 INSERT + UPDATE + DELETE

```sql
START TRANSACTION;

INSERT INTO Account(user_name, balance) VALUES('테스트', 20000);
UPDATE Account SET balance = balance + 500 WHERE user_name = '테스트';
DELETE FROM Account WHERE user_name = '테스트';

COMMIT;
```

→ 모든 작업이 반영 또는 취소되는 원자성 실습

---

# 8️⃣ 트랜잭션 전문가 팁

---

### ✔ 1) 트랜잭션은 “적게, 짧게” 사용해야 한다

긴 트랜잭션은 락을 오래 유지 → 서비스 정체.

---

### ✔ 2) REPEATABLE READ가 MySQL 기본

→ Phantom Read 방지됨 (MVCC 덕분)

---

### ✔ 3) SELECT FOR UPDATE / LOCK IN SHARE MODE 는 락이 걸린다

동시성 영향 주의!

---

### ✔ 4) 트랜잭션 내에서 Auto-Commit OFF는 필수

---

### ✔ 5) UPDATE/DELETE 시 WHERE 절 빠지면 대형 사고

→ 반드시 ROLLBACK 준비 후 실행


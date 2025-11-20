# 🚀 SQL RENAME

---

# 1️⃣ RENAME 이란?

SQL에서 **RENAME** 은 데이터베이스 객체의 이름을 변경하는 동작입니다.

변경 대상:

* 테이블 이름 (RENAME TABLE)
* 컬럼 이름 (ALTER … RENAME COLUMN)
* 인덱스 이름
* 제약조건 이름

MySQL에서는 다음 두 가지 방식이 존재합니다.

---

# 2️⃣ RENAME TABLE (MySQL)

```
RENAME TABLE 기존이름 TO 새이름;
```

✔ 테이블 이름만 변경
✔ 실제 데이터 이동 없음 → 메타데이터 변경
✔ DDL → 자동 COMMIT
✔ 테이블에 Exclusive Lock 발생

---

# 3️⃣ RENAME COLUMN (MySQL 8+)

컬럼 이름을 변경하려면 ALTER TABLE 문법을 사용해야 한다.

```
ALTER TABLE 테이블명 RENAME COLUMN 기존컬럼명 TO 새컬럼명;
```

---

# 4️⃣ DBMS별 RENAME 차이

| DBMS       | 테이블명 변경                 | 컬럼명 변경                      | 비고            |
| ---------- | ----------------------- | --------------------------- | ------------- |
| MySQL      | RENAME TABLE            | ALTER TABLE … RENAME COLUMN | 둘 다 지원        |
| PostgreSQL | ALTER TABLE … RENAME TO | ALTER TABLE … RENAME COLUMN | 표준 SQL 준수     |
| Oracle     | RENAME old TO new       | ALTER TABLE RENAME COLUMN   | 테이블 RENAME 가능 |
| SQL Server | sp_rename               | sp_rename                   | 시스템 프로시저      |

→ SQL Server는 전용 함수 `sp_rename` 사용.

---

# 5️⃣ RENAME이 중요한 이유 (실무 기준)

✔ 테이블명 변경 시 API/프론트/백엔드 모든 연동 코드가 바뀜
✔ 컬럼명 변경 시 전체 코드 영향 → Big Refactoring
✔ ENUM/제약조건/인덱스 이름도 함께 바꿔야 하는 경우 많음
✔ 운영 DB에서 RENAME은 위험 → 의존성 확인 필수

---

# 6️⃣ 실습용 테이블 생성

```sql
CREATE TABLE Member (
    id      INT NOT NULL AUTO_INCREMENT,
    name    VARCHAR(50),
    email   VARCHAR(100),
    reg_date DATE,
    PRIMARY KEY (id)
);
```

---

# 7️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Member (name, email, reg_date) VALUES
('김철수', 'kim@example.com', '2024-01-01'),
('이영희', 'lee@example.com', '2024-01-05'),
('박민수', 'minsu@example.com', '2024-01-10');
```

---

# 8️⃣ RENAME 실습 예제

---

## ✔ 예제 1) 테이블 이름 변경 (가장 기본)

```sql
RENAME TABLE Member TO MemberInfo;
```

### 결과

DESCRIBE Member → 에러
DESCRIBE **MemberInfo** → OK

---

## ✔ 예제 2) 테이블 이름 다시 변경

```sql
RENAME TABLE MemberInfo TO UserMember;
```

---

## ✔ 예제 3) 컬럼 이름 변경

```sql
ALTER TABLE UserMember
RENAME COLUMN reg_date TO registered_at;
```

### 결과 (DESC UserMember)

| Field         | Type |
| ------------- | ---- |
| registered_at | date |

---

## ✔ 예제 4) 컬럼 이름 변경 + 자료형 변경 (CHANGE 방식)

구 MySQL 스타일:

```sql
ALTER TABLE UserMember
CHANGE COLUMN email email_address VARCHAR(150);
```

---

## ✔ 예제 5) 컬럼 순서 변경(이름 변경과 동시)

```sql
ALTER TABLE UserMember
CHANGE COLUMN name full_name VARCHAR(100) AFTER id;
```

---

## ✔ 예제 6) 여러 테이블 동시 RENAME

```sql
RENAME TABLE
    UserMember TO UserAccount,
    Orders TO OrderHistory;
```

---

## ✔ 예제 7) 컬럼명 잘못 변경 후 되돌리기

```sql
ALTER TABLE UserAccount
RENAME COLUMN full_name TO name;
```

---

## ✔ 예제 8) 외래키(FK) 컬럼 이름 변경 시 실무 문제

1. FK 삭제
2. 컬럼 이름 변경
3. FK 재추가

```sql
ALTER TABLE Payments DROP FOREIGN KEY fk_user_id;
ALTER TABLE Payments RENAME COLUMN user_id TO member_id;
ALTER TABLE Payments ADD CONSTRAINT fk_member_id
    FOREIGN KEY (member_id) REFERENCES UserAccount(id);
```

---

## ✔ 예제 9) 인덱스 이름 변경

MySQL에서는 DROP + ADD 방식이어야 함.

```sql
ALTER TABLE UserAccount DROP INDEX idx_email;
ALTER TABLE UserAccount ADD INDEX idx_emailaddress (email_address);
```

---

## ✔ 예제 10) 제약조건 이름 변경

CHECK / UNIQUE 이름 재설정:

```sql
ALTER TABLE UserAccount DROP INDEX u_email;
ALTER TABLE UserAccount ADD CONSTRAINT u_emailaddress UNIQUE(email_address);
```

---

## ✔ 예제 11) RENAME 테이블 후 데이터 확인

```sql
SELECT * FROM UserAccount;
```

### 결과

| id | name | email_address                                 | registered_at |
| -- | ---- | --------------------------------------------- | ------------- |
| 1  | 김철수  | [kim@example.com](mailto:kim@example.com)     | 2024-01-01    |
| 2  | 이영희  | [lee@example.com](mailto:lee@example.com)     | 2024-01-05    |
| 3  | 박민수  | [minsu@example.com](mailto:minsu@example.com) | 2024-01-10    |

---

## ✔ 예제 12) 존재하지 않는 테이블 RENAME (오류)

```sql
RENAME TABLE NoSuchTable TO Test;
```

→ ERROR: Table doesn’t exist

---

## ✔ 예제 13) 컬럼 이름 충돌 테스트

```sql
ALTER TABLE UserAccount
RENAME COLUMN name TO email_address;
```

→ ERROR: Duplicate column name

---

## ✔ 예제 14) RENAME TABLE + TEMPORARY 패턴 (Zero-Downtime 전략의 기초)

```sql
RENAME TABLE UserAccount TO UserAccount_old,
             UserAccount_new TO UserAccount;
```

실무에서 스키마 교체할 때 자주 사용되는 패턴.

---

## ✔ 예제 15) RENAME + VIEW 의존성 문제 확인

```sql
RENAME TABLE UserAccount TO UserMember;
```

→ 해당 테이블을 사용하는 VIEW, FUNCTION, PROCEDURE가 깨질 수 있음
(필수 점검 항목)

---

# 9️⃣ RENAME 사용 시 실무 전문가 팁

---

### ✔ 1) RENAME TABLE은 초경량 작업이지만 Lock은 발생한다

데이터 이동 없음 → 메타데이터만 수정
하지만 테이블은 잠김.

---

### ✔ 2) 운영 환경에서 RENAME은 큰 위험

API / 서버 / 백엔드 전체 영향 → 의존성 분석 필수.

---

### ✔ 3) 컬럼 이름 변경은 “빅 리팩토링”

백엔드 모델, DTO, JPA, Query, 프론트 모두 수정 필요.

---

### ✔ 4) SQL Server는 sp_rename

MySQL/Oracle/PostgreSQL과 문법이 다름.

---

### ✔ 5) 트랜잭션 ROLLBACK 불가능

RENAME은 DDL → Auto-Commit.

---

### ✔ 6) VIEW / FK / PROCEDURE / FUNCTION 종속성 반드시 점검

Rename 후 종속 객체 깨질 위험 높음.

---

### ✔ 7) RENAME TABLE을 활용한 Zero-Downtime Migration 패턴 존재

새 테이블 → 데이터 마이그 → RENAME → 빠른 교체.


# ✅ CONSTRAINT란 무엇인가?

`CONSTRAINT`는 SQL 표준에서 **테이블의 무결성 제약 조건을 명시하고, 이름을 부여할 수 있게 해주는 선언적 구문**입니다.

> ✅ 목적: \*\*데이터 무결성(Data Integrity)\*\*을 보장  
> ✅ 방식: 선언적으로 제한조건을 설정함으로써 DBMS 수준에서 일관성과 정확성을 유지  
> ✅ 효과: 비즈니스 로직이 아닌 DB 레벨에서 데이터 품질을 강제

# 📚 CONSTRAINT의 주요 종류와 의미

| 종류 | 키워드 | 설명 | 주요 특징 |
| --- | --- | --- | --- |
| **PRIMARY KEY** | `PRIMARY KEY` | 테이블의 유일한 식별자 | 자동으로 `NOT NULL` + `UNIQUE`, 인덱스 생성 |
| **UNIQUE** | `UNIQUE` | 중복 허용 안함 | 여러 개 지정 가능 (단, PK는 1개) |
| **FOREIGN KEY** | `FOREIGN KEY` | 다른 테이블의 PK 또는 Unique를 참조 | 관계형 무결성 보장 |
| **CHECK** | `CHECK` | 조건식 만족 여부 제한 | DB마다 기능 차이 큼 |
| **NOT NULL** | `NOT NULL` | NULL 값 금지 | 단순한 열 수준 제약 |

## ✅ CONSTRAINT 작성 구문

### 🔹 1. 테이블 정의 시 인라인 제약

```
CREATE TABLE user_account (
    id INT PRIMARY KEY,                    -- PK
    email VARCHAR(255) UNIQUE,             -- UNIQUE
    status VARCHAR(20) CHECK (status IN ('active', 'inactive')),  -- CHECK
    created_at TIMESTAMP NOT NULL          -- NOT NULL
);
```

### 🔹 2. 명시적 이름 지정 (CONSTRAINT 사용)

```
CREATE TABLE user_account (
    id INT,
    email VARCHAR(255),
    status VARCHAR(20),
    CONSTRAINT pk_user_account PRIMARY KEY (id),
    CONSTRAINT uq_user_email UNIQUE (email),
    CONSTRAINT chk_status CHECK (status IN ('active', 'inactive'))
);
```

### 🔹 3. ALTER TABLE로 추가

```
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(id)
ON DELETE CASCADE;
```

# 🔧 내부 동작과 저장 메타데이터

### 🔹 무결성 검사는 언제 일어나는가?

-   `INSERT`, `UPDATE`, `DELETE` 시점에 제약 조건을 DBMS가 **자동 검사**
-   트랜잭션 컨텍스트 내에서 수행됨
-   **즉시 검사(Immediate)** 또는 **지연 검사(Deferred)** 설정 가능 (PostgreSQL, Oracle 지원)

### 🔹 메타데이터 시스템 뷰 (예: Oracle)

| DBMS | 제약 조건 정보 테이블 |
| --- | --- |
| Oracle | `USER_CONSTRAINTS`, `ALL_CONSTRAINTS` |
| MySQL | `information_schema.TABLE_CONSTRAINTS` |
| PostgreSQL | `pg_constraint`, `information_schema.table_constraints` |
| SQL Server | `sys.check_constraints`, `sys.foreign_keys`, `sys.key_constraints` |

### 예: Oracle에서 제약 조건 보기

```
SELECT constraint_name, constraint_type, table_name
FROM user_constraints
WHERE table_name = 'ORDERS';
```

# 🚦 CONSTRAINT 동작 특성 및 설계 전략

| 구분 | 설명 |
| --- | --- |
| 제약 이름은 필수는 아니지만 **명시적으로 설정하는 것이 권장** |   |
| **PRIMARY KEY**는 자동으로 인덱스를 생성하지만, **CHECK**는 인덱스를 생성하지 않음 |   |
| 제약 조건 위반 시 **ROLLBACK** 트리거 가능성 |   |
| 트랜잭션에 따라 **지연 검사 가능 (DEFERRABLE)** |   |
| **Foreign Key**는 **Join 성능을 위해 인덱스 반드시 생성** 권장 |   |
| **CHECK 제약은 DB마다 해석 방식 차이 존재** (e.g., MSSQL은 UDF 호출 불가) |   |

# 📉 성능 측면에서의 고려사항

| 제약 조건 | 성능 영향 | 설명 |
| --- | --- | --- |
| PRIMARY KEY | 중간 | 인덱스 생성, 중복 검사 |
| UNIQUE | 중간 | 인덱스 생성, 중복 검사 |
| FOREIGN KEY | 높음 | 참조 무결성 검사: 부모 테이블 스캔 필요 |
| CHECK | 낮음 ~ 중간 | 복잡한 조건일수록 성능 저하 가능 |
| NOT NULL | 매우 낮음 | 거의 영향 없음 (NULL 여부만 확인) |

> ❗ **대량 데이터 이관 시**에는 `CONSTRAINT`를 잠시 DROP 후 INSERT 후 다시 ADD하는 방식도 사용됨 (트랜잭션 단위 고려)

# 🧠 실무 팁

1.  ✅ **CONSTRAINT 이름은 항상 명시하라**
    -   디버깅, 제약 수정/삭제 시 추적 용이
2.  ✅ **외래키는 반드시 인덱싱하라**
    -   특히 자식 테이블의 FK 컬럼에 인덱스가 없으면 성능 이슈 발생
3.  ❌ **CHECK로 복잡한 논리 표현 금지**
    -   단순 조건(값 범위 등)만 허용, 복잡한 비즈니스 로직은 애플리케이션에서 처리
4.  ❗ **비정규화 시 제약을 제거하는 전략적 선택도 존재**
    -   속도와 무결성 간 트레이드오프 분석 후 결정

# 🧾 요약

| 항목 | 설명 |
| --- | --- |
| CONSTRAINT 키워드 | 제약 조건 정의와 이름 부여를 위한 SQL 문법 |
| 종류 | PRIMARY KEY, UNIQUE, FOREIGN KEY, CHECK, NOT NULL |
| 실무 활용 | 데이터 무결성 유지, 관계 설계, 트랜잭션 보호 |
| 성능 영향 | FK가 가장 크고, NOT NULL이 가장 적음 |
| 관리 전략 | 메타데이터 조회/명시적 이름 지정/ALTER로 동적 제어 |

다음은 각 **제약 조건별로 DDL 생성 → 위반 상황 발생 → 예외 메시지 확인 → 제약 수정 → 제약 삭제**까지의 전체 과정을 **시나리오 기반**으로 자세히 설명드리겠습니다.  
모든 시나리오는 표준 SQL에 근거하되, 실습은 **MySQL (InnoDB)** 기준으로 작성하겠습니다.

# 🧪 시나리오 전제

```
-- 초기 고객 테이블
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);
```

이제 아래 5가지 제약 조건에 대해 차례로 진행합니다:

1.  PRIMARY KEY
2.  UNIQUE
3.  FOREIGN KEY
4.  CHECK
5.  NOT NULL

## ① `PRIMARY KEY` 제약 시나리오

### ✅ 1단계. 생성

```
CREATE TABLE product (
    id INT,
    name VARCHAR(50),
    CONSTRAINT pk_product PRIMARY KEY (id)
);
```

### ❌ 2단계. 위반 사례

```
INSERT INTO product (id, name) VALUES (1, 'Coffee');
INSERT INTO product (id, name) VALUES (1, 'Tea'); -- 중복된 id
```

### ⚠ 3단계. 예외 메시지

```
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```

### 🔧 4단계. 수정

```
ALTER TABLE product DROP PRIMARY KEY;
ALTER TABLE product ADD CONSTRAINT pk_product_new PRIMARY KEY (id, name);
```

### ❎ 5단계. 삭제

```
ALTER TABLE product DROP PRIMARY KEY;
```

## ② `UNIQUE` 제약 시나리오

### ✅ 1단계. 생성

```
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    CONSTRAINT uq_users_email UNIQUE (email)
);
```

### ❌ 2단계. 위반 사례

```
INSERT INTO users VALUES (1, 'a@example.com');
INSERT INTO users VALUES (2, 'a@example.com'); -- 이메일 중복
```

### ⚠ 3단계. 예외 메시지

```
ERROR 1062 (23000): Duplicate entry 'a@example.com' for key 'uq_users_email'
```

### 🔧 4단계. 수정

```
ALTER TABLE users DROP INDEX uq_users_email;
ALTER TABLE users ADD CONSTRAINT uq_users_email_new UNIQUE (email, id);
```

### ❎ 5단계. 삭제

```
ALTER TABLE users DROP INDEX uq_users_email;
```

## ③ `FOREIGN KEY` 제약 시나리오

### ✅ 1단계. 생성

```
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE RESTRICT
);
```

### ❌ 2단계. 위반 사례

1.  **없는 고객 ID를 삽입**

```
INSERT INTO orders VALUES (1, 999); -- customers 테이블에 id=999 없음
```

2.  **참조된 고객을 삭제**

```
DELETE FROM customers WHERE id = 1; -- orders 테이블에 참조 중이면 삭제 불가
```

### ⚠ 3단계. 예외 메시지

```
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails
```

### 🔧 4단계. 수정

```
ALTER TABLE orders DROP FOREIGN KEY fk_orders_customer;
ALTER TABLE orders ADD CONSTRAINT fk_orders_customer_cascade
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE CASCADE;
```

### ❎ 5단계. 삭제

```
ALTER TABLE orders DROP FOREIGN KEY fk_orders_customer_cascade;
```

## ④ `CHECK` 제약 시나리오

> 💡 참고: MySQL 8.0.16 이상에서 `CHECK` 정식 지원

### ✅ 1단계. 생성

```
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT,
    CONSTRAINT chk_age CHECK (age >= 18 AND age <= 65)
);
```

### ❌ 2단계. 위반 사례

```
INSERT INTO employees VALUES (1, 10); -- 나이 18 미만
```

### ⚠ 3단계. 예외 메시지

```
ERROR 3819 (HY000): Check constraint 'chk_age' is violated.
```

### 🔧 4단계. 수정

```
ALTER TABLE employees DROP CHECK chk_age;
ALTER TABLE employees ADD CONSTRAINT chk_age_2 CHECK (age >= 16 AND age <= 65);
```

### ❎ 5단계. 삭제

```
ALTER TABLE employees DROP CHECK chk_age_2;
```

## ⑤ `NOT NULL` 제약 시나리오

### ✅ 1단계. 생성

```
CREATE TABLE payments (
    id INT PRIMARY KEY,
    amount DECIMAL(10,2) NOT NULL
);
```

### ❌ 2단계. 위반 사례

```
INSERT INTO payments (id, amount) VALUES (1, NULL); -- amount가 NULL
```

### ⚠ 3단계. 예외 메시지

```
ERROR 1048 (23000): Column 'amount' cannot be null
```

### 🔧 4단계. 수정

```
ALTER TABLE payments MODIFY amount DECIMAL(10,2) NULL;
```

### ❎ 5단계. 삭제는 별도로 없음

→ `NOT NULL`은 삭제가 아니라 `NULL 허용`으로 **타입 수정**

# 🧾 요약: 제약 조건별 관리 명령어 정리

| 제약 종류 | 생성 방식 | 위반 시 오류 코드 | 수정 방법 | 삭제 방법 |
| --- | --- | --- | --- | --- |
| PRIMARY KEY | `PRIMARY KEY (...)` | 1062 | `DROP` 후 `ADD` | `ALTER TABLE ... DROP PRIMARY KEY` |
| UNIQUE | `UNIQUE (...)` | 1062 | `DROP INDEX` 후 `ADD` | `DROP INDEX` |
| FOREIGN KEY | `FOREIGN KEY (...)` | 1452 | `DROP FOREIGN KEY` 후 다시 `ADD` | `DROP FOREIGN KEY` |
| CHECK | `CHECK (...)` | 3819 | `DROP CHECK` 후 다시 `ADD` | `DROP CHECK` |
| NOT NULL | 열 수준에서 `NOT NULL` | 1048 | `MODIFY`로 `NULL` 허용 | `MODIFY`로 변경 |

좋습니다! 지금부터 앞서 설명한 \*\*제약 조건별 시나리오 (DDL 생성 → 위반 → 예외 확인 → 수정 → 삭제)\*\*를 모두 **MySQL (InnoDB, 8.0 이상)** 기준으로 실습 가능한 형태의 **SQL 스크립트**로 정리해드리겠습니다.

# ✅ 0. 환경 초기화

```
-- 초기 테이블 및 데이터 정리 (안전 실행)
DROP TABLE IF EXISTS payments, employees, orders, users, product, customers;

-- 부모 테이블 생성
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

-- 테스트용 고객 1명 삽입
INSERT INTO customers VALUES (1, 'Alice');
```

# ✅ ① PRIMARY KEY 제약 시나리오

```
-- 1단계. 테이블 생성
CREATE TABLE product (
    id INT,
    name VARCHAR(50),
    CONSTRAINT pk_product PRIMARY KEY (id)
);

-- 2단계. 중복된 PRIMARY KEY 입력 시도
INSERT INTO product VALUES (1, 'Coffee');
INSERT INTO product VALUES (1, 'Tea'); -- ❌ 중복

-- 3단계. 예외 메시지
-- ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

-- 4단계. PRIMARY KEY 수정 (id -> id + name 복합키로)
ALTER TABLE product DROP PRIMARY KEY;
ALTER TABLE product ADD CONSTRAINT pk_product_new PRIMARY KEY (id, name);

-- 5단계. 삭제
ALTER TABLE product DROP PRIMARY KEY;
```

# ✅ ② UNIQUE 제약 시나리오

```
-- 1단계. 테이블 생성
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    CONSTRAINT uq_users_email UNIQUE (email)
);

-- 2단계. 중복된 이메일 삽입 시도
INSERT INTO users VALUES (1, 'a@example.com');
INSERT INTO users VALUES (2, 'a@example.com'); -- ❌ 중복

-- 3단계. 예외 메시지
-- ERROR 1062 (23000): Duplicate entry 'a@example.com' for key 'uq_users_email'

-- 4단계. UNIQUE 제약 수정 (이메일+id로 확장)
ALTER TABLE users DROP INDEX uq_users_email;
ALTER TABLE users ADD CONSTRAINT uq_users_email_new UNIQUE (email, id);

-- 5단계. 삭제
ALTER TABLE users DROP INDEX uq_users_email_new;
```

# ✅ ③ FOREIGN KEY 제약 시나리오

```
-- 1단계. 테이블 생성
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE RESTRICT
);

-- 2-1단계. 존재하지 않는 고객 ID 참조 시도
INSERT INTO orders VALUES (1, 999); -- ❌ customers에 id=999 없음

-- 2-2단계. 참조된 고객 삭제 시도
-- (먼저 정상 참조 추가)
INSERT INTO orders VALUES (2, 1); -- ✅ 존재하는 고객

-- 이제 고객 삭제 시도
DELETE FROM customers WHERE id = 1; -- ❌ 참조 중

-- 3단계. 예외 메시지
-- ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails

-- 4단계. ON DELETE CASCADE로 제약 재설정
ALTER TABLE orders DROP FOREIGN KEY fk_orders_customer;
ALTER TABLE orders ADD CONSTRAINT fk_orders_customer_cascade
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE CASCADE;

-- 5단계. 삭제
ALTER TABLE orders DROP FOREIGN KEY fk_orders_customer_cascade;
```

# ✅ ④ CHECK 제약 시나리오 (MySQL 8.0.16 이상)

```
-- 1단계. 테이블 생성
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT,
    CONSTRAINT chk_age CHECK (age >= 18 AND age <= 65)
);

-- 2단계. 위반 데이터 입력
INSERT INTO employees VALUES (1, 10); -- ❌ 18 미만

-- 3단계. 예외 메시지
-- ERROR 3819 (HY000): Check constraint 'chk_age' is violated.

-- 4단계. 제약 수정
ALTER TABLE employees DROP CHECK chk_age;
ALTER TABLE employees ADD CONSTRAINT chk_age_2 CHECK (age >= 16 AND age <= 65);

-- 5단계. 삭제
ALTER TABLE employees DROP CHECK chk_age_2;
```

# ✅ ⑤ NOT NULL 제약 시나리오

```
-- 1단계. 테이블 생성
CREATE TABLE payments (
    id INT PRIMARY KEY,
    amount DECIMAL(10,2) NOT NULL
);

-- 2단계. NULL 삽입 시도
INSERT INTO payments VALUES (1, NULL); -- ❌ NOT NULL 위반

-- 3단계. 예외 메시지
-- ERROR 1048 (23000): Column 'amount' cannot be null

-- 4단계. NULL 허용으로 수정
ALTER TABLE payments MODIFY amount DECIMAL(10,2) NULL;

-- 5단계. NOT NULL 삭제는 위의 MODIFY로 처리됨
```

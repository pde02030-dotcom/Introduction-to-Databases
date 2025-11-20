# 🧩 데이터베이스의 Cascade: 참조 무결성을 자동으로 유지하는 강력한 무기

## ✅ 들어가며

관계형 데이터베이스에서 여러 테이블은 종종 **외래 키(Foreign Key)** 를 통해 연결됩니다. 예를 들어, `회원(Member)`과 `주문(Order)` 관계에서는 하나의 회원이 여러 개의 주문을 가질 수 있으므로, `Order` 테이블은 `Member` 테이블을 참조하게 됩니다.

그렇다면 회원이 삭제될 경우, 해당 회원의 주문은 어떻게 처리해야 할까요? 이때 수동으로 자식 테이블을 먼저 정리할 수도 있지만, 더 효과적인 방법이 있습니다. 바로 **CASCADE(카스케이드)** 입니다.

---

## 🔍 CASCADE란 무엇인가?

**CASCADE**는 외래 키 제약 조건에 따라 **부모 테이블에서 일어난 삭제(DELETE) 또는 갱신(UPDATE) 작업을 자식 테이블로 전파**하는 동작입니다.

### 📌 적용 대상

* `ON DELETE CASCADE`
* `ON UPDATE CASCADE`

즉, 부모 레코드가 삭제되거나 갱신되면,

* **자동으로 자식 테이블의 관련 레코드도 삭제되거나 갱신**되도록 하는 기능입니다.

---

## 🧪 사용 예제 (MySQL 기준)

### 1. 테이블 생성

```sql
CREATE TABLE Member (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Orders (
    id INT PRIMARY KEY,
    member_id INT,
    FOREIGN KEY (member_id) REFERENCES Member(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

### 2. 데이터 삽입

```sql
INSERT INTO Member VALUES (1, '홍길동');
INSERT INTO Orders VALUES (101, 1);
INSERT INTO Orders VALUES (102, 1);
```

### 3. 부모 삭제 시 자식도 삭제됨

```sql
DELETE FROM Member WHERE id = 1;
-- 👉 Orders 테이블의 member_id = 1인 레코드도 함께 삭제됨
```

### 4. 부모 ID 변경 시 자식도 업데이트됨

```sql
UPDATE Member SET id = 2 WHERE id = 1;
-- 👉 Orders 테이블의 member_id도 2로 갱신됨
```

---

## 🧠 옵션 정리

| 옵션                  | 설명                                  |
| ------------------- | ----------------------------------- |
| `ON DELETE CASCADE` | 부모 레코드 삭제 시 자식 레코드도 자동 삭제           |
| `ON UPDATE CASCADE` | 부모 기본키 변경 시 자식 외래키도 자동 변경           |
| `SET NULL`          | 부모 삭제/변경 시 자식 외래키를 `NULL`로 설정       |
| `SET DEFAULT`       | 자식 외래키를 기본값으로 설정                    |
| `NO ACTION`         | 참조 무결성 위반이 발생하면 **예외 발생**           |
| `RESTRICT`          | 부모 삭제/변경 **시도 자체를 차단** (자식 존재 시 불가) |

---

## 🔄 실무 적용 시나리오

### ✔ 유효한 시나리오

| 시나리오                      | 권장 설정                              |
| ------------------------- | ---------------------------------- |
| 회원 탈퇴 시 주문 기록도 삭제         | `ON DELETE CASCADE`                |
| 상품 정보가 갱신되면 관련 테이블도 자동 반영 | `ON UPDATE CASCADE`                |
| 부모가 삭제되어도 자식 데이터는 남겨야 함   | `ON DELETE SET NULL` 또는 `RESTRICT` |

---

## ⚠️ 실무에서의 주의 사항

### 1. **Cascade Delete 폭탄을 조심하라**

부모 테이블의 한 행 삭제가 **수십 개 테이블에 영향을 줄 수 있습니다.** 무분별한 `ON DELETE CASCADE` 설정은 데이터 유실로 이어질 수 있습니다.

### 2. **로그 기반 복구 시 복잡성 증가**

Cascade는 여러 테이블을 연쇄적으로 수정하기 때문에, 트랜잭션 로그가 복잡해지고 **장애 발생 시 복구 난이도가 증가**합니다.

### 3. **트리거와 중첩될 경우 충돌 가능**

Cascade와 `AFTER DELETE` 트리거를 함께 사용하는 경우 **의도하지 않은 중복 처리나 오류**가 발생할 수 있습니다.

---

## 📌 JPA와의 비교

JPA에서는 Java 코드에서 다음처럼 설정합니다:

```java
@OneToMany(mappedBy = "member", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Order> orders = new ArrayList<>();
```

* `CascadeType.ALL`: `persist`, `merge`, `remove` 등 모든 연산 전파
* `orphanRemoval = true`: 부모와의 연관관계가 끊어진 자식 엔티티 자동 삭제

### ✅ 차이점

| 항목     | DB Cascade    | JPA Cascade          |
| ------ | ------------- | -------------------- |
| 동작 위치  | 데이터베이스 레벨     | 자바 애플리케이션 레벨         |
| 트리거 시점 | SQL 실행 시      | `EntityManager` 연산 시 |
| 장점     | 성능 우수, 명확한 선언 | 객체 그래프 기반 연산 가능      |
| 단점     | 디버깅 어려움       | 영속성 컨텍스트 제어 필요       |

---

## 🧵 마무리

Cascade는 데이터베이스의 강력한 참조 무결성 기능으로, 복잡한 연관 관계를 **자동화**하고 **유지보수를 단순화**할 수 있습니다. 그러나 **설정이 잘못되면 전체 시스템에 치명적인 데이터 손실**을 유발할 수도 있으므로, 반드시 **데이터 흐름과 비즈니스 요구에 따라 세심하게 설계**해야 합니다.

> ✅ 설계의 핵심:
> ❌ “일단 CASCADE 걸어놓자” →
> ✅ “관계 및 비즈니스 로직에 기반한 정교한 제약 조건 설정”

---

# 🔧 추천 실습

---

## 🧪 실습 1: `ON DELETE SET NULL` – 부모 삭제 시 자식 외래키를 NULL로 설정

### ✅ 목적

부모가 삭제되어도 자식 데이터를 보존하되, 외래 키는 `NULL`로 설정.

---

### 🛠️ MySQL SQL 실습

```sql
CREATE TABLE Department (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE Employee (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES Department(id)
        ON DELETE SET NULL
);
```

```sql
-- 데이터 삽입
INSERT INTO Department VALUES (1, 'IT');
INSERT INTO Employee VALUES (100, 'Alice', 1), (101, 'Bob', 1);

-- 부서 삭제
DELETE FROM Department WHERE id = 1;

-- Employee 테이블에서 department_id는 NULL이 된다.
SELECT * FROM Employee;
```

---

### ☕ JPA 실습

```java
@Entity
public class Department {
    @Id
    private Integer id;
    private String name;
}

@Entity
public class Employee {
    @Id
    private Integer id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id", foreignKey = @ForeignKey(ConstraintMode.CONSTRAINT))
    private Department department;
}
```

※ `ON DELETE SET NULL`은 DDL 생성 시 명시되지 않으므로 직접 테이블을 생성하거나 JPA DDL을 비활성화하고 수동으로 설정해야 합니다.

---

## 🧪 실습 2: `ON DELETE RESTRICT` – 부모 삭제 제한

### ✅ 목적

자식이 존재하면 부모를 삭제할 수 없도록 막기.

---

### 🛠️ MySQL SQL 실습

```sql
CREATE TABLE Category (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE Product (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    category_id INT,
    FOREIGN KEY (category_id)
        REFERENCES Category(id)
        ON DELETE RESTRICT
);
```

```sql
-- 데이터 삽입
INSERT INTO Category VALUES (10, 'Food');
INSERT INTO Product VALUES (1000, 'Banana', 10);

-- 부모 삭제 시도 → 실패!
DELETE FROM Category WHERE id = 10;
```

> ❌ 오류: Cannot delete or update a parent row: a foreign key constraint fails

---

### ☕ JPA 실습

JPA에서는 `RESTRICT`도 마찬가지로 DDL에서는 명시되지 않기 때문에 수동 생성 필요:

```java
@Entity
public class Category {
    @Id
    private Integer id;
    private String name;
}

@Entity
public class Product {
    @Id
    private Integer id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category;
}
```

---

## 🧪 실습 3: JPA Cascade vs DB Cascade 병행 비교 실습

### ✅ 목적

* JPA의 `CascadeType.REMOVE`와
* DB의 `ON DELETE CASCADE`가 어떻게 다르게 작동하는지 비교

---

### ☕ JPA 엔티티 설정

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "member", cascade = CascadeType.REMOVE, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    private String item;

    @ManyToOne
    @JoinColumn(name = "member_id", foreignKey = @ForeignKey(ConstraintMode.CONSTRAINT))
    private Member member;
}
```

---

## 💡 시나리오 테스트

1. **JPA Cascade**

```java
em.remove(member); // ➜ Member와 연관된 Order도 JPA 내부에서 delete됨
```

2. **DB Cascade**
   DDL 수동 생성:

```sql
CREATE TABLE Order (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    item VARCHAR(100),
    member_id BIGINT,
    FOREIGN KEY (member_id)
        REFERENCES Member(id)
        ON DELETE CASCADE
);
```

JPA에서 `@OneToMany`의 `cascade = {}`로 설정 후,

```java
em.remove(member); // ➜ DB 레벨에서 Order 자동 삭제 (Hibernate는 delete Order 쿼리 생략)
```

---

## 🧠 비교 분석 요약

| 항목       | JPA Cascade             | DB Cascade    |
| -------- | ----------------------- | ------------- |
| 동작 위치    | Java 코드 (EntityManager) | DB 엔진         |
| 삭제 쿼리 주체 | Hibernate               | MySQL 등 RDBMS |
| 트랜잭션 컨트롤 | 완벽히 관리 가능               | 예측 어려움        |
| 성능       | 대량 처리에 불리할 수 있음         | 매우 빠름         |
| 로그 및 디버깅 | 추적 용이                   | DB 로그 필요      |

---

## 🔚 마무리

각 실습은 아래 목적에 적합합니다:

* ✅ `SET NULL`: 데이터 보존
* ✅ `RESTRICT`: 데이터 안전
* ✅ `CASCADE`: 데이터 자동 처리



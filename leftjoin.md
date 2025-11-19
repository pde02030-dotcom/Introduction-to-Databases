
# 🔥 `LEFT JOIN`

---

## ✅ 1. LEFT JOIN이란?

### 📌 정의

`LEFT JOIN` 또는 `LEFT OUTER JOIN`은 SQL의 조인 방식 중 하나로,
**왼쪽 테이블(A)의 모든 행을 유지하고**, 오른쪽 테이블(B)의 행 중에 **조건이 일치하는 행만 병합**합니다.

조건이 일치하지 않으면 **오른쪽 테이블의 컬럼들은 NULL로 채워집니다.**

### 📐 수학적 모델

```
A LEFT JOIN B ON A.x = B.y
= σ(A.x = B.y)(A × B) ∪ (A - πA(σ(A.x = B.y)(A × B)))
```

---

## ✅ 2. 예제로 배우는 LEFT JOIN

### 🧾 테이블 정의

```sql
-- 고객 테이블
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 주문 테이블
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    product VARCHAR(50)
);
```

---

### 💾 데이터 삽입

```sql
INSERT INTO customers VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');

INSERT INTO orders VALUES
(101, 1, 'Laptop'),
(102, 1, 'Mouse'),
(103, 2, 'Keyboard');
```

---

## ✅ 3. 테이블 상태 확인

### 📊 customers (왼쪽 테이블)

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

---

### 📊 orders (오른쪽 테이블)

| id  | customer\_id | product  |
| --- | ------------ | -------- |
| 101 | 1            | Laptop   |
| 102 | 1            | Mouse    |
| 103 | 2            | Keyboard |

---

## ✅ 4. LEFT JOIN 기본 예제

```sql
SELECT c.id, c.name, o.product
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

---

### 🔍 결과

| id | name    | product  |
| -- | ------- | -------- |
| 1  | Alice   | Laptop   |
| 1  | Alice   | Mouse    |
| 2  | Bob     | Keyboard |
| 3  | Charlie | NULL     |

### ✅ 해석

* **Alice**는 주문 2건 → 2행
* **Bob**은 주문 1건 → 1행
* **Charlie**는 주문 없음 → `product`는 NULL

---

## ✅ 5. LEFT JOIN 작동 원리: 내부 처리 흐름

### 👇 내부 순서

1. A 테이블(customers)의 **각 행을 기준으로**
2. B 테이블(orders)의 **모든 행과 ON 조건 비교**
3. 조건이 **TRUE인 경우 병합**
4. 조건을 만족하는 B의 행이 없다면 → **B의 모든 컬럼은 NULL**

---

## ✅ 6. 조건이 WHERE에 있을 경우 주의

### ❌ 잘못된 예

```sql
SELECT c.name, o.product
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.product = 'Mouse';
```

→ 결과:

| name  | product |
| ----- | ------- |
| Alice | Mouse   |

### ❗ 문제점

* WHERE에서 `o.product = 'Mouse'`는 **NULL을 제외함**
* → 사실상 INNER JOIN과 동일한 결과

---

### ✅ 올바른 방법

```sql
SELECT c.name, o.product
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id AND o.product = 'Mouse';
```

→ 결과:

| name    | product |
| ------- | ------- |
| Alice   | Mouse   |
| Alice   | NULL    |
| Bob     | NULL    |
| Charlie | NULL    |

---

## ✅ 7. 고급 LEFT JOIN 예제

### 🎯 1. 고객별 총 주문 수

```sql
SELECT c.name, COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

| name    | order\_count |
| ------- | ------------ |
| Alice   | 2            |
| Bob     | 1            |
| Charlie | 0            |

---

### 🎯 2. 주문이 없을 경우 'No Orders'로 표시

```sql
SELECT c.name, COALESCE(o.product, 'No Orders') AS product
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

| name    | product   |
| ------- | --------- |
| Alice   | Laptop    |
| Alice   | Mouse     |
| Bob     | Keyboard  |
| Charlie | No Orders |

---

## ✅ 8. 시각적으로 정리된 JOIN 결과

```text
CUSTOMERS              ORDERS              결과 (LEFT JOIN)
----------             ---------           -----------------------------
1, Alice               101, 1, Laptop      1, Alice, Laptop
                       102, 1, Mouse       1, Alice, Mouse
2, Bob                 103, 2, Keyboard    2, Bob, Keyboard
3, Charlie             없음                3, Charlie, NULL
```

---

## ✅ 9. LEFT JOIN vs INNER JOIN

| 항목         | LEFT JOIN                 | INNER JOIN       |
| ---------- | ------------------------- | ---------------- |
| 기준         | 왼쪽 테이블                    | 양쪽 테이블 모두        |
| NULL 포함 여부 | 오른쪽 테이블 조건 불일치 시 NULL로 채움 | 조건 불일치 행은 아예 제거됨 |
| 대표적인 용도    | 기준 테이블의 행을 모두 보고 싶을 때     | 매칭되는 데이터만 필요할 때  |

---

## ✅ 10. 마무리 요약

* **LEFT JOIN은 정보 유실 방지용 조인**이다.
* **항상 왼쪽 테이블의 각 행을 기준**으로 조건을 평가한다.
* `WHERE`절의 조건이 **NULL을 제거하는지 주의**하자.
* **집계함수 + COALESCE + 조건부 JOIN**으로 실무에서 다양하게 쓰인다.

---

# 📦 실전 비즈니스 케이스: 배송 상태 보고서

## 🎯 목적

> 고객들의 주문에 대해 **배송 여부**를 확인하고,
> **배송이 안 된 주문은 어떤 상태인지**,
> **주문이 없는 고객도 포함**해서 전체 상황을 보고서로 작성한다.

---

## ✅ 1. 테이블 정의

```sql
-- 고객
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 주문
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- 배송
CREATE TABLE deliveries (
    order_id INT PRIMARY KEY,
    delivery_date DATE,
    status VARCHAR(20), -- e.g., 'delivered', 'pending', 'canceled'
    FOREIGN KEY (order_id) REFERENCES orders(id)
);
```

---

## ✅ 2. 샘플 데이터 삽입

```sql
-- customers
INSERT INTO customers VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');

-- orders
INSERT INTO orders VALUES
(101, 1, '2024-06-01', 250.00),
(102, 1, '2024-06-03', 70.00),
(103, 2, '2024-06-02', 150.00);

-- deliveries
INSERT INTO deliveries VALUES
(101, '2024-06-05', 'delivered'),
(103, '2024-06-07', 'pending');
```

---

## ✅ 3. 현재 테이블 상태 요약

### 📊 customers

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

### 📊 orders

| id  | customer\_id | order\_date | total  |
| --- | ------------ | ----------- | ------ |
| 101 | 1            | 2024-06-01  | 250.00 |
| 102 | 1            | 2024-06-03  | 70.00  |
| 103 | 2            | 2024-06-02  | 150.00 |

### 📊 deliveries

| order\_id | delivery\_date | status    |
| --------- | -------------- | --------- |
| 101       | 2024-06-05     | delivered |
| 103       | 2024-06-07     | pending   |

---

## ✅ 4. 배송 상태 보고서 쿼리

```sql
SELECT 
    c.name AS customer_name,
    o.id AS order_id,
    o.order_date,
    o.total,
    COALESCE(d.status, 'not shipped') AS delivery_status,
    d.delivery_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
LEFT JOIN deliveries d ON o.id = d.order_id
ORDER BY c.name, o.order_date;
```

---

## ✅ 5. 결과

| customer\_name | order\_id | order\_date | total  | delivery\_status | delivery\_date |
| -------------- | --------- | ----------- | ------ | ---------------- | -------------- |
| Alice          | 101       | 2024-06-01  | 250.00 | delivered        | 2024-06-05     |
| Alice          | 102       | 2024-06-03  | 70.00  | not shipped      | NULL           |
| Bob            | 103       | 2024-06-02  | 150.00 | pending          | 2024-06-07     |
| Charlie        | NULL      | NULL        | NULL   | not shipped      | NULL           |

---

## ✅ 6. 해설

* `LEFT JOIN`을 2단계로 연결해 **고객 → 주문 → 배송** 순으로 병합
* `COALESCE(d.status, 'not shipped')`로 배송 상태가 없는 경우 `"not shipped"` 처리
* 주문이 없는 **Charlie**도 유지됨 (LEFT JOIN의 핵심 효과)

---

## ✅ 7. 실무 활용 포인트

| 목적               | LEFT JOIN 활용 이유                   |
| ---------------- | --------------------------------- |
| 전체 고객 현황 보고서     | 주문 여부와 관계없이 고객은 전부 포함해야 하므로       |
| 배송 누락 확인         | 배송 상태 없는 주문 추적 가능 (COALESCE로 표현)  |
| 집계 및 경영 판단 근거 제공 | 배송률 %, 누락률 %, 고객별 미배송율 등 추가 분석 가능 |


---

# 🧩 예제: `students` 테이블과 `clubs` 테이블

## 🎓 students

| student\_id | name    |
| ----------- | ------- |
| 1           | Alice   |
| 2           | Bob     |
| 3           | Charlie |

## 🏀 clubs

| club\_id | club\_name |
| -------- | ---------- |
| 1        | Chess      |
| 2        | Soccer     |

---

## 📌 1단계: 카티시안 곱 (students × clubs)

**`SELECT * FROM students, clubs;`**
(3 rows × 2 rows = 6 rows)

| student\_id | name    | club\_id | club\_name |
| ----------- | ------- | -------- | ---------- |
| 1           | Alice   | 1        | Chess      |
| 1           | Alice   | 2        | Soccer     |
| 2           | Bob     | 1        | Chess      |
| 2           | Bob     | 2        | Soccer     |
| 3           | Charlie | 1        | Chess      |
| 3           | Charlie | 2        | Soccer     |

> 🔄 모든 조합이 생성됨 — 필터링 없고, 조건 없음 = **카티시안 곱**

---

## 📌 2단계: 조건 추가 → `INNER JOIN`의 형태

```sql
SELECT *
FROM students s
JOIN clubs c
  ON s.student_id = c.club_id;
```

조건이 부합하는 행만 남기기

| student\_id | name  | club\_id | club\_name |
| ----------- | ----- | -------- | ---------- |
| 1           | Alice | 1        | Chess      |
| 2           | Bob   | 2        | Soccer     |

> 🔍 `student_id = club_id`일 때만 출력됨
> **카티시안 곱 + 필터링 조건 → INNER JOIN**

---

## 📌 3단계: LEFT JOIN = **카티시안 곱 + 조건 필터 + 왼쪽 테이블 보존**

```sql
SELECT *
FROM students s
LEFT JOIN clubs c
  ON s.student_id = c.club_id;
```

결과:

| student\_id | name    | club\_id | club\_name |
| ----------- | ------- | -------- | ---------- |
| 1           | Alice   | 1        | Chess      |
| 2           | Bob     | 2        | Soccer     |
| 3           | Charlie | NULL     | NULL       |

> 🎯 **Charlie**는 `student_id = 3`과 일치하는 `club_id`가 없음
> → 왼쪽 테이블(`students`)의 모든 행은 보존됨
> → 매칭 안 되면 **오른쪽 테이블 컬럼은 NULL**


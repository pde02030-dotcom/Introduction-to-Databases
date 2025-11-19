# 🔍 `RIGHT JOIN`

---

## ✅ 1. RIGHT JOIN이란?

### 📌 정의

`RIGHT JOIN` 또는 `RIGHT OUTER JOIN`은 SQL 조인의 한 종류로,

> **오른쪽 테이블의 모든 행을 기준으로 유지하고**,
> 왼쪽 테이블에서 조건이 **일치하는 행만 병합**합니다.
> 조건을 만족하지 않으면 왼쪽 테이블의 컬럼은 **NULL로 채웁니다.**

---

### 🔄 요약 비교 (LEFT vs RIGHT)

| 항목         | LEFT JOIN            | RIGHT JOIN          |
| ---------- | -------------------- | ------------------- |
| 기준 테이블     | **왼쪽 테이블(A)**        | **오른쪽 테이블(B)**      |
| 필수로 유지되는 행 | 왼쪽 테이블의 모든 행         | 오른쪽 테이블의 모든 행       |
| 조건 불일치 시   | 오른쪽 테이블의 컬럼 → `NULL` | 왼쪽 테이블의 컬럼 → `NULL` |

---

## ✅ 2. 실습용 테이블

### 📊 customers (왼쪽 테이블 A)

```sql
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO customers VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');
```

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

---

### 📊 orders (오른쪽 테이블 B)

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    product VARCHAR(50)
);

INSERT INTO orders VALUES
(101, 1, 'Laptop'),
(102, 1, 'Mouse'),
(103, 2, 'Keyboard'),
(104, 4, 'Tablet');
```

| id  | customer\_id | product  |
| --- | ------------ | -------- |
| 101 | 1            | Laptop   |
| 102 | 1            | Mouse    |
| 103 | 2            | Keyboard |
| 104 | 4            | Tablet   |

→ customer\_id = 4인 주문은 고객 테이블에 없음!

---

## ✅ 3. `RIGHT JOIN` 기본 쿼리

```sql
SELECT c.name, o.id AS order_id, o.product
FROM customers c
RIGHT JOIN orders o ON c.id = o.customer_id;
```

---

## ✅ 4. 내부 처리 순서 분석

---

### ✅ \[1단계] 카디시안 곱 (A × B)

조인 조건 없이 가능한 모든 조합:

| c.id | c.name | o.id | o.customer\_id | o.product |
| ---- | ------ | ---- | -------------- | --------- |
| 1    | Alice  | 101  | 1              | Laptop    |
| 1    | Alice  | 102  | 1              | Mouse     |
| 1    | Alice  | 103  | 2              | Keyboard  |
| 1    | Alice  | 104  | 4              | Tablet    |
| 2    | Bob    | 101  | 1              | Laptop    |
| 2    | Bob    | 102  | 1              | Mouse     |
| ...  | ...    | ...  | ...            | ...       |

→ 총 3 × 4 = **12행**

---

### ✅ \[2단계] ON 조건 필터링

조건: `c.id = o.customer_id`

→ 조건 만족 행만 남음

| c.id | c.name | o.id | o.customer\_id | o.product |
| ---- | ------ | ---- | -------------- | --------- |
| 1    | Alice  | 101  | 1              | Laptop    |
| 1    | Alice  | 102  | 1              | Mouse     |
| 2    | Bob    | 103  | 2              | Keyboard  |

→ 주문 104는 customer\_id = 4 → `조건 불일치`

---

### ✅ \[3단계] RIGHT JOIN이므로 **오른쪽 테이블 기준 유지**

조건 불일치인 주문 104도 **결과에 포함**, 고객 정보는 `NULL`

| name  | order\_id | product  |
| ----- | --------- | -------- |
| Alice | 101       | Laptop   |
| Alice | 102       | Mouse    |
| Bob   | 103       | Keyboard |
| NULL  | 104       | Tablet   |

---

## ✅ 5. 결과 정리

| customers.name | orders.id | orders.product |
| -------------- | --------- | -------------- |
| Alice          | 101       | Laptop         |
| Alice          | 102       | Mouse          |
| Bob            | 103       | Keyboard       |
| NULL           | 104       | Tablet         |

→ 고객 정보가 없는 주문 104도 **삭제되지 않고 보존됨**

---

## ✅ 6. 시각적 요약

```text
RIGHT JOIN = A × B → 조건 필터 → 오른쪽(B) 행은 무조건 남김

                   조인 조건: A.id = B.customer_id

CUSTOMERS           ORDERS                  결과
-----------         ------------            ------------------------
1, Alice            101, 1, Laptop          Alice, 101, Laptop
1, Alice            102, 1, Mouse           Alice, 102, Mouse
2, Bob              103, 2, Keyboard        Bob, 103, Keyboard
❌ 없음             104, 4, Tablet          NULL, 104, Tablet   👈 주목
```

---

## ✅ 7. 실무 예제: 고객 없는 유령 주문 확인

```sql
SELECT o.id AS order_id, o.product
FROM customers c
RIGHT JOIN orders o ON c.id = o.customer_id
WHERE c.id IS NULL;
```

→ 고객 테이블에 존재하지 않는 주문을 필터링

| order\_id | product |
| --------- | ------- |
| 104       | Tablet  |

---

## ✅ 8. LEFT JOIN vs RIGHT JOIN 한눈에 비교

| 항목           | LEFT JOIN                          | RIGHT JOIN                         |
| ------------ | ---------------------------------- | ---------------------------------- |
| 유지되는 테이블     | 왼쪽 테이블(A)의 모든 행                    | 오른쪽 테이블(B)의 모든 행                   |
| NULL이 채워지는 쪽 | 오른쪽 테이블(B)의 컬럼                     | 왼쪽 테이블(A)의 컬럼                      |
| 주로 사용되는 케이스  | 고객이 주문했는지 확인 등                     | 주문이 유효한 고객인지 확인 등                  |
| 대칭 구조        | `A LEFT JOIN B` = `B RIGHT JOIN A` | `A RIGHT JOIN B` = `B LEFT JOIN A` |

---

## ✅ 9. 결론 요약

* `RIGHT JOIN`은 **오른쪽 테이블의 모든 행을 유지**하며
  왼쪽과 조건이 맞지 않으면 **왼쪽 컬럼을 NULL로 채움**
* **JOIN의 핵심은 카디시안 곱 후 조건 필터링**
* `RIGHT JOIN`은 실제 쿼리에서 **좌우 순서 때문에 사용 빈도는 낮지만**,
  JOIN 기준을 바꾸는 데 유용한 **논리적 도구**

---

## ✅ 선택 과제 (심화용)

* `RIGHT JOIN` + `GROUP BY`로 **고객 없는 주문 수 집계**
* `RIGHT JOIN` + `COALESCE`로 **가짜 고객 이름 지정**
* `RIGHT JOIN` + `ON ... AND` 조건 결합
* `LEFT JOIN`으로 동일한 결과를 낼 수 있도록 쿼리 변환

---


# 🧠 RIGHT JOIN의 관계 대수식

```text
A RIGHT JOIN B ON A.x = B.y
=
σ(A.x = B.y)(A × B)
∪
(B - πB(σ(A.x = B.y)(A × B)))
```

---

## 🧩 1. 예제 테이블 구성

### 📘 테이블 A: `employees`

| emp\_id | name  |
| ------- | ----- |
| 1       | Alice |
| 2       | Bob   |
| 3       | Carol |

### 📗 테이블 B: `departments`

| dept\_id | dept\_name |
| -------- | ---------- |
| 1        | HR         |
| 2        | Dev        |
| 4        | Sales      |

---

## 📌 2. SQL RIGHT JOIN

```sql
SELECT *
FROM employees A
RIGHT JOIN departments B
  ON A.emp_id = B.dept_id;
```

---

## 🔍 3. 관계 대수로 변환 및 단계별 분석

### 📍 Step 1: **카티시안 곱**

```
A × B
= employees × departments
```

| emp\_id | name  | dept\_id | dept\_name |
| ------- | ----- | -------- | ---------- |
| 1       | Alice | 1        | HR         |
| 1       | Alice | 2        | Dev        |
| 1       | Alice | 4        | Sales      |
| 2       | Bob   | 1        | HR         |
| 2       | Bob   | 2        | Dev        |
| 2       | Bob   | 4        | Sales      |
| 3       | Carol | 1        | HR         |
| 3       | Carol | 2        | Dev        |
| 3       | Carol | 4        | Sales      |

---

### 📍 Step 2: **조건 필터링**

```
σ(A.emp_id = B.dept_id)(A × B)
```

조건을 만족하는 튜플만 필터링:

| emp\_id | name  | dept\_id | dept\_name |
| ------- | ----- | -------- | ---------- |
| 1       | Alice | 1        | HR         |
| 2       | Bob   | 2        | Dev        |

(이게 **INNER JOIN 결과**와 같음)

---

### 📍 Step 3: **조건에 참여한 B만 추출**

```
πB(σ(A.emp_id = B.dept_id)(A × B))
```

→ 조건을 만족한 `B`만 추출:

| dept\_id | dept\_name |
| -------- | ---------- |
| 1        | HR         |
| 2        | Dev        |

---

### 📍 Step 4: **B 중에서 조건 불만족한 것만 찾기**

```
B - πB(...)
```

→ 매칭 실패한 `B`:

| dept\_id | dept\_name |
| -------- | ---------- |
| 4        | Sales      |

---

### 📍 Step 5: **NULL-padding된 A와 결합**

> 이 단계는 수식에는 직접 안 보이지만 개념상 반드시 적용됩니다:

* A의 속성은 `NULL`로 채우고
* 해당 B를 붙임

| emp\_id | name | dept\_id | dept\_name |
| ------- | ---- | -------- | ---------- |
| NULL    | NULL | 4        | Sales      |

---

### 📍 Step 6: 최종 RIGHT JOIN 결과 (∪)

```
= 조인 성공 결과
∪
  B와 매칭되지 않은 B 튜플 + A의 NULL
```

| emp\_id | name  | dept\_id | dept\_name |
| ------- | ----- | -------- | ---------- |
| 1       | Alice | 1        | HR         |
| 2       | Bob   | 2        | Dev        |
| NULL    | NULL  | 4        | Sales      |

---

## ✅ 요약: RIGHT JOIN 관계 대수 흐름

```
RIGHT JOIN = INNER JOIN
           ∪
           (조건에 매칭되지 못한 B 튜플에 A 속성을 NULL로 채워 합침)
```

---

## 📚 관계 대수식 다시 보기

```
A RIGHT JOIN B ON A.x = B.y
=
σ(A.x = B.y)(A × B)
∪
(B - πB(σ(A.x = B.y)(A × B)))
```

* `σ(...)`: 조인 조건 충족 (INNER JOIN)
* `πB(...)`: 매칭된 B만 추출
* `B - πB(...)`: 매칭 실패한 B
* ∪: 둘 합쳐 최종 RIGHT JOIN 완성



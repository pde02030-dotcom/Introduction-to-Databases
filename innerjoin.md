# 🔍 INNER JOIN

---

## 1. INNER JOIN 개념

### ✅ 정의

두 테이블의 조인 조건을 만족하는 **교집합(Intersection)** 결과만 반환.
양쪽 테이블에 모두 **존재하는 매칭되는 행**이 있어야만 결과에 포함됨.

### ✅ 관계형 대수 표현

```
R ⨝ S ≡ σ(R.a = S.b)(R × S)
```

* `R × S` = 모든 가능한 조합 (카티시안 곱)
* `σ(R.a = S.b)` = 조인 조건에 해당하는 행만 선택

---

## 2. 기본 문법

```sql
SELECT *
FROM table1
INNER JOIN table2
ON table1.col = table2.col;
```

※ `INNER`는 생략 가능

```sql
SELECT *
FROM table1
JOIN table2
ON table1.col = table2.col;
```

---

## 3. 예제 1: 단순한 외래 키 관계 조인

### 테이블 구조:

**students**

| student\_id | name  | dept\_id |
| ----------- | ----- | -------- |
| 1           | Alice | 10       |
| 2           | Bob   | 20       |
| 3           | Carol | NULL     |

**departments**

| dept\_id | dept\_name |
| -------- | ---------- |
| 10       | CS         |
| 20       | Math       |
| 30       | Physics    |

### 쿼리:

```sql
SELECT s.name, d.dept_name
FROM students s
INNER JOIN departments d
ON s.dept_id = d.dept_id;
```

### 결과:

| name  | dept\_name |
| ----- | ---------- |
| Alice | CS         |
| Bob   | Math       |

**설명:**

* `Carol`은 `dept_id`가 NULL → 조건 `s.dept_id = d.dept_id` 불충족 → 제외
* `dept_id = 30`인 Physics는 학생과 매칭되는 것이 없음 → 제외

---

## 4. 예제 2: 여러 조건 조인

```sql
SELECT *
FROM orders o
JOIN customers c
ON o.customer_id = c.id
AND o.status = 'DELIVERED';
```

* `JOIN 조건`에 여러 조건을 추가할 수 있음
* 옵티마이저는 조건을 `ON`절 안/밖에서 처리하는 위치에 따라 다른 전략 선택

---

## 5. 예제 3: 자기 자신과 조인 (SELF JOIN)

```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employee e1
JOIN employee e2
ON e1.manager_id = e2.id;
```

**용도:** 계층 구조(조직도), 트리 구조, 상하 관계 모델링

---

## 6. 예제 4: N\:M 관계 조인

**authors**

| id | name      |
| -- | --------- |
| 1  | Hemingway |
| 2  | Orwell    |

**books**

| id | title               |
| -- | ------------------- |
| 1  | The Old Man and Sea |
| 2  | 1984                |

**author\_book (관계 테이블)**

| author\_id | book\_id |
| ---------- | -------- |
| 1          | 1        |
| 2          | 2        |

```sql
SELECT a.name, b.title
FROM authors a
JOIN author_book ab ON a.id = ab.author_id
JOIN books b ON ab.book_id = b.id;
```

→ 관계형 모델에서 **다대다(N\:M)** 를 처리하는 전형적인 방식

---

## 7. INNER JOIN에서 NULL의 처리

```sql
SELECT *
FROM A
JOIN B ON A.id = B.a_id;
```

* A 또는 B 중에 조인 조건 컬럼이 NULL이면, **JOIN 결과에서 제외됨**
* 이유: `NULL = 10` → `FALSE`가 아니라 `UNKNOWN` → JOIN 조건 불만족

---

## 8. INNER JOIN vs WHERE 절 조건 비교

### 예:

```sql
SELECT * FROM A, B WHERE A.id = B.id;
```

* **고전 스타일** 조인 (ANSI-89)
* `INNER JOIN`과 완전히 동일한 결과
* **권장 방식은 ANSI JOIN (`JOIN ... ON`)**

---

## 9. 성능 최적화: 인덱스 활용

### ◾ 좋은 인덱스가 있을 경우:

```sql
CREATE INDEX idx_students_dept_id ON students(dept_id);
```

→ 옵티마이저는 Nested Loop Join 시 `students.dept_id`를 빠르게 검색

### ◾ 조인 키가 양쪽 테이블에 인덱스가 없다면:

* 옵티마이저는 Table Scan을 하게 되어 성능 급락
* 특히 대용량에서 매우 심각한 IO 발생

---

## 10. 옵티마이저 조인 전략

### MySQL 기준 `EXPLAIN`

```sql
EXPLAIN SELECT s.name, d.dept_name
FROM students s
JOIN departments d
ON s.dept_id = d.dept_id;
```

| id | table       | type | key              | rows | Extra       |
| -- | ----------- | ---- | ---------------- | ---- | ----------- |
| 1  | students    | ALL  | NULL             | 1000 |             |
| 1  | departments | ref  | idx\_departments | 1    | Using where |

* `type = ref` → index 기반 JOIN
* `ALL`은 Full Table Scan을 의미 → 비효율적일 수 있음

---

## 11. INNER JOIN 최적화 실전 팁

| 항목      | 설명                                          |
| ------- | ------------------------------------------- |
| 인덱스     | 양쪽 조인 키에 적절한 인덱스 부여                         |
| 통계      | `ANALYZE TABLE` 등으로 테이블 통계 최신화              |
| 조건 위치   | WHERE vs ON 조건 → OUTER JOIN 시 특히 중요         |
| 힌트 사용   | `USE INDEX`, `STRAIGHT_JOIN` 등 힌트로 옵티마이저 제어 |
| JOIN 순서 | MySQL은 기본적으로 **최적 조인 순서 재정렬** 수행            |

---

## 12. INNER JOIN 실무 사례: 주문/고객/상품

```sql
SELECT c.name AS customer, p.name AS product, o.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE o.status = 'PAID';
```

* 여러 테이블을 체인처럼 조인 가능
* JOIN은 항상 2개씩 연결되므로 `JOIN a JOIN b JOIN c`는 내부적으로 **(a ⨝ b) ⨝ c**로 해석됨

---

## ✅ 결론: INNER JOIN 핵심 요약

| 항목      | 내용                          |
| ------- | --------------------------- |
| 역할      | 교집합, 양쪽 모두 일치하는 데이터만 출력     |
| NULL 처리 | JOIN 조건 컬럼이 NULL이면 제외됨      |
| 인덱스 중요성 | 성능을 좌우하는 핵심 요소              |
| 실무 사용빈도 | 가장 보편적이고 필수적인 JOIN 방식       |
| 고급 활용   | SELF JOIN, N\:M 조인, 서브쿼리 결합 |

---


# 🔍 `JOIN ... USING` vs `JOIN ... ON` 차이

---

## ✅ 1. 문법 차이

### 🔸 `JOIN ... ON`

* **조인 조건**을 명시적으로 정의
* **컬럼 이름이 달라도 조인 가능**

```sql
SELECT ...
FROM table1
JOIN table2
ON table1.col1 = table2.col2;
```

---

### 🔸 `JOIN ... USING`

* **양쪽 테이블에 같은 이름의 컬럼이 있어야만 사용 가능**
* 이 **동일한 컬럼명**을 기준으로 조인하며, 결과에서는 **한 번만 출력**

```sql
SELECT ...
FROM table1
JOIN table2
USING (shared_column);
```

---

## ✅ 2. 조건 컬럼 이름이 다를 때 사용 가능 여부

| 항목               | 가능 여부 | 설명                   |
| ---------------- | ----- | -------------------- |
| `JOIN ... ON`    | ✅ 가능  | 서로 다른 이름도 조건으로 지정 가능 |
| `JOIN ... USING` | ❌ 불가능 | 컬럼 이름이 반드시 동일해야 함    |

---

## ✅ 3. 예제 비교

### 🔹 테이블 구조

**employees**

| emp\_id | name  | dept\_id |
| ------- | ----- | -------- |
| 1       | Alice | 10       |
| 2       | Bob   | 20       |

**departments**

| dept\_id | dept\_name  |
| -------- | ----------- |
| 10       | HR          |
| 20       | Engineering |

---

### ✅ A. JOIN ... ON 예제

```sql
SELECT *
FROM employees e
JOIN departments d
ON e.dept_id = d.dept_id;
```

📌 결과:

| emp\_id | name  | dept\_id | dept\_id | dept\_name  |
| ------- | ----- | -------- | -------- | ----------- |
| 1       | Alice | 10       | 10       | HR          |
| 2       | Bob   | 20       | 20       | Engineering |

※ **`dept_id` 컬럼이 양쪽에서 모두 출력**됨

---

### ✅ B. JOIN ... USING 예제

```sql
SELECT *
FROM employees
JOIN departments
USING (dept_id);
```

📌 결과:

| emp\_id | name  | dept\_id | dept\_name  |
| ------- | ----- | -------- | ----------- |
| 1       | Alice | 10       | HR          |
| 2       | Bob   | 20       | Engineering |

※ `dept_id`는 결과에서 **한 번만 출력**

---

## ✅ 4. `SELECT *` 시 출력 컬럼 차이

| 구문               | 공통 컬럼 출력 방식  |
| ---------------- | ------------ |
| `JOIN ... ON`    | **양쪽 모두 출력** |
| `JOIN ... USING` | **한 번만 출력**  |

---

## ✅ 5. 정렬(ORDER BY) 또는 GROUP BY 시 차이

`USING` 구문을 사용할 경우, 공통 컬럼은 **중복이 제거되어 단일 이름으로**만 참조 가능:

```sql
-- USING 사용 시
ORDER BY dept_id  -- 가능 (단일 컬럼으로만 존재)

-- ON 사용 시
ORDER BY e.dept_id OR d.dept_id  -- 둘 다 가능
```

---

## ✅ 6. 실무 권장 사항

| 항목            | 권장 구문            | 이유            |
| ------------- | ---------------- | ------------- |
| 컬럼 이름이 다름     | `JOIN ... ON`    | 유일한 선택지       |
| 명시적 제어 필요     | `JOIN ... ON`    | 가독성, 디버깅 용이   |
| 컬럼 중복 제거 원할 때 | `JOIN ... USING` | SELECT \*에 적합 |
| 유지보수, 확장성     | `JOIN ... ON`    | 명확함이 중요함      |

---

## ✅ 7. WHERE 조건 추가 시 동일

두 JOIN 방식 모두 이후 `WHERE`, `GROUP BY`, `ORDER BY` 사용에는 **기능적 차이 없음**
다만, 컬럼 이름의 **접두사 유무**에 주의 필요

---

## ✅ 결론 요약

| 항목                 | JOIN ... ON           | JOIN ... USING  |
| ------------------ | --------------------- | --------------- |
| 컬럼 이름 같아야 함        | ❌ 필요 없음               | ✅ 동일한 이름 필요     |
| SELECT \* 결과       | 양쪽 컬럼 모두 출력           | 중복 컬럼 한 번만 출력   |
| 유연성                | 매우 높음                 | 낮음              |
| 실무에서의 선호           | 복잡한 조인에서는 더 명확함       | 단순 조인에서 가독성은 좋음 |
| WHERE/GROUP BY 호환성 | 동일하나 컬럼 접근 방식에 유의해야 함 | 동일하나 접두사 접근 불가  |

---

## 🧠 보너스 팁

MySQL, PostgreSQL 모두 `USING`을 지원하며,
SQL 표준에서는 `USING`은 **동일한 이름의 컬럼만 허용**하는 특수한 케이스의 `ON` 구문입니다.

```sql
JOIN table2 ON table1.col = table2.col
```

과

```sql
JOIN table2 USING (col)
```

은 **기능적으로 동일하지만**, 출력 컬럼 및 참조 방식이 다릅니다.


# 🔍 INNER JOIN + GROUP BY + HAVING ?

---

## ✅ 1. 기본 구조

```sql
SELECT 컬럼, 집계함수
FROM 테이블1
JOIN 테이블2 ON ...
GROUP BY 그룹핑 컬럼
HAVING 집계조건;
```

### INNER JOIN:

* 먼저 **두 테이블을 조인**하여 하나의 중간 결과 집합 생성
* **조건에 맞는 행만 남김 (교집합)**

### GROUP BY:

* 이 조인 결과를 기준으로 **그룹핑**
* **같은 값을 가진 행끼리 묶어서 하나의 그룹으로 취급**

### HAVING:

* **그룹에 대한 조건**을 필터링
* `WHERE`은 "행 기준", `HAVING`은 "그룹 기준"임

---

## ✅ 2. 실전 예제: 주문과 고객 테이블

### 테이블 정의

**customers**

| id | name  |
| -- | ----- |
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |

**orders**

| id | customer\_id | amount |
| -- | ------------ | ------ |
| 1  | 1            | 100    |
| 2  | 1            | 150    |
| 3  | 2            | 200    |
| 4  | 3            | NULL   |

---

### 💡 고객별 총 주문 금액 구하기

```sql
SELECT c.name, SUM(o.amount) AS total_amount
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

📌 결과:

| name  | total\_amount |
| ----- | ------------- |
| Alice | 250           |
| Bob   | 200           |
| Carol | NULL          |

---

### 💡 주문 금액이 200 이상인 고객만 보기

```sql
SELECT c.name, SUM(o.amount) AS total_amount
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name
HAVING SUM(o.amount) >= 200;
```

📌 결과:

| name  | total\_amount |
| ----- | ------------- |
| Alice | 250           |
| Bob   | 200           |

❗ **Carol**은 `NULL`이므로 제외됨
❗ `HAVING`은 `GROUP BY` 이후 실행되므로 **집계 함수 결과로 조건** 가능

---

## ✅ 3. `WHERE` vs `HAVING`

| 구분     | 적용 대상         | 예                            |
| ------ | ------------- | ---------------------------- |
| WHERE  | **행(row)**    | `WHERE o.amount > 100`       |
| HAVING | **그룹(group)** | `HAVING SUM(o.amount) > 300` |

👉 `WHERE`은 **JOIN 전에 필터링**,
👉 `HAVING`은 **GROUP BY 후 필터링**

---

## ✅ 4. JOIN + GROUP BY 실전 확장 예제

### 💡 고객이 주문한 상품 수, 금액의 평균, 최대/최소

```sql
SELECT c.name,
       COUNT(o.id) AS order_count,
       AVG(o.amount) AS avg_amount,
       MAX(o.amount) AS max_amount,
       MIN(o.amount) AS min_amount
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

---

### 💡 HAVING으로 조건 추가

```sql
-- 주문 건수가 2건 이상인 고객만 보기
HAVING COUNT(o.id) >= 2;
```

---

## ✅ 5. GROUP BY 대상은 SELECT에 반드시 포함?

* **SQL 표준**: `GROUP BY`에 지정되지 않은 컬럼은 **집계 함수로만 사용 가능**
* MySQL은 `ONLY_FULL_GROUP_BY`가 꺼져 있으면 허용하지만, **PostgreSQL, Oracle**은 **오류 발생**

```sql
-- 오류 (PostgreSQL 등에서는 name이 GROUP BY에 없음)
SELECT c.name, o.amount
FROM customers c
JOIN orders o ON ...
GROUP BY c.id;
```

✅ 고급 실무 팁: `GROUP BY`에는 **기본 키** 또는 **UNIQUE 제약이 있는 컬럼**을 우선 지정

---

## ✅ 6. 내부 동작 순서 (SQL Logical Processing Order)

| 순서 | 절        | 설명                    |
| -- | -------- | --------------------- |
| 1  | FROM     | JOIN 포함: 테이블 간 결합 수행  |
| 2  | WHERE    | 행 기준 필터링              |
| 3  | GROUP BY | 같은 값을 가지는 행들을 그룹으로 묶음 |
| 4  | HAVING   | 그룹 기준으로 필터링           |
| 5  | SELECT   | 출력 컬럼 결정, 집계 함수 적용    |
| 6  | ORDER BY | 결과 정렬                 |

👉 즉, `JOIN → WHERE → GROUP BY → HAVING → SELECT` 순서로 해석

---

## ✅ 7. JOIN + GROUP BY 성능 팁

| 전략                       | 설명                               |
| ------------------------ | -------------------------------- |
| 조인 조건에 인덱스 적용            | JOIN 최적화 핵심                      |
| WHERE 조건 최대한 적용          | 불필요한 JOIN을 줄여 GROUP BY 이전 행 수 감소 |
| SELECT에 필요한 컬럼만 명시       | 불필요한 I/O 감소                      |
| 인라인 뷰(서브쿼리)로 GROUP BY 분리 | 가독성과 성능 튜닝에 도움                   |

---

## ✅ 8. 예제: 인라인 뷰로 JOIN + GROUP BY 분리

```sql
SELECT c.name, x.total_amount
FROM customers c
JOIN (
  SELECT customer_id, SUM(amount) AS total_amount
  FROM orders
  GROUP BY customer_id
) x ON c.id = x.customer_id
WHERE x.total_amount > 300;
```

→ **JOIN 전에 GROUP BY 적용** → 성능 향상

---

## ✅ 결론 요약

| 항목              | 내용                             |
| --------------- | ------------------------------ |
| INNER JOIN      | 조건 만족하는 행만 교집합 추출              |
| GROUP BY        | 조인 결과를 그룹으로 묶음                 |
| HAVING          | 그룹에 대한 조건 필터링                  |
| WHERE vs HAVING | WHERE = 행 필터링, HAVING = 그룹 필터링 |
| 성능 팁            | WHERE로 조인 전 필터링, 인덱스, 인라인 뷰 활용 |



---

# 🔍 `INNER JOIN` vs `IN` vs `EXISTS`: 비교 분석

---

## ✅ 1. 개념 요약

| 구문                  | 설명                        |
| ------------------- | ------------------------- |
| `INNER JOIN`        | 두 테이블을 교집합처럼 결합           |
| `IN (subquery)`     | 서브쿼리 결과에 포함된 값과 일치하는 행 추출 |
| `EXISTS (subquery)` | 서브쿼리가 **행이라도 반환하면 TRUE**  |

---

## ✅ 2. 기본 구조

### 🔸 INNER JOIN

```sql
SELECT e.name, d.name
FROM employee e
JOIN department d ON e.dept_id = d.id;
```

→ 양쪽 테이블의 **매칭되는 행**만 결과로

---

### 🔸 IN

```sql
SELECT name
FROM employee
WHERE dept_id IN (
    SELECT id FROM department
);
```

→ `employee.dept_id`가 `department.id` 중 하나와 **일치하면** 포함

---

### 🔸 EXISTS

```sql
SELECT name
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE e.dept_id = d.id
);
```

→ **행이 존재하기만 하면 TRUE**

---

## ✅ 3. 예제 데이터

**employee**

| id | name  | dept\_id |
| -- | ----- | -------- |
| 1  | Alice | 10       |
| 2  | Bob   | 20       |
| 3  | Carol | 30       |

**department**

| id | name        |
| -- | ----------- |
| 10 | HR          |
| 20 | Engineering |

---

## ✅ 4. 결과 비교

| 방법           | 결과         |
| ------------ | ---------- |
| `INNER JOIN` | Alice, Bob |
| `IN`         | Alice, Bob |
| `EXISTS`     | Alice, Bob |

→ `Carol`은 `dept_id = 30`인데, `department`에 없음 → 제외

---

## ✅ 5. 성능 비교 (실행 계획 관점)

### 📌 옵티마이저의 전략

| 구문           | 옵티마이저 동작 예시                             |
| ------------ | --------------------------------------- |
| `INNER JOIN` | 조인 알고리즘 (Nested Loop / Hash / Merge) 적용 |
| `IN`         | 가능하면 **세미 조인(semi-join)** 으로 변환         |
| `EXISTS`     | 가능하면 **Anti Join or Semi Join** 으로 변환   |

👉 대부분의 RDBMS는 `IN`, `EXISTS`, `JOIN`을 **동등한 실행 계획**으로 변환함
👉 단, **조건 복잡도, 인덱스, 서브쿼리 크기**에 따라 선택이 달라짐

---

## ✅ 6. 차이점 요약

| 항목       | INNER JOIN          | IN                  | EXISTS                          |
| -------- | ------------------- | ------------------- | ------------------------------- |
| 반환 행     | 매칭된 컬럼 + 조인된 테이블 컬럼 | 기본 테이블의 컬럼만         | 기본 테이블의 컬럼만                     |
| 중복 제거    | 필요 시 `DISTINCT` 필요  | 자동 없음               | 자동 없음                           |
| NULL 처리  | JOIN 조건 NULL 시 제외   | `IN (NULL)` → 결과 없음 | `EXISTS (NULL)` → 관계 없음이면 FALSE |
| 성능       | 중간 결과 큼, 인덱스 중요     | 소규모 테이블에 유리         | 대규모 서브쿼리에 유리                    |
| 옵티마이저 변환 | 조인 전략 선택            | 세미조인으로 변환 가능        | 세미조인 또는 인라인 View로 변환            |

---

## ✅ 7. 성능 비교 실전 예

### 예: 1억 건 테이블에서 특정 조건을 만족하는 사용자만 필터링

**A. JOIN 사용**

```sql
SELECT u.id, u.name
FROM users u
JOIN user_roles r ON u.id = r.user_id
WHERE r.role = 'ADMIN';
```

* `user_roles.role`에 **인덱스** 있을 경우 매우 효율적
* `user_roles`가 크면 JOIN으로 인해 **중간 결과가 매우 커질 수 있음**

---

**B. IN 사용**

```sql
SELECT id, name
FROM users
WHERE id IN (
  SELECT user_id
  FROM user_roles
  WHERE role = 'ADMIN'
);
```

* `user_roles.user_id`에 인덱스 있으면 `Semi Join`으로 최적화
* MySQL 5.x에서는 성능 하락 가능, 8.x에서는 개선

---

**C. EXISTS 사용**

```sql
SELECT id, name
FROM users u
WHERE EXISTS (
  SELECT 1
  FROM user_roles r
  WHERE r.user_id = u.id
  AND r.role = 'ADMIN'
);
```

* **서브쿼리 내부 조건이 복잡하거나 상관관계 많을 때 EXISTS 유리**
* 옵티마이저가 조건을 **Predicate Pushdown**으로 최적화 가능

---

## ✅ 8. 실무에서의 선택 기준

| 상황                       | 추천 구문                 |
| ------------------------ | --------------------- |
| 결과에서 두 테이블의 컬럼이 모두 필요할 때 | `INNER JOIN`          |
| 조건 컬럼만 매칭하고 결과는 한 테이블일 때 | `IN` 또는 `EXISTS`      |
| 서브쿼리가 크고 인덱스가 있는 경우      | `EXISTS` 우위           |
| 서브쿼리 결과가 작을 경우           | `IN` 사용 용이            |
| NULL 고려가 필요한 경우          | `JOIN` 또는 `EXISTS` 선호 |

---

## ✅ 9. NULL 관련 주의사항

```sql
-- department에 NULL 있는 경우
SELECT name
FROM employee
WHERE dept_id IN (NULL);
```

→ 결과 없음 (NULL은 어떤 값과도 비교 불가)

```sql
-- EXISTS는 서브쿼리 자체가 비어 있는지만 판단
WHERE EXISTS (SELECT 1 FROM department WHERE id IS NULL);
```

→ NULL도 행이 있으면 EXISTS는 TRUE

---

## ✅ 10. 전문가 요약 정리

| 구문         | 핵심 특징                          | 추천 시점               |
| ---------- | ------------------------------ | ------------------- |
| INNER JOIN | 관계형 모델의 정석, 교집합을 명시적으로 구성      | 두 테이블을 함께 보고 싶을 때   |
| IN         | 결과 컬럼이 하나, 조건 비교만 필요           | 작은 서브쿼리, 간단한 필터링 시  |
| EXISTS     | 조건만 만족하는지 확인, 서브쿼리 내용은 중요하지 않음 | 서브쿼리가 복잡하고 행 수가 클 때 |

---

## 💡 결론

* 세 방법은 **논리적으로 유사**하지만,
* **사용 목적**, **서브쿼리 크기**, **인덱스 유무**, **조건 복잡도**에 따라 성능과 가독성이 달라집니다.
* 실무에서는 옵티마이저가 이들을 **JOIN 또는 Semi-Join** 형태로 내부 변환하므로,
  **실행 계획(EXPLAIN)** 을 꼭 확인하고 선택하는 것이 중요합니다.





# 📌 1. **Primary Index (기본키 인덱스)**

PK를 만들면 자동으로 **클러스터드 인덱스(B-Tree)** 생성됨.

### ✔ 테이블 생성 시

```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);
```

### ✔ ALTER TABLE 로 생성

```sql
ALTER TABLE Users
ADD PRIMARY KEY (user_id);
```

**주의:**
MySQL InnoDB에서는 PK 인덱스가 **클러스터드 인덱스**이며,
한 테이블당 1개만 존재 가능.

---

# 📌 2. **Unique Index (유니크 인덱스)**

중복을 허용하지 않는 인덱스.

### ✔ UNIQUE 인덱스 추가

```sql
CREATE UNIQUE INDEX idx_users_email
ON Users (email);
```

### ✔ 테이블 생성 시

```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```

* 이메일, 주민번호, 전화번호 등에 사용
* NULL은 여러 개 허용(MySQL)

---

# 📌 3. **Composite Index (복합 인덱스)**

두 개 이상의 컬럼으로 구성된 인덱스.

### ✔ 예: (last_name, first_name)

```sql
CREATE INDEX idx_users_name
ON Users (last_name, first_name);
```

주의: **인덱스 컬럼 순서가 매우 중요함**

* (last_name, first_name)는
  last_name 단독 조건에서는 사용 O
  first_name 단독 조건에서는 사용 X

---

# 📌 4. **Covering Index (커버링 인덱스)**

쿼리에서 필요한 모든 컬럼이 인덱스에 포함되어
테이블 접근(Back to Table)을 생략하는 인덱스.

### ✔ INDEX에 컬럼을 모두 포함시키면 됨

예: 아래 쿼리 커버링 가능

```sql
SELECT email, name FROM Users
WHERE email = 'aaa@aaa.com';
```

→ email + name 둘 다 인덱스에 있어야 함

```sql
CREATE INDEX idx_users_email_name
ON Users (email, name);
```

---

# 📌 5. **Hash Index (해시 인덱스)**

MySQL에서는 **MEMORY 엔진**에서만 지원.

### ✔ MEMORY 테이블 생성 시 HASH 지정

```sql
CREATE TABLE CacheTable (
    key_col VARCHAR(50),
    value_col VARCHAR(200),
    INDEX key_index (key_col) USING HASH
) ENGINE=MEMORY;
```

특징:

* = (등가) 조건 매우 빠름
* 범위 검색 불가
* ORDER BY / BETWEEN 사용 불가

예:

```sql
SELECT * FROM CacheTable WHERE key_col = 'ABC';
```

→ O

```sql
SELECT * FROM CacheTable WHERE key_col > 'ABC';
```

→ 불가 (HASH 인덱스가 도움 안 됨)

---

# 📌 6. **Fulltext Index (전문 검색)**

자연어 또는 Boolean 검색 가능.

### ✔ Fulltext 인덱스 생성

```sql
CREATE FULLTEXT INDEX idx_users_content
ON Users (content);
```

InnoDB는 MySQL 5.6 이상에서 FULLTEXT 지원.

---

### ✔ Fulltext 검색 예제

```sql
SELECT * FROM Users
WHERE MATCH(content) AGAINST ('database tuning');
```

---

# 📌 7. **Spatial Index (공간 인덱스, GIS 인덱스)**

좌표, 지형, 경로, 위치 기반 서비스에서 사용
(MySQL: GEOMETRY 타입 기반, R-Tree 인덱스)

### ✔ Spatial 인덱스 생성

```sql
CREATE SPATIAL INDEX idx_location_geom
ON Locations (geom);
```

### ✔ 테이블 생성 시

```sql
CREATE TABLE Locations (
    id INT PRIMARY KEY,
    geom GEOMETRY NOT NULL,
    SPATIAL INDEX(geom)
);
```

**주의:**

* InnoDB에서는 `geom` 컬럼에 반드시 `NOT NULL` 필요
* Polygon, Point, LineString 같은 GIS 타입 지원

---

# 📌 8. (추가) Full Index Syntax Cheat Sheet

| 인덱스 종류    | CREATE INDEX 예제                             |
| --------- | ------------------------------------------- |
| Primary   | `PRIMARY KEY (col)`                         |
| Unique    | `CREATE UNIQUE INDEX idx ON t(col);`        |
| Composite | `CREATE INDEX idx ON t(c1, c2);`            |
| Covering  | `CREATE INDEX idx ON t(c1, c2, c3);`        |
| Hash      | `USING HASH` (MEMORY 엔진 한정)                 |
| Fulltext  | `CREATE FULLTEXT INDEX idx ON t(text_col);` |
| Spatial   | `CREATE SPATIAL INDEX idx ON t(geom);`      |

---

# 🎯 결론

여기까지의 인덱스 생성 SQL은 모두 **MySQL 기준의 실무에서 바로 사용 가능한 형태**이며,
각 인덱스의 특징과 한계까지 함께 정리되어 있습니다.


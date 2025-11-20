# 🚀 SQL INDEX

---

# 1️⃣ INDEX 란?

Index는 **테이블의 특정 컬럼에 대해 빠르게 검색할 수 있도록 만들어진 데이터 구조(B-Tree 기반)** 이다.

일상적 비유:
✔ 책의 “목차(Index)”
✔ 사전의 “ㄱ ~ ㅎ 정렬된 위치 정보”

→ 즉, 테이블 전체를 뒤지지 않고 **바로 목적지로 이동**할 수 있게 해준다.

---

# 2️⃣ INDEX의 내부 구조 (중요)

대부분의 RDBMS(MySQL InnoDB 등) 인덱스는 **B-Tree(Binary Tree의 확장 형태)** 로 구성된다.

특징:

* 정렬 상태 유지
* 검색 속도 O(log N)
* 범위 검색 지원
* 중복 허용
* INSERT/UPDATE 시 트리 재배치 비용 발생(과도한 인덱스는 오히려 느림)

---

# 3️⃣ INDEX 종류

| 종류                  | 설명                            |
| ------------------- | ----------------------------- |
| **BTREE Index**     | 기본 인덱스, 범위 검색 가능, 대부분 상황에서 사용 |
| **HASH Index**      | 정확히 일치 검색(=) 전용, 범위 검색 불가     |
| **UNIQUE Index**    | 값의 중복 불가 보장                   |
| **PRIMARY KEY**     | 테이블의 대표 인덱스                   |
| **FULLTEXT Index**  | 텍스트 검색 용도 (검색엔진 방식)           |
| **Composite Index** | 여러 컬럼 묶어서 만드는 인덱스             |

---

# 4️⃣ 인덱스를 사용해야 하는 경우 / 안 해야 하는 경우

### ✔ 사용해야 하는 경우

* WHERE 검색이 자주 발생
* JOIN 조인이 자주 발생하는 FK 컬럼
* ORDER BY / GROUP BY가 자주 발생하는 컬럼
* 데이터가 많이 쌓이는 테이블

### ❌ 사용하면 안 되는 경우

* UPDATE/INSERT가 매우 많은 테이블 (인덱스 유지 비용 증가)
* “카디널리티 낮은 컬럼”(예: 성별, YES/NO)
* 너무 많은 인덱스 → DML 성능 저하 및 디스크 낭비

---

# 5️⃣ 실습용 테이블 생성

```sql
CREATE TABLE Member (
    id        INT NOT NULL AUTO_INCREMENT,
    name      VARCHAR(50),
    email     VARCHAR(100),
    age       INT,
    city      VARCHAR(50),
    PRIMARY KEY (id)
);
```

---

# 6️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Member (name, email, age, city) VALUES
('김철수', 'kim@test.com', 25, 'Seoul'),
('이영희', 'lee@test.com', 32, 'Busan'),
('박민수', 'minsu@test.com', 19, 'Seoul'),
('최가영', 'ga@test.com', 28, 'Daegu'),
('홍길동', 'hong@test.com', 45, 'Seoul');
```

---

# 7️⃣ INDEX 실습 예제

---

## ✔ 예제 1) 단일 컬럼 인덱스 생성

```sql
CREATE INDEX idx_member_name
ON Member(name);
```

EXPLAIN:

```sql
EXPLAIN SELECT * FROM Member WHERE name='김철수';
```

→ key: idx_member_name
→ type: ref (빠른 검색)

---

## ✔ 예제 2) 인덱스 생성 후 정렬 성능 향상 확인

```sql
EXPLAIN SELECT * FROM Member ORDER BY name;
```

→ key: idx_member_name
→ filesort 사용 안함

---

## ✔ 예제 3) 인덱스 삭제

```sql
DROP INDEX idx_member_name ON Member;
```

---

## ✔ 예제 4) UNIQUE 인덱스 생성

```sql
CREATE UNIQUE INDEX idx_member_email ON Member(email);
```

중복 시도:

```sql
INSERT INTO Member (name,email,age,city)
VALUES ('테스트','kim@test.com', 33, 'Incheon');
```

→ ❌ Duplicate entry 에러

---

## ✔ 예제 5) Composite Index(복합 인덱스)

```sql
CREATE INDEX idx_city_age
ON Member(city, age);
```

활용:

```sql
EXPLAIN SELECT * FROM Member WHERE city='Seoul' AND age > 20;
```

→ key: idx_city_age
→ 범위 조건 지원(BTREE)

---

## ✔ 예제 6) Composite Index 왼쪽 우선 법칙

```sql
EXPLAIN SELECT * FROM Member WHERE age > 20;
```

→ idx_city_age 사용 ❌
(왼쪽 컬럼 city를 사용하지 않으면 인덱스 사용 불가)

---

## ✔ 예제 7) LIKE 검색 최적화

```sql
CREATE INDEX idx_member_email2 ON Member(email);

EXPLAIN SELECT * FROM Member WHERE email LIKE 'kim%';
```

→ 인덱스 사용됨 (문자열 앞부분 고정)

---

## ✔ 예제 8) LIKE '%문자' 는 인덱스 사용 안 됨

```sql
EXPLAIN SELECT * FROM Member WHERE email LIKE '%test.com';
```

→ full scan
→ key=NULL

---

## ✔ 예제 9) FULLTEXT Index 생성

```sql
ALTER TABLE Member
ADD FULLTEXT INDEX ft_name (name);
```

검색:

```sql
SELECT * FROM Member
WHERE MATCH(name) AGAINST('홍길동');
```

---

## ✔ 예제 10) 인덱스 활용한 JOIN 최적화

```sql
CREATE INDEX idx_city ON Member(city);

EXPLAIN SELECT m.name, m.city
FROM Member m 
JOIN CityInfo c ON m.city = c.city_name;
```

→ key: idx_city
→ ref join

---

## ✔ 예제 11) ORDER BY + LIMIT 최적화

```sql
CREATE INDEX idx_age ON Member(age);

EXPLAIN SELECT * FROM Member ORDER BY age LIMIT 3;
```

→ 인덱스 범위 스캔으로 빠름

---

## ✔ 예제 12) GROUP BY 최적화

```sql
EXPLAIN SELECT city, COUNT(*) 
FROM Member GROUP BY city;
```

→ idx_city 사용 가능

---

## ✔ 예제 13) 인덱스 있는 컬럼 UPDATE 비용 증가 시나리오

```sql
UPDATE Member SET name='TEST_NAME' WHERE id=1;
```

→ idx_member_name 재정렬 필요 → 비용 증가

---

## ✔ 예제 14) WHERE 절에서 함수 사용 시 인덱스 사용 안 됨

```sql
EXPLAIN SELECT * FROM Member WHERE LOWER(name)='김철수';
```

→ key=NULL
→ 인덱스 사용 안 함

---

## ✔ 예제 15) 인덱스 힌트 사용

```sql
SELECT * FROM Member FORCE INDEX (idx_city)
WHERE city='Seoul';
```

→ 강제로 인덱스 사용하도록 지시

---

# 8️⃣ INDEX 사용 시 실무 전문가 팁

---

### ✔ 1) 인덱스는 적게 만들수록 좋다

많으면 INSERT/UPDATE 성능 하락.

---

### ✔ 2) WHERE 절이 자주 쓰이는 컬럼 우선

조회 빈도 > 데이터 크기 > 카디널리티 중요도 순으로 고려.

---

### ✔ 3) Composite Index는 “왼쪽 우선 법칙”이 무조건 적용

(city, age) → city 조건이 없으면 인덱스 못 씀.

---

### ✔ 4) LIKE '%검색어' 는 인덱스 사용 불가

ELK, FULLTEXT, Search Engine 고려.

---

### ✔ 5) ORDER BY / GROUP BY 는 인덱스 여부에 큰 영향

정렬방식(filesort) 회피 가능.

---

### ✔ 6) 대규모 테이블 변경 시 Online DDL 필요

MySQL 8.0의 ALGORITHM=INPLACE 옵션 고려.

---

### ✔ 7) 인덱스가 많으면 디스크 용량 증가

인덱스 파일 별도 존재.

---

### ✔ 8) JOIN 컬럼은 반드시 인덱스 생성

JOIN 성능의 90%는 인덱스가 결정.


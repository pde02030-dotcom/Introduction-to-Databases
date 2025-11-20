# 🚀 SQL Optimizer

### — RDBMS의 두뇌, 실행계획을 만들어내는 결정 엔진의 모든 것

많은 개발자가 SQL 옵티마이저를 단순히
“인덱스를 선택해주는 엔진” 정도로 이해합니다.

하지만 실제 옵티마이저는 다음을 모두 고려하는
**복잡한 비용 기반 최적화 엔진(Cost Based Optimizer, CBO)** 입니다.

> ✔ 어떤 인덱스를 사용할지
> ✔ 어떤 JOIN 순서를 사용할지
> ✔ 어떤 조인 알고리즘을 쓸지
> ✔ 어떤 범위 검색이 빠른지
> ✔ 필터 순서를 어떻게 구성할지
> ✔ 정렬을 할지, 인덱스 정렬을 이용할지
> ✔ 임시 테이블이 필요한지
> ✔ 병목이 발생하지 않는 조합은 무엇인지

즉, 옵티마이저는 SQL 쿼리를 최소 비용으로 수행하기 위한
**최적의 실행 계획(Execution Plan)을 찾는 두뇌**입니다.

---

# 1️⃣ Optimizer란 무엇인가?

### ✔ Optimizer = 실행계획 생성 엔진

Optimizer는 SQL 문을 분석하여 다음 정보를 만든다:

* 어떤 테이블을 먼저 읽을지
* 어떤 인덱스를 사용할지
* 어떤 Join 알고리즘을 사용할지
* 어떤 정렬 방식(filesort vs index-order)
* 어떤 필터 순서를 적용할지
* 어떤 임시 테이블을 만들지

이 모든 결과가
`EXPLAIN` 명령으로 출력되는 실행계획이다.

---

# 2️⃣ Optimizer의 핵심 목표

### 🔥 **“가장 비용이 낮은 실행계획 선택”**

비용(cost)은 단순 시간이 아니라 다음 요소를 포함한다:

* 디스크 I/O 횟수
* 메모리 사용
* CPU 연산
* 인덱스 depth
* 예상되는 row 수
* 랜덤 I/O vs 순차 I/O
* nested loop join 비용
* 정렬 비용

즉, 옵티마이저는
“가장 빨라 보이는 실행계획”이 아니라
“예상 비용이 가장 낮은 실행계획”
을 선택한다.

---

# 3️⃣ 옵티마이저는 어떻게 실행계획을 만들까? (내부 동작)

MySQL 옵티마이저의 실행계획 생성 과정은 6단계로 이뤄진다:

---

## ✔ Step 1. 파싱(Parsing)

SQL 문법을 분석하여 파스 트리(Abstract Syntax Tree)를 생성.

---

## ✔ Step 2. 재작성(Query Rewrite)

옵티마이저가 SQL을 더 최적화된 형태로 “자동 변환”한다.

예:

* WHERE 조건식 재배치
* JOIN 순서 재구성
* Subquery → Semijoin 변환
* OR 조건 → UNION 변환
* 불필요한 서브쿼리 제거

MySQL 8.0 이후 많은 rewrite가 자동으로 수행된다.

---

## ✔ Step 3. 통계 정보 읽기 (Statistics)

옵티마이저가 비용을 계산하기 위해 테이블/인덱스 통계를 읽는다.

통계 포함:

* 각 컬럼의 카디널리티(유니크 비율)
* 인덱스 깊이(B-tree depth)
* 데이터 분포 히스토그램(histograms)
* row count
* NULL 비율

이 정보가 잘못되면 **잘못된 실행 계획**이 나온다.

---

## ✔ Step 4. 가능한 모든 실행계획 후보 생성

옵티마이저는 가능한 모든 조합을 탐색한다.

예:

* 사용할 수 있는 모든 인덱스 조합
* 가능한 JOIN 순서
* 가능한 JOIN 알고리즘
* WHERE 조건의 적용 순서
* 정렬 방식(filesort vs index order)

단, MySQL은 완전 탐색이 아닌 **비용 기반 휴리스틱 탐색**(Pruning) 방식 사용.

---

## ✔ Step 5. 비용(Cost) 계산

각 실행계획 후보에 대해 비용을 계산한다.

비용 산정 요소:

* 인덱스 탐색 비용
* Range Scan 비용
* Row Estimate * CPU 비용
* Join buffer 크기
* Nested loop 비용
* Random I/O vs Sequential I/O
* 정렬 비용
* 임시 테이블 비용

MySQL에서 cost는 “절대 시간”이 아니라 “추정된 상대 비용”을 의미.

---

## ✔ Step 6. 최소 비용의 실행계획 선택

모든 후보 중 가장 비용이 낮은 계획이 최종 실행 계획이 된다.

→ 이 결과가 `EXPLAIN`으로 출력되는 것이다.

---

# 4️⃣ Optimizer가 고려하는 요소 (아주 중요)

### ✔ 1) 카디널리티(Cardinality)

유니크한 값의 수가 많을수록 인덱스 효율이 높다.

예: email 주소 → 인덱스 효율 최고
성별 M/F → 인덱스 효율 최악

---

### ✔ 2) selectivity(선택도)

“조건이 얼마나 row를 좁히는가”

예:

```sql
WHERE paid = 1  (50만 / 100만)
→ 선택도 = 약 50% = 나쁨
```

```sql
WHERE email = 'x'
→ 선택도 ≈ 0.000001% = 매우 좋음
```

---

### ✔ 3) 인덱스 구성 및 우선순위

복합 인덱스는 “왼쪽 컬럼”이 핵심.

---

### ✔ 4) Join 순서

옵티마이저는 항상:

> “가장 조건이 좁혀지는 테이블을 먼저 읽는다”

---

### ✔ 5) Join 알고리즘

MySQL는 보통 Nested Loop Join을 사용.

옵티마이저는:

* 어떤 테이블을 드라이빙 테이블로 할지
* 어떤 인덱스를 통해 반복할지

모두 계산한다.

---

# 5️⃣ Optimizer가 사용하는 통계 정보

MySQL 8.0 이후 통계는 매우 정교해졌다:

* **Persistent Statistics**
* **Histogram Statistics**
* **Dynamic Sampling**
* **Cardinality estimation** 개선

특히 **히스토그램(histogram)** 활성화 시 정확도가 크게 올랐다:

```sql
ANALYZE TABLE member UPDATE HISTOGRAM ON age;
```

---

# 6️⃣ Optimizer가 실패하는 경우 (잘못된 실행계획의 원인)

### 1) 통계 정보 오래됨

→ 옵티마이저가 “행이 적다”고 판단해 잘못된 인덱스 선택

### 2) 조건식에 함수 사용

→ 인덱스 무력화

### 3) OR 조건

→ 인덱스 사용 어려움

### 4) 데이터 기울기(skew)

예:
99%는 status=0, 1%는 status=1
인데 status=1을 찾는 쿼리에서 인덱스를 안 쓰는 경우

### 5) 복합 인덱스 왼쪽 컬럼 조건을 누락

(city, age) 인덱스에

```sql
WHERE age > 20
```

→ 인덱스 작동 안 함

---

# 7️⃣ 옵티마이저 힌트 (Optimizer Hints)

MySQL은 다음과 같은 힌트를 제공:

* **USE INDEX**
* **FORCE INDEX**
* **IGNORE INDEX**
* **STRAIGHT_JOIN**
* **OPTIMIZER_TRACE**
* **SET optimizer_switch**

예:

```sql
SELECT * FROM Member FORCE INDEX(idx_city) WHERE city='Seoul';
```

하지만 힌트는 “옵티마이저를 이긴다”는 의미.
→ 최후의 수단으로만 사용.

---

# 8️⃣ Optimizer 관련 실무 튜닝 전략 20개

1. 무조건 먼저 EXPLAIN으로 실행계획 파악
2. 조건식에서 함수 제거 (LOWER, YEAR 등)
3. 복합 인덱스는 왼쪽 컬럼 우선
4. 카디널리티 낮은 컬럼에 인덱스 만들지 말 것
5. Histogram 업데이트로 통계 향상
6. Index Condition Pushdown 활용
7. Covering Index 적극 활용
8. JOIN 컬럼 반드시 인덱스
9. OR 조건 피하기 → UNION으로 분리
10. LIMIT + ORDER BY 최적화는 “인덱스 정렬”로
11. LIKE ‘%abc’ 피하기
12. 큰 테이블의 COUNT(*)는 인덱스 활용
13. JOIN 순서를 옵티마이저에 맡기되 필요시 STRAIGHT_JOIN
14. 인덱스 너무 많으면 INSERT/UPDATE 느려짐
15. 조건식 복잡하면 서브쿼리로 분리
16. WHERE + ORDER BY에 동일 인덱스 사용
17. 옵티마이저가 파일 정렬(filesort)일 경우 인덱스로 해결
18. 임시 테이블(using temporary)은 빨간 불
19. 풀스캔(type=ALL) 발생 시 반드시 인덱스 검토
20. EXPLAIN ANALYZE 로 실제 시간 측정하여 교차 검증

---

# ✨ 최종 요약

> **Optimizer는 SQL의 실행 계획을 자동으로 생성하고, 그 중 가장 비용이 낮은 실행 방법을 선택하는 데이터베이스의 두뇌다.
> 통계 정보, 인덱스, 조인 순서, 필터 조건, 정렬 방식 등을 모두 계산해 최적의 plan을 만들고, 그 결과가 EXPLAIN으로 나타난다.**



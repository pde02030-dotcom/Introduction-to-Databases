# 🚀 ERD(Entity–Relationship Diagram)

ERD는 단순히 “표를 그리는 도구”가 아닙니다.
ERD는 **비즈니스를 데이터 관점으로 추상화하고, 논리적 모델과 물리적 모델을 연결하는**
데이터베이스 설계의 핵심 아티팩트입니다.

ERD는:

* **업무 프로세스**를 데이터로 재해석하고
* **엔터티(Entity)** 를 도출하고
* **속성(Attribute)** 을 정의하고
* 엔터티 간의 **카디널리티(Cardinality)** 와 **관계(Relationship)** 를 설정하고
* **정규화(Normalization)** 를 적용하며
* **Primary Key / Foreign Key 전략**을 결정하고
* **식별관계/비식별관계** 를 판단하고
* 최종적으로 **물리 모델로 변환**하기 위한 블루프린트입니다.

---

# 1️⃣ ERD란 무엇인가? (정확한 정의)

**ERD(Entity-Relationship Diagram)** 는
현실 세계의 개체(Entity)와 그 사이 관계(Relationship)를
데이터베이스 구조로 표현하는 **모델링 다이어그램**.

ERD는 세 가지 구성 요소로 이루어져 있다:

1. **Entity (엔터티)**
2. **Attribute (속성)**
3. **Relationship (관계)**

하지만 **세부 구현 관점** 다음 요소가 필수로 포함된다:

* 식별관계 / 비식별관계
* 카디널리티(1:1, 1:N, N:M)
* PK / FK 키 전략
* 정규화(1NF, 2NF, 3NF, BCNF) 
* 후보키 / 대체키
* 도메인 모델링
* 논리 모델 → 물리 모델 변환

---

# 2️⃣ 엔터티(Entity) — 무엇을 저장할 것인가?

엔터티는 “저장해야 할 정보의 집합”이다.

엔터티 판단 기준(5가지):

1. **업무에서 필요하다**
2. **고유 식별자(PK)가 존재한다**
3. **속성이 2개 이상 존재한다**
4. **다른 엔터티와 관계가 존재한다**
5. **2건 이상 존재한다(집합 개념)**

예:

* 회원(Member)
* 주문(Order)
* 결제(Payment)
* 강의(Course)
* 수강(StudentCourse)

엔터티는 반드시 **명사형**으로 정의한다.

---

# 3️⃣ 속성(Attribute) — 엔터티를 설명하는 정보

속성은 엔터티가 가지는 데이터.

종류:

### ✔ 기본 속성

일반 데이터(이름, 나이 등)

### ✔ 식별 속성

엔터티를 유일하게 식별하는 속성(PK 후보)

### ✔ 외래 속성

다른 엔터티의 PK를 참조하는 FK

### ✔ 파생 속성

계산으로 만들어지는 속성 (total_price, age 등)
→ 물리 모델에서는 대부분 컬럼으로 저장하지 않는다.

---

# 4️⃣ 관계(Relationship) — 엔터티 사이의 연결

세 가지 관계가 있다:

| 관계  | 설명                                                      |
| --- | ------------------------------------------------------- |
| 1:1 | 한 엔터티의 한 행이 다른 엔터티의 한 행과 연결                             |
| 1:N | 가장 흔함. 한 엔터티가 여러 개와 연결                                  |
| N:M | 직접 저장 불가 → 반드시 **조인 엔터티(Bridge, Associative)** 로 풀어야 한다 |

예:
학생 – 강의: **N:M**
→ 수강(StudentCourse) 엔터티가 필요

---

# 5️⃣ 카디널리티(Cardinality) — 관계의 정량적 표현

카디널리티는 관계에서 한 엔터티가 다른 엔터티와 연결되는 수량.

종류:

* 1:1
* 1:N
* N:M
* (0,1), (1,N) 같은 최소/최대 표현도 존재

보다 면밀히 따져보면 다음이 중요함:

* (1,1) Mandatory (필수 관계)
* (0,N) Optional (선택 관계)
* (1,N) Mandatory Many
* (0,1) Optional One

예:
Order → Payment

* 주문은 반드시 1개의 결제를 가진다: (1,1)
* 결제는 반드시 1개의 주문에 속한다: (1,1)

---

# 6️⃣ PK / FK / 키 전략

### ✔ Primary Key (PK)

* 테이블을 **식별**하는 유일한 키
* 변경되면 안 됨
* NULL 불가
* 인덱스 반드시 생성됨

### ✔ Candidate Key(후보키)

PK가 될 수 있는 여러 키들

### ✔ Alternate Key(대체키)

PK로 선택되지 않은 후보키

### ✔ Surrogate Key(대리 PK)

실제 업무와 무관한 키(AUTO_INCREMENT, UUID)

전문가들은 PK를 다음 기준으로 선택한다:

✔ 안정성(Stable)
✔ 불변성(Immutable)
✔ 유일성(Unique)
✔ 최소성(Minimal)

---

# 7️⃣ 식별관계 vs 비식별관계 (DB 모델링의 핵심)

## ✔ 식별관계(Identifying Relationship)

부모 PK가 자식의 **PK에 포함**되는 관계
→ 강한 종속 관계
→ ERD에서 **실선**

예:
OrderItem

* order_id (PK + FK)
* product_id (PK + FK)

→ 복합 PK 구성

---

## ✔ 비식별관계(Non-identifying Relationship)

부모 PK가 자식의 **일반 컬럼(FK)** 으로만 존재
→ ERD에서 **점선**

예:
Order
Payment

* order_id (FK)
* payment_id (PK)

---

# 8️⃣ 정규화(Normalization) — ERD의 질을 결정하는 원칙

ERD의 품질은 **정규화 수준**과 거의 비례합니다.

### ✔ 1NF: 원자성

컬럼은 더 이상 분해되지 않는다.
"주소"를 하나의 컬럼에 넣으면 비정규.

### ✔ 2NF: 부분 함수 종속 제거

복합 PK의 일부에만 종속되는 속성 제거

### ✔ 3NF: 이행 함수 종속 제거

PK에 종속되지 않고 다른 일반 속성에 종속되는 속성 제거

### ✔ BCNF: 결정자(Determinant)가 모두 후보키일 것

실무에서는 3NF + 일부 역정규화 전략 사용.

---

# 9️⃣ 논리 모델 → 물리 모델 변환 과정

ERD는 크게 두 단계가 있다:

### ✔ Logical ERD (논리 모델)

* 비즈니스 중심
* PK 후보 정의
* 관계 설계
* 정규화 적용
* 데이터 타입 세부 미정

### ✔ Physical ERD (물리 모델)

* 실제 DBMS에 맞춘 모델
* 데이터 타입 지정
* PK/FK 물리적 구현
* 인덱스 설계
* 파티션링
* Auto Increment / Sequence
* Not Null / Default 설정

(논리 모델은 DBMS 무관, 물리 모델은 MySQL/Oracle/PostgreSQL 등 DBMS별로 따름)

---

# 🔟 실무 ERD 설계 예시 (전문가 수준 모델링)

### ▶ 회원(Member)

* member_id (PK)
* email (AK, unique)
* password
* name
* reg_date

### ▶ 주문(Order)

* order_id (PK)
* member_id (FK)
* order_date
* total_price

### ▶ 주문상품(OrderItem) — 식별관계

* order_id (PK + FK)
* product_id (PK + FK)
* quantity
* price

### ▶ 결제(Payment)

* payment_id (PK)
* order_id (FK)
* amount
* status

### 관계:

* Member 1:N Order
* Order 1:N OrderItem
* Order 1:1 Payment
* OrderItem N:1 Product

---

# 1️⃣1️⃣ 좋은 ERD의 기준 (상위 DBA 기준)

✔ 엔터티가 업무를 정확히 반영해야 한다 <br>
✔ 속성은 최소 원자성을 지녀야 한다 <br>
✔ 식별자 규칙이 일관적이어야 한다 <br>
✔ 모든 관계는 FK로 구현되어야 한다 <br>
✔ 정규화 뒤, 필요한 경우 역정규화 판단해야 한다 <br>
✔ 인덱싱 전략을 고려해 물리 모델을 작성해야 한다 <br>
✔ 확장성과 변경 용이성을 갖춰야 한다 <br>
✔ 중복 데이터 최소화 <br>
✔ FK 삭제/갱신 정책(on delete, on update) 일관성 유지 <br>
✔ Nullability 설계 명확 <br>
✔ 카디널리티 정확 <br>

---

# ✨ 최종 요약

> **ERD는 엔터티·속성·관계·키·정규화에 기반한 데이터 구조의 공식적 설계 문서이며,
> 논리 모델과 물리 모델을 연결하여 안정적이고 확장 가능한 데이터베이스를 구축하기 위한 필수 아키텍처 도구이다.**


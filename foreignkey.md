🔑 외래키(Foreign Key)란?

외래키는 **한 테이블의 컬럼이 다른 테이블의 기본키(Primary Key)나 유니크키(Unique Key)를 참조**하도록 설정된 제약조건입니다.

✅ **정의:** 한 테이블의 값이 다른 테이블의 기본키(또는 유니크키)로부터만 입력될 수 있도록 보장하는 제약 조건  
✅ **목적:** 데이터 간의 관계(relationship)와 참조 무결성(referential integrity)을 유지하기 위해 사용

📘 외래키 구성 요소

| 요소 | 설명 |
| --- | --- |
| **자식 테이블(child)** | 외래키를 가진 테이블 (참조하는 쪽) |
| **부모 테이블(parent)** | 외래키가 참조하는 테이블 (기준이 되는 쪽) |
| **외래키 컬럼(FK column)** | 자식 테이블의 특정 컬럼, 부모 테이블의 기본키나 유니크키를 참조 |
| **참조 대상(Referenced column)** | 부모 테이블의 기본키 또는 유니크 제약이 설정된 컬럼 |

예:

```
CREATE TABLE DEPARTMENT (
    DEPT_ID INT PRIMARY KEY,
    NAME VARCHAR(100)
);

CREATE TABLE EMPLOYEE (
    EMP_ID INT PRIMARY KEY,
    NAME VARCHAR(100),
    DEPT_ID INT,
    FOREIGN KEY (DEPT_ID) REFERENCES DEPARTMENT(DEPT_ID)
);
```

```
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT
);

ALTER TABLE orders ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id) REFERENCES customers(id)
ON DELETE CASCADE
ON UPDATE CASCADE;
```

```
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customer
FOREIGN KEY (customer_id)
REFERENCES customers(id)
ON DELETE CASCADE
ON UPDATE CASCADE;
```

📏 외래키의 제약 조건

1\. 참조 무결성(Referential Integrity)

외래키는 자식 테이블이 **존재하지 않는 부모 값**을 가질 수 없도록 제한합니다.

-   자식 테이블의 외래키 값은 반드시 부모 테이블에 존재해야 함
-   부모 테이블의 값이 삭제/변경되면 자식 테이블에도 영향을 줌

2\. 제약 옵션 (ON DELETE / ON UPDATE)

| 옵션 | 설명 |
| --- | --- |
| `CASCADE` | 부모 삭제/수정 시, 자식도 같이 삭제/수정 |
| `SET NULL` | 부모 삭제/수정 시, 자식의 외래키를 `NULL`로 설정 |
| `SET DEFAULT` | 자식의 외래키 값을 기본값으로 변경 (거의 사용 안 함) |
| `RESTRICT` / `NO ACTION` | 자식이 있으면 부모 삭제/수정 금지 (기본값) |

예:

```
FOREIGN KEY (DEPT_ID) REFERENCES DEPARTMENT(DEPT_ID)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

🧠 실무 설계 전략

1\. 외래키는 반드시 인덱싱하라

-   **자식 테이블의 FK 컬럼에는 인덱스가 필요**
-   조인 성능 향상
-   ON DELETE/UPDATE 시 무결성 검사 효율

```
CREATE INDEX idx_emp_dept_id ON EMPLOYEE(DEPT_ID);
```

2\. 대용량 트랜잭션에서는 외래키를 잠시 제거하기도 함

-   대규모 데이터 이관/삭제 시 **외래키 제약 때문에 성능 저하** 발생
-   데이터 검증이 충분히 이루어진 경우에 한해 외래키 제거 후 작업

3\. 관계 모델링 시 방향을 신중하게 설계하라

-   외래키는 **데이터 흐름을 제한**하므로, 지나치게 연결된 구조는 피해야 함
-   **회전 관계(cyclic dependency)**는 반드시 피해야 함

🧪 외래키로 인해 발생 가능한 오류들

| 오류 메시지 | 원인 |
| --- | --- |
| `Cannot add or update a child row: a foreign key constraint fails` | 부모 테이블에 없는 값을 자식 테이블에 넣으려 함 |
| `Cannot delete or update a parent row: a foreign key constraint fails` | 자식 테이블에 관련 레코드가 남아 있어 삭제 불가 |

📉 외래키와 성능 고려사항

-   외래키는 SELECT에는 성능 영향 없음
-   INSERT/UPDATE/DELETE에는 무결성 검사 비용 존재
-   자식 테이블의 외래키 컬럼이 인덱스 없으면 **DELETE/UPDATE 시 Full Table Scan** 발생

🧩 외래키와 ERD(Entity-Relationship Diagram)

-   ERD에서 외래키는 \*\*관계선(line)\*\*으로 표현됨
-   1:N, N:1, N:M 관계 표현에 있어 핵심
-   설계 초기부터 FK 설정이 명확해야 데이터 무결성과 추후 확장성이 좋아짐

✅ 요약

| 항목 | 설명 |
| --- | --- |
| 외래키 목적 | 참조 무결성 보장 |
| 참조 대상 | 부모 테이블의 PK/Unique Key |
| 제약 옵션 | CASCADE, SET NULL, RESTRICT 등 |
| 실무 팁 | FK 컬럼에 인덱스 필수, 순환 참조 피하기 |
| 주의사항 | 성능에 영향, 트랜잭션 처리 시 조심 |

📚 부록: 실전 SQL 예제

```
-- 1. 부모 테이블
CREATE TABLE CATEGORY (
    ID INT PRIMARY KEY,
    NAME VARCHAR(50)
);

-- 2. 자식 테이블
CREATE TABLE PRODUCT (
    ID INT PRIMARY KEY,
    NAME VARCHAR(100),
    CATEGORY_ID INT,
    FOREIGN KEY (CATEGORY_ID) REFERENCES CATEGORY(ID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

**왜 'N' 쪽에 외래키를 두나요?**

이해를 돕기 위해 예를 들어볼게요. '학과'와 '학생'의 관계를 생각해 봅시다.

-   **1 (학과):** 하나의 학과에는 여러 명의 학생이 소속될 수 있습니다. (1:N 관계에서 '1'에 해당)
-   **N (학생):** 한 명의 학생은 반드시 하나의 학과에만 소속될 수 있습니다. (1:N 관계에서 'N'에 해당)

이 경우, 외래키는 **'학생' 테이블**에 위치합니다. 즉, '학생' 테이블에 학과\_ID라는 컬럼을 만들고, 이 컬럼이 '학과' 테이블의 학과\_ID(기본키)를 참조하도록 설정하는 거죠.

**외래키를 'N' 쪽에 두는 실질적인 이유:**

1.  **데이터 무결성 유지**: 각 학생은 반드시 존재하는 학과에만 소속되어야 합니다. '학생' 테이블에 외래키를 두면, 데이터베이스가 유효하지 않은 학과\_ID가 입력되는 것을 막아줍니다.
2.  **확장성 및 효율성**:
    -   새로운 학생이 입학하여 특정 학과에 배정될 때, '학생' 테이블에 학과\_ID만 넣어주면 됩니다. '학과' 테이블을 수정할 필요가 없죠.
    -   만약 반대로 '학과' 테이블에 외래키를 둔다면 어떨까요? '학과' 테이블에 소속된 모든 학생들의 ID를 다 기록해야 할 겁니다. 학생이 추가될 때마다 '학과' 테이블을 계속 수정해야 하고, 한 학과에 학생이 너무 많아지면 테이블 구조도 복잡해지겠죠? 이는 매우 비효율적입니다.
3.  **쿼리 용이성**: 일반적으로 '학생' 정보를 조회하면서 '학생이 어떤 학과에 소속되어 있는지'를 함께 보고 싶은 경우가 많습니다. 외래키가 '학생' 테이블에 있으면, 두 테이블을 쉽게 조인하여 원하는 정보를 얻을 수 있습니다.

# 🚀 SELECT 쿼리의 출력 결과: “Result Set(결과 집합)

### — SQL 표준 관점 · DB 내부 구조 · JDBC ResultSet · Cursor 기반 처리까지

`SELECT` 쿼리를 실행하면 DB는 “출력 결과”를 반환합니다.
이 출력 결과를 SQL 표준에서는 **Result Set(결과 집합)** 이라고 부릅니다.

Result Set을 정확히 이해하면

* SQL 처리 방식
* JDBC / ORM의 내부 동작
* Cursor 기반 스트리밍
* 대용량 조회 시 메모리 전략
  을 완벽히 이해할 수 있습니다.

---

# 1️⃣ Result Set이란? (SQL 표준 공식 정의)

SQL 표준(ISO/IEC 9075)에서 Result Set은 다음과 같이 정의됩니다:

> **SELECT문의 실행 결과로 반환되는 "행(row)의 집합"이며,
> 각 행은 동일한 컬럼 구조를 가진 테이블 형태의 논리적 데이터 집합이다.**

즉,

✔ SELECT 결과 전체 = Result Set
✔ 테이블처럼 생겼지만 실제 테이블은 아님
✔ “조회 시점”의 스냅샷(Snapshot)

---

# 2️⃣ Result Set = “가상의 테이블 Virtual Table”

Result Set은 다음 특징을 갖습니다:

| 특징                | 설명                             |
| ----------------- | ------------------------------ |
| **정형 구조**         | 테이블처럼 row와 column으로 구성         |
| **스키마 존재**        | SELECT 절에서 정의한 컬럼 순서·데이터 타입    |
| **일관성**           | 쿼리 실행 시점의 데이터 스냅샷              |
| **불변(Immutable)** | Result Set은 결과일 뿐, 수정을 반영하지 않음 |
| **일시적 객체**        | 메모리 혹은 서버 버퍼에서 잠시 존재           |

이 때문에 Result Set을 “**조회 결과 테이블**” 또는 “**가상의 테이블**”이라고 부른다.

---

# 3️⃣ Result Set의 내부 구성

Result Set은 크게 세 요소로 구성됩니다:

1. **Column Metadata**

   * 컬럼 이름
   * 데이터 타입
   * 순서
   * Null 여부

2. **Row Data**

   * 각 행의 실제 값들

3. **Cursor(커서) 상태 정보**

   * 현재 읽고 있는 row 위치
   * next(), previous() 가능 여부
   * streaming 여부

이 세 개가 합쳐져 Result Set이 된다.

---

# 4️⃣ Result Set을 만드는 과정 (DB 내부)

SELECT 실행 흐름:

```
SQL 파싱 → 옵티마이저 → 실행 계획 → 실행 엔진 → Result Set 생성
```

### ✔ Step 1) 파싱

SQL 문법 검사 후 파스 트리 생성

### ✔ Step 2) 옵티마이저

인덱스, join 순서, 필터 전략 결정

### ✔ Step 3) 실행 엔진

테이블에서 행을 읽어옴

### ✔ Step 4) Result Set 생성

SQL 엔진이 결과를 row 단위로 버퍼에 채우고
클라이언트(드라이버)로 스트리밍

✔ 결과 = DB서버가 만든 “정렬된 행들의 집합”

---

# 5️⃣ Result Set과 Cursor(커서)의 관계

Result Set은 단독으로 존재하지 않습니다.
**항상 Cursor(커서)** 와 함께 동작합니다.

> **Cursor = Result Set의 행(row)을 하나씩 순회하기 위한 포인터**

DB는 Result Set 전체를 무조건 메모리에 올리지 않습니다.
대부분 **커서 기반 Streaming 방식**을 사용합니다.

---

# 6️⃣ JDBC에서 ResultSet이란?

Java에서 SELECT 실행 시 반환되는 객체는:

```java
ResultSet rs = statement.executeQuery();
```

JDBC ResultSet은 SQL Result Set을 자바 객체로 래핑(wrapping)한 것이다.

### 주요 특징:

| 기능                      | 설명                           |
| ----------------------- | ---------------------------- |
| **커서 기반**               | next() 호출 시 다음 row를 읽고 커서 이동 |
| **forward only**        | 기본적으로 앞으로만 이동                |
| **type-safe get() 메소드** | getString, getInt, getDate 등 |
| **Lazy Loading**        | 필요한 시점에 row 가져옴              |
| **Streaming 가능**        | 대용량 처리 시 fetchSize 조절 가능     |

---

# 7️⃣ Result Set의 유형(중요)

JDBC에서 ResultSet은 두 가지 축으로 구분된다.

---

## ✔ 1) Cursor 방향 (Scroll Type)

| 타입                          | 설명                      |
| --------------------------- | ----------------------- |
| **TYPE_FORWARD_ONLY**       | 앞으로만 이동 (가장 빠르고 일반적)    |
| **TYPE_SCROLL_INSENSITIVE** | 앞/뒤로 이동 가능. 변경 데이터 반영 X |
| **TYPE_SCROLL_SENSITIVE**   | 앞/뒤 이동 + 변경 반영          |

---

## ✔ 2) ResultSet 동작 방식 (Concurrency Type)

| 타입                   | 설명                |
| -------------------- | ----------------- |
| **CONCUR_READ_ONLY** | 읽기 전용 (일반적)       |
| **CONCUR_UPDATABLE** | rs.updateRow() 가능 |

일반적으로 ResultSet은:

```
TYPE_FORWARD_ONLY + CONCUR_READ_ONLY
```

이 조합이 가장 빠르고 안정적이다.

---

# 8️⃣ Result Set은 언제 사라지는가?

Result Set은 영구 객체가 아니다.

* Statement/PreparedStatement를 닫으면 사라짐
* Connection을 닫으면 당연히 사라짐
* JDBC 드라이버가 버퍼를 폐기하면 사라짐
* 서버 커서가 끝까지 읽히면 사라짐

즉, **일시적으로만 존재하는 “조회 결과 버퍼”** 이다.

---

# 9️⃣ Result Set이 성능에 미치는 영향

대규모 조회 시 Result Set은 다음 문제를 일으킬 수 있다:

---

### ✔ 1) 결과가 너무 크면 Out Of Memory

ResultSet 전체 fetch → JVM Heap 과다 사용
→ Streaming 모드 또는 fetchSize 필요

---

### ✔ 2) 네트워크 비용 증가

DB서버 → App 서버 간 패킷 수 증가, 지연 증가

---

### ✔ 3) 대량 row는 반드시 Streaming 조회 필요

MySQL JDBC:

```java
stmt.setFetchSize(Integer.MIN_VALUE);
```

PostgreSQL JDBC:

```java
stmt.setFetchSize(50);
```

---

# 🔟 DB 이론 관점 Result Set = “릴레이션 인스턴스”

관계형 데이터베이스 이론(릴레이션 모델)에서는:

> SELECT 결과는 **릴레이션(Relation)의 인스턴스(instance)** 이다.

즉:

* 테이블처럼 생겼고
* 스키마가 있으며
* 튜플(tuple, row)들의 집합

그 자체가 하나의 “릴레이션”이다.

---

# 1️⃣1️⃣ Result Set과 테이블의 차이

| 항목     | 테이블              | Result Set          |
| ------ | ---------------- | ------------------- |
| 저장 여부  | DB에 저장           | 메모리/버퍼에 일시적 저장      |
| 스키마    | 고정               | SELECT문 구조에 따라 생성   |
| 수정 가능성 | INSERT/UPDATE 가능 | 불가능                 |
| 지속성    | 영구               | 일시적(쿼리 생명주기 동안만 존재) |

---

# 1️⃣2️⃣ SELECT OUTPUT = “Result Set” 요약

* SELECT 실행 결과물
* row + column 형태의 가상 테이블
* DB서버가 생성한 일시적 Snapshot
* JDBC ResultSet 형태로 스트리밍 처리
* 커서를 사용해 한 row씩 읽기
* 메모리/성능 고려 매우 중요



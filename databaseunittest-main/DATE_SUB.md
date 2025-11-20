

# 🚀 1. NOW() 함수란?

### ✔ 역할: **현재 날짜 + 현재 시간(타임스탬프)을 반환하는 함수**

MySQL 기준:

```sql
SELECT NOW();
```

예시 결과:

```
2025-01-19 20:32:10
```

✔ 서버의 현재 시간(YYYY-MM-DD hh:mm:ss)
✔ DATETIME 타입 반환
✔ `CURRENT_TIMESTAMP()` 와 동일한 의미

자주 쓰이는 곳:

* 로그 저장 시 (created_at = NOW())
* 업데이트 시 (updated_at = NOW())
* 날짜 계산

---

# 🚀 2. DATE_SUB() 함수란?

### ✔ 역할: **특정 날짜에서 “기간(interval)”을 빼는 함수**

형식:

```
DATE_SUB(기준_날짜, INTERVAL 값 단위)
```

예제:

```sql
SELECT DATE_SUB('2025-01-19', INTERVAL 10 DAY);
```

결과:

```
2025-01-09
```

즉,

* DAY
* MONTH
* YEAR
* HOUR
* MINUTE
* SECOND
* WEEK

등 거의 모든 단위로 계산 가능.

추가 예:

```sql
SELECT DATE_SUB('2025-01-19 20:00:00', INTERVAL 2 HOUR);
-- → 2025-01-19 18:00:00
```

---

# 🚀 3. DATE_SUB + NOW 결합

### ✔ `DATE_SUB(NOW(), INTERVAL 7 DAY)`

= “현재 시간에서 7일을 뺀 날짜”

코드:

```sql
SELECT DATE_SUB(NOW(), INTERVAL 7 DAY);
```

예상 결과:

```
2025-01-12 20:32:10
```

즉,

* 오늘이 2025-01-19 20:32
* 7일 빼면 2025-01-12 20:32

---

# 🚀 4. 왜 DATE_SUB(NOW(), INTERVAL 7 DAY)를 쓰나?

### ✔ 최근 7일 데이터를 조회할 때 사용됨

예:

```sql
SELECT *
FROM Orders
WHERE order_date >= DATE_SUB(NOW(), INTERVAL 7 DAY);
```

의미:

> “오늘 기준 최근 7일동안 주문된 데이터만 조회해라"

정말 많이 쓰는 패턴:

* 최근 1일
* 최근 7일
* 최근 30일
* 최근 3개월
* 최근 1년

---

# 🚀 5. 비슷한 시간 함수 비교

| 함수        | 의미                  |
| --------- | ------------------- |
| **NOW()** | 현재 날짜+시간 (DATETIME) |
| CURDATE() | 오늘 날짜만 (DATE)       |
| CURTIME() | 현재 시간만 (TIME)       |
| SYSDATE() | 실행 시점의 실제 시스템 시간    |

---

# 🚀 6. DATE_SUB의 반대 함수

### DATE_ADD

“날짜에 더하기”

예:

```sql
SELECT DATE_ADD(NOW(), INTERVAL 3 DAY);
```

---

# 🔥 최종 정리

### ✔ NOW()

: “지금”을 DATETIME으로 반환
예) 2025-01-19 20:32:00

### ✔ DATE_SUB()

: 특정 날짜에서 기간을 빼는 함수
예) DATE_SUB('2025-01-19', INTERVAL 7 DAY) → 1월 12일

### ✔ DATE_SUB(NOW(), INTERVAL 7 DAY)

: “지금으로부터 7일 전”
→ 최근 7일 데이터를 조회할 때 가장 많이 사용하는 패턴



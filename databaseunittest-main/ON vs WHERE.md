# 🚀 ON vs WHERE — JOIN에서 완전히 다른 역할을 한다

많은 초보자들이 “ON이나 WHERE나 어차피 조건만 넣으면 되지 않나요?” 라고 묻지만
**JOIN에서 ON과 WHERE는 절대 같은 의미가 아닙니다.**

JOIN 구조에서는 두 절의 실행 시점과 역할이 아예 다릅니다.

---

# 🔥 1. 핵심 요약 (가장 중요한 차이)

| 항목                   | ON                      | WHERE               |
| -------------------- | ----------------------- | ------------------- |
| **사용 목적**            | 조인 조건                   | 최종 필터 조건            |
| **실행 시점**            | JOIN 수행할 때(중간 결과 생성 단계) | JOIN이 끝난 후 최종 결과 단계 |
| **OUTER JOIN에서의 역할** | NULL을 유지할지 제거할지 결정      | NULL을 필터링해버림        |
| **잘못 사용 시**          | 데이터 남음                  | 데이터 사라짐             |

OUTER JOIN에서 특히 큰 차이가 발생합니다.

---

# 💡 2. ON 절 = “조인 조건을 적용하여 두 테이블을 결합하는 단계”

JOIN의 논리적 순서는 다음과 같습니다:

```
1) FROM
2) JOIN … ON
3) WHERE
4) SELECT
```

즉, ON 절은 **JOIN을 수행하는 과정에서 어떤 행과 어떤 행을 결합할지를 결정**합니다.

예:

```sql
FROM A
LEFT JOIN B ON A.id = B.a_id
```

→ ON 실행 후 A의 모든 행은 유지되며 B가 매칭되면 붙고, 매칭되지 않으면 NULL로 채워짐.

---

# 💡 3. WHERE 절 = “JOIN 결과가 만들어진 뒤 결과를 필터링”

JOIN이 끝나고 전체 Result Set이 만들어진 다음
WHERE 절이 적용되어 **필터링**이 이루어집니다.

예:

```sql
WHERE B.value > 10
```

→ JOIN 결과에서 B.value가 10보다 큰 행만 남김.

---

# 🔥 4. INNER JOIN에서는 ON과 WHERE가 거의 동일하게 동작

다음 두 쿼리는 INNER JOIN일 때는 같은 결과를 만듭니다.

### 방식 1 (ON 사용)

```sql
SELECT *
FROM A
JOIN B ON A.id = B.a_id AND B.value > 10;
```

### 방식 2 (WHERE 사용)

```sql
SELECT *
FROM A
JOIN B ON A.id = B.a_id
WHERE B.value > 10;
```

INNER JOIN은
**매칭되지 않는 행을 아예 제거하기 때문**에
ON이든 WHERE든 필터링 조건이 비슷하게 적용됩니다.

하지만 이것은 **INNER JOIN에서만** 해당됩니다.

---

# ❌ 5. OUTER JOIN에서는 ON과 WHERE를 잘못 쓰면 완전히 다른 결과

## ✔ 예시 테이블

A:

| id | name |
| -- | ---- |
| 1  | A    |
| 2  | B    |

B:

| a_id | value |
| ---- | ----- |
| 1    | 10    |
| 1    | NULL  |

---

# ✔ ① ON에서 필터링하는 경우 (LEFT JOIN 유지됨)

```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.a_id AND B.value = 10;
```

결과:

| A.id | A.name | B.value |                     |
| ---- | ------ | ------- | ------------------- |
| 1    | A      | 10      |                     |
| 2    | B      | NULL    |                     |
| 1    | A      | NULL    | ← ON에서 매칭 실패 → NULL |

````

A의 모든 행은 **무조건 유지**됨  
(LEFT JOIN의 원래 의미 지켜짐)

---

# ✔ ② WHERE에서 필터링하는 경우 (OUTER JOIN 의미 사라짐)

```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.a_id
WHERE B.value = 10;
````

결과:

| A.id | A.name | B.value |
| ---- | ------ | ------- |
| 1    | A      | 10      |

→ B.value가 NULL인 행은 WHERE에서 전부 제거됨
→ 사실상 **INNER JOIN처럼 동작**해버림

---

# 🧠 6. 왜 이런 차이가 발생하나?

JOIN 처리 과정은 이렇게 이뤄집니다:

```
1) ON → 테이블 결합
2) OUTER JOIN일 경우 NULL 결정
3) WHERE → 결과에서 행 제거
```

따라서 **OUTER JOIN에서 WHERE는 조인 결과를 망가뜨릴 수 있습니다.**

---

# 💡 7. 정리 — 언제 ON, 언제 WHERE를 사용할까?

### ✔ ON 을 사용하는 경우

* JOIN 조건
* OUTER JOIN에서 매칭 조건
* 조인할 때 필터링해야 성능이 좋은 조건

### ✔ WHERE 를 사용하는 경우

* 최종 결과를 필터링할 조건
* JOIN 조건과 무관한 필터링
  (예: 날짜 범위, 점수 조건)

---

# 🏁 최종 요약 (가장 중요한 문장)

> ✔ ON = JOIN을 어떻게 결합할지 결정하는 조건
> ✔ WHERE = JOIN 이후 최종 결과를 필터링하는 조건
>
> ✔ INNER JOIN에서는 비슷해 보이지만
> ✔ OUTER JOIN에서는 완전히 다른 결과를 만들어낸다.


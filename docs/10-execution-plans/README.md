# 10. 실행계획

---

## 10.1 히스토그램 (HISTOGRAM)

-   컬럼의 데이터 분포도를 참조할 수 있는 히스토그램 정보를 활용
-   MySQL 서버가 실행될 때 딕셔너리의 히스토그램 정보를 `information_schema.column_statistics` 테이블로 로드
-   자동 분석하지 않는다

> **💡 핵심 포인트**
>
> **히스토그램 정보가 있으면 옵티마이저가 어떤 테이블을 먼저 읽을지, 어떤 키를 사용할지 등 더 정확히 판단이 가능**
>
> **히스토그램 정보가 없으면 옵티마이저는 데이터가 균등하게 분포되어 있을 것으로 가정**

### 히스토그램 수집 및 조회

```sql
-- 히스토그램 수집
ANALYZE TABLE employees
UPDATE HISTOGRAM on gender, hire_date;

-- 히스토그램 조회
SELECT *
FROM information_schema.column_statistics
WHERE schema_name = 'employees'
  AND table_name = 'employees';
```

---

## 10.2 쿼리의 실행 시간 확인 (EXPLAIN ANALYZE)

-   `EXPLAIN ANALYZE`로 실행 단계 및 소요 정보의 시간이 확인 가능하다
-   `actual time`: 소요시간
-   `rows`: 레코드 건수
-   `loops`: 반복 횟수

> **⚠️ 실행 순서 주의사항**
>
> -   들여쓰기가 같은 레벨은 **상단에 위치한 라인이 먼저 실행**
> -   들여쓰기가 다른 경우는 **가장 안쪽의 라인이 먼저 실행**

#### EXPLAIN ANALYZE 예제

```sql
EXPLAIN ANALYZE
SELECT
    e.hire_date,
    AVG(s.salary)
FROM employees AS e
    INNER JOIN salaries s
        ON s.emp_no = e.emp_no
        AND s.salary > 50000
        AND s.from_date <= '1990-01-01'
        AND s.to_date > '1990-01-01'
WHERE e.first_name = 'Matt'
GROUP BY e.hire_date;
```

#### 실행 결과 분석

```sh
-> Table scan on <temporary>  (actual time=6.15..6.16 rows=48 loops=1)
    -> Aggregate using temporary table  (actual time=6.14..6.14 rows=48 loops=1)
        -> Nested loop inner join  (cost=472 rows=124) (actual time=0.247..3.32 rows=48 loops=1)
            -> Index lookup on e using ix_firstname (first_name='Matt')  (cost=121 rows=233) (actual time=0.212..0.972 rows=233 loops=1)
            -> Filter: ((s.salary > 50000) and (s.from_date <= DATE'1990-01-01') and (s.to_date > DATE'1990-01-01'))  (cost=0.549 rows=0.532) (actual time=0.00855..0.00976 rows=0.206 loops=233)
                -> Index lookup on s using PRIMARY (emp_no=e.emp_no)  (cost=0.549 rows=9.58) (actual time=0.00573..0.008 rows=9.53 loops=233)
```

#### 실행 순서 상세 분석

1. **Index lookup on e using ix_firstname**

    - `ix_firstname` 인덱스를 통해 `first_name='Matt'` 조건에 맞는 레코드 검색

2. **Index lookup on s using PRIMARY**

    - PRIMARY 키를 통해 `s.emp_no`가 `e.emp_no`와 같은 레코드 검색

3. **Filter: ((s.salary > 50000) and (s.from_date <= DATE'1990-01-01') and (s.to_date > DATE'1990-01-01'))**

    - 조건에 맞는 레코드만 불러와서

4. **Nested loop inner join**

    - 테이블 조인

5. **Aggregate using temporary table**

    - GROUP BY 집계하면서 임시 테이블에 저장

6. **Table scan on temporary table**
    - 임시테이블을 불러와 결과를 반환

---

## 10.3 실행 계획 분석 (EXPLAIN)

### 10.3.1 id

-   SELECT 쿼리별로 부여되는 식별자 값
-   **id가 같으면 조인으로 실행**

### 10.3.2 select_type

-   어떤 타입의 쿼리인지 표시

| select_type          | 설명                                                                 |
| -------------------- | -------------------------------------------------------------------- |
| **SIMPLE**           | 서브쿼리나 UNION이 없는 단순 SELECT                                  |
| **PRIMARY**          | 서브쿼리를 포함한 쿼리에서 최상위 SELECT                             |
| UNION                | UNION 안에서 두 번째 이후 SELECT                                     |
| DEPENDENT UNION      | 상위 쿼리에 의존하는 UNION                                           |
| UNION RESULT         | UNION 연산의 최종 결과 집합                                          |
| **SUBQUERY**         | FROM절 이외의 SUBQUERY (외부 쿼리와 독립적으로 실행 가능한 서브쿼리) |
| DEPENDENT SUBQUERY   | 외부 쿼리 값에 의존하는 서브쿼리                                     |
| **DERIVED**          | FROM 절 안의 파생 테이블 (임시 테이블 생성)                          |
| DEPENDENT DERIVED    | 외부 쿼리 값에 의존하는 파생 테이블                                  |
| **MATERIALIZED**     | 서브쿼리 결과를 캐싱 후 재사용                                       |
| UNCACHEABLE SUBQUERY | 캐시 불가능한 서브쿼리 (비결정적 함수, 변수 등)                      |
| UNCACHEABLE UNION    | 캐시 불가능한 UNION                                                  |

### 10.3.3 table

-   테이블의 별칭
-   테이블 컬럼의 `<DERIVED `id`>`으로 표시되는 경우는 id의 임시테이블을 의미한다

#### 📊 실행 계획 분석 예제

| id  | select_type | table      |
| --- | ----------- | ---------- |
| 1   | PRIMARY     | <derived2> |
| 1   | PRIMARY     | e          |
| 2   | DERIVED     | de         |

**분석 과정:**

1. 첫 번째 라인의 테이블이 `<derived2>`라는 것으로 보아 이 라인보다 id 값이 2인 라인이 먼저 실행되고 그 결과가 파생 테이블로 준비돼야 한다는 것을 알 수 있다.

2. 세 번째 라인(id 값이 2인 라인)을 보면 `select_type` 칼럼의 값이 `DERIVED`로 표시돼 있다. 즉, 이 라인은 `table` 칼럼에 표시된 `dept_emp` 테이블을 읽어서 파생 테이블을 생성하는 것을 알 수 있다.

3. 세 번째 라인의 분석이 끝났으므로 다시 실행 계획의 첫 번째 라인으로 돌아가자.

4. 첫 번째 라인과 두 번째 라인은 같은 id 값을 가지고 있는 것으로 봐서 2개 테이블(첫 번째 라인의 `<derived2>`와 두 번째 라인의 `e` 테이블)이 조인되는 쿼리라는 사실을 알 수 있다. 그런데 `<derived2>` 테이블이 `e` 테이블보다 먼저(윗 라인에 표시됐기 때문에) `<derived2>`가 **드라이빙 테이블**이 되고, `e` 테이블이 **드리븐 테이블**이 된다는 것을 알 수 있다. 즉, `<derived2>` 테이블을 먼저 읽어서 `e` 테이블로 조인을 실행했다는 것을 알 수 있다.

```sql
EXPLAIN
SELECT *
FROM (
    SELECT de.emp_no
    FROM dept_emp AS de
    GROUP BY de.emp_no
) AS tb
INNER JOIN employees AS e
WHERE e.emp_no = tb.emp_no;
```

### 10.3.4 partitions

-   해당 쿼리가 어떤 파티션(partition)에서 데이터를 읽는지
-   **파티션 이름이 표시되면, 해당 파티션만 읽는다는 뜻 (파티션 프루닝 적용)**

### 10.3.5 type

-   각 테이블의 레코드를 어떤 방식으로 읽었는지

| type            | 설명                                                                 | 효율 |
| --------------- | -------------------------------------------------------------------- | ---- |
| system          | 한 행만 있는 테이블 (또는 상수 테이블), 매우 빠름                    | 1    |
| **const**       | **기본 키나 UNIQUE 인덱스로 단일 행을 즉시 조회**                    | 1    |
| **eq_ref**      | **JOIN 시, 기본 키나 UNIQUE 인덱스를 통해 한 행만 매칭**             | 1    |
| **ref**         | **인덱스에서 특정 값(= 비교)으로 여러 행 조회**                      | 2    |
| fulltext        | FULLTEXT 인덱스를 사용한 검색 (전문 검색 인덱스)                     | 2    |
| ref_or_null     | `ref` 방식 + NULL 값 검색까지 포함                                   | 2    |
| unique_subquery | 서브쿼리에서 고유 인덱스를 사용해 단일 값 반환 (`= (SELECT pk ...)`) | 3    |
| index_subquery  | 서브쿼리에서 인덱스를 사용해 여러 값 반환                            | 3    |
| **range**       | **인덱스 범위 검색 (`BETWEEN`, `<`, `>`, `IN` 등)**                  | 3    |
| index_merge     | 둘 이상의 인덱스를 병합해 사용                                       | 3    |
| index           | 인덱스 풀 스캔 (테이블 전체를 인덱스 순서로 읽음)                    | 3    |
| **ALL**         | **풀 테이블 스캔 (가장 느린 방식)**                                  | 3    |

### 10.3.6 possible_keys

-   **키의 후보**

### 10.3.7 key

-   **최종 실행된 인덱스**

### 10.3.8 key_len

-   다중 컬럼 인덱스에서 몇 개의 컬럼까지 사용했는지
-   인덱스의 각 레코드에서 몇 바이트를 사용했는지
-   바이트 수는 데이터 타입 + NULL 가능 여부에 따라 계산됨 (NULL은 +1)

> **📝 key_len 예시**
>
> ```
> key: ix_hiredate, ix_firstname
> key_len: 3, 58
> ```
>
> -   첫 번째 인덱스(`ix_hiredate`) → **3바이트 사용**
> -   두 번째 인덱스(`ix_firstname`) → **58바이트 사용**

### 10.3.9 ref

-   참조조건이 어떤 값으로 제공되었는지 표현
-   **function인 경우만 주의**

### 10.3.10 rows

-   처리 방식에 얼마나 많은 레코드를 읽고 비교해야 하는지의 값

### 10.3.11 filtered

-   해당 단계에서 조건에 의해 걸러져서 다음 단계로 넘어가는 행(row)의 비율
-   **filtered 값이 작을수록 좋음**
-   filtered 값이 더 정확하게 예측할 수 있도록 히스토그램이 도입되었다
-   **`filtered(10) × rows(1000) = 실제 처리되는 예상 행 수 (100)`**

### 10.3.12 extra

| Extra 값              | 설명                                                                    |
| --------------------- | ----------------------------------------------------------------------- |
| **Using index**       | **커버링 인덱스를 사용해 테이블 접근 없이 인덱스만 읽음 (성능 좋음)**   |
| **Using where**       | **WHERE 조건을 사용해 필터링함**                                        |
| **Using filesort**    | **MySQL이 결과 정렬을 위해 별도의 정렬 작업 수행 (성능 저하 가능)**     |
| **Using temporary**   | **임시 테이블을 만들어 처리함 (복잡한 GROUP BY, DISTINCT 등에서 발생)** |
| Using join buffer     | 조인 시 조인 버퍼를 사용함                                              |
| Using index condition | 인덱스 조건 푸시다운 (Index Condition Pushdown) 사용                    |
| Using MRR             | Multi-Range Read 최적화 사용                                            |
| Impossible WHERE      | WHERE 조건이 절대 참이 될 수 없어 결과가 없음                           |
| Loose Scan            | 느슨한 인덱스 스캔 (특정 그룹핑 최적화)                                 |

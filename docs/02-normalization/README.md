# 정규화 (Normalization)

---

-   정규화는 데이터 중복을 제거하고 이상(Anomaly)을 방지하여 데이터 무결성을 높이고 유지보수 비용을 줄이는 설계 기법이다.
-   목표는 다음과 같다.
    -   업데이트 이상(Update Anomaly) 방지: 동일 정보가 여러 곳에 중복되어 생기는 불일치 제거
    -   삽입 이상(Insert Anomaly) 방지: 일부 정보만으로는 레코드를 추가할 수 없는 문제 제거
    -   삭제 이상(Delete Anomaly) 방지: 불필요한 부수 데이터 삭제로 인해 필요한 정보까지 유실되는 문제 제거

> -   정규화는 "함수적 종속(Functional Dependency)"을 식별하고, 그에 맞추어 테이블을 분해하는 작업이다.
> -   분해는 정보 손실 없이(무손실 분해) 되며, 가능한 한 의존성을 보존한다.
> -   실무에서는 3NF 또는 BCNF를 목표로 하되, 쿼리/성능 요구에 맞춰 부분 비정규화를 선택적으로 적용한다.

#### 용어 정리

-   함수적 종속(Functional Dependency, FD): X → Y는 속성 집합 X가 주어지면 Y가 유일하게 결정됨을 뜻한다.
-   후보키(Candidate Key): 레코드를 유일하게 식별하는 최소 속성 집합. 여러 개 존재할 수 있다.
-   기본키(Primary Key): 후보키 중 선택된 대표 키.

---

## 시나리오

#### Table

`account`
| bank_name | account_num | account_id | class | ratio | empl_id | empl_name | card_id |
| --------- | ----------- | ---------- | ------ | ----- | ------- | --------- | ---------- |
| woori | 110-23-01 | 10001 | bronze | 0.1 | e1 | park | c101 |
| woori | 110-34-33 | 10002 | silver | 0.2 | e1 | park | c102 |
| kookmin | 1989-33-12 | 10003 | loyal | 0.7 | e1 | park | c103 |
| kookmin | 1989-32-23 | 10004 | loyal | 0.1 | e2 | kim | c201, c202 |

#### Candidate Key

-   `{account_id}`
-   `{bank_name, account_num}`

---

### 1NF

-   속성 하나는 하나의 속성값만을 가져야 한다 (**도메인 원자값**)

`account`
| bank_name | account_num | account_id | class | ratio | empl_id | empl_name | card_id |
| --------- | ----------- | ---------- | ------ | ----- | ------- | --------- | ------- |
| woori | 110-23-01 | 10001 | bronze | 0.1 | e1 | park | c101 |
| woori | 110-34-33 | 10002 | silver | 0.2 | e1 | park | c102 |
| kookmin | 1989-33-12 | 10003 | loyal | 0.7 | e1 | park | c103 |
| kookmin | 1989-32-23 | 10004 | loyal | 0.1 | e2 | kim | c201 |
| kookmin | 1989-32-23 | 10004 | loyal | 0.1 | e2 | kim | c202 |

> ⚠️ **부분 종속 위배**

### 2NF

-   기본키 일부에만 종속된 속성을 제거한다 (**부분 종속**)

> **💡 핵심 포인트**
>
> **2NF는 key가 compositive key(복합키)가 아니라면 2NF는 자동으로 만족한다**

`account`
| account_id(pk) | bank_name | account_num | class | ratio | empl_id | empl_name |
| ----------- | ---------- | ------------ | ------ | ----- | -------- | ---------- |
| 10001 | woori | 110-23-01 | bronze | 0.1 | e1 | park |
| 10002 | woori | 110-34-33 | silver | 0.2 | e1 | park |
| 10003 | kookmin | 1989-33-12 | loyal | 0.7 | e1 | park |
| 10004 | kookmin | 1989-32-23 | loyal | 0.1 | e2 | kim |

`account_card`
| account_id | card_id(pk) |
| ----------- | -------- |
| 10001 | c101 |
| 10002 | c102 |
| 10003 | c103 |
| 10004 | c201 |
| 10004 | c202 |

> ⚠️ **이행 종속 위배**
>
> if x→y & y→z holds, then x→z is transitive FD
> unless either y or z is not subset of any key

> { account_id → empl_id }, {empl_id → empl_name}
> **account_id → empl_id → empl_name**

### 3NF

-   기본키가 아닌 속성 간의 이행 종속을 제거한다 (**이행 종속**)

`account`
| account_id(pk) | bank_name | account_num | class | ratio | empl_id |
| ----------- | ---------- | ------------ | ------ | ----- | -------- |
| 10001 | woori | 110-23-01 | bronze | 0.1 | e1 |
| 10002 | woori | 110-34-33 | silver | 0.2 | e1 |
| 10003 | kookmin | 1989-33-12 | loyal | 0.7 | e1 |
| 10004 | kookmin | 1989-32-23 | loyal | 0.1 | e2 |

`employee`
| empl_id(pk) | empl_name |
| -------- | ---------- |
| e1 | park |
| e2 | kim |

`account_card`
| account_id | card_id(pk) |
| ----------- | -------- |
| 10001 | c101 |
| 10002 | c102 |
| 10003 | c103 |
| 10004 | c201 |
| 10004 | c202 |

> ⚠️ **결정자 위배**
>
> 모든 유효한 non-trivial FD x→y는 x가 super key 여야 한다

> {class → bank_name}
> **class는 super key 여야 한다**

### BCNF

-   모든 결정자가 후보키여야 한다 (**결정자**)

`account`
| account_id(pk) | account_num | class | ratio | empl_id |
| ----------- | ------------ | ------ | ----- | -------- |
| 10001 | 110-23-01 | bronze | 0.1 | e1 |
| 10002 | 110-34-33 | silver | 0.2 | e1 |
| 10003 | 1989-33-12 | loyal | 0.7 | e1 |
| 10004 | 1989-32-23 | loyal | 0.1 | e2 |

`account_class`
|class(pk)|bank_name|
|--|--|
|bronze|woori|
|silver|woori|
|prestige|kookmin|
|loyal|kookmin|

`employee`
| empl_id(pk) | empl_name |
| -------- | ---------- |
| e1 | park |
| e2 | kim |

`account_card`
| account_id | card_id(pk) |
| ----------- | -------- |
| 10001 | c101 |
| 10002 | c102 |
| 10003 | c103 |
| 10004 | c201 |
| 10004 | c202 |

### 4NF

-   다치 종속을 제거한다 (**다치 종속**)

### 5NF

-   조인 시 데이터 손실이 없게 한다 (**조인 종속**)

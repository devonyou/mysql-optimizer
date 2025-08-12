# 📚 Real MySQL

-   [도서](https://product.kyobobook.co.kr/detail/S000060313997)
-   [GitHub](https://github.com/wikibook/realmysql80)

## feature

| 제목                 | 링크                                                 |
| -------------------- | ---------------------------------------------------- |
| 03 사용자 및 권한    | [바로가기](./docs/03-user-and-permission/README.md)  |
| 04 아키텍처          | [바로가기](./docs/04-architecture/README.md)         |
| 05 트랜잭션과 잠금   | [바로가기](./docs/05-transaction-and-lock/README.md) |
| 06 데이터 압축       | [바로가기](./docs/06-data-compression/README.md)     |
| 07 데이터 암호화     | [바로가기](./docs/07-data-encryption/README.md)      |
| 08 인덱스            | [바로가기](./docs/08-indexes/README.md)              |
| 09 옵티마이저와 힌트 | [바로가기](./docs/09-optimizer-and-hints/README.md)  |
| 10 실행 계획         | [바로가기](./docs/10-execution-plans/README.md)      |

## Getting Started

```sh
# docker run
docker compose up --build -d

docker exec -it mysql-optimizer-mysql-1 sh
```

## dump

```sh
mysql> source ./sqls/dump.sql
```

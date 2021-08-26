PostgreSQL partition table - 2020/12
===

###### tags: `tech workshop`


# PostgresOpen 2019 PostgreSQL Partitioning

- [PostgresOpen 2019 PostgreSQL Partitioning](https://www.youtube.com/watch?v=JWQVDKw1HVk&ab_channel=PostgresOpen)

## Why partition?

![](https://i.imgur.com/jI6qXAl.png)

![](https://i.imgur.com/pFHTezX.png)

## Partition Use Case

![](https://i.imgur.com/LWssmdS.png)

![](https://i.imgur.com/6Aq5ujH.png)


## No Partition Standard

![](https://i.imgur.com/tsdZen3.png)


## Time-based Table Example

![](https://i.imgur.com/zWQ4c8i.png)

## Time-Series Partitioning 

![](https://i.imgur.com/kctyXM4.png)

![](https://i.imgur.com/aIwSRa0.png)

## Partition Types

![](https://i.imgur.com/010bDZQ.png)


## Partition in Pictures

![](https://i.imgur.com/XXZJw6j.png)

![](https://i.imgur.com/nMbZk33.png)

## Partitioning Syntax

![](https://i.imgur.com/MtT0WEo.png)

## Partitioning Locking

![](https://i.imgur.com/T9eWXfW.png)

## Partitioning Performance in PG11 v.s. PG12

![](https://i.imgur.com/sodvsbm.png)

# Table Partitioning in Postgres: How Far We've Come 2019

- https://www.postgresql.eu/events/pgconfeu2019/sessions/session/2685/slides/225/pgconf-eu-2019.pdf


# Table Partitioning in Postgres: How Far We've Come 2020

- [Table Partitioning in Postgres: How Far We've Come](https://www.youtube.com/watch?v=YxbNN2mxAkU&ab_channel=EDB)

## Old partitioning

![](https://i.imgur.com/r6Donrb.png)

## New partitioning (Declatative partitioning)

![](https://i.imgur.com/spdiWem.png)

## Attach/Detach partition

![](https://i.imgur.com/dNNqO9l.png)

## indexes

![](https://i.imgur.com/uM1CSNA.png)

## optimizations

![](https://i.imgur.com/sNfZILX.png)

![](https://i.imgur.com/uB1cKHA.png)

## Partitioning best practices and pitfalls

![](https://i.imgur.com/FUjb5dT.png)


# 其他

- partition difference 10 v.s. 11
    - https://severalnines.com/database-blog/how-take-advantage-new-partitioning-features-postgresql-11

## index

- 在 partition table create index 會沒辦法使用 concurrently 這點要注意，那如果不想 lock table 的話該怎麼做？
    - 參考 https://www.postgresql.org/docs/12/ddl-partitioning.html

## 問題

- partition table 在 pg 11, 12 支援比較好，可能要先做升級
- 把 pg 繼續放在 k8s 上，還是拉出來另外開一台機器放 pg？
- 決定 partition table 要用哪個欄位的什麼值去切開 table，因為決定後未來要再更動應該會很麻煩
- customers table 如果鎖住不能更新會影響哪些地方？


# pg 其他

## pgbench 
https://gitpress.io/@chchang/postgresql-pgbench-benchmark
https://docs.postgresql.tw/reference/client-applications/pgbench

## 其他
https://severalnines.com/blog/tuning-io-operations-postgresql
https://www.bookstack.cn/read/pgsql-12-tw/the-sql-language-performance-tips-14.5.-dan-xing-she-ding.md
https://thoughtbot.com/blog/advanced-postgres-performance-tips

[Declarative Caching with Postgres and Redis](https://www.youtube.com/watch?v=IID2LQVztIM&ab_channel=SouthernCaliforniaLinuxExpo)


# 指令、流程

```sql=
create table customers_hash 
	(like customers including all excluding indexes) 
	partition by hash(project_id);

insert into customers_hash select * from customers;

select count(*) from customers_hash;

create table customers_hash_0 partition of customers_hash for values with (modulus 5, remainder 0);

create table customers_hash_1 partition of customers_hash for values with (modulus 5, remainder 1);

create table customers_hash_2 partition of customers_hash for values with (modulus 5, remainder 2);

create table customers_hash_3 partition of customers_hash for values with (modulus 5, remainder 3);

create table customers_hash_4 partition of customers_hash for values with (modulus 5, remainder 4);

create table customers_hash_default partition of customers_hash default;

;;;;;;;;;;;;;;;;;;;

create unique index test0 on customers_hash_0 (id);
create unique index test1 on customers_hash_1 (id);
create unique index test2 on customers_hash_2 (id);
create unique index test3 on customers_hash_3 (id);
create unique index test4 on customers_hash_4 (id);
```
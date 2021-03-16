PostgreSQL tuning - 2021/3
===

###### tags: `tech workshop`

# autovacuum

- [autovacuum 會用到的參數](https://docs.postgresql.tw/server-administration/server-configuration/automatic-vacuuming)

- 可以透過類似 `ALTER TABLE <table_name> SET (autovacuum_vacuum_cost_limit = <large_value>)`，針對某些 table 個別給定需要的參數值
    - 可以用 `\d+ <table_name>` 下面的 options 觀察目前 table 是否有 custom options
    - 或是 `SELECT relname, reloptions FROM pg_class;
`


- To check if the autovacuum daemon is running always:
    - `$ps -axww | grep autovacuum`
    - `SELECT name, setting FROM pg_settings WHERE name = 'autovacuum';`

- 如何觀察 last_autovacuum/last_autoanalyze 的時間？
    - `SELECT * FROM pg_stat_user_tables;`


## reference


- [PostgreSQL VACUUM and ANALYZE Best Practice Tips](https://www.2ndquadrant.com/en/blog/postgresql-vacuum-and-analyze-best-practice-tips/)
- [Tuning Autovacuum in PostgreSQL and Autovacuum Internals](https://www.percona.com/blog/2018/08/10/tuning-autovacuum-in-postgresql-and-autovacuum-internals/)
- [Monitoring PostgreSQL VACUUM processes](https://www.datadoghq.com/blog/postgresql-vacuum-monitoring/)
- [Tuning PostgreSQL Autovacuum to Prevent Table Bloat](https://blog.newrelic.com/product-news/tuning-postgresql-autovacuum/)

# fillfactor

## ref

- https://zhangeamon.top/postgres/fillfactor/
- https://dzone.com/articles/what-is-the-best-value-for-the-fill-factor
- https://www.dbrnd.com/2016/03/postgresql-the-awesome-table-fillfactor-to-speedup-update-and-select-statement/
- https://www.luoow.com/dc_tw/100860106
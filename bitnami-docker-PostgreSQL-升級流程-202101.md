bitnami docker PostgreSQL 升級流程 - 2021/1
===

###### tags: `tech workshop`


1. 把 bitnami image 改成 12.5.0，並讓 bitnami 12.5 產生新版的 datadir
2. 把 container command 改成 bash, tty
    - 因為升級的時候 server 必須是關掉的狀態
3. 以 root 身份進去 pod
    - 為了安裝 postgresql-10 和 12
    - `apt update && apt install -y gnupg wget && sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt buster-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -`
    - `apt update && apt install -y postgresql-10 postgresql-12`
4. 拿掉 postgresql 12 repl_xxx 的 ROLE
    - 因為 pg_upgrade 只允許新的 cluster 有 postgres 這個 role，不然會噴出 `Only the install user can be defined in the new cluster.`
    - https://stackoverflow.com/questions/41854387/database-user-postgres-is-not-the-install-user
    - ```=# SELECT * FROM pg_catalog.pg_roles WHERE rolname !~ '^pg_';```
    - `=# drop role xxx;`

* bitnami postgres command: 
    - `/opt/bitnami/postgresql/bin/postgres -D /bitnami/postgresql/data --config-file=/opt/bitnami/postgresql/conf/postgresql.conf --external_pid_file=/opt/bitnami/postgresql/tmp/postgresql.pid --hba_file=/opt/bitnami/postgresql/conf/pg_hba.conf`

5. 更改 datadir owner，換成 `postgres:postgres`
    - `chown -R postgres:postgres <datadir>`

6. 執行 pg_upgrade

```
$ /usr/lib/postgresql/12/bin/pg_upgrade \
-d /bitnami/postgresql/data-pg10 \
-b /usr/lib/postgresql/10/bin \
-o "-c config_file=/opt/bitnami/postgresql/conf/postgresql.conf --hba_file=/opt/bitnami/postgresql/conf/pg_hba.conf" \
-D /bitnami/postgresql/data \
-B /usr/lib/postgresql/12/bin \
-O "-c config_file=/opt/bitnami/postgresql/conf/postgresql.conf --hba_file=/opt/bitnami/postgresql/conf/pg_hba.conf" \
-c
```

- 有可能會遇到語系問題 https://stackoverflow.com/questions/48612313/pg-upgrade-lc-collate-values-for-database-postgres-do-not-match

7. 把 datadir owner 改回去 `1001:1001`
    - `chown -R 1001:1001 <datadir>`

8. 把 pod 的設定全部改回去，並且用 helm upgrade 升級 pg 版本
9. 升級 slave postgresql 版本

10. 最後執行 kurator-bot 的攻略
    - https://hackmd.io/UI1k6IPYS8OsYkuTDX59rQ

## 參考

- https://gitlab.com/gitlab-com/gl-infra/infrastructure/-/issues/9097
- http://reader.epubee.com/books/mobile/e1/e1a1610e13fe691f77a875db6fe51c87/text00099.html


## pg_upgrade

https://dev.to/rafaelbernard/postgresql-pgupgrade-from-10-to-12-566i

https://github.com/tianon/docker-postgres-upgrade
- https://github.com/tianon/docker-postgres-upgrade/blob/master/10-to-12/docker-upgrade

升級討論 https://github.com/docker-library/postgres/issues/37
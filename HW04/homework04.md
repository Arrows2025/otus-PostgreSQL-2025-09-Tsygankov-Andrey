# Домашнее задание 04
## Работа с базами данных, пользователями и правами

В базе данных PostgreSQL 18 проверяю наличие кластеров `pg_lsclusters`
```diff
+18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```
Cоздаю и сразу стартую новый кластер `sudo pg_createcluster 18 otus --start`
```diff
+18  otus    5433 online postgres /var/lib/postgresql/18/otus /var/log/postgresql/postgresql-18-otus.log
```
В списке кластеров базы данных PostgreSQL 18 появляется новый кластер `otus`
```diff
+18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
+18  otus    5433 online postgres /var/lib/postgresql/18/otus /var/log/postgresql/postgresql-18-otus.log
```

<img width="1689" height="1091" alt="image" src="https://github.com/user-attachments/assets/0495641c-d212-4492-a09b-d2ebe99c0425" /><br>

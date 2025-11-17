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

Для настройки удалённого доступа к новому кластеру редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/otus/postgresql.conf` для прослушивания всех адресов

`#listen_addresses = 'localhost'          # what IP address(es) to listen on;`  
меняю на  
`listen_addresses = '*'                   # what IP address(es) to listen on;`

И редактирую конфигурационный файл PostgreSQL: `sudo nano /etc/postgresql/18/otus/pg_hba.conf` для доступа к БД PostgreSQL с моей локальной машины

добавляю в конец файла строку
`host    all             all             192.168.0.129/24            scram-sha-256`

Меняю пароль для пользователя базы данных `postgres`
```
sudo -u postgres psql --cluster 18/otus
\password
\q
```

Перезагрузка кластера базы данных `sudo pg_ctlcluster 18 otus restart` и тест соединения с БД PostgreSQL через приложение DBeaver прошёл успешно

<img width="893" height="775" alt="image" src="https://github.com/user-attachments/assets/10b9d991-ed8b-4eed-976b-45d10cf8dede" /><br>

* Захожу в созданный кластер `otus` под пользователем `postgres` и создаю новую базу данных `tetsdb`

```
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE testdb;
CREATE DATABASE
```

* Подключаюсь к созданной базе данных `tetsdb` под пользователем `postgres` и создаю новую схему `testnm`
```
postgres=# \c testdb;
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# CREATE SCHEMA IF NOT EXISTS testnm;
CREATE SCHEMA
```

* Создаю новую таблицу `t1` с одной колонкой `с1` типа `integer` и вставляю в неё строку со значением `c1=1`
```
testdb=# CREATE TABLE t1(с1 int);
CREATE TABLE
testdb=# INSERT INTO t1 VALUES (1);
INSERT 0 1
```

* Создаю новую роль `readonly`, даю новой роли права:
	* на подключение к базе данных `testdb`
	* на использование схемы `testnm`
	* на выборку данных для всех таблиц схемы `testnm`
```
testdb=# CREATE role readonly;
CREATE ROLE
testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT
testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
```

<img width="1081" height="731" alt="image" src="https://github.com/user-attachments/assets/a562a137-a547-4738-a926-251f771d7055" /><br>


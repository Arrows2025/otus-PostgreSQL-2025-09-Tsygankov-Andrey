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

* Создаю пользователя `testread` с паролем `test123`, даю пользователю `testread` роль `readonly`
```
testdb=# CREATE USER testread WITH PASSWORD 'test123';
CREATE ROLE
testdb=# GRANT readonly TO testread;
GRANT ROLE
```
<img width="1081" height="821" alt="image" src="https://github.com/user-attachments/assets/7fadd6d5-e067-4760-b82c-6aeeb704034e" /><br>

* При попытке переключиться на пользователя `testread` получаю ошибку
```
testdb=# \c testdb testread
подключиться к серверу через сокет "/var/run/postgresql/.s.PGSQL.5433" не удалось: ВАЖНО:  пользователь "testread" не прошёл проверку подлинности (Peer)
Сохранено предыдущее подключение
```
<img width="2361" height="1001" alt="image" src="https://github.com/user-attachments/assets/15b9d0d1-d9fe-4064-844f-161c7c806992" /><br>

* Выхожу из терминального клиента PostgreSQL Shell и подключаюсь к базе данных `testdb` пользователем `testread` напрямую через терминальный клиент, запрашиваю данные таблицы `t1` и получаю ошибку `нет доступа к таблице t1`
```
psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> SELECT * FROM t1;
ОШИБКА:  нет доступа к таблице t1
testdb=> \dt;
           Список таблиц
 Схема  | Имя |   Тип   | Владелец
--------+-----+---------+----------
 public | t1  | таблица | postgres
(1 строка)

testdb=> SHOW search_path;
   search_path
-----------------
 "$user", public
(1 строка)
```
<img width="1593" height="701" alt="image" src="https://github.com/user-attachments/assets/4e2aaa68-9628-480f-a43f-655334a12e81" /><br>

* Так как `search_path = '"$user", public'`, а схемы `$user` нет, то таблица по умолчанию создалась в схеме `public`. Пользователь `testread` не смог получить доступ к таблице `t1`, потому что роль `readonly`, которой он обладает, не имеет прав на выборку данных из схемы `public`

* Чтобы избежать этой ошибки, таблицу `t1` необходимо было создать в схеме `testnm`. Для этого надо было либо:
	* перед созданием таблицы `t1` указать схему для текущей сессии `SET search_path = testnm;`
	* перед созданием таблицы `t1` настроить путь поиска на уровне базы данных `ALTER DATABASE testdb SET search_path = 'testnm,public';`
	* перед созданием таблицы `t1` настроить путь поиска для пользователя (роли) `ALTER ROLE testread SET search_path = 'testnm,public';` (не для нашего случая, т.к. пользователя мы создавали после создания таблицы)
	* создать таблицу с явным указанием префикса схемы базы данных `CREATE TABLE testnm.t1(с1 int);`






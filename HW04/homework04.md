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

* Чтобы избежать этой ошибки, таблицу `t1` необходимо было создать в схеме `testnm`, для этого надо было:
	* либо перед созданием таблицы `t1` указать схему для текущей сессии `SET search_path = testnm;`
	* либо перед созданием таблицы `t1` настроить путь поиска на уровне базы данных `ALTER DATABASE testdb SET search_path = 'testnm,public';`
	* либо перед созданием таблицы `t1` настроить путь поиска для пользователя (роли) `ALTER ROLE testread SET search_path = 'testnm,public';` (не для нашего случая, т.к. пользователя мы создавали после создания таблицы)
	* либо создать таблицу с явным указанием префикса схемы базы данных `CREATE TABLE testnm.t1(с1 int);`

* Возвращаюсь в базу данных `testdb` под пользователем `postgres`, удаляю таблицу `t1` и создаю её заново с явным указанием префикса схемы базы данных `testnm.t1` и добавляю одну строку со значением `c1=1`
```
testdb=> \c testdb postgres
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# DROP TABLE t1;
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 int);
CREATE TABLE
testdb=# INSERT INTO testnm.t1 VALUES (1);
INSERT 0 1
testdb=# \q
```
<img width="1593" height="1031" alt="image" src="https://github.com/user-attachments/assets/0a86a814-e192-403f-b6a9-32c39474bb33" /><br>

* Захожу под пользователем `testread` в базу данных `testdb` и делаю выборку из таблицы `testnm.t1`, опять получаю ошибку - нет доступа к таблице, потому что `GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;` дал право выборки для роли `readonly` для всех уже существующих на тот момент таблиц, для новой таблицы `testnm.t1` такого права нет. От пользователя `postgres` выдаю право роли `readonly` на выборку из всех новых таблиц схемы `testnm` - `ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;`
```
psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> SELECT * FROM testnm.t1;
ОШИБКА:  нет доступа к таблице t1
testdb=> \c testdb postgres;
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;
ALTER DEFAULT PRIVILEGES
testdb=# \q
```
<img width="1593" height="521" alt="image" src="https://github.com/user-attachments/assets/e6850d0f-09d2-41ad-b2fe-643569e2eb30" /><br>

* Снова захожу под пользователем `testread` в базу данных `testdb` и делаю выборку из таблицы `testnm.t1` и опять получаю ошибку - нет доступа к таблице, теперь причина ошибки в том, что право `ALTER DEFAULT PRIVILEGES` будет действовать для всех вновь создаваемых таблиц и не действует на уже созданную `testnm.t1`, для выборки из таблицы `testnm.t1` пользователю `testread` надо повторно дать право `GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;` от пользователя `postgres`
```
psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> SELECT * FROM testnm.t1;
ОШИБКА:  нет доступа к таблице t1
testdb=> \c testdb postgres;
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
testdb=# \q
```
<img width="1593" height="971" alt="image" src="https://github.com/user-attachments/assets/a3197667-328f-452a-93e6-b89fab6690a2" /><br>

* И ещё раз захожу захожу под пользователем `testread` в базу данных `testdb`, делаю выборку из таблицы `testnm.t1` и запрос отрабатывает без ошибки, так как пользователь `testread` наконец-то получил необходимые права для выборки из таблицы `testnm.t1` :thumbsup:
```
psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> SELECT * FROM testnm.t1;
 c1
----
  1
(1 строка)
```
<img width="1593" height="1331" alt="image" src="https://github.com/user-attachments/assets/39220f11-c30f-4a3f-963a-bc2fe1d743d2" /><br>

* Пробую выполнить команду на создание таблицы t2 и вставку в неё одной записи, получаю ошибку доступа к схеме `public`, т.к. без указания префикса схемы таблица опять создаётся в схеме public, которая указана по умолчанию в переменной `search_path`, а прав на создание таблицы в схеме public у пользователя `testread` нет.
```
testdb=> CREATE TABLE t2(c1 integer); INSERT INTO t2 VALUES (2);
ОШИБКА:  нет доступа к схеме public
СТРОКА 1: CREATE TABLE t2(c1 integer);
                       ^
ОШИБКА:  отношение "t2" не существует
СТРОКА 1: INSERT INTO t2 VALUES (2);
                      ^
```
<img width="1593" height="641" alt="image" src="https://github.com/user-attachments/assets/5e93d7a8-101a-46bc-9dc6-93c177577cf3" /><br>

* Но судя по информации из задания таблица t2 якобы должна была создаться, поэтому из интернета узнаю, что в PostgreSQL версии 14 и ниже роли `PUBLIC` предоставлена возможность создавать объекты (CREATE) в схеме `public` по умолчанию, но БД PostgreSQL версии 15 и выше не позволяют это делать. Теперь понятно, почему в первом пункте было задание: `создайте новый кластер PostgresSQL 14`, но так как в начале курса нам сказали не обращать внимания на версии операционных систем и БД PostgreSQL, в первом пункте был создан новый кластер PostgresSQL версии 18. Это недоразумение можно исправить выдачей нового гранта пользователю `testread` (роли `readonly`) на создание объектов в схеме `public` - `GRANT CREATE ON SCHEMA public TO readonly;` и создать необходимую таблицу t2 для дальнейших экспериментов
```
\c testdb postgres;
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# GRANT CREATE ON SCHEMA public TO readonly;
GRANT
testdb=# \q

psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> CREATE TABLE t2(c1 integer); INSERT INTO t2 VALUES (2);
CREATE TABLE
INSERT 0 1

```
<img width="1593" height="1121" alt="image" src="https://github.com/user-attachments/assets/f86575a2-691f-4969-853d-b8fc61ffbedc" /><br>

* Но как мы выдали права на создание объектов в схеме `public` для пользователя `testread`, таким же образом мы можем их и отозвать, чтобы избежать случайных ошибок создания таблиц не в своих схемах, а в схеме `public`, таких как таблица `t3` без явного указания префикса схемы базы данных и без редактирования переменной `search_path`, в которой схема `public` прописана по умолчанию
```
testdb=> \c testdb postgres;
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# REVOKE CREATE ON SCHEMA public FROM readonly;
REVOKE
testdb=# \q

psql -h 127.0.0.1 --cluster 18/otus -U testread -d testdb -W

testdb=> CREATE TABLE t3(c1 integer); INSERT INTO t3 VALUES (3);
ОШИБКА:  нет доступа к схеме public
СТРОКА 1: CREATE TABLE t3(c1 integer);
                       ^
ОШИБКА:  отношение "t3" не существует
СТРОКА 1: INSERT INTO t3 VALUES (3);
                      ^
testdb=> \q
```
<img width="1593" height="1380" alt="image" src="https://github.com/user-attachments/assets/2a13e937-0e3d-403b-955a-3528cae6a8a8" /><br>

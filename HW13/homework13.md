# Домашнее задание 13
## Бэкапы

Для выполнения домашнего задания подключаюсь в БД PostgreSQL, создаю базу данных `test_db`, в ней создаю схему `my_schema` и две одинаковых таблицы `table1` и `table2`, первую таблицу заполняю случайными сгенерированными данными в размере 100 строк
```
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE test_db;
CREATE DATABASE

postgres=# \c test_db;
Вы подключены к базе данных "test_db" как пользователь "postgres".

test_db=# CREATE SCHEMA my_schema;
CREATE SCHEMA

test_db=# CREATE TABLE my_schema.table1 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE

test_db=# CREATE TABLE my_schema.table2 (LIKE my_schema.table1 INCLUDING ALL);
CREATE TABLE

test_db=# INSERT INTO my_schema.table1 (Data) SELECT md5(random()::text) FROM generate_series(1, 100);
INSERT 0 100
```
<img width="1673" height="551" alt="image" src="https://github.com/user-attachments/assets/ce2faa47-e30d-45cc-af4f-7fae9eb35856" /><br>

Создаю новый каталог для бекапа под пользователем postgres `/var/lib/postgresql/backups/`, подключаюсь с базе данных `test_db` и выгружаю таблицу `table1` в файл CSV командой COPY, проверяю наличие файла на диске. Далее восстанавливаю данные в таблицу `table2` путём загрузки файла CSV с помощью команды COPY, проверяю наличие данных в таблице `table2`
```
sudo -u postgres mkdir /var/lib/postgresql/backups/
sudo -u postgres psql --cluster 18/otus -d test_db

test_db=# \copy my_schema.table1 TO '/var/lib/postgresql/backups/table1_copy.csv' WITH CSV HEADER
COPY 100

test_db=# \! ls -la /var/lib/postgresql/backups/
total 12
drwxrwxr-x 2 postgres postgres 4096 мар  2 00:04 .
drwxr-xr-x 7 postgres postgres 4096 мар  2 00:00 ..
-rw-rw-r-- 1 postgres postgres 3600 мар  2 00:04 table1_copy.csv

test_db=# \copy my_schema.table2 FROM '/var/lib/postgresql/backups/table1_copy.csv' WITH CSV HEADER
COPY 100

test_db=# select count(*) from my_schema.table2;
 count
-------
   100
(1 строка)
```

<img width="1625" height="671" alt="image" src="https://github.com/user-attachments/assets/f0df207e-7ae9-4d40-a3b8-2f630e6ae9df" /><br>



<img width="2297" height="431" alt="image" src="https://github.com/user-attachments/assets/e7e56b27-ba46-4878-97d1-f6641f065be5" /><br>



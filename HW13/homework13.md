# Домашнее задание 13
## Бэкапы

:one: Для выполнения домашнего задания подключаюсь к БД PostgreSQL, создаю базу данных `test_db`, в ней создаю схему `my_schema` и две одинаковых таблицы `table1` и `table2`, первую таблицу заполняю случайными сгенерированными данными в размере 100 строк
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

test_db=# CREATE TABLE my_schema.table2 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE

test_db=# INSERT INTO my_schema.table1 (Data) SELECT md5(random()::text) FROM generate_series(1, 100);
INSERT 0 100
```
<img width="1673" height="551" alt="image" src="https://github.com/user-attachments/assets/8fb7a0ae-fae4-4efd-8ab7-fe8fb6678a17" /><br>

Создаю новый каталог для бекапа под пользователем postgres `/var/lib/postgresql/backups/`, подключаюсь с базе данных `test_db` и выгружаю таблицу `table1` в файл CSV командой COPY, проверяю наличие файла на диске. Далее восстанавливаю данные в таблицу `table2` путём загрузки файла CSV командой COPY, проверяю наличие данных в таблице `table2`
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

<img width="1625" height="671" alt="image" src="https://github.com/user-attachments/assets/f0df207e-7ae9-4d40-a3b8-2f630e6ae9df" /><br><br>

:two: С помощью утилиты `PG_DUMP` создаю сжатый дамп (-Fc) только схемы `my_schema`
```
sudo -u postgres pg_dump --cluster 18/otus -d test_db -n my_schema -Fc -f /var/lib/postgresql/backups/my_schema.gz
```
Создаю новую базу данных `restored_db` и в ней создаю схему `my_schema`
```
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE restored_db;
CREATE DATABASE
postgres=# \c restored_db;
Вы подключены к базе данных "restored_db" как пользователь "postgres".
restored_db=# CREATE SCHEMA my_schema;
CREATE SCHEMA
restored_db=# \q
```

Из созданного дампа c помощью утилиты `PG_RESTORE` в новую базу данных `restored_db` восстанавливаю только таблицу `my_schema.table2`
```
sudo -u postgres pg_restore --cluster 18/otus -d restored_db -n my_schema -t table2 /var/lib/postgresql/backups/my_schema.gz
```
Проверяю наличие восстановленой таблицы и её содержимое
```
sudo -u postgres psql --cluster 18/otus -d restored_db

restored_db=# \dt my_schema.*
              Список таблиц
   Схема   |  Имя   |   Тип   | Владелец
-----------+--------+---------+----------
 my_schema | table2 | таблица | postgres
(1 строка)

restored_db=# select count(*) from my_schema.table2;
 count
-------
   100
(1 строка)
```
<img width="2425" height="971" alt="image" src="https://github.com/user-attachments/assets/692f31bb-b649-412a-b584-38899906e0a5" /><br>

Прилагаю скрин созданных и заполненых таблиц, использованных в домашнем задании

<img width="1651" height="1093" alt="image" src="https://github.com/user-attachments/assets/6c62fa0c-7e6e-459d-9a85-7cc94595be2d" />

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


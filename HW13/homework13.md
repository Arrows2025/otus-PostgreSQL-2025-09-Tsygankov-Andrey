# Домашнее задание 13
## Бэкапы

Для выполнения домашнего задания подключаюсь в БД PostgreSQL, создаю базу данных `test_db`, в ней создаю схему `my_schema` и две одинаковых таблицы `table1` и `table2`
```
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE test_db;
CREATE DATABASE

postgres=# \c test_db;
Вы подключены к базе данных "test_db" как пользователь "postgres".

test_db=# CREATE SCHEMA my_schema;
CREATE SCHEMA

test_db=# CREATE TABLE table1 (
    id SERIAL PRIMARY KEY,
    Data TEXT
);
CREATE TABLE

test_db=# CREATE TABLE table2 (
    id SERIAL PRIMARY KEY,
    Data TEXT
);
CREATE TABLE

test_db=# INSERT INTO table1 (Data) SELECT md5(random()::text) FROM generate_series(1, 100);
INSERT 0 100
```

<img width="1513" height="731" alt="image" src="https://github.com/user-attachments/assets/b52acc99-a7f6-49c4-a190-7f30666d502a" /><br>


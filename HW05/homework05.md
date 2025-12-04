# Домашнее задание 05
## Нагрузочное тестирование и тюнинг PostgreSQL
Для выполнения домашнего задания использую уже установленный на виртуальной машине сервер Ubuntu 24.04.3 и установленную базу данных PostgreSQL 18

Для инициализации Pgbench создаю для тестов базу данных `pbtest`, для инициализации Sysbench (задание со :star:) создаю для тестов базу данных sbtest и пользователя sbtest и выдаю ему все привилегии на базу данных sbtest
```
sudo -u postgres psql

postgres=# CREATE DATABASE pbtest;
CREATE DATABASE
postgres=# CREATE DATABASE sbtest;
CREATE DATABASE
postgres=# CREATE USER sbtest WITH PASSWORD 'sbtest';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbtest;
GRANT
postgres=# \q
```
<img width="1373" height="471" alt="image" src="https://github.com/user-attachments/assets/93ce7684-7993-42d2-a2ae-36d8bcb844c1" /><br>

Инициализирую Pgbench
```
sudo su postgres
pgbench -i pbtest
```
<img width="1633" height="333" alt="image" src="https://github.com/user-attachments/assets/1b3c8a4c-34c7-477e-83d8-b5c853c415fa" /><br>

Устанавливаю утилиту Sysbench :star:
```
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```

<img width="1724" height="609" alt="image" src="https://github.com/user-attachments/assets/6e338e8f-6591-46d4-92e1-3364f5bac52b" />
<img width="1724" height="1069" alt="image" src="https://github.com/user-attachments/assets/39d1903e-1903-4211-8487-61c0a258ad11" />


Таблица результатов нагрузочного тестирования
|Параметры PostgreSQL|Pgbench|Sysbench|
|:-|:-|:-|
|Равнение по левому краю|Равнение по центру|Равнение по правому краю|
|Запись|Запись|Запись|
|Запись|Запись|Запись|
|Запись|Запись|Запись|
|Запись|Запись|Запись|

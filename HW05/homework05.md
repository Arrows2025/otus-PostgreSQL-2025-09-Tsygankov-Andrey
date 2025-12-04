# Домашнее задание 05
## Нагрузочное тестирование и тюнинг PostgreSQL
Для выполнения домашнего задания использую уже установленный на виртуальной машине сервер Ubuntu 24.04.3 и установленную базу данных PostgreSQL 18. Основное домашнее задание Pgbench и задание со звёздочкой Sysbench :star: буду выполнять параллельно для одинаковых параметров настройки базы данных

Для инициализации Pgbench создаю для тестов базу данных `pbtest`, для инициализации Sysbench :star: создаю для тестов базу данных `sbtest`, пользователя `sbtest` и выдаю все привилегии этому пользователю на базу данных `sbtest`
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

Инициализирую утилиту Pgbench
```
sudo su postgres
pgbench -i pbtest
```
<img width="1633" height="333" alt="image" src="https://github.com/user-attachments/assets/d191b140-5aa8-4c4b-9454-49517cdd2ba4" /><br>

Устанавливаю утилиту Sysbench :star:
```
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```

<img width="1724" height="609" alt="image" src="https://github.com/user-attachments/assets/6e338e8f-6591-46d4-92e1-3364f5bac52b" />
<img width="1724" height="1069" alt="image" src="https://github.com/user-attachments/assets/39d1903e-1903-4211-8487-61c0a258ad11" /><br><br>

Инициализирую утилиту Sysbench :star: на 10 таблиц по 1 миллиону записей в каждой
```
sysbench --db-driver=pgsql --pgsql-host=192.168.0.50 --pgsql-port=5432 --pgsql-db=sbtest --pgsql-user=postgres --pgsql-password=postgres --tables=10 --table-size=1000000 oltp_read_write prepare
```
<img width="1672" height="839" alt="image" src="https://github.com/user-attachments/assets/e292b27e-7fa9-4279-8b29-19a1bba8eeaf" /><br>


<img width="1789" height="954" alt="image" src="https://github.com/user-attachments/assets/d392536f-43e7-499d-8b1f-6060252a3880" />

Запускаю тест утилит Pgbench и Sysbench :star: на базовых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
```
pgbench -c 50 -j 2 -P 10 -T 60 pbtest

sysbench --db-driver=pgsql --pgsql-host=192.168.0.50 --pgsql-port=5432 --pgsql-db=sbtest --pgsql-user=postgres --pgsql-password=postgres --threads=2 --time=60 oltp_read_write run
```
<img width="1659" height="563" alt="image" src="https://github.com/user-attachments/assets/4461ab25-3e42-4066-9d4d-eba57762bc17" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/afe88c28-4f29-4211-8307-c802cc414b24" /><br>


Таблица результатов нагрузочного тестирования
|Параметры PostgreSQL|Pgbench|Sysbench :star:|
|:-|-:|-:|
|Базовые параметры|297.55 tps|189.86 tps|
|Запись|Запись|Запись|
|Запись|Запись|Запись|
|Запись|Запись|Запись|
|Запись|Запись|Запись|

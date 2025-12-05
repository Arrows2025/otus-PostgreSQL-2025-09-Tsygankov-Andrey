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

:one: Запускаю тест утилит Pgbench и Sysbench :star: на базовых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
```
pgbench -c 50 -j 2 -P 10 -T 60 pbtest

sysbench --db-driver=pgsql --pgsql-host=192.168.0.50 --pgsql-port=5432 --pgsql-db=sbtest --pgsql-user=postgres --pgsql-password=postgres --threads=2 --time=60 oltp_read_write run
```
<img width="1659" height="563" alt="image" src="https://github.com/user-attachments/assets/4461ab25-3e42-4066-9d4d-eba57762bc17" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/afe88c28-4f29-4211-8307-c802cc414b24" /><br><br>

:two: Для повышения производительности кластера отключаю параметр fsync = off. Этот параметр отвечает за сброс данных из кэша на диск при завершении транзакций. Если его отключить, данные не будут записываться на дисковые накопители сразу после завершения операций. Это может повысить скорость операций insert и update. Однако отключение fsync может повредить базу, если произойдёт сбой (неожиданное отключение питания, сбой ОС, сбой дисковой подсистемы). В домашнем задании мы настраиваем кластер на максимальную производительность, не обращая внимания на надёжность.
```
postgres@ubuntu24server:/home/arrows$ psql

postgres=# show fsync;
postgres=# alter system set fsync = off;
postgres=# select pg_reload_conf();
postgres=# show fsync;
postgres=# \q
```

<img width="1074" height="632" alt="image" src="https://github.com/user-attachments/assets/730d817d-50ae-49fb-8edd-79b2f3ca9c40" /><br>

Запускаю повторно тест утилит Pgbench и Sysbench :star: на новых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
<img width="1672" height="563" alt="image" src="https://github.com/user-attachments/assets/14ae53cc-49a9-40f4-9826-765f8485b002" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/735ab713-40c0-450e-9460-b854b98d4a4d" /><br><br>


:three: Получаю заметное увеличение числа транзакций в секунду и дополнительно отключаю ещё один параметр synchronous_commit = off. Этот параметр отключает синхронную запись в журнал предзаписи (WAL) в момент коммита транзакции.
```
postgres@ubuntu24server:/home/arrows$ psql

postgres=# show synchronous_commit;
postgres=# alter system set synchronous_commit = off;
postgres=# select pg_reload_conf();
postgres=# show synchronous_commit;
postgres=# \q
```

<img width="1672" height="632" alt="image" src="https://github.com/user-attachments/assets/83926b3a-17e7-449e-bbbc-db9339ef2773" /><br>

Запускаю повторно тест утилит Pgbench и Sysbench :star: на новых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
<img width="1672" height="563" alt="image" src="https://github.com/user-attachments/assets/a1385607-c43a-458a-8aed-55cb4221e3e5" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/92b11271-5740-4ad9-be5c-461eb4abda35" /><br><br>

:four: Изменение параметра synchronous_commit не дало увеличения производительности, далее меняю параметр random_page_cost = 1.25, на рекомендуемое значение для SSD дисков по верхней планке. Этот параметр планировщика запросов задаёт примерную стоимость чтения с диска одной страницы данных при произвольном доступе
```
postgres@ubuntu24server:/home/arrows$ psql
postgres=# show random_page_cost;
postgres=# alter system set random_page_cost = 1.25;
postgres=# select pg_reload_conf();
postgres=# show random_page_cost;
postgres=# \q
```

<img width="1672" height="632" alt="image" src="https://github.com/user-attachments/assets/d93fce4b-6de0-403a-a9c4-e160a2b27751" /><br>

Запускаю повторно тест утилит Pgbench и Sysbench :star: на новых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
<img width="1672" height="563" alt="image" src="https://github.com/user-attachments/assets/5a303453-313f-47f6-8f1d-6b689317b32a" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/5fcbc64e-97fc-40a2-8d05-3f13963a8f4a" /><br><br>

:five: Изменение параметра random_page_cost = 1.25 так же на дало увеличения производительности, пробую поставить этот параметр в значение random_page_cost = 1.1 на рекомендуемое значение для SSD дисков по нижней планке
```
postgres@ubuntu24server:/home/arrows$ psql
postgres=# show random_page_cost;
postgres=# alter system set random_page_cost = 1.1;
postgres=# select pg_reload_conf();
postgres=# show random_page_cost;
postgres=# \q
```
<img width="1672" height="632" alt="image" src="https://github.com/user-attachments/assets/dd19c8dd-663e-4d60-a722-bf0a087744f6" /><br>

Запускаю повторно тест утилит Pgbench и Sysbench :star: на новых параметрах БД PostgreSQL, результаты TPS - количество транзакций в секунду заношу в таблицу
<img width="1672" height="563" alt="image" src="https://github.com/user-attachments/assets/d4dbcc0d-949a-402a-b21d-25d0a4bf57ec" />
<img width="1672" height="954" alt="image" src="https://github.com/user-attachments/assets/20ca8bfd-e145-4275-bb1f-faf32380e8e5" /><br><br>

Таблица результатов нагрузочного тестирования
|Параметры PostgreSQL|Pgbench|Sysbench :star:|
|:-|-:|-:|
|Базовые параметры|297.55 tps|189.86 tps|
|fsync = off|978.09 tps|300.48 tps|
|synchronous_commit = off|973.43 tps|289.42 tps|
|random_page_cost = 1.25|935.48 tps|286.08 tps|
|random_page_cost = 1.1|947.36 tps|260.44 tps|

Как видно по результатам из таблицы самое большое повышение производительности дало отключение параметра fsync = off, другие параметры существенного изменения производительности не дали.

# Домашнее задание 07
## Работа с журналами

Для выполнения домашнего задания использую уже установленный сервер Ubuntu 24.04.3 на виртуальной машине с процессором на 2 ядра и 4 Гб оперативной памяти и установленную базу данных PostgreSQL 18 с базовыми параметрами

Для инициализации Pgbench создаю для тестов базу данных pbtest и инициализирую утилиту Pgbench
```
sudo -u postgres psql

postgres=# CREATE DATABASE pbtest;
CREATE DATABASE
postgres=# \q

sudo su postgres
pgbench -i pbtest
```
<img width="2009" height="791" alt="image" src="https://github.com/user-attachments/assets/082ed790-7350-45df-bc0e-0b34ab5b050b" /><br>

Сбрасываю статистику, чтобы измерить какой объём журнальных файлов был сгенерирован за время работы утилиты Pgbench, и настраиваю выполнение контрольной точки раз в 30 секунд
```
sudo -u postgres psql

postgres=# SELECT pg_stat_statements_reset();
postgres=# ALTER SYSTEM SET checkpoint_timeout = 30;
postgres=# select pg_reload_conf();
postgres=# show checkpoint_timeout;
postgres=# \q
```
<img width="1299" height="873" alt="image" src="https://github.com/user-attachments/assets/d65e1c19-d1a7-4598-ae40-55af6a030dd6" /><br>

10 минут с помощью утилиты Pgbench подаю нагрузку на БД PostgreSQL
```
sudo su postgres
pgbench -c 50 -j 2 -P 60 -T 600 pbtest
```
<img width="1267" height="903" alt="image" src="https://github.com/user-attachments/assets/cbf5927c-7dfe-4845-8a09-1f3b7721ff98" /><br>

С помощью запроса нахожу идентификатор базы данных утилиты Pgbench: dbid = 40990
```
SELECT dbid,
       queryid,
       query,
       total_exec_time,
       wal_records,
       wal_fpi,
       wal_bytes,
       stats_since
FROM pg_stat_statements;
```
<img width="1297" height="622" alt="image" src="https://github.com/user-attachments/assets/515afeeb-3529-460c-be2a-a54b95e2eb6b" /><br>

Считаю сумму байт журнальных файлов, которые были сгенерированы утилитой Pgbench - `358 083 342 байт ~ 342 Мб`. На одну контрольную точку приходится объём данных `342 Мб / 20 = 17.1 Мб`
```
SELECT sum(wal_bytes)
FROM pg_stat_statements
where dbid = 40990;
```
<img width="1267" height="753" alt="image" src="https://github.com/user-attachments/assets/ac355cb4-d77f-41a5-80b9-349cb2518716" /><br>


Вывожу из лога БД PostgreSQL строки с временем запуска контрольных точек, за 10 минут работы утилиты Pgbench все 20 запусков контрольных точек произошли по расписанию через 30 секунд - в 08 и 38 секунд каждой минуты
```
tail -f -n52 /var/log/postgresql/postgresql-18-main.log | grep "начата контрольная точка: time"
```
<img width="1961" height="701" alt="image" src="https://github.com/user-attachments/assets/bd603f88-a9bc-461e-ae25-e9af653cadad" /><br>


Отключаю синхронный режим работы кластера
```
sudo -u postgres psql

postgres=# ALTER SYSTEM SET synchronous_commit = off;
postgres=# select pg_reload_conf();
postgres=# show synchronous_commit;
postgres=# \q
```
<img width="1267" height="663" alt="image" src="https://github.com/user-attachments/assets/2cfb4a89-5a37-472d-98a9-d0570276e781" />

И вновь запускаю утилиту Pgbench с такими же параметрами, чтобы измерить количество транзакций в секунду в асинхронном режиме. В синхронном режиме количество транзакций в секунду TPS = 250.744716, в асинхронном количество транзакций в секунду более чем вдвое выше TPS = 517.283712. В асинхронном режиме транзакции считаются зафиксированными сразу после записи в журнал WAL, без ожидания записи на диск, это кратно увеличивает производительность, но повышает риск потери данных
```
sudo su postgres
pgbench -c 50 -j 2 -P 60 -T 600 pbtest
```
<img width="1267" height="903" alt="image" src="https://github.com/user-attachments/assets/380fc918-f60f-42fe-84b5-8adadfff98b5" /><br>




<img width="1267" height="423" alt="image" src="https://github.com/user-attachments/assets/5fd17c19-a323-4d48-bc8d-dfbbf0fe6364" />

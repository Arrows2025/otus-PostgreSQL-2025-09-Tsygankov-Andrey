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
<img width="1321" height="851" alt="image" src="https://github.com/user-attachments/assets/cb2485a5-8c4c-4564-9a8c-2f2cd430af7b" /><br>

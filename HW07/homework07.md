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

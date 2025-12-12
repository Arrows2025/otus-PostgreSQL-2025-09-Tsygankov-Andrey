# Домашнее задание 06
## Настройка autovacuum с учётом особенностей производительности

Для выполнения домашнего задания использую уже установленный на виртуальной машине сервер Ubuntu 24.04.3 и установленную базу данных PostgreSQL 18 с базовыми параметрами. Настраиваю процессор витруальной машины на 2 ядра и ставлю 4 Гб оперативной памяти.

Для инициализации Pgbench создаю для тестов базу данных `pbtest` и инициализирую утилиту Pgbench
```
sudo -u postgres psql

postgres=# CREATE DATABASE pbtest;
CREATE DATABASE
postgres=# \q

sudo su postgres
pgbench -i pbtest
```

<img width="2009" height="791" alt="image" src="https://github.com/user-attachments/assets/082ed790-7350-45df-bc0e-0b34ab5b050b" /><br>

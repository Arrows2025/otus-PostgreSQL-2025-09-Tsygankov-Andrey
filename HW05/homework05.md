# Домашнее задание 05
## Нагрузочное тестирование и тюнинг PostgreSQL
Для выполнения домашнего задания использую уже установленный на виртуальной машине сервер Ubuntu 24.04.3 и установленную базу данных PostgreSQL 18

Для инициализации Pgbench создаю для тестов базу данных `test`
```
sudo -u postgres psql

postgres=# CREATE DATABASE test;
CREATE DATABASE
postgres=# \q

sudo su postgres
pgbench -i test
```
<img width="2009" height="791" alt="image" src="https://github.com/user-attachments/assets/e65e6d21-1f9f-439a-8773-8a16cddfabec" /><br>


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

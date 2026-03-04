# Домашнее задание 14
## Репликация

Для выполнения домашнего используются три ранее установленные виртуальные машины с процессором на 1 ядро и 2 Гб оперативной памяти, на каждую виртуальную машину установлена серверная операционная система Ubuntu 24.04.3:
* Ubuntu24Server - IP 192.168.0.50
* Ubuntu24Node1 - IP 192.168.0.51
* Ubuntu24Node2 - IP 192.168.0.52

<img width="1089" height="624" alt="image" src="https://github.com/user-attachments/assets/4181101a-49b5-4d3e-a5e3-50335b5bb097" /><br>

:one: Подключаюсь к базе данных на ВМ1 и устанавливаю `wal_level = logical` для логической репликации
```
sudo -u postgres psql --cluster 18/otus

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# \q
```
Перегружаю кластер и заново подключаюсь, создаю базу данных `replica`, в ней две таблицы `test` и `test2`, на таблицу `test` создаю публикацию `test_pub`
```
sudo pg_ctlcluster 18 otus restart
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE replica;
CREATE DATABASE
postgres=# \c replica;
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# CREATE TABLE test (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
```
<img width="1097" height="731" alt="image" src="https://github.com/user-attachments/assets/9bfbb29f-588b-4406-a5aa-0b3eecc43044" /><br>

:two: На ВМ2 создаю и стартую новый кластер `otus2`, настраиваю конфигурационные файлы PostgreSQL `postgresql.conf` и `pg_hba.conf`, подключаюсь к кластеру, устанавливаю пароль для пользователя `postgres` и `wal_level = logical` для логической репликации
```
sudo pg_createcluster 18 otus2 --start

18  otus2   5433 online postgres /var/lib/postgresql/18/otus2 /var/log/postgresql/postgresql-18-otus2.log

sudo nano /etc/postgresql/18/otus2/postgresql.conf
sudo nano /etc/postgresql/18/otus2/pg_hba.conf
sudo -u postgres psql --cluster 18/otus2

postgres=# \password
postgres=# ALTER SYSTEM SET wal_level = logical;
postgres=# \q
```

Перегружаю кластер и заново подключаюсь, создаю базу данных `replica`, в ней две таблицы `test2` и `test`, на таблицу `test2` создаю публикацию `test2_pub`, а на публикацию `test_pub` таблицы `test` с ВМ1 создаю подписку `test_sub`
```
sudo pg_ctlcluster 18 otus2 restart
sudo -u postgres psql --cluster 18/otus2

postgres=# CREATE DATABASE replica;
CREATE DATABASE
postgres=# \c replica;
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE TABLE test (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE PUBLICATION test2_pub FOR TABLE test2;
CREATE PUBLICATION
replica=# CREATE SUBSCRIPTION test_sub
CONNECTION 'host=192.168.0.50 port=5433 user=postgres password=postgres dbname=replica'
PUBLICATION test_pub WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test_sub"
CREATE SUBSCRIPTION
```
<img width="2073" height="1121" alt="image" src="https://github.com/user-attachments/assets/67b150fb-7b9a-4788-bb5f-23dba7ae9e5d" />
<img width="2073" height="671" alt="image" src="https://github.com/user-attachments/assets/2929af9d-b5f5-42c7-97d6-663fe3736d45" /><br><br>

:three: На ВМ1 подключаюсь к кластеру и к базе данных `replica` и на публикацию `test2_pub` таблицы `test2` с ВМ2 создаю подписку `test2_sub`
```
sudo -u postgres psql --cluster 18/otus

postgres=# \c replica
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# CREATE SUBSCRIPTION test2_sub
CONNECTION 'host=192.168.0.51 port=5433 user=postgres password=postgres dbname=replica'
PUBLICATION test2_pub WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test2_sub"
CREATE SUBSCRIPTION
```
<img width="1433" height="401" alt="image" src="https://github.com/user-attachments/assets/10216c52-8688-49c9-9136-02db42be83d8" /><br>

:four: На ВМ3 создаю и стартую новый кластер `otus3`, настраиваю конфигурационные файлы PostgreSQL `postgresql.conf` и `pg_hba.conf`, подключаюсь к кластеру, устанавливаю пароль для пользователя `postgres`. Параметр `wal_level` не меняю, так как этот кластер используется для чтения объединённых данных и резервного копирования, для публикаций он не используется
```
sudo pg_createcluster 18 otus3 --start

18  otus3   5433 online postgres /var/lib/postgresql/18/otus3 /var/log/postgresql/postgresql-18-otus3.log

sudo nano /etc/postgresql/18/otus3/postgresql.conf
sudo nano /etc/postgresql/18/otus3/pg_hba.conf
sudo -u postgres psql --cluster 18/otus3

postgres=# \password
Введите новый пароль для пользователя "postgres":
Повторите его:
postgres=# \q
```

Перегружаю кластер и заново подключаюсь, создаю базу данных `replica`, в ней две таблицы `test` и `test2`, на публикацию `test_pub` таблицы `test` с ВМ1 создаю подписку `test_sub3`, на публикацию `test2_pub` таблицы `test2` с ВМ2 создаю подписку `test2_sub3`. При попытке создать подписку и именем `test_sub` получаю ошибку, что слот репликации `test_sub` уже существует
```
sudo pg_ctlcluster 18 otus3 restart
sudo -u postgres psql --cluster 18/otus3

postgres=# CREATE DATABASE replica;
CREATE DATABASE
postgres=# \c replica;
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# CREATE TABLE test (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE SUBSCRIPTION test_sub
CONNECTION 'host=192.168.0.50 port=5433 user=postgres password=postgres dbname=replica'
PUBLICATION test_pub WITH (copy_data = true);
ОШИБКА:  не удалось создать слот репликации "test_sub": ОШИБКА:  слот репликации "test_sub" уже существует
replica=# CREATE SUBSCRIPTION test_sub3
CONNECTION 'host=192.168.0.50 port=5433 user=postgres password=postgres dbname=replica'
PUBLICATION test_pub WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test_sub3"
CREATE SUBSCRIPTION
replica=# CREATE SUBSCRIPTION test2_sub3
CONNECTION 'host=192.168.0.51 port=5433 user=postgres password=postgres dbname=replica'
PUBLICATION test2_pub WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test2_sub3"
CREATE SUBSCRIPTION
```
<img width="2073" height="1091" alt="image" src="https://github.com/user-attachments/assets/f2d3c95c-1afc-4111-bc14-79ddd48b5cd6" />
<img width="2073" height="881" alt="image" src="https://github.com/user-attachments/assets/f7819315-58d6-4567-80e5-5e960b2a7c6a" /><br><br>

:five: Проверка работы логической репликации. Проверяю списки созданных подписок на всех ВМ
```
sudo -u postgres psql --cluster 18/otus

postgres=# \c replica
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# \dRs
               Список подписок
    Имя    | Владелец | Включён | Публикация
-----------+----------+---------+-------------
 test2_sub | postgres | t       | {test2_pub}
(1 строка)
```
<img width="2533" height="559" alt="image" src="https://github.com/user-attachments/assets/fccaf263-9aa3-4da2-a1f1-4b9f66011923" /><br>


Вставляю строку в таблицу `test` на ВМ1 и проверяю наличие этой строки в таблицах `test` на ВМ2 и ВМ3 - строка реплицировалась
```
sudo -u postgres psql --cluster 18/otus

postgres=# \c replica
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# select * from test;
 id | data
----+------
(0 строк)

replica=# insert into test(data) values ('ВМ1');
INSERT 0 1

replica=# select * from test;
 id | data
----+------
  1 | ВМ1
(1 строка)
```
<img width="2530" height="707" alt="image" src="https://github.com/user-attachments/assets/3a52644f-b5b8-4d7d-a38a-632aee5b841e" /><br>

Вставляю строку в таблицу `test2` на ВМ2 и проверяю наличие этой строки в таблицах `test2` на ВМ1 и ВМ3 - строка реплицировалась
```
sudo -u postgres psql --cluster 18/otus2

postgres=# \c replica
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# select * from test2;
 id | data
----+------
(0 строк)

replica=# insert into test2(data) values ('ВМ2');
INSERT 0 1

replica=# select * from test2;
 id | data
----+------
  1 | ВМ2
(1 строка)
```

<img width="2533" height="707" alt="image" src="https://github.com/user-attachments/assets/1acbd77c-8244-4048-a8f6-51be1ee2edc1" /><br>

⭐ Настроить физическую репликацию с ВМ4, используя ВМ3 в качестве источника

На ВМ3 подключаюсь к кластеру `otus3`, проверяю параметр wal_level, он по умолчанию установлен в значение `replica`, ничего не меняю, настраиваю конфигурационный файл PostgreSQL `pg_hba.conf`, добавляю строку для репликации: `host    replication     all             192.168.0.52/24         scram-sha-256`, перегружаю кластер и проверяю его статус
```
sudo -u postgres psql --cluster 18/otus3

postgres=# show wal_level;
 wal_level
-----------
 replica
(1 строка)

postgres=# \q

sudo nano /etc/postgresql/18/otus3/pg_hba.conf
sudo pg_ctlcluster 18 otus3 restart
pg_lsclusters

Ver Cluster Port Status         Owner    Data directory               Log file
18  main    5432 online,patroni postgres /var/lib/postgresql/18/main  /var/log/postgresql/postgresql-18-main.log
18  otus3   5433 online         postgres /var/lib/postgresql/18/otus3 /var/log/postgresql/postgresql-18-otus3.log
```
<img width="1849" height="581" alt="image" src="https://github.com/user-attachments/assets/dd12d601-9997-434e-bc9e-80753b94de1d" /><br>



Устанавливаю сервер Ubuntu 24.04.3 на четвёртую виртуальную машину и настраиваю ему статический IP-адрес 192.168.0.53:
* Ubuntu24Node3 - IP 192.168.0.53

<img width="1089" height="624" alt="image" src="https://github.com/user-attachments/assets/0c8792fa-50ad-4347-87b7-01f4f1885e40" /><br>

Устанавливаю PostgreSQL с пакетами дополнительных программ: `sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc`

На ВМ4 создаю новый кластер `otus4`, настраиваю конфигурационные файлы PostgreSQL `postgresql.conf` и `pg_hba.conf`, останавливаю кластер, удаляю папку /var/lib/postgresql/18/otus4 и проверяю её наличие на диске, с помощью утилиты pg_basebackup делаю бэкап кластера `otus3` с ВМ3 с ключом -R, который создаст заготовку управляющего файла recovery.conf
```
sudo pg_createcluster 18 otus4 --start

18  otus4   5433 online postgres /var/lib/postgresql/18/otus4 /var/log/postgresql/postgresql-18-otus4.log

sudo nano /etc/postgresql/18/otus4/postgresql.conf
sudo nano /etc/postgresql/18/otus4/pg_hba.conf
sudo pg_ctlcluster 18 otus4 stop

ls -la /var/lib/postgresql/18/
total 16
drwxr-xr-x  4 postgres postgres 4096 Mar  4 03:54 .
drwxr-xr-x  3 postgres postgres 4096 Mar  4 03:28 ..
drwx------ 19 postgres postgres 4096 Mar  4 03:34 main
drwx------ 19 postgres postgres 4096 Mar  4 04:04 otus4

sudo rm -rf /var/lib/postgresql/18/otus4

ls -la /var/lib/postgresql/18/
total 12
drwxr-xr-x  3 postgres postgres 4096 Mar  4 04:06 .
drwxr-xr-x  3 postgres postgres 4096 Mar  4 03:28 ..
drwx------ 19 postgres postgres 4096 Mar  4 03:34 main

sudo -u postgres pg_basebackup -h 192.168.0.52 -p 5433 -R -D /var/lib/postgresql/18/otus4

ls -la /var/lib/postgresql/18/
total 16
drwxr-xr-x  4 postgres postgres 4096 Mar  4 04:06 .
drwxr-xr-x  3 postgres postgres 4096 Mar  4 03:28 ..
drwx------ 19 postgres postgres 4096 Mar  4 03:34 main
drwx------ 19 postgres postgres 4096 Mar  4 04:06 otus4

sudo pg_ctlcluster 18 otus4 start
pg_lsclusters

Ver Cluster Port Status Owner    Data directory               Log file
18  main    5432 online postgres /var/lib/postgresql/18/main  /var/log/postgresql/postgresql-18-main.log
18  otus4   5433 online postgres /var/lib/postgresql/18/otus4 /var/log/postgresql/postgresql-18-otus4.log
```
<img width="2073" height="791" alt="image" src="https://github.com/user-attachments/assets/7a961a7d-31bf-428e-a24c-3fb66c18b365" />
<img width="2073" height="911" alt="image" src="https://github.com/user-attachments/assets/bc810952-2661-4022-b264-e4cc26af71f5" /><br><br>



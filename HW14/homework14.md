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

:two: На ВМ2 создаю и стартую новый кластер `otus2`, настраиваю конфигурационный файлы PostgreSQL `postgresql.conf` и `pg_hba.conf`, подключаюсь к кластеру, устанавливаю пароль для пользователя `postgres` и `wal_level = logical` для логической репликации
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


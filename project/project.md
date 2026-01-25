# Проектная работа
## Создание и тестирование высоконагруженного отказоустойчивого кластера PostgreSQL с использованием Patroni, etcd, HAProxy

### 1. Установка и настройка PostgreSQL
Для проектной работы в Oracle VirtualBox Manager созданы три виртуальные машины с процессором на 1 ядро и 2 Гб оперативной памяти, на каждую виртуальную машину установлена серверная операционная система Ubuntu 24.04.3:
* Ubuntu24Server - IP 192.168.0.50
* Ubuntu24Node1 - IP 192.168.0.51
* Ubuntu24Node2 - IP 192.168.0.52

Устанавливаю на каждую виртуальную машину базу данных PostgreSQL 18 с базовыми параметрами и пакетом дополнительных программ: `sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc`

Редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/main/postgresql.conf` для прослушивания всех адресов
```
#listen_addresses = 'localhost'          # what IP address(es) to listen on;
изменить на
listen_addresses = '*'                   # what IP address(es) to listen on;
```
Редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/main/pg_hba.conf` для доступа к БД PostgreSQL с моей локальной машины
```
добавить в конец файла строки
host    all             all             0.0.0.0/0               scram-sha-256
host    replication     replicator      0.0.0.0/0               scram-sha-256
```
Устанавливаю пароль для пользователя базы данных postgres и создаю нового пользователя replicator
```
sudo -u postgres psql

postgres=# \password
postgres=# create user replicator replication login encrypted password 'replicator';
postgres=# \q
```

Перезагрузка кластера базы данных `sudo pg_ctlcluster 18 main restart` и тест соединения с БД PostgreSQL через приложение DBeaver прошёл успешно. Содержимое базы данных на первом узле следующее:
<img width="2099" height="843" alt="image" src="https://github.com/user-attachments/assets/fdf7f70b-8f19-40f1-926d-35fcd6b2adcc" /><br>


Останавливаю PostgreSQL, отключаю автоматический запуск сервиса PostgreSQL, на второй и третьей ноде удаляю содержимое каталога pgdata
```
sudo systemctl stop postgresql
sudo systemctl disable postgresql.service
sudo rm -rf /var/lib/postgresql/18/main/
```
<img width="1801" height="251" alt="image" src="https://github.com/user-attachments/assets/f7a31696-e6d1-4989-acfa-6b91bfd15542" /><br>

### 2. Установка и настройка etcd

На трёх узлах устанавливаю etcd
```
sudo apt-get update                  # обновление системы
sudo apt-get install etcd-server     # установка etcd-server
sudo apt-get install etcd-client     # установка etcd-client
etcd --version                       # проверка установки
```
<img width="2537" height="641" alt="image" src="https://github.com/user-attachments/assets/e083d8bd-7fba-424e-9b06-9ba2d0293149" />
<img width="2537" height="1361" alt="image" src="https://github.com/user-attachments/assets/664912f2-e361-4080-bb49-db7a219b3d48" />
<img width="2537" height="1031" alt="image" src="https://github.com/user-attachments/assets/b7553652-3db2-401f-be5f-a343db2a7c6d" />
<img width="2537" height="221" alt="image" src="https://github.com/user-attachments/assets/491ca40e-1520-4799-bb42-dfb63cea3c67" /><br><br>

Останавливаю etcd, выключаю сервис etcd из автозагрузки systemd и удаляю конфигурацию etcd по умолчанию
```
sudo systemctl stop etcd
sudo systemctl disable etcd
sudo rm -rf /var/lib/etcd/default
```
<img width="1705" height="281" alt="image" src="https://github.com/user-attachments/assets/eddc7e25-5f64-4c52-9bb0-ab53c9afad1b" /><br>

Настраиваю etcd на трёх узлах `sudo nano /etc/default/etcd`
```
ETCD_NAME="etcd-node-0"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.50:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.50:2379"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
```

```
ETCD_NAME="etcd-node-1"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.51:2380"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.51:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
```

```
ETCD_NAME="etcd-node-2"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.52:2380"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.52:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
```

Запускаю etcd на трёх узлах `sudo systemctl start etcd' и проверяю состояние кластера etcd, кластер стартован успешно
```
etcdctl member list
etcdctl endpoint --cluster health
etcdctl endpoint status --cluster -w table
```
<img width="2393" height="551" alt="image" src="https://github.com/user-attachments/assets/21eea86f-67d2-46b8-b1ed-d0e97d42ea81" /><br>

Останавливаю etcd на трёх узлах для корректировки параметра ETCD_INITIAL_CLUSTER_STATE="new" на ETCD_INITIAL_CLUSTER_STATE="existing" и вновь запускаю кластер, включаю сервис etcd в автозагрузку systemd
```
sudo systemctl start etcd
sudo systemctl enable etcd
etcdctl endpoint status --cluster -w table
```
<img width="2393" height="491" alt="image" src="https://github.com/user-attachments/assets/03fc81aa-696d-42d7-a696-24aedacf036f" /><br>

Проверка работоспособности кластера - лидером на текущий момент является узел 192.168.0.51, останавливаю etdc на узле 192.168.0.51 и проверяю состояние кластера - лидером становится узел 192.168.0.52
<img width="2393" height="401" alt="image" src="https://github.com/user-attachments/assets/f2b090d0-70f6-4aa0-b580-a07ad36bb9d2" /><br>

Стартую etcd на узле 192.168.0.51, узел возвращается в состав кластера
<img width="2393" height="311" alt="image" src="https://github.com/user-attachments/assets/37bf7bf7-656c-4cd1-b17d-332628424fc5" /><br>

Достаточно много времени ушло на решение следующей проблемы: скопировав настройки etcd из презентации, не сразу понял, что в параметр ETCD_DATA_DIR надо ещё добавить значение ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster", чтобы получился следующий путь ETCD_DATA_DIR="/var/lib/etcd/pg-cluster", пока этого не сделал, кластер не запускался.

### 3. Установка и настройка Patroni

На трёх узлах устанавливаю Patroni
```
sudo apt install python3.12-venv -- установка модуля для создания виртуальных окружений
sudo mkdir -p /opt/patroni -- создание каталога для Patroni
sudo chown postgres:postgres /opt/patroni -- передача прав владения каталогом пользователю postgres
sudo -u postgres python3 -m venv /opt/patroni/venv -- создание виртуального окружения от имени postgres
sudo -u postgres /opt/patroni/venv/bin/pip install 'patroni[etcd3]' -- установка Patroni с поддержкой etcd3
sudo -u postgres /opt/patroni/venv/bin/pip install 'psycopg2-binary' -- установка драйвера для работы Patroni с PostgreSQL
```
<img width="2553" height="1301" alt="image" src="https://github.com/user-attachments/assets/74957c10-a731-403e-80fa-f9fcee47ad43" />
<img width="2553" height="1301" alt="image" src="https://github.com/user-attachments/assets/32a4d2f9-0ae5-4e45-ac79-ef0779012057" />
<img width="2553" height="1271" alt="image" src="https://github.com/user-attachments/assets/71d8debd-7e8e-42c7-84d6-b453b791ba10" />
<img width="2553" height="281" alt="image" src="https://github.com/user-attachments/assets/afa05548-b806-42b4-ad7d-ea6be8cd8c9d" /><br><br>

Создаю файлы конфигураций Patroni на трёх узлах, аналогично представленному ниже для первой ноды
```
sudo mkdir -p /etc/patroni
sudo nano /etc/patroni/config.yml -- итоговый файл конфигурации с учётом всех правок ниже по ошибочным запускам Patroni
```
```
scope: patroni-cluster
namespace: /patroni/
name: node0
restapi:
  listen: 192.168.0.50:8008
  connect_address: 192.168.0.50:8008
etcd3:
  protocol: http
  hosts: 192.168.0.50:2379,192.168.0.51:2379,192.168.0.52:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      pg_hba:
      - host    all             all             0.0.0.0/0               scram-sha-256
      - host    replication     replicator      0.0.0.0/0               scram-sha-256
      parameters:
        wal_level: hot_standby
        hot_standby: "on"
  initdb:
  - encoding: UTF8
  - data-checksums
  users:
    arrows:
      password: ********
      options:
      - createrole
      - createdb
postgresql:
  listen: 192.168.0.50:5432
  connect_address: 192.168.0.50:5432
  data_dir: /var/lib/postgresql/18/main
  bin_dir: /usr/lib/postgresql/18/bin
  config_dir: /etc/postgresql/18/main
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres
  parameters:
    unix_socket_directories: /var/run/postgresql
tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
```

Определяю Patroni как службу, на каждом узле создаю файл `sudo nano /etc/systemd/system/patroni.service` с одинаковым содержимым:
```
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/opt/patroni/venv/bin/patroni /etc/patroni/config.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.target
```

Перевожу Patroni в автозапуск и стартую сервис Patroni на первой ноде
```
systemctl daemon-reload
systemctl enable patroni
systemctl start patroni
systemctl status patroni
```

<img width="2537" height="1391" alt="image" src="https://github.com/user-attachments/assets/fe48ba03-6fc7-4260-92eb-123aad225f9b" /><br>

Меняю переменную PATH для текущей сессии `export PATH="$PATH:/opt/patroni/venv/bin/"` и добавляю в файл `sudo nano /etc/environment` в переменную PATH путь установки Patroni `PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/opt/patroni/venv/bin/"`, так как после установки Patroni файл patronictl система не видит

Проверяю состояние кластера `patronictl -c /etc/patroni/config.yml list`, появилась первая нода, но она не запустилась
<img width="1545" height="311" alt="image" src="https://github.com/user-attachments/assets/f857adf7-5036-449b-a81d-6532cd5fe587" /><br>

Смотрю статус Patroni
<img width="2030" height="441" alt="image" src="https://github.com/user-attachments/assets/96f03202-5951-4d84-924d-ca97ddb92ac4" /><br>

Ошибка: postgres не может открыть файл конфигурации сервера "/var/lib/postgresql/18/main/postgresql.conf", добавляю в конфигурационный файл Patroni `sudo nano /etc/patroni/config.yml` параметр `postgresql.config_dir: /etc/postgresql/18/main`

Останавливаю Patroni, удаляю ноду из кластера и заново стартую сервис Patroni на первой ноде
```
systemctl stop patroni
patronictl -c /etc/patroni/config.yml remove patroni-cluster
systemctl daemon-reload
systemctl enable patroni
systemctl start patroni
systemctl status patroni
```
<img width="2489" height="1151" alt="image" src="https://github.com/user-attachments/assets/691f8b2b-5f14-44a0-8345-c6c388a35abc" /><br>
<img width="2489" height="971" alt="image" src="https://github.com/user-attachments/assets/bb351748-9890-45b9-bfb3-7f56454c180b" /><br>
<img width="1497" height="311" alt="image" src="https://github.com/user-attachments/assets/b9d3a5e0-712a-402d-8bd2-9d16d2332a17" />

Первая нода стартовала, но зайти БД PostgreSQL не удалось - `psql: ошибка: преобразовать имя "." в адрес не удалось: No address associated with hostname` :-1:

В файле `sudo nano /etc/patroni/config.yml` правлю параметр `parameters.unix_socket_directories: '.'` на значение `/var/run/postgresql`, перезапускаю Patroni и проверяю коннект к базе - `psql: ошибка: подключиться к серверу через сокет "/var/run/postgresql/.s.PGSQL.5432" не удалось: ВАЖНО:  в pg_hba.conf нет записи для компьютера "[local]", пользователя "postgres", базы "postgres", без шифрования` :-1:

Командой редактирования конфигурации кластера добавляю строку локального коннекта для пользователя `postgres`, подключение к базе данных прошло успешно :+1:
```
patronictl -c /etc/patroni/config.yml edit-config patroni-cluster
```
<img width="2521" height="1121" alt="image" src="https://github.com/user-attachments/assets/0f45265d-accd-4fbd-8bd3-e816f3b6b8d9" /><br>



Перевожу Patroni в автозапуск, стартую сервис Patroni и правлю переменную PATH на второй ноде - Patroni не запустился, забыл поменять в файле конфигурации Patroni параметр `name` :astonished::-1:
<img width="1390" height="300" alt="image" src="https://github.com/user-attachments/assets/8dccfa06-919d-4406-848f-7dd3a7539576" /><br>

Правлю параметр `name` в файле `sudo nano /etc/patroni/config.yml` на второй и третьей ноде, перезапускаю Patroni на второй ноде и проверяю состояние кластера, вторая нода добавилась в режиме создания реплики :+1:
<img width="1609" height="281" alt="image" src="https://github.com/user-attachments/assets/dee351fb-9e67-4005-afa7-ab6b8df05aa9" /><br>

На второй ноде начал заполняться каталог pgdata `/var/lib/postgresql/18/main`, размер базы данных на первой ноде ~ 6,5 Гб, там находятся базы инициализации утилит Pgbench и Sysbench. Примерно через пять минут вторая нода вышла в режим потоковой репликации `streaming`
<img width="1497" height="281" alt="image" src="https://github.com/user-attachments/assets/3a123b5a-9581-4945-885d-f0751c9b4c30" /><br>

Повторяю процедуру запуска Patroni на третьей ноде - перевожу Patroni в автозапуск, стартую сервис Patroni и правлю переменную PATH на третьей ноде, проверяю состояние кластера, третья нода добавилась в режиме создания реплики, примерно через пять минут третья нода также вышла в режим потоковой репликации `streaming` :+1:
<img width="1609" height="551" alt="image" src="https://github.com/user-attachments/assets/0063eb6a-3d49-4831-90b6-d6ecd5dfeb35" /><br>

Кластер Patroni успешно развёрнут

### 4. Установка и настройка HAProxy

На первом узле устанавливаю HAProxy
```
sudo apt-get install haproxy         # установка etcd-server
haproxy -v                           # проверка установки
```
<img width="1929" height="1331" alt="image" src="https://github.com/user-attachments/assets/f4d7ef59-ca02-4675-a1a6-5856b1164864" />
<img width="1929" height="221" alt="image" src="https://github.com/user-attachments/assets/81a9d5e9-c0ad-47ea-845d-8a807b91faf1" /><br>

Редактирую файл конфигурации HAProxy `sudo nano /etc/haproxy/haproxy.cfg`
```
global
    maxconn 100

defaults
    log    global
    mode    tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen stats
    mode http
    bind 192.168.0.50:7000
    stats enable
    stats uri /

listen primary
    bind *:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node0 192.168.0.50:5432 maxconn 100 check port 8008
    server node1 192.168.0.51:5432 maxconn 100 check port 8008
    server node2 192.168.0.52:5432 maxconn 100 check port 8008

listen standbys
    balance roundrobin
    bind *:5001
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node0 192.168.0.50:5432 maxconn 100 check port 8008
    server node1 192.168.0.51:5432 maxconn 100 check port 8008
    server node2 192.168.0.52:5432 maxconn 100 check port 8008
```

Стартую сервис HAProxy, определяю HAProxy как службу и проверяю статус HAProxy
```
sudo systemctl start haproxy.service
sudo systemctl enable haproxy.service
sudo systemctl status haproxy.service
```
<img width="2537" height="971" alt="image" src="https://github.com/user-attachments/assets/0d847f70-72a6-4605-9a70-ea4e144453ab" /><br>

HAProxy успешно стартован, в браузере по адресу http://192.168.0.50:7000/ получаю следующую статистику
<img width="1242" height="937" alt="image" src="https://github.com/user-attachments/assets/79d3baae-d7c7-44bf-8a62-ee1d3a63dd24" /><br>

В клиенте DBeaver 25.3.3 добавляю новое соединение к PostgreSQL по адресу HAProxy 192.168.0.50:5000 и проверяю на какой узел в данный момент ходит клиент - IP-адрес node1 192.168.0.51, т.к. лидером в данный момент является node1
<img width="1497" height="281" alt="image" src="https://github.com/user-attachments/assets/ae99e4eb-027d-4b88-98bb-cf0eb8b2c575" />
<img width="704" height="241" alt="image" src="https://github.com/user-attachments/assets/49c3ce4b-cc93-439f-9bde-1ad5825b1585" /><br>

Для проверки работы HAProxy останавливаю Patroni на узле 192.168.0.51 `systemctl stop patroni`, лидером становится node0 с IP 192.168.0.50, эта информация отображается в статистике и DBeaver 25.3.3 обращается к узлу 192.168.0.50
<img width="1497" height="281" alt="image" src="https://github.com/user-attachments/assets/1aac19c9-5cf3-4c5f-a8b8-4bc976aee8cf" />
<img width="1242" height="937" alt="image" src="https://github.com/user-attachments/assets/1641698e-adab-473e-bbb9-234f7a2d132a" />
<img width="695" height="245" alt="image" src="https://github.com/user-attachments/assets/287bb430-ba1c-44ed-8adf-9102cd70a8d8" />



### 5. Тестирование

:arrow_right: Для проверки работы репликации создаю на первой ноде-лидере дополнительную базу данных в PostgreSQL `test_replication` и проверяю её наличие на репликах
```
patronictl -c /etc/patroni/config.yml list
sudo -u postgres psql

postgres=# \l
postgres=# create database test_replication;
CREATE DATABASE
postgres=# \l
```

<img width="2201" height="1331" alt="image" src="https://github.com/user-attachments/assets/de594bd8-7a0c-4bbe-a443-9c6085abc6d5" /><br>

Новая база данных `test_replication` появилась на обеих нодах-репликах
<img width="2201" height="1001" alt="image" src="https://github.com/user-attachments/assets/83945208-af54-49c1-959f-4e822c9ec55c" />
<img width="2201" height="1001" alt="image" src="https://github.com/user-attachments/assets/a7f5f6f0-553b-45af-88d9-175976c9686a" /><br>

:arrow_right: Для проверки аварийного переключения имитирую выход из строя ноды-лидера путём выключения виртуальной машины `poweroff`
<img width="1497" height="521" alt="image" src="https://github.com/user-attachments/assets/e99b9461-2cf2-4c52-b466-92aa00de5101" /><br>

Лидером становится node2
<img width="1497" height="641" alt="image" src="https://github.com/user-attachments/assets/d16f97ea-e27f-4710-8b4e-0a707c1fa8c4" /><br>

Включаю отключённую виртуальную машину и отключенная нода возвращается в кластер в режиме режиме потоковой репликации
<img width="1497" height="881" alt="image" src="https://github.com/user-attachments/assets/0b76898c-39d8-4723-9a5d-16c1d684800c" /><br>




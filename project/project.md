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

На трёх узнал устанавливаю Patroni
```
sudo apt install python3.12-venv -- установка модуля для создания виртуальных окружений
sudo mkdir-p /opt/patroni -- создание каталога для Patroni
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
      - host replication replicator 0.0.0.0/0 scram-sha-256
      - host all all 0.0.0.0/0 scram-sha-256
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
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres
  parameters:
    unix_socket_directories: '.'
tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
```



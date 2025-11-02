# Домашнее задание 03
## Установка и настройка PostgreSQL

Для выполнения домашнего задания установлен сервер Ubuntu 24.04.3 на виртуальной машине, используя Oracle VirtualBox Manager. Подключаюсь к серверу через программу PuTTY посредством ввода своего пользователя с авторизацией с помощью ключа

<img width="1305" height="1061" alt="image" src="https://github.com/user-attachments/assets/0d40a3d3-4e12-4cd8-9673-f190177fc8d3" /><br>

Устанавливаю PostgreSQL 18 и набор пакетов дополнительных программ
```sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc```

Проверка статуса кластера PostgreSQL: `pg_lsclusters`

<img width="1305" height="941" alt="image" src="https://github.com/user-attachments/assets/aa296430-3964-4375-ab8c-0e8a61b5831e" /><br>

Захожу в БД PostgreSQL под пользователем postgres `sudo -u postgres psql` и создаю таблицу `ns_prgr` с небольшим содержимым
```
postgres=# CREATE TABLE ns_prgr (
        prgr int2 NOT NULL,
        nam_prgr bpchar(8) NULL,
        namprgrs bpchar(3) NULL,
        grup numeric NULL,
        tip int2 DEFAULT 0 NOT NULL,
        npp_graf numeric NULL,
        cod_color numeric NULL,
        CONSTRAINT xpkds007 PRIMARY KEY (prgr)
);
CREATE TABLE
postgres=# INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(0, 'Всего   ', 'Всг', NULL, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(1, 'Порожний', 'Пор', 0, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(2, 'Груженый', 'Гр.', 0, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(3, 'Перегруз', 'Пгр', 2, 0, NULL, NULL);
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
postgres=# \q
```

<img width="2281" height="791" alt="image" src="https://github.com/user-attachments/assets/0c7f59ac-4fad-4e77-9bc6-09fc3a0f73d1" /><br>

Проверяю статус кластера `pg_lsclusters` и останавливаю PostgreSQL `sudo systemctl stop postgresql@18-main`

<img width="1689" height="281" alt="image" src="https://github.com/user-attachments/assets/d0b7908c-3dd8-4f6a-a436-b5bf59e982e3" /><br>

Проверяю наличие дисков в системе `lsblk`
<img width="1033" height="311" alt="image" src="https://github.com/user-attachments/assets/aeba3b11-8892-4e7d-8cbf-3cf0f4aeb6fe" /><br>

Останавливаю виртуальную машину и с помощью панели управления VirtualBox добавляю новый виртуальный диск объёмом 10 Gb
<img width="777" height="454" alt="image" src="https://github.com/user-attachments/assets/0f40fb2c-571a-4761-b4cb-d1e0a8e6507b" /><br>

<img width="856" height="498" alt="image" src="https://github.com/user-attachments/assets/11d34c8a-178d-4769-8645-5fd8fd75bc4f" /><br>

<img width="1203" height="504" alt="image" src="https://github.com/user-attachments/assets/05811dda-2ac2-49fd-9401-adcc7203282a" /><br>

Запускаю виртуальную машину и проверяю наличие дисков в системе `lsblk`, в системе появился новый неразмеченный диск `sdb`
<img width="1033" height="341" alt="image" src="https://github.com/user-attachments/assets/3213fd2d-2206-4170-8991-baf349e71a6f" /><br>

Размечаю диск, добавляю раздел, форматирую его под файловую систему `ext4` и монтирую диск в систему
```
sudo parted /dev/sdb mklabel gpt -- создаю таблицу разделов для диска sdb
sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100% -- создаю раздел на диске sdb
sudo mkfs.ext4 -L new-data /dev/sdb1 - создаю файловую систему для раздела
sudo mkdir -p /mnt/new-data -- создаю директорию для монтирования файловой системы
sudo mount -o defaults /dev/sdb1 /mnt/new-data
df -h -- проверка примонтированных устройств в системе, новый раздел диска примонтирован по пути /mnt/new-data
```
<img width="1321" height="971" alt="image" src="https://github.com/user-attachments/assets/5ad07dbe-d276-486b-aa5c-21c18f5e31c7" /><br>



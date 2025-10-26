# Домашнее задание 02
## Установка и настройка PostgreSQL в контейнере Docker

Для выполнения домашнего задания создана виртуальная машина на платформе облачного сервиса Yandex Cloud с параметрами: операционная система Ubuntu 24.04 LTS, 2 ядра, 4 Гб памяти, SSD-диск, публичный ключ SSH для авторизации

<img width="917" height="999" alt="image" src="https://github.com/user-attachments/assets/822054c2-3373-4d57-80ec-3967f2f48db8" />
<img width="917" height="658" alt="image" src="https://github.com/user-attachments/assets/8645fd72-579f-43a5-8328-eb7414c4f671" />
<img width="919" height="540" alt="image" src="https://github.com/user-attachments/assets/e1519bb5-0e09-4b25-8125-570b06d9a60f" />
<img width="916" height="804" alt="image" src="https://github.com/user-attachments/assets/41da6f59-049d-4cb1-aaa1-1b6221a8edb7" /><br>

После создания виртуальной машины подключение к серверу через программу PuTTY посредством ввода своего пользователя с авторизацией с помощью ключа прошло успешно

<img width="1305" height="1121" alt="image" src="https://github.com/user-attachments/assets/a551b85e-defa-4c25-8e69-7732c65e5e5e" /><br>

Устанавливаю Docker Engine
```
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
```
<img width="2560" height="1380" alt="image" src="https://github.com/user-attachments/assets/8c73c6a4-06f9-4ddf-ac71-2dbcb0575085" />
<img width="2560" height="1380" alt="image" src="https://github.com/user-attachments/assets/9c325f81-20c6-46f3-a656-22e4853db966" /><br>

Создаю Docker-сеть
```
sudo docker network create pg-net
```
<img width="1241" height="131" alt="image" src="https://github.com/user-attachments/assets/117ce8f2-22ce-45eb-a2a9-27bcdc6c1404" /><br>

Создаю каталог ```/var/lib/postgres```
```
sudo mkdir -p -v /var/lib/postgres
```
<img width="1257" height="131" alt="image" src="https://github.com/user-attachments/assets/909860bb-9d16-4e6b-9ed5-09699282fb3c" /><br>

Разворачиваю контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
```
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
```
<img width="1257" height="761" alt="image" src="https://github.com/user-attachments/assets/1eebf932-5542-46cc-b082-15818d32821b" /><br>

Запускаю отдельный контейнер с клиентом PostgreSQL в общей сети с сервером БД и создаю новую базу данных ```CREATE DATABASE YandexCloud;```
```
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
```
<img width="1257" height="311" alt="image" src="https://github.com/user-attachments/assets/7dc71811-729c-4ced-bdb8-43cefd419b45" /><br>


Создаю таблицу с данными
```
CREATE TABLE ns_prgr (
	prgr int2 NOT NULL,
	nam_prgr bpchar(8) NULL
);
INSERT INTO ns_prgr
(prgr, nam_prgr)
VALUES(0, 'Всего');
INSERT INTO ns_prgr
(prgr, nam_prgr)
VALUES(1, 'Порожний');
INSERT INTO ns_prgr
(prgr, nam_prgr)
VALUES(2, 'Груженый');
commit;
```





<img width="1305" height="551" alt="image" src="https://github.com/user-attachments/assets/a714431e-5cb4-409e-9784-7091395c008c" />


<img width="872" height="697" alt="image" src="https://github.com/user-attachments/assets/8d3a92d5-365c-45ab-9eb4-8c55fed36aae" />

<img width="792" height="333" alt="image" src="https://github.com/user-attachments/assets/355d23e6-4347-4e23-a97b-eb128e591461" />




<img width="2549" height="581" alt="image" src="https://github.com/user-attachments/assets/4dc05e08-cef9-4538-92cc-0f06e2dfae2b" />





<img width="2537" height="1271" alt="image" src="https://github.com/user-attachments/assets/297a00ad-859c-43dc-ac21-ae8a39ce5e96" />

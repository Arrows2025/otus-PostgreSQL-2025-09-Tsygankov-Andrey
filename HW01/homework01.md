# Домашнее задание 01
## Работа с уровнями изоляции транзакции в PostgreSQL

Для выполнения домашнего задания установлен сервер Ubuntu 24.04.3 на виртуальной машине, используя Oracle VirtualBox Manager

<img width="1280" height="800" alt="VirtualBox_Ubuntu24Server_21_10_2025_15_43_56" src="https://github.com/user-attachments/assets/a0c5768d-8cff-46b6-ad11-0bc024c7ff78" /><br>

Дополнительно установлен пакет net-tools: `sudo apt install net-tools`

<img width="1280" height="800" alt="VirtualBox" src="https://github.com/user-attachments/assets/53161354-d398-4564-87a5-3f3f1f6ecafe" /><br>

В настройка сетевого адаптера виртуальной машины тип подключения NAT изменил на Сетевой мост, чтобы увидеть локальную сеть и сделать доступ к серверу Ubuntu по доступному IP-адресу

<img width="1414" height="795" alt="image" src="https://github.com/user-attachments/assets/29110acf-14c1-4dd4-8d47-7f090778d3e1" /><br>

Установка статического IP-адреса 192.168.0.50 через редактирование файла: `sudo nano /etc/netplan/01-netcfg.yaml` и применение новых сетевых настроек: `sudo netplan apply`  
Содержимое файла 01-netcfg.yaml:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.50/24
      gateway4: 192.168.0.1
      nameservers:
          addresses: [192.168.0.1, 8.8.8.8]
```

Динамический IP-адрес 192.168.0.131 изменён на статический IP-адрес 192.168.0.50

<img width="1280" height="800" alt="VirtualBox" src="https://github.com/user-attachments/assets/be8118e1-ff39-4db8-a4ae-673a8ecede01" /><br>

Настраиваю параметры (16-ый размер и белый цвет приоритетного шрифта) и подключение к серверу по IP-адресу 192.168.0.50 в программе PuTTY

<img width="602" height="543" alt="image" src="https://github.com/user-attachments/assets/2b4e3442-f897-42e3-9252-36be5e25f026" /><br>

Подключаюсь к серверу через программу PuTTY посредством ввода своего пользователя и пароля

<img width="1305" height="851" alt="image" src="https://github.com/user-attachments/assets/b03d867f-35a9-46a2-860f-ead27f8eebdd" /><br>

Создаю ключи командой `ssh-keygen -t ed25519` и записываю публичный ключ в авторизованные ключи сервера: `cat .ssh/arrows.key.pub >> .ssh/authorized_keys`

<img width="1369" height="731" alt="image" src="https://github.com/user-attachments/assets/6c64407b-fbb2-4b91-9901-b9ab5adeea1f" /><br>

На основе созданного приватного ключа с помощью программы PuTTY Key Generator сформирован новый приватный ключ формата PuTTY для авторизации без пароля и добавлен в настройки PuTTY

<img width="602" height="543" alt="image" src="https://github.com/user-attachments/assets/a8138ad3-0094-4464-bc20-f60023b6f3fa" /><br>

Подключаюсь к серверу через программу PuTTY посредством ввода своего пользователя с авторизацией с помощью ключа (первая сессия)

<img width="1305" height="851" alt="image" src="https://github.com/user-attachments/assets/483178d3-6348-4589-88f7-14ac9641c796" /><br>

Установка PostgreSQL и пакетов дополнительных программ: ```sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc```

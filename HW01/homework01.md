# Домашнее задание 01
## Работа с уровнями изоляции транзакции в PostgreSQL

Для выполнения домашнего задания установлен сервер Ubuntu 24.04.3 на виртуальной машине, используя Oracle VirtualBox Manager

<img width="1280" height="800" alt="VirtualBox_Ubuntu24Server_21_10_2025_15_43_56" src="https://github.com/user-attachments/assets/a0c5768d-8cff-46b6-ad11-0bc024c7ff78" /><br>

Дополнительно установлен пакет net-tools: `sudo apt install net-tools`

<img width="1280" height="800" alt="VirtualBox" src="https://github.com/user-attachments/assets/53161354-d398-4564-87a5-3f3f1f6ecafe" /><br>

В настройка сетевого адаптера виртуальной машины тип подключения NAT изменил на Сетевой мост, чтобы увидеть локальную сеть и сделать доступ к серверу Ubuntu по доступному IP-адресу

<img width="1414" height="795" alt="image" src="https://github.com/user-attachments/assets/29110acf-14c1-4dd4-8d47-7f090778d3e1" /><br>

Установка статического IP-адреса 192.168.0.50 через редактирование файла: `sudo nano /etc/netplan/01-netcfg.yaml` и применения новых сетевых настроек: `sudo netplan apply`  
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

Создаю ключи командой `ssh-keygen -t ed25519`

<img width="1305" height="731" alt="image" src="https://github.com/user-attachments/assets/b36d57f5-c2b4-445b-a23b-ce34807e1271" /><br>

и записываю публичный ключ в авторизованные ключи сервера: `cat arrows_key.pub > /home/arrows/.ssh/authorized_keys`

<img width="985" height="221" alt="image" src="https://github.com/user-attachments/assets/08e9965e-9223-45a5-86e8-b0fd8173b5c6" />

<img width="1033" height="911" alt="image" src="https://github.com/user-attachments/assets/f8b7ac76-0022-47c9-9e4f-14df677ca732" />

<img width="1305" height="761" alt="image" src="https://github.com/user-attachments/assets/7f7a4138-5f70-4fde-bc6d-290d7424baf7" />

<img width="1305" height="761" alt="image" src="https://github.com/user-attachments/assets/d0cb95e2-cfdb-4e3d-ae4d-121557b611f5" />

<img width="602" height="543" alt="image" src="https://github.com/user-attachments/assets/a8138ad3-0094-4464-bc20-f60023b6f3fa" />





<img width="1369" height="731" alt="image" src="https://github.com/user-attachments/assets/6c64407b-fbb2-4b91-9901-b9ab5adeea1f" />

На основе созданного приватного ключа с помощью программы PuTTY Key Generator сформирован приватный ключ для авторизации на сервер с помощью клиента PuTTY без пароля
<img width="1305" height="851" alt="image" src="https://github.com/user-attachments/assets/483178d3-6348-4589-88f7-14ac9641c796" />


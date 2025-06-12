University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://itmo-ict-faculty.github.io/network-programming)
Year: 2024/2025
Group: K3323
Author: Manomenov Ivan
Lab: Lab1
Date of create: 08.06.25
Date of finished: - 

# Ход работы
Виртулка Ubuntu запускалась при помощи Yandex Cloud (Ubuntu 24.04) - далее сервер

Виртуалка Mikrotik запускалась при помощи OracleVM (RouterOs 7.19.1) - далее клиент

В качестве VPN сервиса был выбран Wireguard

**Установка Wireguard на сервер:**
1. Выполняем обновление системы
    ```sudo apt update && sudo apt upgrade -y```
3.  скачиваем wireguard  ```sudo apt install wireguard -y```

3. Генерируем ключи:
```
wg genkey | tee ~/privatekey | wg pubkey > ~/publickey
sudo mv ~/privatekey /etc/wireguard/privatekey
sudo mv ~/publickey /etc/wireguard/publickey
sudo chmod 600 /etc/wireguard/privatekey
sudo chmod 644 /etc/wireguard/publickey
```
4. Настраиваем интерфейс WireGuard (wg0):
  ``` sudo nano /etc/wireguard/wg0.conf```
```[Interface]
PrivateKey = <тут мой приватный ключ, не покажу>
Address = 10.0.0.1/24
ListenPort = 51820

# Разрешим трафик
PostUp = ufw allow 51820/udp
PostDown = ufw delete allow 51820/udp
```
5. Включаем и запускаем WireGuard:
```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```
6. Проверяем работу:

   ![image](https://github.com/user-attachments/assets/8fc701b4-de77-4491-8897-8f92479af47e)

   
**Установка Wireguard на клиента:**
1. Создаем интерфейс: ```/interface wireguard add name=wg0  ```
2. Запоминаем ключи отсюда ```/interface wireguard print```
3. Добавляем IP-адрес на интерфейс:  ``` /ip address add address=10.0.0.2/24 interface=wg0```
4. Добавляем peer ```/interface/wireguard/peers/add interface=wg0 public-key="публик ключ моего сервера" allowed-address=10.10.10.1/32 endpoint-address=<получаем через curl ifconfig.me
на сервере> endpoint-port=51820```
5. На сервере добавляем peer в конфиг
```
[Peer]
PublicKey = <публик ключ с интерфеса клиента>
AllowedIPs = 10.10.10.2/32
```
**Проверка работы:**

Пингуем с клиента:

![image](https://github.com/user-attachments/assets/ceaa2d0d-4474-49c0-a953-b07b322b6beb)

Пингуем с сервера:

![image](https://github.com/user-attachments/assets/86b78305-0ae7-4475-b863-01f5a73e78dc)

```sudo wg show```

![image](https://github.com/user-attachments/assets/c85d8a23-e413-4f7e-9f63-bfa0a71f6147)








[]!(pics/1.png)

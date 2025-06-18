University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://itmo-ict-faculty.github.io/network-programming)

Year: 2024/2025

Group: K3323

Author: Manomenov Ivan

Lab: Lab1

Date of create: 08.06.25

# Ход работы
1. Проверяем связанность:
   
![image](https://github.com/user-attachments/assets/c37ed251-1611-4bfc-918c-1bc443f53fc0)
![image](https://github.com/user-attachments/assets/8500e857-9558-4447-b5b0-981643d95a11)
![image](https://github.com/user-attachments/assets/8a103294-4dca-442d-b014-47c74b721404)

2. Установка NetBox:
   
   2.1 На дополнительной вм устанавливаем git, docker.
   
   2.2 По интрукции поднимаем [NetBox](https://github.com/netbox-community/netbox-docker)
   
  ```
  git clone -b release https://github.com/netbox-community/netbox-docker.git
  cd netbox-docker
  docker compose pull
  docker compose up
  ```

   ![image](https://github.com/user-attachments/assets/5ce44f43-cf58-47fa-a84c-f1f1c8c1a348)

3. Создаем устройства через интерфейс NetBox

   ![image](https://github.com/user-attachments/assets/36ca3bcf-b081-4b61-8ce2-9f0b34b55e82)

4. Сбор данных из netbox:
   Playbook:
   
   ```
   - name: Save Netbox Data
     hosts: localhost
     gather_facts: no
     vars:
       api_endpoint: http://158.160.185.212:8000
       token: тут мой токен
     tasks:
   
       - name: Get netbox devices data
         set_fact:
           devices: "{{ query('netbox.netbox.nb_lookup', 'devices', api_endpoint=api_endpoint, token=token, validate_certs=false) }}"
   
       - name: Save data to file
         copy:
           content:
             - "{{ devices }}"
           dest: devices.json
   ```

   ![image](https://github.com/user-attachments/assets/1e4acf89-0c40-4c2f-b470-a463cbaf0455)

   Получили данные об устройствах в [devices.json](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab3/devices.json)

5. Настройка CHR на основе данных Netbox
   
   5.1 Настроим инвентарь, как в прошлой лабе [hosts.ini](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab2/hosts.ini)
   
   5.2 Заведем playbook [netbox_to_chr.yml](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab3/netbox_to_chr.yml)
   
   5.3 Запускаем

   ![image](https://github.com/user-attachments/assets/cd80e44f-ecb6-4072-8158-b564f531e46e)

   5.4 Проверяем, что поменялось имя и добавился IP:

   ![image](https://github.com/user-attachments/assets/837f31aa-06ed-44a9-9b49-0c37676dec4c)

7. Добавление серийного номера в Netboox:
   
   6.1 Создадим playbook [update_serial.yml](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab3/update_serial.yml)
   
   6.2 Запустим

   ![image](https://github.com/user-attachments/assets/066b9511-1cd8-42bc-92ee-0e02e4692f48)

   6.3 Проверим, что серийный номер поменялся

   ![image](https://github.com/user-attachments/assets/94702208-a510-4dcb-9392-612e880fd5b8)
   ![image](https://github.com/user-attachments/assets/b9d5c986-9bc8-4c21-ac30-c23fcf1880ce)

8. Схема:
   
![image](https://github.com/user-attachments/assets/c8431eac-96a8-4825-8a22-c43e5bb6b852)



   

   
   


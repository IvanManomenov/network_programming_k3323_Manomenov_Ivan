# Ход работы:
1. Анологично первой лабе создаем второго клиента, проверяем, что поинги проходят
   - с клиента

     ![image](https://github.com/user-attachments/assets/1a33a853-5d44-4c8c-82aa-872b5f797f4f)
     
   - с сервера
     
     ![image](https://github.com/user-attachments/assets/3ffa2538-be91-4127-9a08-a6bc776bee2c)
2. Создаем inventory-файл:
   ```
    [all]
    chr1 ansible_host=10.0.0.2
    chr2 ansible_host=10.0.0.3
    
    [all:vars]
    ansible_network_os=routeros
    ansible_connection=ansible.netcommon.network_cli
    ansible_user=admin

   ```
3. Создаем Playbook:
```
---
- name: Configure MikroTik CHR devices
  hosts: all
  gather_facts: false
  connection: community.routeros.api
  vars:
    router_ids:
      chr1: 1.1.1.1
      chr2: 2.2.2.2
    ntp_primary: "pool.ntp.org"
    ospf_area: "backbone"
    ospf_network: "10.0.0.0/24"
    current_password: "admin"
    new_password: "admin"

  tasks:
    - name: Change admin password
      community.routeros.command:
        commands:
          - "/user set admin password={{ new_password }}"
      vars:
        ansible_password: "{{ current_password }}"
      no_log: false

    - name: Configure NTP
      community.routeros.command:
        commands:
          - "/system ntp client set enabled=yes"
      vars:
        ansible_password: "{{ new_password }}"

    - name: Configure OSPF Instance
      community.routeros.command:
        commands:
          - "/routing ospf instance set [ find default=yes ] router-id={{ router_ids[inventory_hostname] }}"
          - "/interface bridge add name=lo"
          - "/ip address add address={{ router_ids[inventory_hostname] }}/32 interface=lo"
          - "/routing ospf instance add name=v2inst version=2 router-id={{ router_ids[inventory_hostname] }}"
          - "/routing ospf area add name=backbone_v2 area-id=0.0.0.0 instance=v2inst"
          - "/routing ospf interface-template add network=0.0.0.0/0 area=backbone_v2"
      vars:
        ansible_password: "{{ new_password }}"

    - name: Collect OSPF information
      community.routeros.command:
        commands:
          - "/routing ospf neighbor print"
          - "/routing ospf instance print"
          - "/routing ospf interface print"
      vars:
        ansible_password: "{{ new_password }}"
      register: ospf_data

    - name: Save OSPF output
      ansible.builtin.copy:
        content: "{{ ospf_data.stdout | to_nice_json }}"
        dest: "./ospf_info_{{ inventory_hostname }}.json"

    - name: Get full config
      community.routeros.command:
        commands:
          - "/export"
      vars:
        ansible_password: "{{ new_password }}"
      register: full_config

    - name: Save configuration
      ansible.builtin.copy:
        content: "{{ full_config.stdout }}"
        dest: "./config_backup_{{ inventory_hostname }}.rsc"
```
4. Запуск ```ansible-playbook -i inventory.yml collect_data.yml```

   ![image](https://github.com/user-attachments/assets/c8c3717e-139b-4cf5-b3d3-6da8ee2378c3)

5. Проверяем работу на Микротике:
   - подключение по ssh
     
   ![image](https://github.com/user-attachments/assets/05fa2c79-2a3e-4ad2-9cf2-828f3cd2ae33)

   - NTP Client
  
     ![image](https://github.com/user-attachments/assets/010ad513-21d4-4060-975d-c3797b719d58)
  
   - OSPF
  
     ![image](https://github.com/user-attachments/assets/37f84385-ee35-4eea-8519-3d09d59dfa3e)
  
   - Соседи

  ![image](https://github.com/user-attachments/assets/70825ba2-0bef-4094-9d80-1d1a8521f649)

   - Конфиги

   - ![image](https://github.com/user-attachments/assets/ac097c86-71b7-4c6e-8616-af10d977a4b6)





   

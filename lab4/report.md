# Ход работы:
Запускаем вм

   ![image](https://github.com/user-attachments/assets/b94c0b88-762a-4fcd-aae9-9e59af88327d)


Implementing Basic Forwarding:
1. С коробки ```make run``` естественно блять не работает. Нужно:
    - В utils/Makefile в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Во всех json в triandle-pogo в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Во всех json в pod-topo в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Проверяем командой ```grep -r --include='*' 'txtpb' .```, что этой ебанной хуйни не осталось
2. Запускаем ```make run``` и пингуем (ожидаемо не пингуется):

    ![image](https://github.com/user-attachments/assets/a32e53b8-05bc-4663-b42a-192628cd40ab)

3. Правим скрипт в TODO'шках:
4. Проверяем, что пингуется

   ![image](https://github.com/user-attachments/assets/a298f2f7-df42-40e5-b5ea-40b49113bac3)



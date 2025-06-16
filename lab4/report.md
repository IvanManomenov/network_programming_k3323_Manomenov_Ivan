# Ход работы:
Запускаем вм

   ![image](https://github.com/user-attachments/assets/b94c0b88-762a-4fcd-aae9-9e59af88327d)


**Implementing Basic Forwarding:**
1. С коробки ```make run``` естественно блять не работает. Нужно:
    - В utils/Makefile в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Во всех json в triandle-pogo в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Во всех json в pod-topo в строке с build/basic.p4.p4info.txtpb заменить .txtpb на .txt
    - Проверяем командой ```grep -r --include='*' 'txtpb' .```, что этой ебанной хуйни не осталось
2. Запускаем ```make run``` и пингуем (ожидаемо не пингуется):

    ![image](https://github.com/user-attachments/assets/a32e53b8-05bc-4663-b42a-192628cd40ab)

3. Правим TODO'шки в скрипте [basic.p4](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab4/basic.p4):
   ```
   parser MyParser(packet_in packet,
                   out headers hdr,
                   inout metadata meta,
                   inout standard_metadata_t standard_metadata) {
   
       state start {
           packet.extract(hdr.ethernet);
           transition select(hdr.ethernet.etherType) {
               TYPE_IPV4: parse_ipv4;
               default: accept;
           }
       }
   
       state parse_ipv4 {
           packet.extract(hdr.ipv4);
           transition accept;
       }
   }
   ```

   ```
   control MyIngress(inout headers hdr,
                     inout metadata meta,
                     inout standard_metadata_t standard_metadata) {
   
       action drop() {
           mark_to_drop(standard_metadata);
       }
   
       action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
           hdr.ethernet.dstAddr = dstAddr;
           hdr.ethernet.srcAddr = hdr.ethernet.srcAddr;  // Обычно сюда можно прописать MAC порта свитча
           standard_metadata.egress_spec = port;
           hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
       }
   
       table ipv4_lpm {
           key = {
               hdr.ipv4.dstAddr: lpm;
           }
           actions = {
               ipv4_forward;
               drop;
               NoAction;
           }
           size = 1024;
           default_action = NoAction();
       }
   
       apply {
           if (hdr.ipv4.isValid()) {
               ipv4_lpm.apply();
           }
       }
   }
   ```

   ```
   control MyDeparser(packet_out packet, in headers hdr) {
       apply {
           packet.emit(hdr.ethernet);
           packet.emit(hdr.ipv4);
       }
   }
   ```
5. Проверяем, что пингуется (алилуя блять)

   ![image](https://github.com/user-attachments/assets/a298f2f7-df42-40e5-b5ea-40b49113bac3)


**Implementing Basic Tunneling**:
1. Снова исправляем json'ы, чтобы заработало блять
2. Правим TODO'шки в скрипте [basic_tunnel.p4](https://github.com/IvanManomenov/network_programming_k3323_Manomenov_Ivan/blob/main/lab4/basic_tunnel.p4):
   ```
   parser MyParser(packet_in packet,
                   out headers hdr,
                   inout metadata meta,
                   inout standard_metadata_t standard_metadata) {
   
       state start {
           transition parse_ethernet;
       }
   
       state parse_ethernet {
           packet.extract(hdr.ethernet);
           transition select(hdr.ethernet.etherType) {
               TYPE_MYTUNNEL: parse_myTunnel;
               TYPE_IPV4: parse_ipv4;
               default: accept;
           }
       }
   
       state parse_myTunnel {
           packet.extract(hdr.myTunnel);
           transition select(hdr.myTunnel.proto_id) {
               TYPE_IPV4: parse_ipv4;
               default: accept;
           }
       }
   
       state parse_ipv4 {
           packet.extract(hdr.ipv4);
           transition accept;
       }
   
   }
   ```

   ```
   control MyIngress(inout headers hdr,
                     inout metadata meta,
                     inout standard_metadata_t standard_metadata) {
       action drop() {
           mark_to_drop(standard_metadata);
       }
   
       action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
           standard_metadata.egress_spec = port;
           hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
           hdr.ethernet.dstAddr = dstAddr;
           hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
       }
   
       table ipv4_lpm {
           key = {
               hdr.ipv4.dstAddr: lpm;
           }
           actions = {
               ipv4_forward;
               drop;
               NoAction;
           }
           size = 1024;
           default_action = drop();
       }
   
       action myTunnel_forward(egressSpec_t port) {
           standard_metadata.egress_spec = port;
       }
   
       table myTunnel_exact {
           key = {
               hdr.myTunnel.dst_id: exact;
           }
           actions = {
               myTunnel_forward;
               drop;
           }
           size = 1024;
           default_action = drop();
       }
   
       apply {
           if (hdr.ipv4.isValid() && !hdr.myTunnel.isValid()) {
               // Process only non-tunneled IPv4 packets
               ipv4_lpm.apply();
           }
   
           if (hdr.myTunnel.isValid()) {
               // process tunneled packets
               myTunnel_exact.apply();
           }
       }
   }
   ```

   ```
   control MyDeparser(packet_out packet, in headers hdr) {
       apply {
           packet.emit(hdr.ethernet);
           packet.emit(hdr.myTunnel);
           packet.emit(hdr.ipv4);
       }
   }
   ```
3. Проверяем работу без туннелирования
   - ```mininet> xterm h1 h2```
   - В h2 запускаем ```./recieve.py```

      ![image](https://github.com/user-attachments/assets/80a2b388-e86a-475f-86ce-685d29835839)

     
   - В h1 ```./send.py 10.0.2.2 "P4 is cool"```
     
     ![image](https://github.com/user-attachments/assets/5432fd1d-3533-4c25-809a-7d60a66f461e)

   - Ожидаемо: h2 получает сообщение, пакет содержит Ethernet + IPv4 + TCP + сообщение

4. C туннелированием
   - в h1 запускаем
     
     ![image](https://github.com/user-attachments/assets/a78fc9af-0a98-466b-b45a-8cf88b4368d7)

   - в h2 получаем сообщение

     ![image](https://github.com/user-attachments/assets/69707bd9-5d9d-49c7-8fdf-d6e785a6072a)



     


   




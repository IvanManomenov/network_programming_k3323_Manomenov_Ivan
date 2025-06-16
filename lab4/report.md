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

3. Правим скрипт в TODO'шках:
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



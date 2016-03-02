# Шукач SSID

Було б гарно знати як працювати з більш низьким рівнем роботи сокетів в Рубі й побачити наскільки він потужний. З мого досвіду, варто знати детально протокол з яким ви працюєте.
Я намагався досягнути цієї мети за допомогою гему `Packetfu` але він ще не все вміє що надає протокол. Отже я запустив мій Wireshark(з фільтром: `wlan.fc.type_subtype == 0x08`) та почfв перевіряти структуру безпровідної мережі щоб перевірити як можна зануритися глибше з сокетами в Рубі. в більш низький рівень аби мати більше можливостей ніж просто грати з TCP та  UDP.

Основними задачами було:
- Перейти на найнижчий рівень сокетів(Layer 2)
- Отримати всі до одного пакети незалежно від протоколу
- Отримати сирі пакети такими ж, як я отримував в wireshark 

Я пройшовся по всім методам, що описано нижче, а також заглянув в `/usr/include/linux/if_ether.h` який надихнув мене на ідею використання `ETH_P_ALL` та багато іншого. Додатково, команда `man socket` була дуже корисною для мене.

**Зверніть увагу: **Щоб робити наступне інтерфейс вашої мережевої карти має бути встановленим в режим моніторингу (using airmon-ng)

```bash
# Запустіть вашу мережеву карту в режимі мониторингу
airmon-ng start wls1

# перевірити список запущених інтерфейсів, що мониторяться
airmon-ng
```

```ruby
#!/usr/bin/env ruby
require 'socket'

# Відкриваємо Сокет на дцже низькому рівні, отримуємо сирі данні(Raw) для кожного пакету(ETH_P_ALL)
socket = Socket.new(Socket::PF_PACKET, Socket::SOCK_RAW, 0x03_00)

puts "\n\n"
puts "       BSSID       |       SSID        "  
puts "-------------------*-------------------"
while true
  # Захоплюємо мережу(wire) і переводемо її в hex і робимо її масивом
  packet = socket.recvfrom(2048)[0].unpack('H*').join.scan(/../)
  #
  # Паттерн Beacon Packet:
  # 1- IEEE 802.11 Beacon фрейм починається з 0x08000000h, завжди!
  # 2- значення Beacon фрейму знаходиться на байтах від 10го до 13го
  # 3- кількість байтів перед значеням SSID встановлено в 62 байт
  # 4- 62й байт це розмір SSID за яким іде сама строка SSID 
  # 5- Передавач(BSSID) або AP MAC адреса знаходиться між 34м та 39м байтами 
  #
  if packet.size >= 62 && packet[9..12].join == "08000000"   # Переконуємося зо маємо Beacon фрейм
    ssid_length = packet[61].hex - 1                         # Отримуємо розмір SSID
    ssid  = [packet[62..(62 + ssid_length)].join].pack('H*') # Отримуємо SSID 
    bssid = packet[34..39].join(':').upcase                  # Отримуємо BSSID
    
    puts " #{bssid}" + "    " + "#{ssid}"
  end
  
end
```


<br>
**Література** - *дуже корисна!*
- [raw_socket.rb](https://gist.github.com/k-sone/8036832#file-raw_sock-rb)
- [wifi_sniffer.rb](https://gist.github.com/amejiarosario/5420854)
- [packetter.rb](https://github.com/lrks/packetter/blob/master/ruby/packetter.rb)
- [Another git](https://gist.github.com/sam113101/aad031bcc50746956a29)
- [Programming Ruby1.9](http://media.pragprog.com/titles/ruby3/app_socket.pdf)
- [Rubydocs - class Socket](http://docs.ruby-lang.org/en/2.3.0/Socket.html)
- [Linux Kernel Networking – advanced topics (5)](http://www.haifux.org/lectures/217/netLec5.pdf)
- [PF_PACKET Protocol Family](http://curioushq.blogspot.com/2011/05/pfpacket-protocol-family.html)
- [Ruby Raw Socket for Windows](http://curioushq.blogspot.com/2011/05/ruby-raw-socket-for-windows.html)

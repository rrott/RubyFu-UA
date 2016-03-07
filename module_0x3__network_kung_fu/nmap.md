# Nmap

```
gem install ruby-nmap ronin-scanners
```
Як тільки ви зрозумієте як користуватися nmap-ом і як він працює, ви зрозумієте наскільки проста в використанні ця бібліотека. Ми можемо робити все що вміє справжній nmap.


### Сканування
гем Ruby-nmap це рубі інтерфейс до програми nmap - порт сканру, що надає такий функціонал:

* рубі інтерфейс для запуску nmap
* парсер для розбору XML файлу результату роботи nmap.

Давайте подивимося, як він працює

```ruby
require 'nmap'
scan = Nmap::Program.scan(:targets => '192.168.0.15', :verbose => true)
```
### Скунування SYN

```ruby
require 'nmap/program'

Nmap::Program.scan do |nmap|
  nmap.syn_scan = true
  nmap.service_scan = true
  nmap.os_fingerprint = true
  nmap.xml = 'scan.xml'
  nmap.verbose = true

  nmap.ports = [20,21,22,23,25,80,110,443,512,522,8080,1080,4444,3389]
  nmap.targets = '192.168.1.*'
end
```

Кодна з опцій типу  `nmap.syn_scan` чи `nmap.xml`розглядається як *Задача*  [Документація](http://www.rubydoc.info/gems/ruby-nmap/frames "Official doc") має список [опцій](http://www.rubydoc.info/gems/ruby-nmap/Nmap/Task) що підтримуються цією бібліотекою.


### Повне, всебічне сканування

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'nmap/program'

Nmap::Program.scan do |nmap|

  # Ціль
  nmap.targets = '192.168.0.1'

  # Дебаг та режим виводу інформації
  nmap.verbose = true
  nmap.show_reason = true

  # Методи сканування портів:
  nmap.syn_scan = true          # можете використовувати nmap.all like -A в nmap

  # Визначення сервісів та їх версій:
  nmap.service_scan = true
  nmap.os_fingerprint = true
  nmap.version_all = true

  # Сканування скриптом
  nmap.script = "all"

  nmap.all_ports                # nmap.ports = (0..65535).to_a

  # Firewall/IDS обхіж та спуфінг:
  nmap.decoys = ["google.com","yahoo.com","hotmail.com","facebook.com"]
  nmap.spoof_mac = "00:11:22:33:44:55"
  # Час та швидкодія
  nmap.min_parallelism = 30
  nmap.max_parallelism = 130

  # Вивід результатів скану
  nmap.output_all = 'rubyfu_scan'

end
```

### Розбір файлу XML з рузультатом роботи nmap
Я запустив 'агресивний' скан сайту `scanme.nmap.org`
```
nmap -n -v -A scanme.nmap.org -oX scanme.nmap.org.xml
```

Далі буде код з офіційного сайту проекту (https://github.com/sophsec/ruby-nmap)

```ruby
require 'nmap/xml'

Nmap::XML.new(ARGV[0]) do |xml|
  xml.each_host do |host|
    puts "[#{host.ip}]"
    # Print: Port/Protocol      port_status      service_name
    host.each_port do |port|
      puts "  #{port.number}/#{port.protocol}\t#{port.state}\t#{port.service}"
    end
  end
end
```

Повертає
```
[45.33.32.156]
  22/tcp        open    ssh
  80/tcp        open    http
  9929/tcp      open    nping-echo
```



https://github.com/ronin-ruby/ronin-scanners






<br><br><br>
---














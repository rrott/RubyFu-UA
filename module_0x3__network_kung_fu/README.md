# Модуль 0x3 |Мережеве Кунг-Фу


## Робота за IP адресами
В програмуванні для мереж ми завжди працюємо з IP адресами і ось деякі приклади:
- Обчислення мережевих префіксів для IP адрес із IP адреси та маски підмережі.
- Обчислення хостової(host) частини IP адреси із IP адреси та маски підмережі.
- Підрахунок кількості хостів в підмережі.
- Визначення чи належить якась IP адреса до підмережі чи ні.
- Конвертація маски підмережі з крапкової десятичної анотації в ціле число.

В Рубі є клас IPAddr для простих операція з IP адресами що дозволяє виконувати всі вищенаведені задачі.

```ruby
require 'ipaddr'
ip = IPAddr.new("192.34.56.54/24")
```

###  
### Обчислення мережевих префіксів для IP адрес із IP адреси та маски підмережі.

Простий виклик методу mask поверне нам мережевий префікс для IP адреси і це просто побітовоа маска IP-адреси разом з маскою підмережі.

```ruby
require 'ipaddr'
ip = IPAddr.new(ARGV[0])
network_prefix = ip.mask(ARGV[1])
puts network_prefix
```

```
ruby ip_example.rb 192.168.5.130 24
# Returns 
192.168.5.0
```

### Обчислення хостової(host) частини IP адреси із IP адреси та маски підмережі.

обчислення хостової(host) частини IP адреси це не така проста задача як в попередньому випадку. Нам спочатку треба обчислити доповнення маски підмережі.

Subnet(24) : 11111111.11111111.11111111.00000000

neg_subnet(24) : 00000000.00000000.00000000.11111111

Ми використали заперечення(~) та метод mask аби вирахувати доповнення маски підмережі і потім просто виконали побітове AND між IP та доповненням підмережі.

```ruby
require 'ipaddr'
ip = IPAddr.new(ARGV[0])
neg_subnet = ~(IPAddr.new("255.255.255.255").mask(ARGV[1]))
host = ip & neg_subnet
puts host
```

Запустіть:
```
ruby ip_example.rb 192.168.5.130 24
# Returns 
0.0.0.130
```

### Підрахунок кількості хостів в підмережі.
Ми використаємо метод to_range аби знайти діапазон всіх IP адрес а потім підрахуємо їх кількість в діапазоні. Ми зменшуємо це значення на 2 елементи щоб виключити IP адресу шлюзу та широкомовного(broadcast) IP.


```ruby
require 'ipaddr'
ip=IPAddr.new("0.0.0.0/#{ARGV[0]}")
puts ip.to_range.count-2
```
Запустіть:
```
ruby ip_example.rb 24
254
```

### Визначення чи належить якась IP адреса до підмережі чи ні.

`===` - це аліас методу include? який повертає true якщо IP адреса належить діапазону, або ж false, якщо - ні.


```ruby
require 'ipaddr'
net=IPAddr.new("#{ARG[0]}/#{ARGV[1]}")
puts net === ARGV[2]
```
Запустіть:
```
ruby ip_example.rb 192.168.5.128 24 192.168.5.93
true
```

```
ruby ip_example.rb 192.168.5.128 24 192.168.6.93
false
```

### Конвертація маски підмережі з крапкової десятеричної анотації в ціле число.
Ми взяли маску підмережі як IP адресу та перетворили її в ціле число використовуючи метод `to_i`, потім використали `to_s(2)` аби перетворити  ціле число в бінарну форму. Маючи бінарну форму ми підрахували кількість одиниць за допомогою методу `count("1")`.


```ruby
require 'ipaddr'
subnet_mask = IPAddr.new(ARGV[0])
puts subnet_mask.to_i.to_s(2).count("1").to_s
```

Запустіть:
```
ruby ip_example.rb 255.255.255.0
24
```


### Конвертація IP адреси в інші формати

#### З Десятичного IP в крапкову анотацію

```ruby
require 'ipaddr'
IPAddr.new(3232236159, Socket::AF_INET).to_s
```
або:

```ruby
[3232236159].pack('N').unpack('C4').join('.')
```

#### З крапкової анатації IP в десятичну.

```ruby
require 'ipaddr'
IPAddr.new('192.168.2.127').to_i
```

Цю частину було майже повністю процитовано із статті [Робота з IP адресами в Рубі][1]


## Геолокація за IP 
Вам, можливо буде потрібно дізнатися більше інформації про розташування серверу за його IP адресою для подальших атак чи з будь якої іншої причини.

### GeoIP

The special thing about geoip lib is that it's an API for offline database you download from [www.maxmind.com](http://www.maxmind.com). There are few free databases from MaxMind whoever you can have a subscription database version though. 

- Download one of the free GeoLite country, city or ASN databases
    - [GeoLiteCountry](geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz)
    - [GeoLiteCity](geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz)
    - [GeoIPASNum](geolite.maxmind.com/download/geoip/database/asnum/GeoIPASNum.dat.gz)


- Install geoip gem
```
gem install geoip
```

- Usage

```ruby
#!/usr/bin/env ruby

ip = ARGV[0]
geoip = GeoIP.new('GeoLiteCity.dat')
geoinfo = geoip.country(ip).to_hash

puts "IP address:\t"   + geoinfo[:ip]
puts "Country:\t"      + geoinfo[:country_name]
puts "Country code:\t" + geoinfo[:country_code2]
puts "City name:\t"    + geoinfo[:city_name]
puts "Latitude:\t"     + geoinfo[:latitude]
puts "Longitude:\t"    + geoinfo[:longitude]
puts "Time zone:\t"    + geoinfo[:timezone]
```

```
-> ruby ip2location.rb 108.168.255.243

IP address:     108.168.255.243
Country:        United States
Country code:   US
City name:      Dallas
Latitude:       32.9299
Longitude:      -96.8353
Time zone:      America/Chicago
```



<br><br><br>
---
[1]: http://www.brownfort.com/2014/09/ip-operations-ruby/

- [RubyDoc | IPAddr](http://ruby-doc.org/stdlib-1.9.3/libdoc/ipaddr/rdoc/IPAddr.html)
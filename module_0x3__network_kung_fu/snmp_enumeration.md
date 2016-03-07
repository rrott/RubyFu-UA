# Отримання даних SNMP
- Встановіть [ruby-snmp][1]
```
gem install snmp
```

## Get Запит
Невірно налаштований SNMP сервіс може дати нападаючим дуже багато інформації. Давайте подивимося як отримати деяку інформацію з сервера.

```ruby
# KING SABRI | @KINGSABRI
require 'snmp'

# Підключення до SNMP серверу
manager = SNMP::Manager.new(:host => '192.168.0.17')

# Базова інформація
puts "SNMP Version: " + manager.config[:version]
puts "Community: " + manager.config[:community]
puts "Write Community: " + manager.config[:WriteCommunity]


# Отримуємо ім'я хосту, контактну інформацію, мвсце знаходження
hostname = manager.get("sysName.0").each_varbind.map {|vb| vb.value.to_s}       # manager.get("sysName.0").varbind_list[0]
contact  = manager.get("sysContact.0").each_varbind.map {|vb| vb.value.to_s}    # manager.get("sysContact.0").varbind_list[0]
location = manager.get("sysLocation.0").each_varbind.map {|vb| vb.value.to_s}   # manager.get("sysLocation.0").varbind_list[0]

# Це надасть списоа OIDs
response = manager.get(["sysName.0", "sysContact.0", "sysLocation.0"])
response.each_varbind do |vb|
    puts vb.value.to_s
end

```

> Зверніть увагу: OID імена чутливі до регістру.


## Set Запит
Іноді може пощастити отримати строку управління SNMP і в цей момент ми можемо іже робити й приміняти зміни но всій системі, раутері, свічах, та інше.

```ruby
require 'snmp'
include SNMP

# Підключання до SNMP серверу
manager = SNMP::Manager.new(:host => '192.168.0.17')
# Налаштужмо наш запит в OID
varbind = VarBind.new("1.3.6.1.2.1.1.5.0", OctetString.new("Your System Got Hacked"))
# надсилаємо наші настройки до серверу
manager.set(varbind)
# Перевіряємо чи були застосованими зміни. 
manager.get("sysName.0").each_varbind.map {|vb| vb.value.to_s}
manager.close
```



<br><br><br>
---
[1]: https://github.com/hallidave/ruby-snmp/
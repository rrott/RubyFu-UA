# Сокети в Рубі

## Коротке введення 
### Ієрархія класів для роботи з Сокетами в Рубі

Аби зрозуміти ієрархію сокетів в Рубі, ось вам просте дерево, яке це робить:

```
IO                              # Основа вводу/виводу в Рубі
└── BasicSocket                 # Абстрактний базовий клад для всіх класів сокетів
    ├── IPSocket                # Супер клас для протоколів що використовують Інтернет Протокол(AF_INET)
    │   ├── TCPSocket           # Клас для сокетів Transmission Control Protocol (TCP)
    │   │   ├── SOCKSSocket     # Helper клас для побудови серверних аплікащій з сокетом TC
    │   │   └── TCPServer       # Helper клас для побудови серверів з сокетом TCP
    │   └── UDPSocket           # Клас для сокету User Datagram Protocol (UDP)
    ├── Socket                  # Базовий клас сокету що мімкрує під BSD Sockets API. Він надає більш специффічного для ОС функціоналу сокетів.
    └── UNIXSocket              # Клас що надає IPC використовуючи UNIX domain protocol (AF_UNIX)
        └── UNIXServer          # Helper клас для побудови серверів на сокет з протоколом UNIX domain
```
Я докладно розгляну деякі з`Socket::Constants` оскільки я не знайшов літературу на яку можна було б посилатися окрім [Programming Ruby1.9 *The Pragmatic Programmers' Guide*](http://media.pragprog.com/titles/ruby3/app_socket.pdf); В інакшому випадку вам було б потрібно постійно запускати команду`ri Socket::Constants` з командної строки, що в принципі є гарним способом отримання опису будь-якої константи.


### Socket Types
- SOCK_RAW
- SOCK_PACKET
- SOCK_STREAM
- SOCK_DRAM
- SOCK_RDM
- SOCK_SEQPACKET

### Address Families(Socket Domains)
- AF_APPLETALK
- AF_ATM
- AF_AX25
- AF_CCITT
- AF_CHAOS
- AF_CNT
- AF_COIP
- AF_DATAKIT
- AF_DEC
- AF_DLI
- AF_E164
- AF_ECMA
- AF_HYLINK
- AF_IMPLINK
- AF_INET(IPv4)  
- AF_INET6(IPv6)
- AF_IPX
- AF_ISDN
- AF_ISO
- AF_LAT
- AF_LINK
- AF_LOCAL(UNIX)
- AF_MAX
- AF_NATM
- AF_NDRV
- AF_NETBIOS
- AF_NETGRAPH
- AF_NS
- AF_OSI
- AF_PACKET
- AF_PPP
- AF_PUP
- AF_ROUTE
- AF_SIP
- AF_SNA
- AF_SYSTEM
- AF_UNIX
- AF_UNSPEC

### Socket Protocol
- IPPROTO_SCTP
- IPPROTO_TCP
- IPPROTO_UDP

### Protocol Families
- PF_APPLETALK
- PF_ATM
- PF_AX25
- PF_CCITT
- PF_CHAOS
- PF_CNT
- PF_COIP
- PF_DATAKIT
- PF_DEC
- PF_DLI
- PF_ECMA
- PF_HYLINK
- PF_IMPLINK
- PF_INET
- PF_INET6
- PF_IPX
- PF_ISDN
- PF_ISO
- PF_KEY
- PF_LAT
- PF_LINK
- PF_LOCAL
- PF_MAX
- PF_NATM
- PF_NDRV
- PF_NETBIOS
- PF_NETGRAPH
- PF_NS
- PF_OSI
- PF_PACKET
- PF_PIP
- PF_PPP
- PF_PUP
- PF_ROUTE
- PF_RTIP
- PF_SIP
- PF_SNA
- PF_SYSTEM
- PF_UNIX
- PF_UNSPEC
- PF_XTP

### Socket options
- SO_ACCEPTCONN
- SO_ACCEPTFILTER
- SO_ALLZONES
- SO_ATTACH_FILTER
- SO_BINDTODEVICE
- SO_BINTIME
- SO_BROADCAST
- SO_DEBUG
- SO_DETACH_FILTER
- SO_DONTROUTE
- SO_DONTTRUNC
- SO_ERROR
- SO_KEEPALIVE
- SO_LINGER
- SO_MAC_EXEMPT
- SO_NKE
- SO_NOSIGPIPE
- SO_NO_CHECK
- SO_NREAD
- SO_OOBINLINE
- SO_PASSCRED
- SO_PEERCRED
- SO_PEERNAME
- SO_PRIORITY
- SO_RCVBUF
- SO_RCVLOWAT
- SO_RCVTIMEO
- SO_RECVUCRED
- SO_REUSEADDR
- SO_REUSEPORT
- SO_SECURITY_AUTHENTICATION
- SO_SECURITY_ENCRYPTION_NETWORK
- SO_SECURITY_ENCRYPTION_TRANSPORT
- SO_SNDBUF
- SO_SNDLOWAT
- SO_SNDTIMEO
- SO_TIMESTAMP
- SO_TIMESTAMPNS
- SO_TYPE
- SO_USELOOPBACK
- SO_WANTMORE
- SO_WANTOOBFLAG

## Створення Темплати Сокету

```ruby
Socket.new(domain, socktype [, protocol])
```

**domain(Address/Protocol Families):** like AF_INET, PF_PACKET, etc

**socktype:** like SOCK_RAW, SOCK_STREAM

**protocol: ** by default, it's `0`m it should be a protocol defined (we'll manipulate that later)


## TCP Socket


**Server/Client life cycle **
```
            Client        Server
              |             |                  
   socket     +             +      socket
              |             |
   connect    +--------,    +      bind
              |         |   |
   write ,--> +------,  |   +      listen
         |    |      |  |   |
   read  `----+ <--, |  `-> +      accept
              |    | |      |
   close      +--, | `----> + <--, read <--,
                 | |        |    |         |
                 | `--------+----' write   ٨
                 |                         |
                 `----->------>------->----`
```

### Використання сокетів

#### Отримати список всіх IP адрес

```ruby
require 'socket'
Socket.ip_address_list
```

#### Отримати Hostname
```ruby
Socket.gethostname
```



### TCP Сервер

Тут ми представимо звичайний TCP сервер. Цей сервер буде отримувати підключення з клієнта та відправляти повідомлення до нього в разі підключення, потім закривати клієнтське і серверне підключення.

```ruby
require 'socket'

server = TCPServer.new('0.0.0.0', 9911) # Сервер, слухая всі інтерфейси на порту 9911
client = server.accept                  # Очікує підключення клієнту
rhost  = client.peeraddr.last           # peeraddr повертає інформацію про віддалений клієнт [address_family, port, hostname, numeric_address(ip)]
client.puts "Hi TCP Client! #{rhost}"   # Відправдяє повідомлення клієнтові після підключення
client.gets.chomp                       # Читає вхідні повідомлення від клієнту
client.close                            # Закриває підключення клієнта
server.close                            # Закриває підключення TCP сервера.
```

### TCP Клієнт 

```ruby
require 'socket'

client = TCPSocket.new('127.0.0.1', 9911)   # Клієнт підключається до сервера на порту 9911
rhost  = client.peeraddr.last               # Отримує IP адрусу сервера 
client.gets.chomp
client.puts "Hi, TCP Server #{rhost}"
client.close
```
Ви можете виставити таймаут чи інтервал для підключення якщо відповідь від сервера може прийти з затримкою.

```ruby
timeval = [3, 0].pack("l_2")        # Промізок часу в 3 секунди 
client.setsockopt Socket::SOL_SOCKET, Socket::SO_RCVTIMEO, timeval      # Налаштовуємо сокет отримувати інтервал 
client.setsockopt Socket::SOL_SOCKET, Socket::SO_SNDTIMEO, timeval      # Налаштовуємо сокет на відправку інтервалу
client.getsockopt(Socket::SOL_SOCKET, Socket::SO_RCVTIMEO).inspect      # Опціонально, перевірити чи опцію сокета було встановлено
client.getsockopt(Socket::SOL_SOCKET, Socket::SO_SNDTIMEO).inspect      # Опціонально, перевірити чи опцію сокета було встановлено
```

Є кілька альтернатив для методів `puts` та `gets`. Ви можете бачити різницю та їх класи використовуючи метод method в командному інтерфейсі Pry.

```ruby
>> s = TCPSocket.new('0.0.0.0', 9911)
=> #<TCPSocket:fd 11>
>> s.method :puts
=> #<Method: TCPSocket(IO)#puts>
>> s.method :write
=> #<Method: TCPSocket(IO)#write>
>> s.method :send
=> #<Method: TCPSocket(BasicSocket)#send>
```

```ruby
>> s = TCPSocket.new('0.0.0.0', 9911)
=> #<TCPSocket:fd 11>
>> s.method :gets
=> #<Method: TCPSocket(IO)#gets>
>> s.method :read
=> #<Method: TCPSocket(IO)#read>
>> s.method :recv
=> #<Method: TCPSocket(BasicSocket)#recv>
```


## UDP Socket

### UDP Сервер
```ruby
require 'socket'

server = UDPSocket.new                                  # Запуск UDP сокету
server.bind('0.0.0.0', 9911)                            # слухаємо всі інтерфейси на порту  9911
mesg, addr = server.recvfrom(1024)                      # Отримуємо 1024 байт повідомлення та IP адресу відправника
server puts "Hi, UDP Client #{addr}", addr[3], addr[1]  # Відправка повідомлення клієнтові 
server.recv(1024)                                       # Отримання 1024 байт повідомлення
```

### UDP Клієнт
```ruby
require 'socket'
client = UDPSocket.new
client.connect('localhost', 9911)       # Підключення до серверу на порту 9911
client.puts "Hi, UDP Server!", 0        # Відправка повідомлення 
server.recv(1024)                       # Отримання 1024 байт повідомлення від сервера
```

Тут також є альтернативи для відправлення й отримання повідомлень. Ви можете знайти це в  [RubyDoc](http://ruby-doc.org/stdlib-2.0.0/libdoc/socket/rdoc/UDPSocket.html).




## GServer
GServer standard library implements a generic server, featuring thread pool management, simple logging, and multi-server management. Any kind of application-level server can be implemented using this class:
- It accepts multiple simultaneous connections from clients
- Several services (i.e. one service per TCP port)
    - can be run simultaneously, 
    - can be stopped at any time through the class method `GServer.stop(port)`
- All the threading issues are handled
- All events are optionally logged


- Very basic GServer

```ruby
require 'gserver'

class HelloServer < GServer                 # Inherit GServer class
  def serve(io)
    io.puts("What's your name?")
    line = io.gets.chomp
    io.puts "Hi, #{line}!"
    self.stop if io.gets =~ /shutdown/      # Stop the server if you get shutdown string
  end
end

server = HelloServer.new(1234, '0.0.0.0')   # Start the server on port 1234
server.audit = true     # Enable logging
server.start            # Start the service 
server.join
```












<br><br><br>
---
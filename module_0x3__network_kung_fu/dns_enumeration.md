# Розкриття даних DNS

```
gem install net-dns
```

В Рубі скрипті:

```ruby
require 'net/dns'
```

### Обробка DNS інформації 
Посібник з використання вказує використовувати бібліотеку так:
```ruby
require 'net/dns'
resolver = Net::DNS::Resolver.start("google.com")
```
І отримуємо
`
;; Answer received from 127.0.1.1:53 (260 bytes)
;;
;; HEADER SECTION
;; id = 36568
;; qr = 1       opCode: QUERY   aa = 0  tc = 0  rd = 1
;; ra = 1       ad = 0  cd = 0  rcode = NoError
;; qdCount = 1  anCount = 6     nsCount = 4     arCount = 4

;; QUESTION SECTION (1 record):
;; google.com.                  IN      A

;; ANSWER SECTION (6 records):
google.com.             31      IN      A       64.233.183.102
google.com.             31      IN      A       64.233.183.113
google.com.             31      IN      A       64.233.183.100
google.com.             31      IN      A       64.233.183.139
google.com.             31      IN      A       64.233.183.101
google.com.             31      IN      A       64.233.183.138

;; AUTHORITY SECTION (4 records):
google.com.             152198  IN      NS      ns1.google.com.
google.com.             152198  IN      NS      ns3.google.com.
google.com.             152198  IN      NS      ns4.google.com.
google.com.             152198  IN      NS      ns2.google.com.

;; ADDITIONAL SECTION (4 records):
ns3.google.com.         152198  IN      A       216.239.36.10
ns4.google.com.         152198  IN      A       216.239.38.10
ns2.google.com.         152198  IN      A       216.239.34.10
ns1.google.com.         345090  IN      A       216.239.32.10
```
Як можете бачити з відбовіді вище, маємо 5 секцій

* ** Header section:** секція заголовку DNS
* **Question section:** DNS запит,
* **Answer section:** Масив даних з відповіддю з сервера (основуючись на типові запиту. Наприклад:. A, NS, MX , etc)
* **Authority section:** Список довірених серверів імен(nameserver)
* **Additional section:** Список серверів імен, які біло переглянуто

Оскільки все це об'єкти, ми можемо переглянути кожну секцію десь так:
```ruby
resolver.header
resolver.question
resolver.answer
resolver.authority
resolver.additional
```

#### Запис "A-record"

Оскільки запис *A* встановлено за замовченням, ми можемо отримати його ось так:
```ruby
resolver = Net::DNS::Resolver.start("google.com")
```
Або отримати точний результат в однин рядок:

```ruby
resolver = Net::DNS::Resolver.start("google.com").answer
```

Поверне список з усіма IP адресами, закріпленими за цим доменом

```
[google.com.             34      IN      A       74.125.239.35,
 google.com.             34      IN      A       74.125.239.39,
 google.com.             34      IN      A       74.125.239.33,
 google.com.             34      IN      A       74.125.239.34,
 google.com.             34      IN      A       74.125.239.36,
 google.com.             34      IN      A       74.125.239.32,
 google.com.             34      IN      A       74.125.239.46,
 google.com.             34      IN      A       74.125.239.40,
 google.com.             34      IN      A       74.125.239.38,
 google.com.             34      IN      A       74.125.239.37,
 google.com.             34      IN      A       74.125.239.41]
```

### MX запис

```ruby
mx = Net::DNS::Resolver.start("google.com", Net::DNS::MX).answer
```
returns an array
```
[google.com.             212     IN      MX      40 alt3.aspmx.l.google.com.,
 google.com.             212     IN      MX      30 alt2.aspmx.l.google.com.,
 google.com.             212     IN      MX      20 alt1.aspmx.l.google.com.,
 google.com.             212     IN      MX      50 alt4.aspmx.l.google.com.,
 google.com.             212     IN      MX      10 aspmx.l.google.com.]
```

### All lookup

```ruby
any = Net::DNS::Resolver.start("facebook.com", Net::DNS::ANY).answer
```
returns
```
[facebook.com.           385     IN      A       173.252.120.6,
 facebook.com.           85364   IN      TXT     ,
 facebook.com.           149133  IN      NS      b.ns.facebook.com.,
 facebook.com.           149133  IN      NS      a.ns.facebook.com.]
```

for list of types, please refer to the [gem docs](http://www.rubydoc.info/gems/net-dns/Net/DNS/RR/Types)


### Reverse DNS lookup

```ruby
resolver = Net::DNS::Resolver.new
query = resolver.query("69.171.239.12", Net::DNS::PTR)
```
If you want to specify the nameserver(s) to use, it support an array of nameserver
```ruby
resolver = Net::DNS::Resolver.new(:nameserver => "8.8.8.8")
```
or update the object
```ruby
resolver = Net::DNS::Resolver.new
resolver.nameservers = ["8.8.4.4" , "8.8.8.8"]
```


<br><br><br>
---
http://searchsignals.com/tutorials/reverse-dns-lookup/




# Видобування строк
Видобування строк є однією з основних задач, необхідних всім програмістам. Іноді це може навіть може стати надзвичайно важкою задачею, тому що немає простого представлення строк, з яких можна видобути інформацію. Ось деякі приклади:

## Знаходження мережевої інформації

### Видобування MAC адреси зі строки
Нам потрібно знайти всі MAC адреси в довільній строчці з даними:
```ruby
mac = "ads fs:ad fa:fs:fe: Wind00-0C-29-38-1D-61ows 1100:50:7F:E6:96:20dsfsad fas fa1 3c:77:e6:68:66:e9 f2"
```

**Використовуючи регулярні вирази:**

регулярний вираз має підтримувати і формат mac-адреси Windows так і формат адреси в Linux

Давайте знайдемо нашу адресу:
```ruby
mac_regex = /(?:[0-9A-F][0-9A-F][:\-]){5}[0-9A-F][0-9A-F]/i
mac.scan mac_regex
```
Результат:
```
["00-0C-29-38-1D-61", "00:50:7F:E6:96:20", "3c:77:e6:68:66:e9"]
```

### Видобування IPv4 адрес зі строки
Нам треба знайти всі IPv4 адреси в довільній строчці.

```ruby
ip = "ads fs:ad fa:fs:fe: Wind10.0.4.5ows 11192.168.0.15dsfsad fas fa1 20.555.1.700 f2"
```

```
ipv4_regex = /(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)/
```
Давайте знайдемо наші IP адреси

```ruby
ip.scan ipv4_regex

```
Результат:
```
[["10", "0", "4", "5"], ["192", "168", "0", "15"]]
```

### Видобуток IPv6 адрес із строки
https://gist.github.com/cpetschnig/294476

http://snipplr.com/view/43003/regex--match-ipv6-address/

## Видобуток Web Strings
### Знаходження лінок в файлі
Уявімо у нас є наступна строчка:
```ruby
string = "text here http://foo1.example.org/bla1 and http://foo2.example.org/bla2 and here mailto:test@example.com and here also."
```
<br>
**Використовуючи регулярні вирази**
```ruby
string.scan(/https?:\/\/[\S]+/)
```

**Використовуючи стандартний модуль URI**
Поверне масив посилок:
```ruby
require 'uri'
URI.extract(string, ["http" , "https"])
```

### Видобуток лінок з веб-сторінки
Використовуючи вищенаведені трюки


```ruby
require 'net/http'
URI.extract(Net::HTTP.get(URI.parse("http://rubyfu.net")), ["http", "https"])
```
або використовуючи регулярні вирази:
```ruby
require 'net/http'
Net::HTTP.get(URI.parse("http://rubyfu.net")).scan(/https?:\/\/[\S]+/)
```

### Видобуток списку поштових адрес з веб-сторінки
```ruby
require 'net/http'
Net::HTTP.get(URI.parse("http://pastebin.com/khAmnhsZ")).scan(/\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}\b/i).uniq
```


### Видобуток текстових строк з HTML тегів 

Уявімо у нас є наступний шмат HTML коду й нам потрібно знайти лише текстову складову без HTML тегів.

```ruby

string = "<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>This is a Heading</h1>
<p>This is another <strong>contents</strong>.</p>

</body>
</html>"


puts string.gsub(/<.*?>/,'').strip

```

Результат
```
Page Title



This is a Heading
This is another contents.
```







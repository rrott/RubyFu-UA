# Перетворення

## Перетворення строки в шіснадцятирічний формат

Якщо нема необхідності показувати префікс, що вказує на формат, то можете просто зробити наступне:

```ruby
"Rubyfu".unpack("H*")
```

В іншому випадку використовуйте один з наступних методів:

Для одного символу
```ruby
'\x%02x' % "A".ord
```
**Зверніть увагу:** Набір символів ```*""``` це те саме що й ```.join```

```ruby
"ABCD".unpack('H*')[0].scan(/.{2}/).map {|h| '\x'+h }.join
```
або
```ruby
"ABCD".unpack('C*').map { |c| '\x%02x' % c }.join
```
або
```ruby
"ABCD".split("").map {|h| '\x'+h.unpack('H*')[0] }*""
```
або
```ruby
"ABCD".split("").map {|c|'\x' + c.ord.to_s(16)}.join
```
або
```ruby
"ABCD".split("").map {|c|'\x' + c.ord.to_s(16)}*""
```
або
```ruby
"ABCD".chars.map {|c| '\x' + c.ord.to_s(16)}*""
```
або
```ruby
"ABCD".each_char.map {|c| '\x'+(c.unpack('H*')[0])}.join
```
або ж нарешті
```ruby
"ABCD".chars.map {|c| '\x%x' % c.ord}.join
```
[Джерело: Ruby | Convert ASCII to HEX][1]


Результат:
```
\x41\x42\x43\x44
```



## Перетворення з шіснадцятирічного формату в Строку
```ruby
["41424344"].pack('H*')
```
Результат:
```
ABCD
```

**Примітка про hex(шістнадцятирічний формат):** Іноді вам може трапитись невидимі символи, особливо коли працюватимете з бінарними даними. В цьому випадку додайте **(**`# -*- coding: binary -*-`**)** на початку вашого файлу аби захистити ваш інтерпретатор від можливих проблем і помилок.


## Перетворення з шіснадцятирічного формату(return address - зворотня адреса) в формат Little-Endian
Little-Endian - це просто реверсивна строчка, записана ззаду на перед, наприклад запис "Rubyfu" як "ufybuR" що може біти здійсненим за допомогою методу `reverse` із стандартного класу `String`:
```ruby
"Rubyfu".reverse
```
Для експлуатації це не так просто як це здається оскільки ми працюємо з шістнадцятирічними значеннями в яких можуть бути як видимі так і невидимі(друковані й недруковані) символи.

Допустимо в нас є `0x77d6b141` який нам треба перетворити в формат Little-Endian аби надати CPU можливість правильно його прочитати.

Взагалі, це достатньо проста задача перетворення `0x77d6b141` в `\x41\xb1\xd6\x77` і це лише виклик однієї команди, але не в тому випадку, коли у вас є ланцюжок викликів команд. Для цього просто запакуйте строку в масив і викличте метод `pack` на ньому:

```ruby
[0x77d6b141].pack('V')
```

Іноді ви можете отримувати помилку через те що рядок не в Юнікоді(Unicode). Аби запобігти цій проблемі примусово встановіть кодування в UTF-8, але в більшості випадків ви не зустрінетесь з цією проблемою.

```ruby
[0x77d6b141].pack('V').force_encoding("UTF-8")
```

Якщо ви використовуєте ланцюжок викликів команд(ROP) то використання такого методу не зовсім зручно і красиво, тож користуйтеся першим методом і додавайте строку **(**`# -*- coding: binary -*-`**)** на початок вашого файлу зі скриптом,


## Кодування/Декодування строк в base-64
Покажемо як зробити це кількома методами:

**Кодування строки**
```ruby
["RubyFu"].pack('m0')
```
або 
```ruby
require 'base64'
Base64.encode64 "RubyFu"
```

**Декодування**
```ruby
"UnVieUZ1".unpack('m0')
```
або
```ruby
 require 'base64'
 Base64.decode64 "UnVieUZ1"
```
> **Підказка: **
> Метод unpack з класу String неймовірно зручний і корисний для перетворення даних, які можна прочитати як строку й перетворити назад в їх оригінальну форму. Аби прочитати більше про це, перейдіть до сторінки опису класу String [офіційної документації по Рубі](www.ruby-doc.org/core/classes/String.html).


## Кодування й декодування URL-строки
Кодування та декодування URL це знайома річ для багатьох людей, що користується браузерами. З хакерської точки зору нам це постійно буде в пригоді й найбільше для роботи з вразливостями на клієнтській стороні.

**Кодування строк**
```ruby
require 'uri'
puts URI.encode 'http://vulnerable.site/search.aspx?txt="><script>alert(/Rubyfu/.source)</script>'
```
**Декодування строк**
```ruby
require 'uri'
puts URI.decode "http://vulnerable.site/search.aspx?txt=%22%3E%3Cscript%3Ealert(/Rubyfu/.source)%3C/script%3E"
```
Звичайно ж ви можете кодувати й декодувати не лише URL строки а й будь які інші.

Вищенаведений спосіб кодуватиме лише нестандартні URL символи(такі як. `<>"{}`) але якщо вам потрібно закодувати строку повністю, то використовуйте `URI.encode_www_form_component`

```ruby
puts URI.encode_www_form_component 'http://vulnerable.site/search.aspx?txt="><script>alert(/Rubyfu/.source)</script>'
```

## Кожування та декодування HTML

**Кодування HTML**
```ruby
require 'cgi'
CGI.escapeHTML('"><script>alert("Rubyfu!")</script>')
```
Результат: 
```
&quot;&gt;&lt;script&gt;alert(&quot;Rubyfu!&quot;)&lt;/script&gt;
```

**Декодування HTML**
```ruby
require 'cgi'
CGI.unescapeHTML("&quot;&gt;&lt;script&gt;alert(&quot;Rubyfu!&quot;)&lt;/script&gt;")
```
Результат: 
```
"><script>alert("Rubyfu!")</script>
```

## Кодування та декодування SAML строк


**Декодування SAML**

```ruby
# SAML Request 
saml = "fZJNT%2BMwEIbvSPwHy%2Fd8tMvHympSdUGISuwS0cCBm%2BtMUwfbk%2FU4zfLvSVMq2Euv45n3fd7xzOb%2FrGE78KTRZXwSp5yBU1hpV2f8ubyLfvJ5fn42I2lNKxZd2Lon%2BNsBBTZMOhLjQ8Y77wRK0iSctEAiKLFa%2FH4Q0zgVrceACg1ny9uMy7rCdaM2%2Bs0BWrtppK2UAdeoVjW2ruq1bevGImcvR6zpHmtJ1MHSUZAuDKU0vY7Si2h6VU5%2BiMuJuLx65az4dPql3SHBKaz1oYnEfVkWUfG4KkeBna7A%2Fxm6M14j1gZihZazBRH4MODcoKPOgl%2BB32kFz08PGd%2BG0JJIkr7v46%2BhRCaEpod17DCRivYZCkmkd4N28B3wfNyrGKP5bws9DS6PKDz%2FMpsl36Tyz%2F%2Fax1jeFmi0emcLY7C%2F8SDD0Z7dobcynHbbV3QVbcZW0TlqQemNhoqzJD%2B4%2Fn8Yw7l8AA%3D%3D"

require 'cgi'
require 'base64'
require 'zlib'

inflated = Base64::decode64(CGI.unescape(saml))
# Наступні дві команди непотрібні, якщо строку не було заархівовано(deflated/compressed)
zlib = Zlib::Inflate.new(-Zlib::MAX_WBITS)
zlib.inflate(inflated)

```
Результат:
```ruby
"<?xml version=\"1.0\" encoding=\"UTF-8\"?>\r\n<samlp:AuthnRequest xmlns:samlp=\"urn:oasis:names:tc:SAML:2.0:protocol\" ID=\"agdobjcfikneommfjamdclenjcpcjmgdgbmpgjmo\" Version=\"2.0\" IssueInstant=\"2007-04-26T13:51:56Z\" ProtocolBinding=\"urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST\" ProviderName=\"google.com\" AssertionConsumerServiceURL=\"https://www.google.com/a/solweb.no/acs\" IsPassive=\"true\"><saml:Issuer xmlns:saml=\"urn:oasis:names:tc:SAML:2.0:assertion\">google.com</saml:Issuer><samlp:NameIDPolicy AllowCreate=\"true\" Format=\"urn:oasis:names:tc:SAML:2.0:nameid-format:unspecified\" /></samlp:AuthnRequest>\r\n"
```
[Джерело][2]

[Дізнатися більше про SAML][3]

<br><br><br>
---
[1]: http://king-sabri.net/?p=2613
[2]: http://stackoverflow.com/questions/3253298/base64-decode64-in-ruby-returning-strange-results
[3]: http://dev.gettinderbox.com/2013/12/16/introduction-to-saml/
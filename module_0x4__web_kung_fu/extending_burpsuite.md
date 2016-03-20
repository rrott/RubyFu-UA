# Розширюючи Burp Suite

## Налаштування рубі оточення для розширень Burp 

1. Завантажте стабільну версію JRuby з [JRuby Downloads](http://jruby.org/download)
2. Виберіть jar для Linux (JRuby x.x.x Complete .jar) або запускач для Windows.
3. Зімпотруйте оточення **Burp Suite** >> **Extender** >> **Options** >> **Ruby Environment**.

![](webfu__burp_setenv1.png)

- http://human.versus.computer/buby/
- http://human.versus.computer/buby/rdoc/index.html
- https://github.com/null--/what-the-waf/blob/master/what-the-waf.rb
- https://www.pentestgeek.com/web-applications/burp-suite-tutorial-web-application-penetration-testing-part-2/
- http://blog.opensecurityresearch.com/2014/03/extending-burp.html
- http://www.gotohack.org/2011/05/cktricky-appsec-buby-script-basics-part.html
- https://portswigger.net/burp/extender/

Зімпортуйте Burp Suite Extender Core API `IBurpExtender`

**alert.rb**
```ruby
require 'java'
java_import 'burp.IBurpExtender'

class BurpExtender
  include IBurpExtender

  def registerExtenderCallbacks(callbacks)
    callbacks.setExtensionName("Rubyfu Alert!")
    callbacks.issueAlert("Alert: Ruby goes evil!")
  end
end
```
Завантажте плагін alert.rb
![](webfu__burp-ext1.png)

Перевірте вкладку Alerts
![](webfu__burp-ext2.png)

## Buby
Buby - це суміш JRubyз популярними комерційними інструментами для перевірки на безпеку в Burp Suite від PortSwigger. Курування Burp буде передано до JRuby за допомогою розширень Java використовуючи API від BurpExtender. Це розгирення прагне додати фічі Рубі до Burp Suite з інтерфесом, сумісним з Java інтерфейсом Burp-а







<br><br><br>
---

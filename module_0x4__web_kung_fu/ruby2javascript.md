# Ruby 2 JavaScript



## CoffeeScript
[CoffeeScript][1] це мова програмування яка транскриптується в JavaScript. Вона додає синтаксичний цукор натхненний Ruby, Python та Haskell в спробі покразити JavaScript.
 

### Швидкий перегляд CoffeeScript 

Далі буде швидкий опис того, як ми можемо працювати з CoffeeScript

- Встановлюємо CoffeScript 
```
npm install -g coffee-script
```

- для перетворення коду в JavaScript "на льоту"
```
coffee --watch --compile script.coffee 
```

### Ruby гем для роботи з CoffeScript

CoffeeScript гем для **Ruby** це міст до офіційного компілятору CoffeeScript. 

- Всатноілення гему CoffeeScript
```
gem install coffee-script
```

- Конвертація CoffeeScript файлу в JavaScript 

```ruby
#!/usr/bin/env ruby
require 'coffee-script'
if ARGF
  file = File.open("#{ARGV[0]}.js", 'a')
  file.write CoffeeScript.compile(ARGF.read)
end
```

Запустіть код:
```
ruby coffee2js.rb exploit.coffee 
```




## Opal 
Opal це компілятор з Ruby в JavaScript, який компілює код рядок за рядком і також має підтримку бібліотеки corelib з рубі.

- Встановлюємо Opal
```
gem install opal opal-jquery
```





<br><br><br>
---
[1]: http://coffeescript.org
[5]: http://js2.coffee/
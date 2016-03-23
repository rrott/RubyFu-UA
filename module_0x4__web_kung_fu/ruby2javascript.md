# Ruby 2 JavaScript



## CoffeeScript
[CoffeeScript][1] це мова програмування яка транскриптується в JavaScript. Вона додає синтаксичний цукор натхненний Ruby, Python та Haskell в спробі покразити JavaScript.
 

### Швидкий перегляд CoffeeScript 

Далі буде швидкий опис того, як ми можемо працювати з CoffeeScript

- To install CoffeScript 
```
npm install -g coffee-script
```

- For live conversion 
```
coffee --watch --compile script.coffee 
```

### Ruby CoffeScript gem 
**Ruby** CoffeeScript gem is a bridge to the official CoffeeScript compiler. 

- To install CoffeeScript gem
```
gem install coffee-script
```

- Convert CoffeeScript file to JavaScript 

```ruby
#!/usr/bin/env ruby
require 'coffee-script'
if ARGF
  file = File.open("#{ARGV[0]}.js", 'a')
  file.write CoffeeScript.compile(ARGF.read)
end
```

Run it
```
ruby coffee2js.rb exploit.coffee 
```




## Opal 
Opal is a Ruby to JavaScript source-to-source compiler. It also has an implementation of the Ruby corelib.

- To install Opal
```
gem install opal opal-jquery
```





<br><br><br>
---
[1]: http://coffeescript.org
[5]: http://js2.coffee/
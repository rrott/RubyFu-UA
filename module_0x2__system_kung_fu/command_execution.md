# Запуск системних команд

Є кілька методів як запустити системну команду і обираючи один  з них треба брати до уваги наступне:
1. Вам потрібно знати тільки вивід на екран(stdout) чи вам потрібно ще й знати про помилки в роботі(stderr)?
2. Наскільки великим буде вивід на екран? Чи потрібно вам тримати весь результат в пам'яті?
3. Чи потрібно вам прочитати деякі дані допоки процес все ще працює?
4. Чи вам потрібен результат в коді?
5. Чи потрібні вам Рубі об'єкти, що репрезентують процес і дають вам можливість вбити їх коли вам треба?


Наступні методи будуть працювати на всіх операційних системах.

### Kernel#` (зворотні лапки)
```ruby
>> `date`
#=> "Sun Sep 27 00:38:54 AST 2015\n"
```

### Kernel#exec
```ruby
>> exec('date')
Sun Sep 27 00:39:22 AST 2015
RubyFu( ~ )-> 
```
Зробіть увагу що вас викинуло з інтерпретатора.

### Kernel#system
```ruby
>> system 'date'
Sun Sep 27 00:38:01 AST 2015
#=> true
```


### IO#popen
```ruby
>> IO.popen("date") { |f| puts f.gets }
Sun Sep 27 00:40:06 AST 2015
#=> nil
```


### Open3#popen3
```ruby
require 'open3'
stdin, stdout, stderr = Open3.popen3('dc') 
#=> [#<IO:fd 14>, #<IO:fd 16>, #<IO:fd 18>, #<Process::Waiter:0x00000002f68bd0 sleep>]
>> stdin.puts(5)
#=> nil
>> stdin.puts(10)
#=> nil
>> stdin.puts("+")
#=> nil
>> stdin.puts("p")
#=> nil
>> stdout.gets
#=> "15\n"
```


### Process#spawn
Kernel.spawn запускає задану команду в новому шелі(subshell). Він повертає результат роботи відразу разом з айді процесу.
```ruby
pid = Process.spawn("date")
Sun Sep 27 00:50:44 AST 2015
#=> 12242
```

### %x"", %x[], %x{}, %x$''$ 

```ruby
>> %x"date"
#=> Sun Sep 27 00:57:20 AST 2015\n"
>> %x[date]
#=> "Sun Sep 27 00:58:00 AST 2015\n"
>> %x{date}
#=> "Sun Sep 27 00:58:06 AST 2015\n"
>> %x$'date'$
#=> "Sun Sep 27 00:58:12 AST 2015\n"
```

### Rake#sh
```ruby
require 'rake'
>> sh 'date'
date
Sun Sep 27 00:59:05 AST 2015
#=> true
```



### Додатково
Аби дізнатися статус роботи програми запущеної за допомогою зворотніх лапок, ви можете виконати $?.success?
#### $?
```ruby
>> `date`
=> "Sun Sep 27 01:06:42 AST 2015\n"
>> $?.success?
=> true
```




















<br><br><br>
---
- [Ruby | Виконання системних команд](http://king-sabri.net/?p=2553)
- [5 методів запуска програм в Рубі](http://mentalized.net/journal/2010/03/08/5-ways-to-run-commands-from-ruby/)
- [6 методів запуску Shell команд в РУбі](http://tech.natemurray.com/2007/03/ruby-shell-commands.html)
- [Як вибрати правильний метод](http://stackoverflow.com/a/4413/967283)
- [Запуск команд в Рубі](http://blog.bigbinary.com/2012/10/18/backtick-system-exec-in-ruby.html)
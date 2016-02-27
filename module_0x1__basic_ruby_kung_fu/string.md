# Строки

## Розфарбовуємо вивід команди на екран
Оскільки ми в основному працюватимемо з командним рядком, було б гарно мати більш красиво оформлений вивід результату роботи програм. Ось список основних кольорів, що знадобляться вам в найближчий час. Можете додати цей набір,

```ruby
class String
  def red;          colorize(self, "\e[1m\e[31m");  end
  def green;        colorize(self, "\e[1m\e[32m");  end
  def dark_green;   colorize(self, "\e[32m"     );  end
  def yellow;       colorize(self, "\e[1m\e[33m");  end
  def blue;         colorize(self, "\e[1m\e[34m");  end
  def dark_blue;    colorize(self, "\e[34m"     );  end
  def purple;       colorize(self, "\e[35m"     );  end
  def dark_purple;  colorize(self, "\e[1;35m"   );  end
  def cyan;         colorize(self, "\e[1;36m"   );  end
  def dark_cyan;    colorize(self, "\e[36m"     );  end
  def pure;         colorize(self, "\e[1m\e[35m");  end
  def bold;         colorize(self, "\e[1m"      );  end
  
  def colorize(text, color_code) 
    "#{color_code}#{text}\e[0m" 
  end
end
```
Все що вам потрібно так це викликати метод з іменем відповідного кольору коли використовуєте ```puts```:
```ruby
puts "RubyFu".blue
puts "RubyFu".yellow.bold
```
Пояснимо код в класі, аби вам було зрозуміліше 

```

\033  [0;  31m
 ^     ^    ^    
 |     |    |
 |     |    |--------------------------------------- [Номер кольору]
 |     |-------------------- [модифікатор]           (кінчається на "m")
 |-- [Екранований символ]           | 0 - стандартний                     
     (можете вживати "\e")          | 1 - виділений
                                    | 2 - знову стандартний
                                    | 3 - колір фону
                                    | 4 - підкреслений
                                    | 5 - боимаючий
```
Або ж зможете використовувати сторонній гем [colorized] щоб мати більше модних фіч
```
gem install colorize
```
Потім просто підключіть його в ваш скрипт:

```ruby
require 'colorize'
```

## Перезапис результату виводу в консолі
Це дуже прикольно мати більш гнучкості в вашому терміналі бо іноді нам важливо мати можливість робити більше з нашими скриптами.

Перезапис виводу в консоль робить наші програми більш елегантним і зменшує  інформаційний шум, наприклад від виводу, що часто повторюється, типу підрахунки чи показ прогресу операції,

Я прочитав інструкцію до [bash - переміщення курсору вводу][2] і я знайшов щось, що нам може стати в пригоді аби поліпшити наші скрипти. Ось що там було сказано
```
- Позиція курсору:
  \033[<L>;<C>H
     Or
  \033[<L>;<C>f
  Встановлює курсор на рядок L та колонку C.
- Переміщує курсор на N рядків вгору:
  \033[<N>A
- Переміщує курсор на N рядків вниз:
  \033[<N>B
- Переміщує курсор на N колонок вперед:
  \033[<N>C
- Переміщує курсор на N колонок назад:
  \033[<N>D
- Очищує екран та переводить курсор на позицію (0,0):
  \033[2J
- Стирає все до кінця рядка:
  \033[K
- Збурігає позицію курсору:
  \033[s
- Відновлює позицію курсору:
  \033[u
```
Отже, аби перевірити це я написав ось такий PoC(Proof of Concept - Доказ концепції, ідеї) 
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
(1..3).map do |num|
  print "\rНомер: #{num}"
  sleep 0.5
  print ("\033[1B")	# Переводимо курсор на 1 рядок вниз 
  
  ('a'..'c').map do |char|
    print "\rЛітера: #{char}"
    print  ("\e[K")
    sleep 0.5
    print ("\033[1B")	# Переводимо курсор на 1 рядок вниз 
    
    ('A'..'C').map do |char1|
      print "\rВелика літера: #{char1}"
      print  ("\e[K")
      sleep 0.3
    end
    print ("\033[1A")	# Переводимо курсор на 1 рядок вгору
    
  end

  print ("\033[1A")	# Переводимо курсор на 1 рядок вгору
end

print ("\033[2B")	# Переводимо курсор на 2 рядки вниз 

puts ""
```

Вже непогано, але чому б не зробити це в вигляді методів рубі для більш простого використання? Отже я прийшов до наступного результату:
```ruby
# KING SABRI | @KINGSABRI
class String
  def mv_up(n=1)
    cursor(self, "\033[#{n}A")
  end

  def mv_down(n=1)
    cursor(self, "\033[#{n}B")
  end

  def mv_fw(n=1)
    cursor(self, "\033[#{n}C")
  end

  def mv_bw(n=1)
    cursor(self, "\033[#{n}D")
  end

  def cls_upline
    cursor(self, "\e[K")
  end

  def cls
    # cursor(self, "\033[2J")
    cursor(self, "\e[H\e[2J")
  end

  def save_position
    cursor(self, "\033[s")
  end

  def restore_position
    cursor(self, "\033[u")
  end

  def cursor(text, position)
    "\r#{position}#{text}"
  end
end
```

Далі, для доказу ідеї(PoC) я використав попередній скрипт(після оновлення "на льоту" класу String, запустивши код з прикладу вище)
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
# Перший рівень
(1..3).map do |num|
  print "\rНомер: #{num}"
  sleep 0.7
  # Другий рівень
  ('a'..'c').map do |char|
      print "Літера: #{char}".mv_down
      sleep 0.5
      # Третій рівень
      ('A'..'C').map do |char1|
          print "Велика літера: #{char1}".mv_down
          sleep 0.2
          print "".mv_up
      end
      print "".mv_up
  end
  sleep 0.7
end
print "".mv_down 3
```
Красивіше, чи не так? Скажіть, що так =)

##Приклади
### Показ прогресу процесу у вісотках

```ruby
(1..10).each do |percent|
  print "#{percent*10}% закінчено\r"
  sleep(0.5)
  print  ("\e[K") # Видалення поточного рядку
end
puts "Готово!"
```
Інший приклад:
```ruby
(1..5).to_a.reverse.each do |c|
  print "\rВиконання програми закінчиться через #{c} секунд(у/и)"
  print "\e[K"
  sleep 1
end
```
Використовуючи наш красивий метод(після оновлення "на льоту" класу String)
```ruby
(1..5).to_a.reverse.each do |c|
  print "Виконання програми закінчиться через #{c} секунд(у/и)".cls_upline
  sleep 1
end
puts 
```

<br><br><br>
---
[1]: https://github.com/fazibear/colorize
[2]: http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x361.html





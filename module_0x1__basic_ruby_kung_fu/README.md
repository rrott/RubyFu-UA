# Модуль 0x1 | Основи Кунг-Фу в Ruby

За допомогоою Рубі можна робити купу крутих штук зі строками та масивами. В цьому розділі ми покаемо найбільш відомі трюки, що можуть знадобитися в нашому хакерському житті.


## Термінал 

### Розмір Терміналу
Є багато методів визначити розмір терміналу за допомогою рубі.

- Використовуючи стандартну бібліотеку IO/console

```ruby
require 'io/console'
rows, columns = $stdin.winsize
# Try this now
print "-" * (columns/2) + "\n" + ("|" + " " * (columns/2 -2) + "|\n")* (rows / 2) + "-" * (columns/2) + "\n"
```
- Використовуючи стандарту бібліотеку readline

```ruby
require 'readline'
Readline.get_screen_size
```

- Визначити розмір терміналу через змінну оточення(Environment) інтерпретатора IRB чи Pry

```ruby
[ENV['LINES'].to_i, ENV['COLUMNS'].to_i]
```

- Використовуючи програму Unix оточення tput

```ruby
[`tput lines`.to_i, `tput cols`.to_i ]
```

## Консоль з автодоповненням
В Metasploit є пречудова консоль(msfconsole) в якій ми відпочиваємо від використання ключів консольних програм. На щастя є вихід і ось вам одна з ідей як зробити авто-доповнення в консолі рубі:

- Readline 

Модуль Readline надає інтерфейс для GNU Readline. Він визначає багато методів для роботи з авто-доповненням й надає доступ до історії команд всередині рубі інтерпретатора.

**console-basic1.rb**

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
# 
require 'readline'

# Відключаємо Ctrl+C
trap('INT', 'SIG_IGN')

# Список команд
CMDS = [ 'help', 'rubyfu', 'ls', 'pwd', 'exit' ].sort


completion = proc { |line| CMDS.grep( /^#{Regexp.escape( line )}/ ) }

# Налаштування Консолі
Readline.completion_proc = completion        # Включаємо автодоповнення
Readline.completion_append_character = ' '   # Переконуємося в тому, що встановлено пробіл після автодоповнення

while line = Readline.readline('-> ', true)
  puts line unless line.nil? or line.squeeze.empty?
  break if line =~ /^quit.*/i or line =~ /^exit.*/i
end
```
А тепер запустіть цей скрипт і спробуйте завершити набір команди за допомогою кнопки tab!

Отже, основна ідея в знанні авто-доповнення це спрощення роботи, а не в тім, щоб просто натискання кнопку tab. Ось проста думка:


**console-basic2.rb**

```ruby
require 'readline'

# Відключаємо Ctrl+C
trap('INT', 'SIG_IGN')

# Список команд
CMDS = [ 'help', 'rubyfu', 'ls', 'exit' ].sort


completion = 
    proc do |str|
      case 
      when Readline.line_buffer =~ /help.*/i
	puts "Доступні команди:\n" + "#{CMDS.join("\t")}"
      when Readline.line_buffer =~ /rubyfu.*/i
	puts "Rubyfu, там де рубі стає злим!"
      when Readline.line_buffer =~ /ls.*/i
	puts `ls`
      when Readline.line_buffer =~ /exit.*/i
	puts 'Виходимо..'
	exit 0
      else
	CMDS.grep( /^#{Regexp.escape(str)}/i ) unless str.nil?
      end
    end


Readline.completion_proc = completion        # Включаємо автодоповнення
Readline.completion_append_character = ' '   # Переконуємося в тому, що встановлено пробіл після автодоповнення

while line = Readline.readline('-> ', true)  # Починаємо з символу -> та виставляємо add_hist = true
  puts completion.call
  break if line =~ /^quit.*/i or line =~ /^exit.*/i
end

```

Тепер все стало набагато краще, як в *msfconsole*! 




<br><br><br>
---
- [Ruby Readline - Документація та підручник](http://bogojoker.com/readline/)





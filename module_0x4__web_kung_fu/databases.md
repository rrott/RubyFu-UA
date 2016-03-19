# Бази даних
Робота з базами даних потребує знань в тестуванні тенет і далі ми пройдемося по найбільш відомим базам даних та тим, як з ними працюівати.

## SQLite 

- Щоб встановит гем sqlite3 виконайте наступну команду: 
```
gem install sqlite3
```
Вам потрібно буде мати встановленими бібіліотеки розробника для sqlite3

```
apt-get install libsqlite3-dev
```

- основні операції

```ruby
require "sqlite3"

# Open/Create a database
db = SQLite3::Database.new "rubyfu.db"

# Create a table
rows = db.execute <<-SQL 
  CREATE TABLE attackers (
   id   INTEGER PRIMARY KEY   AUTOINCREMENT,
   name TEXT    NOT NULL,
   ip   CHAR(50)
);
SQL

# Execute a few inserts
{
  'Anonymous'    => "192.168.0.7",
  'LulzSec'      => "192.168.0.14",
  'Lizard Squad' => "192.168.0.253"
}.each do |attacker, ip|
  db.execute("INSERT INTO attackers (name, ip) 
	          VALUES (?, ?)", [attacker, ip])
end

# Find a few rows
db.execute "SELECT id,name,ip FROM attackers"

# List all tables
db.execute  "SELECT * FROM sqlite_master where type='table'"
```


## Active Record
- Щоб встановити ActiveRecord виконайте команду
```
gem install activerecord 
```

### база даних MySQL 
- Аби встановити адаптер MySQL виконайте:
```
gem install mysql 
```

Увійдіть в консоль mysql та створіть базу даних *rubyfu_db* з таблицею *attackers*

```
create database rubyfu_db;

grant all on rubyfu_db.* to 'root'@'localhost';

create table attackers (
  id int not null auto_increment,
  name varchar(100) not null, 
  ip text not null,  
  primary key (id)  
);  

exit
```

Вивід матиме десь такий вигляд:
```
mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 41
Server version: 5.5.44-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> create database rubyfu_db;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on rubyfu_db.* to 'root'@'localhost'; 
Query OK, 0 rows affected (0.00 sec)

mysql> use rubyfu_db;
Database changed
mysql> create table attackers (
    ->   id int not null auto_increment,
    ->   name varchar(100) not null, 
    ->   ip text not null,  
    ->   primary key (id)  
    -> );  
Query OK, 0 rows affected (0.01 sec)

mysql> exit
```

Тепер, давайте підключемося до бази *rubyfu_db*  
```ruby
require 'active_record'  
ActiveRecord::Base.establish_connection(  
:adapter  => "mysql",
:username => "root",
:password => "root",
:host     => "localhost",  
:database => "rubyfu_db"  
)  
  
class Attackers < ActiveRecord::Base  
end  
```

- Використаємо бібліотеку ActiveRecord, яка доступна як гем activerecord
- Використаємо ActiveRecord адаптер для *mysql*
- Виконаємо під'єднання до бази даних *rubyfu_db*
- Створимо клас з назвою *Attackers* вкиористовуючи домовленності вказані вище (attacker)

```ruby
Attackers.create(:name => 'Anonymous',    :ip => "192.168.0.7")  
Attackers.create(:name => 'LulzSec',      :ip => "192.168.0.14")  
Attackers.create(:name => 'Lizard Squad', :ip => "192.168.0.253")  
```

Ви спостерігатимете за тим, як ActiveRecord перевірятиме наявні таблиці бази даних, досліджує, щоб з'ясувати, які стовпці доступні. Саме через це ми матимемо змогу використовувати аксесори з іменами таблиць не визначаючи їх заздалегідь - ми визначили їх в базі даних й ActiveRecord автоматично підхопив їх.

Ви можете знайти їх 
- по айді
```
Attackers.find(1)
```
- за іменем
```
Attackers.find_by(name: "Anonymous")
```
Результат 
```ruby
#<Attackers:0x000000010a6ad0 id: 1, name: "Anonymous", ip: "192.168.0.7">
```

або ж ви можете працювати з ними як зі звичайними об'єктами
```ruby
attacker = Attackers.find(3)
attacker.id
attacker.name
attacker.ip
```

Якщо вам треба видалити елемент з бази, ви можете використати команду destroy з ActiveRecord::Base:


```ruby
Attackers.find(2).destroy  
```
Отже, напишемо повний скрипт:

```ruby
#!/usr/bin/env ruby
# KING SABRI | @KINGSABRI
# ActiveRecord with MySQL
#
require 'active_record'  

# Connect to database
ActiveRecord::Base.establish_connection(
                                        :adapter  => "mysql",
                                        :username => "root",
                                        :password => "root",
                                        :host     => "localhost",  
                                        :database => "rubyfu_db"  
                                       )  

# Create Active Record Model for the table 
class Attackers < ActiveRecord::Base  
end

# Create New Entries to the table 
Attackers.create(:name => 'Anonymous',    :ip => "192.168.0.7")  
Attackers.create(:name => 'LulzSec',      :ip => "192.168.0.14")  
Attackers.create(:name => 'Lizard Squad', :ip => "192.168.0.253")

# Interact with table items 
attacker = Attackers.find(3)
attacker.id
attacker.name
attacker.ip

# Delete a table Item
Attackers.find(2).destroy
```


### База даних Oracle 

- Передумови

аби зібрати [ruby-oci8](http://www.rubydoc.info/gems/ruby-oci8/file/docs/install-full-client.md) (що є основною задежністю лоя роботи з драйвером від oracle) вам треба виконати наступні кроки:

 - Завантажте архів програми для вашої системи: [Linux](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html) | [Windows](http://www.oracle.com/technetwork/topics/winsoft-085727.html) | [Mac](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html) 
 - instantclient-basic-[OS].[Arch]-[VERSION].zip
 - instantclient-sqlplus-[OS].[Arch]-[VERSION].zip
 - instantclient-sdk-[OS].[Arch]-[VERSION].zip


- Роапакуйте викачаний архів: 

```
unzip -qq instantclient-basic-linux.x64-12.1.0.2.0.zip
unzip -qq instantclient-sdk-linux.x64-12.1.0.2.0.zip
unzip -qq instantclient-sqlplus-linux.x64-12.1.0.2.0.zip
```

- Створіть системні папки
від імені рута чи за допомогою sudo 

```
mkdir -p /usr/local/oracle/{network,product/instantclient_64/12.1.0.2.0/{bin,lib,jdbc/lib,rdbms/jlib,sqlplus/admin/}}
```

Файлова система має виглядати приблизно так:
```
/usr/local/oracle/
├── admin
│   └── network
└── product
    └── instantclient_64
        └── 12.1.0.2.0
            ├── bin
            ├── jdbc
            │   └── lib
            ├── lib
            ├── rdbms
            │   └── jlib
            └── sqlplus
                └── admin
```

- Перемістіть/скопіюйте розпаковані файли:

```
cd instantclient_12_1

mv ojdbc* /usr/local/oracle/product/instantclient_64/12.1.0.2.0/jdbc/lib/
mv x*.jar /usr/local/oracle/product/instantclient_64/12.1.0.2.0/rdbms/jlib/
# rename glogin.sql to login.sql
mv glogin.sql /usr/local/oracle/product/instantclient_64/12.1.0.2.0/sqlplus/admin/login.sql
mv sdk /usr/local/oracle/product/instantclient_64/12.1.0.2.0/lib/
mv *README /usr/local/oracle/product/instantclient_64/12.1.0.2.0/
mv * /usr/local/oracle/product/instantclient_64/12.1.0.2.0/bin/
# Symlink of instantclient
cd /usr/local/oracle/product/instantclient_64/12.1.0.2.0/bin
ln -s libclntsh.so.12.1 libclntsh.so
ln -s ../lib/sdk sdk
cd -
```

- Налоштуйте оточення системи


Додайте змінні оточення oracle до файлу `~/.bashrc` Потім додайте наступне:

```
# Oracle Environment 
export ORACLE_BASE=/usr/local/oracle
export ORACLE_HOME=$ORACLE_BASE/product/instantclient_64/12.1.0.2.0
export PATH=$ORACLE_HOME/bin:$PATH
LD_LIBRARY_PATH=$ORACLE_HOME/bin
export LD_LIBRARY_PATH
export TNS_ADMIN=$ORACLE_BASE/admin/network
export SQLPATH=$ORACLE_HOME/sqlplus/admin
```
Та виконайте команду:
```
source ~/.bashrc
```

- Щоб встановити адаптер Oracle виконайте:
```
gem install ruby-oci8 activerecord-oracle_enhanced-adapter
```

Тепер спробуємо підключитися:
```
require 'active_record'

ActiveRecord::Base.establish_connection(
					  :adapter  => "oracle_enhanced",
					  :database => "192.168.0.13:1521/XE",
					  :username => "SYSDBA",
					  :password => "welcome1"
				       )

class DBAUsers < ActiveRecord::Base
end


```



### MSSQL database


- To install MSSQL adapter

```
gem install tiny_tds activerecord-sqlserver-adapter
```
<!--
Let's connect
```ruby
require 'tiny_tds'
require 'activerecord-sqlserver-adapter' 

development:
  adapter: sqlserver
  mode: dblib
  host: localhost
  port: 1433
  username: sa
  password: P@ssw0rd
  database: sharepoint_dev
```
-->





<br><br><br>
---


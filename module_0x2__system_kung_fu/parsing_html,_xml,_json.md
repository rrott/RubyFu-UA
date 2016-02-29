# Розбір HTML, XML, JSON

Чесно кажучи, простіший та кращий метод розбору HTML та XML коду це використання бібліотеки **Nokogiri**

- Інсталяція Nokogiri:
```
gem install nokogiri
```

## HTML

Тут ми використовуватимемо nokogiri аби перелік вмісту книги з  `http://rubyfu.net/content/`

### Використовуючи CSS селектори
```ruby
require 'nokogiri'
require 'open-uri'

page = Nokogiri::HTML(open("http://rubyfu.net/content/"))
page.css(".book .book-summary ul.summary li a, .book .book-summary ul.summary li span").each { |css| puts css.text.strip.squeeze.gsub("\n", '')}
```

Результат 
```
RubyFu
Module 0x0 | Introduction
0.1.  Contribution
0.2.  Beginners
0.3.  Required Gems
1.  Module 0x1 | Basic Ruby Kung Fu
1.1.  String
1.1.1.  Conversion
1.1.2.  Extraction
1.2.  Array
2.  Module 0x2 | System Kung Fu
2.1.  Command Execution
2.2.  File manipulation
2.2.1.  Parsing HTML, XML, JSON
2.3.  Cryptography
2.4.  Remote Shell
2.4.1.  Ncat.rb
2.5.  VirusTotal
3.  Module 0x3 | Network Kung Fu
3.1.  Ruby Socket
3.2.  FTP
3.3.  SSH
3.4.  Email
3.4.1.  SMTP Enumeration
3.5.  Network Scanning
.
.
..snippet..
```

## XML
Тут ми маємо два способи і ми б хотіли показати обидва: стандартна бібліотека `rexml` та зовнішня бібліотека `nokogiri` 

Наприклад у нас є такий XML файл
```xml
<?xml version="1.0"?>
<collection shelf="New Arrivals">
<movie title="Enemy Behind">
   <type>War, Thriller</type>
   <format>DVD</format>
   <year>2003</year>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Talk about a US-Japan war</description>
</movie>
<movie title="Transformers">
   <type>Anime, Science Fiction</type>
   <format>DVD</format>
   <year>1989</year>
   <rating>R</rating>
   <stars>8</stars>
   <description>A scientific fiction</description>
</movie>
   <movie title="Trigun">
   <type>Anime, Action</type>
   <format>DVD</format>
   <episodes>4</episodes>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Vash the Stampede!</description>
</movie>
<movie title="Ishtar">
   <type>Comedy</type>
   <format>VHS</format>
   <rating>PG</rating>
   <stars>2</stars>
   <description>Viewable boredom</description>
</movie>
</collection>
```

### REXML

```ruby
require 'rexml/document'
include REXML

file = File.read "file.xml"
xmldoc = Document.new(xmlfile)

# Беремо кореневий елемент
root = xmldoc.root
puts "Root element : " + root.attributes["shelf"]


# Список назв стрічок.
xmldoc.elements.each("collection/movie") do |e|
  puts "Movie Title : " + e.attributes["title"] 
end

# Список типів стрічок.
xmldoc.elements.each("collection/movie/type") do |e|
  puts "Movie Type : " + e.text 
end

# Список описів стрічок.
xmldoc.elements.each("collection/movie/description") do |e|
  puts "Movie Description : " + e.text
end

# Список кіно-зірок
xmldoc.elements.each("collection/movie/stars") do |e|
  puts "Movie Stars : " + e.text
end
```


### Nokogiri
```ruby
require 'nokogiri'


```


#### Slop
```ruby
require 'nokogiri'
# Parse XML file
doc = Nokogiri::Slop file

puts doc.search("type").map {|f| t.text}        # List of Types
puts doc.search("format").map {|f| f.text}      # List of Formats
puts doc.search("year").map {|y| y.text}        # List of Year
puts doc.search("rating").map {|r| r.text}      # List of Rating
puts doc.search("stars").map {|s| s.text}       # List of Stars
doc.search("description").map {|d| d.text}      # List of Descriptions
```


## JSON

# WordPress API

Ruby має [стандартну бібліотеку][2] що називається `xmlrpc`. Вона бере на себе відповідальність працювати з усіма що пов'язано з xmlrpc. Ви, навіть можете створити свій XML-RPC сервер, використовуючи її. Давайте використаємо її на практиці.

Шукаючи якусь дуже відому програму, що використовує XML-RPC я наштовхнувся на WordPress, який, звичайно ж, є першим кандидатом на перевірку.

Отже, зо ми хочемо зробити?
- Сказати "привіт" ВордПресові 
- Отримати список всіх доступних методів.
- Отримати список всіх наявних користувачів
- Отримати список публікацій
- Створити нову публікацію
- Отримати нашу публікацію для роботи з нею
- Отримати всі коментарів до нашої публікації


```ruby
require 'xmlrpc/client'

opts =
    {
        host: '172.17.0.2',
        path: '/xmlrpc.php',
        port: 80,
        proxy_host: nil,
        proxy_port: nil,
        user: 'admin',
        password: '123123',
        use_ssl: false,
        timeout: 30
    }

# Create a new instance 
server = XMLRPC::Client.new(
    opts[:host], opts[:path], opts[:port],
    opts[:proxy_host], opts[:proxy_port],
    opts[:user], opts[:password],
    opts[:use_ssl], opts[:timeout]
)

# Create a new instance takes a hash
server = XMLRPC::Client.new3(opts)

# Say hello to WordPress
response = server.call("demo.sayHello")

# List all available methods
server.call('system.listMethods', 0)

# List all available users
server.call('wp.getAuthors', 0, opts[:user], opts[:password])

# List all available post
response = server.call('wp.getPosts', 0, opts[:user], opts[:password])

# Create a new post!
post =
    {
        "post_title"     => 'Rubyfu vs WP XML-RPC',
        "post_name"      => 'Rubyfu vs WordPress XML-RPC',
        "post_content"   => 'This is Pragmatic Rubyfu Post. Thanks for reading',
        "post_author"    => 2,
        "post_status"    => 'publish',
        "comment_status" => 'open'
    }
response = server.call("wp.newPost", 0, opts[:user], opts[:password], post)

# Retrieve created post
response =  server.call('wp.getPosts', 0, opts[:user], opts[:password], {"post_type" => "post", "post_status" => "published", "number" => "2", "offset" => "2"})

# List all comments on a specific post
response =  server.call('wp.getComments', 0, opts[:user], opts[:password], {"post_id" => 4})

```

Результат 

```ruby
>> # Say hello to WordPress
>> response = server.call("demo.sayHello")
=> "Hello!"
>> 
>> # List all available methods
>> server.call('system.listMethods', 0)
=> ["system.multicall",
 "system.listMethods",
 "system.getCapabilities",
 "demo.addTwoNumbers",
 "demo.sayHello",
 "pingback.extensions.getPingbacks",
 "pingback.ping",
 "mt.publishPost",
 "mt.getTrackbackPings",
 "mt.supportedTextFilters",
 ...skipping...
 "metaWeblog.newMediaObject",
 "metaWeblog.getCategories",
 "metaWeblog.getRecentPosts",
 "metaWeblog.getPost",
 "metaWeblog.editPost",
 "metaWeblog.newPost",
 ...skipping...
 "blogger.deletePost",
 "blogger.editPost",
 "blogger.newPost",
 "blogger.getRecentPosts",
 "blogger.getPost",
 "blogger.getUserInfo",
 "blogger.getUsersBlogs",
 "wp.restoreRevision",
 "wp.getRevisions",
 "wp.getPostTypes",
 "wp.getPostType",
 ...skipping...
 "wp.getPost",
 "wp.deletePost",
 "wp.editPost",
 "wp.newPost",
 "wp.getUsersBlogs"]
>> 
>> # List all available users
>> server.call('wp.getAuthors', 0, opts[:user], opts[:password])
=> [{"user_id"=>"1", "user_login"=>"admin", "display_name"=>"admin"}, {"user_id"=>"3", "user_login"=>"galaxy", "display_name"=>"Galaxy"}, {"user_id"=>"2", "user_login"=>"Rubyfu", "display_name"=>"Rubyfu"}]
>> 
>> # List all available post
>> response = server.call('wp.getPosts', 0, opts[:user], opts[:password])
=> [{"post_id"=>"4",
  "post_title"=>"Rubyfu vs WP XMLRPC",
  "post_date"=>#<XMLRPC::DateTime:0x0000000227f3b0 @day=1, @hour=19, @min=44, @month=11, @sec=31, @year=2015>,
  "post_date_gmt"=>#<XMLRPC::DateTime:0x0000000227d178 @day=1, @hour=19, @min=44, @month=11, @sec=31, @year=2015>,
  "post_modified"=>#<XMLRPC::DateTime:0x000000021d6ee0 @day=1, @hour=19, @min=52, @month=11, @sec=25, @year=2015>,
  "post_modified_gmt"=>#<XMLRPC::DateTime:0x000000021d4ca8 @day=1, @hour=19, @min=52, @month=11, @sec=25, @year=2015>,
  "post_status"=>"publish",
  "post_type"=>"post",
  "post_name"=>"rubyfu-vs-wordpress-xmlrpc",
  "post_author"=>"2",
  "post_password"=>"",
  "post_excerpt"=>"",
  "post_content"=>"This is Pragmatic Rubyfu Post. Thanks for reading",
  "post_parent"=>"0",
  "post_mime_type"=>"",
  "link"=>"http://172.17.0.2/2015/11/01/rubyfu-vs-wordpress-xmlrpc/",
  "guid"=>"http://172.17.0.2/?p=4",
  "menu_order"=>0,
  "comment_status"=>"open",
  "ping_status"=>"open",
  "sticky"=>false,
  "post_thumbnail"=>[],
  "post_format"=>"standard",
  "terms"=>
   [{"term_id"=>"1", "name"=>"Uncategorized", "slug"=>"uncategorized", "term_group"=>"0", "term_taxonomy_id"=>"1", "taxonomy"=>"category", "description"=>"", "parent"=>"0", "count"=>2, "filter"=>"raw"}],
  "custom_fields"=>[]},
 {"post_id"=>"1",
  "post_title"=>"Hello world!",
  "post_date"=>#<XMLRPC::DateTime:0x00000002735580 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_date_gmt"=>#<XMLRPC::DateTime:0x0000000226b130 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified"=>#<XMLRPC::DateTime:0x00000002268de0 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified_gmt"=>#<XMLRPC::DateTime:0x000000021aea58 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_status"=>"publish",
  "post_type"=>"post",
  "post_name"=>"hello-world",
  "post_author"=>"1",
  "post_password"=>"",
  "post_excerpt"=>"",
  "post_content"=>"Welcome to WordPress. This is your first post. Edit or delete it, then start writing!",
  "post_parent"=>"0",
  "post_mime_type"=>"",
  "link"=>"http://172.17.0.2/2015/11/01/hello-world/",
  "guid"=>"http://172.17.0.2/?p=1",
  "menu_order"=>0,
  "comment_status"=>"open",
  "ping_status"=>"open",
  "sticky"=>false,
  "post_thumbnail"=>[],
  "post_format"=>"standard",
  "terms"=>
   [{"term_id"=>"1", "name"=>"Uncategorized", "slug"=>"uncategorized", "term_group"=>"0", "term_taxonomy_id"=>"1", "taxonomy"=>"category", "description"=>"", "parent"=>"0", "count"=>2, "filter"=>"raw"}],
  "custom_fields"=>[]}]
>> 
>> # Create a new post!
>> post =
 | {    
 |   "post_title"     => 'Rubyfu vs WP XML-RPC',        
 |   "post_name"      => 'Rubyfu vs WordPress XML-RPC',        
 |   "post_content"   => 'This is Pragmatic Rubyfu Post. Thanks for reading',        
 |   "post_author"    => 2,        
 |   "post_status"    => 'publish',        
 |   "comment_status" => 'open'        
 | }      
=> {"post_title"=>"Rubyfu vs WP XML-RPC",
 "post_name"=>"Rubyfu vs WordPress XML-RPC",
 "post_content"=>"This is Pragmatic Rubyfu Post. Thanks for reading",
 "post_author"=>2,
 "post_status"=>"publish",
 "comment_status"=>"open"}
>> response = server.call("wp.newPost", 0, opts[:user], opts[:password], post)
=> "7"
>> # Retrieve created post
>> response =  server.call('wp.getPosts', 0, opts[:user], opts[:password], {"post_type" => "post", "post_status" => "published", "number" => "2", "offset" => "2"})
=> [{"post_id"=>"3",
  "post_title"=>"Auto Draft",
  "post_date"=>#<XMLRPC::DateTime:0x0000000225bcd0 @day=1, @hour=19, @min=22, @month=11, @sec=29, @year=2015>,
  "post_date_gmt"=>#<XMLRPC::DateTime:0x00000002259a98 @day=1, @hour=19, @min=22, @month=11, @sec=29, @year=2015>,
  "post_modified"=>#<XMLRPC::DateTime:0x0000000256b808 @day=1, @hour=19, @min=22, @month=11, @sec=29, @year=2015>,
  "post_modified_gmt"=>#<XMLRPC::DateTime:0x000000025695d0 @day=1, @hour=19, @min=22, @month=11, @sec=29, @year=2015>,
  "post_status"=>"auto-draft",
  "post_type"=>"post",
  "post_name"=>"",
  "post_author"=>"1",
  "post_password"=>"",
  "post_excerpt"=>"",
  "post_content"=>"",
  "post_parent"=>"0",
  "post_mime_type"=>"",
  "link"=>"http://172.17.0.2/?p=3",
  "guid"=>"http://172.17.0.2/?p=3",
  "menu_order"=>0,
  "comment_status"=>"open",
  "ping_status"=>"open",
  "sticky"=>false,
  "post_thumbnail"=>[],
  "post_format"=>"standard",
  "terms"=>[],
  "custom_fields"=>[]},
 {"post_id"=>"1",
  "post_title"=>"Hello world!",
  "post_date"=>#<XMLRPC::DateTime:0x00000002617298 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_date_gmt"=>#<XMLRPC::DateTime:0x00000002615038 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified"=>#<XMLRPC::DateTime:0x000000025e6d28 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified_gmt"=>#<XMLRPC::DateTime:0x000000025e4aa0 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_status"=>"publish",
  "post_type"=>"post",
  "post_name"=>"hello-world",
  "post_author"=>"1",
  "post_password"=>"",
  "post_excerpt"=>"",
  "post_content"=>"Welcome to WordPress. This is your first post. Edit or delete it, then start writing!",
  "post_parent"=>"0",
  "post_mime_type"=>"",
  "link"=>"http://172.17.0.2/2015/11/01/hello-world/",
  "guid"=>"http://172.17.0.2/?p=1",
  "menu_order"=>0,
  "comment_status"=>"open",
  "ping_status"=>"open",
  "sticky"=>false,
  "post_thumbnail"=>[],
  "post_format"=>"standard",
  "terms"=>
   [{"term_id"=>"1", "name"=>"Uncategorized", "slug"=>"uncategorized", "term_group"=>"0", "term_taxonomy_id"=>"1", "taxonomy"=>"category", "description"=>"", "parent"=>"0", "count"=>3, "filter"=>"raw"}],
  "custom_fields"=>[]}]
...skipping...
  "post_format"=>"standard",
  "terms"=>[],
  "custom_fields"=>[]},
 {"post_id"=>"1",
  "post_title"=>"Hello world!",
  "post_date"=>#<XMLRPC::DateTime:0x00000002617298 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_date_gmt"=>#<XMLRPC::DateTime:0x00000002615038 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified"=>#<XMLRPC::DateTime:0x000000025e6d28 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_modified_gmt"=>#<XMLRPC::DateTime:0x000000025e4aa0 @day=1, @hour=17, @min=54, @month=11, @sec=14, @year=2015>,
  "post_status"=>"publish",
  "post_type"=>"post",
  "post_name"=>"hello-world",
  "post_author"=>"1",
  "post_password"=>"",
  "post_excerpt"=>"",
  "post_content"=>"Welcome to WordPress. This is your first post. Edit or delete it, then start writing!",
  "post_parent"=>"0",
  "post_mime_type"=>"",
  "link"=>"http://172.17.0.2/2015/11/01/hello-world/",
  "guid"=>"http://172.17.0.2/?p=1",
  "menu_order"=>0,
  "comment_status"=>"open",
  "ping_status"=>"open",
  "sticky"=>false,
  "post_thumbnail"=>[],
  "post_format"=>"standard",
  "terms"=>
   [{"term_id"=>"1", "name"=>"Uncategorized", "slug"=>"uncategorized", "term_group"=>"0", "term_taxonomy_id"=>"1", "taxonomy"=>"category", "description"=>"", "parent"=>"0", "count"=>3, "filter"=>"raw"}],
  "custom_fields"=>[]}]

```

І ось наше нове повідомлення:
![](webfu__xmlrpc1.png)

Джерело: [HOW TO PROGRAMATICALLY CONTROL WORDPRESS WITH RUBY USING XML-RPC][3]

Більше про [WordPress XML-RPC][3]








<br><br><br>
---
[2]: http://ruby-doc.org/stdlib-2.2.3/libdoc/xmlrpc/rdoc/XMLRPC/Client.html
[3]: http://notes.jerzygangi.com/how-to-programatically-control-wordpress-with-ruby-using-xml-rpc/
[4]: https://codex.wordpress.org/XML-RPC_WordPress_API

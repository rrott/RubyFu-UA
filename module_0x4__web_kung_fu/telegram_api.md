# Telegram API

Як відомо, Telegram - програма обміну повідомленнями прив'язана до мобільного номеру користувачів. Telegram має свою API -*і Ruby має гем-обгортку до нього* [*Telegram's Bot API*](https://core.telegram.org/bots/api) *що називається [telegram-bot-ruby](https://github.com/atipugin/telegram-bot-ruby)* - що дохволяє інтегруватися  іншими сервісами, створювати утиліти, багатокористувацьки чи одиночні ігри, соціальні сервіси чи будь що інше. Відчули злу посмішку на обличчі? 

- Встановимо гем telegram-bot 
```
gem install telegram-bot-ruby
```


- Використання

Як і для більшості API вам потрібно мати [token](https://core.telegram.org/bots#botfather) для роботи з ботами. ось як його використовувати:  

```ruby
require 'telegram/bot'

token = 'YOUR_TELEGRAM_BOT_API_TOKEN'

Telegram::Bot::Client.run(token) do |bot|
  bot.listen do |message|
    case message.text
    when '/start'
      bot.api.send_message(chat_id: message.chat.id, text: "Hello, #{message.from.first_name}")
    when '/stop'
      bot.api.send_message(chat_id: message.chat.id, text: "Bye, #{message.from.first_name}")
    end
  end
end
```

- Боти
Якщо ви помітили що щле посміхаєтесь після перегляду вищенаведеного коду, ви можливо подумали про використання боту для виклику вашого власного боту з запитом зробити щось цікаве.

```ruby
require 'telegram/bot'

bot.listen do |message|
  case message
  when Telegram::Bot::Types::InlineQuery
    results = [
      Telegram::Bot::Types::InlineQueryResultArticle
        .new(id: 1, title: 'First article', message_text: 'Very interesting text goes here.'),
      Telegram::Bot::Types::InlineQueryResultArticle
        .new(id: 2, title: 'Second article', message_text: 'Another interesting text here.')
    ]
    bot.api.answer_inline_query(inline_query_id: message.id, results: results)
  when Telegram::Bot::Types::Message
    bot.api.send_message(chat_id: message.chat.id, text: "Hello, #{message.from.first_name}!")
  end
end
```

Є дуєе багато інформації про використання цього [гему](https://github.com/atipugin/telegram-bot-ruby) та [API](https://core.telegram.org/bots) до нього. 



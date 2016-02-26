# Співробітництво
Цю книгу видано за [CC BY-NC-SA ліцензією][0] а це значить, що вітається будь-яке співробітництво та дистрибуція, книга безкоштовна й це назавжди!

Увага!: Код в цій книжці було перевірено лише з Рубі версії 2.2.0 або новіше.

## Методи співробітництва
Є кілька видів співробітництва, які можуть допомогти цій книзі здобути кращих результатів: 

* Додавання цікавих фрагментів коду,
* Додавання більш розширеного тлумачення поточного коду.
* Поліпшення якості коду або додавання альтернатив.
* Поліпшення якості самої книги, в саме:
    * Структурні поліпшення
    * Вичитка та виправлення граматичних помилок
    * Зміна дизайну
    * Ідеї або пропозиції
    * таке інше
* Поширення книги в соціальних мережах та серед спеціалістів з Інформаційної Безпеки.
    * Twitter: [@Rubyfu][8] та хештег `#Rubyfu`
    * Google+: [Rubyfu page][9]
* Додавання більше ресурсів або посилань.
* 
* Contribution by adding more resources and references.
* Пожертвування


## Як саме?

### Початок співробітництва
Ви можете знайти все, що вам треба знати про GitBook та Markdown в розділі [Посилання][1]. Гарним стартом буде відвідування посилки ["як користуватися GitBook" з офіційного сайту][2]. Ви також можете використовувати [програму GitBook][3] для вашого комп'ютера.

1. Створіть новий аккаунт в [GitHub][5](якщо у вас його ще немає).
2. Щробіть форк сховища [RubyFu][4] або [RubyFu-UA][10]
3. Зробіть клон сховища RubyFu на вашому комп'ютері (`git clone https://github.com/[YourGithubAccount]/RubyFu`) 
4. Створіть аккаунт і [GitBook][6](якщо у вас його ще немає).
4. Відкрийте [**редактор GitBook**][3] та увійдіть в нього з вашим аккаунтом.
5. Натисніть кнопку **Import** щоб імпортувати проект в редактор. Після цього ви знайдете книгу в розділі **LOCAL LIBRARY**.
3. Додайте посилання на ваше форкнуте сховище:  **Toolbar** >> **File** >> **Preferences** >> **GIT**.
4. Внесіть свої зміни чи доповнення.
5. Після цього в редакторі GitBook натисніть кнопку **Sync** щоб вивантажити зміни в git сховище форкнуте вами на другому кроці.
6. З інтерфейсу GitHub, надішліть новий **Pull Request(PR)** в **Master** бренч.

Не знаєте чим саме допомогти? Перейдіть до [TODO list](contributors/todo.md) та перевірте що з наведеного ви можете зробити.

### Contributing with Code

##### Ruby code
* Use the triple ticks ` ``` `  followed by `ruby` then your code in between then ` ``` ` to get ruby code highlighted. e.g.

        ```ruby
        puts "Ruby Code here"
        ```
* Explain the main idea -with some details- of the code, if you explain every line that would be great but it's not a must.
* Choose the correct Module.
* Make your title clear.
* Use Text editor/ide for code identification before pasting your code
* Mention the source, if you copied or developed a code that created by others please mention the source in the footer. e.g.

        ```ruby
        puts "Your good code"
        ```
        [Source][1]
    Then add the following to the footer

        [1]: http://TheSouceCodeURL

    Your notes should be under the footer's line. Add the following to initiate the footer if it does not yet exist

        <br><br><br>
        ---
        YOUR NOTES SHALL BE HERE

* Try to use readable code, if you have to add more tricky/skilled code then explain it well
    > **Remember!** Hacker's code **=!** Cryptic code


##### Command-line
Use triple ticks to highlight your command-line. ex. 
    ```
    ls
    ``` 


### General Contribution
General contribution might be topic requests, proofreading, spilling, book organization and style. All these contributions are welcome however has to be discussed on [Rubyfu issues][7] especially things regarding to topics and/or book organization and styling. At the same time don't hesitate to report even single word observation about the book, it's for you at the end of the day.


> **Note:** Since this book is enhanced dynamically and unordered, it's hard to make the footer notes with order series of numbers for the whole book so -until I find better solution- I'll make the number order separated for each page separately. 


<br><br><br>
---
[0]: https://creativecommons.org/licenses/by-nc-sa/3.0/
[1]: references/README.md
[2]: https://github.com/GitbookIO/gitbook
[3]: https://www.gitbook.com/editor
[4]: https://github.com/rubyfu/RubyFu
[5]: https://github.com
[6]: http://gitbook.com
[7]: https://github.com/rubyfu/RubyFu/issues
[8]: https://twitter.com/Rubyfu
[9]: https://plus.google.com/114358908164154763697
[10]: https://github.com/rrott/RubyFu-UA





\# Специфікація: Інтернет-магазин книг



\## Сутності та атрибути

1\. \*\*Customer\*\* (Клієнт): id(int), name(string), email(string).

2\. \*\*Book\*\* (Книга): id(int), title(string), price(DECIMAL).

3\. \*\*Author\*\* (Автор): id(int), name(string), country(string).

4\. \*\*Order\*\* (Замовлення): id(int), order\_date(date), status(string), total\_amount(DECIMAL).

5\. \*\*OrderItem\*\* (Позиція замовлення): quantity, price\_at\_purchase.



\## Зв'язки

* Клієнт може мати багато замовлень (1:N).
* Книга може мати кількох авторів, а автор написати кілька книг (M:N).
* Замовлення містить багато книг, книга може бути в багатьох замовленнях (M:N). Цей зв'язок реалізується через асоціативну сутність OrderItem, оскільки вона має власні атрибути.



\## Критерії прийняття

* Для всіх грошових полів (price, price\_at\_purchase, total\_amount) використовувати тип DECIMAL.


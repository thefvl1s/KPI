# DEFENSE - Завдання 3: Взаємодія (Sequence Diagrams)

**Намір і критерії:**
Деталізувати внутрішню взаємодію компонентів інтернет-магазину книг для всіх 4 сценаріїв UC «Оформлення замовлення (Checkout)» із Завдання 2. Критерії: окрема Sequence-діаграма (+ PNG) на кожен сценарій, узгодженість учасників із ER-моделлю (LAB1) та кроків із Gherkin (LAB2), коректні activation bars та комбіновані фрагменти (`alt`/`opt`/`loop`).

**Як діаграми деталізують сценарії та узгоджуються з моделлю даних:**
- Учасники-репозиторії (`CartRepository`, `BookRepository`, `OrderRepository`) напряму відповідають сутностям ER-моделі з LAB1 (`Book.price`, `Order.order_date`, `Order.status`, `Order.total_amount`, `OrderItem.price_at_purchase`).
- У Сценарії 1 цикл `loop` реалізує зв'язок `1:N` між `Order` та `OrderItem`, фіксуючи історичну ціну `price_at_purchase` з `BookRepository`.
- Блокування (`lockCart()`) та очищення/розблокування кошика (`clearCart()` / `unlockCart()`) в усіх платіжних сценаріях реалізують вимогу REQ-4 із LAB2.
- У Сценаріях 2 та 3 статус `Order` у БД не змінюється (залишається «Очікує на оплату»), а в Сценарії 3 явно показано тайм-аут `> 30 с` (`TimeoutException`).

**Топ-3 розбіжності (spec / моделі ↔ артефакт AI) + коміт-виправлення:**
1. *Відсутність атрибута `order_date` у Сценарії 1:* при першій генерації `OrderRepository.createOrder()` не фіксував `order_date`, що суперечило ER-моделі LAB1. Я оновив `LAB3/spec.md` і додав передачу `order_date` у `scenario1_success.puml`. (Коміт: `29acadc`)
2. *Порушення stateless-логіки `OrderService` у Сценарії 1:* після деактивації сервісу між `createOrder()` та `processPayment()` діаграма передавала `total_amount` у `PaymentGateway.charge()` без читання замовлення з БД. Я уточнив крок у `LAB3/spec.md` та додав `OrderRepository.getOrderById(orderId)` у `scenario1_success.puml`. (Коміт: `3db7809`)
3. *Некоректна семантика `alt` у Сценаріях 2 та 4:* AI використав фрагмент `alt` з однією гілкою без `else`. Я зафіксував правило в `LAB3/spec.md`, замінив `alt` на `opt` у `scenario2_insufficient_funds.puml` і додав гілку `else` у `scenario4_empty_cart.puml`. (Коміт: `a1a511c`)

**Перевірка activation bars:**
У всіх 4 діаграмах активації (`activate`/`deactivate`) збалансовані: `CheckoutUI` та `OrderService` активуються до входу в умовні фрагменти (`opt`/`alt`/`loop`) і деактивуються після їх завершення, не обриваючись усередині окремої гілки.
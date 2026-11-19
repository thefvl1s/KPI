# Слід роботи зі ШІ (Prompts Trace) — Завдання 3

Посилання на чат: https://share.gemini.google/p7GjN0y5ZOmj

Примітка (DoD §1): `LAB3/spec.md` та `LAB3/DEFENSE.md` написані та відредаговані мною власноруч. AI використовувався для аналізу прогалин,форматування і перефразування тексту в файлах, постановки уточнювальних питань і генерації коду PlantUML за моєю специфікацією.

## Крок 1: Аналіз власноруч написаного spec.md
**Мій промпт (вхідний текст написано власноруч):**
Я роблю Завдання (Sequence Diagrams у PlantUML) для інтернет магазину книг Нижче мій spec.md.
Поки що НЕ генеруй діаграми і НЕ переписуй spec.md
Проаналізуй мій текст знайди прогалини у взаємодії між компонентами (наприклад, щодо роботи з БД, блокування кошика під час оплати, циклів або обробки помилок) і постав мені 3–5 конкретних уточнювальних питань, щоб я міг допрацювати специфікацію перед генерацією діаграм.

# Специфікація: Взаємодія для UC «Оформлення замовлення (Checkout)»

## Контекст (із Завдання 1 та 2)
Обраний прецедент: **Оформлення замовлення (Checkout)** в інтернет-магазині книг.
Ключові сутності з ER-моделі (LAB1): `Customer`, `Book`, `Order` (статуси: "Очікує на оплату", "Оплачено", `total_amount: DECIMAL`), `OrderItem` (`quantity`, `price_at_purchase: DECIMAL`).

## Учасники взаємодії (Participants)
- `Customer` (Актор) — авторизований клієнт.
- `CheckoutUI` — інтерфейс сторінки оформлення замовлення.
- `OrderService` — бізнес-логіка створення замовлення та перевірки кошика.
- `CartRepository` — доступ до поточного кошика клієнта.
- `OrderRepository` — збереження сутностей `Order` та `OrderItem` у БД.
- `PaymentGateway` — зовнішній платіжний шлюз.

## Сценарії з Gherkin (LAB2) та кроки

### Сценарій 1: Успішне оформлення замовлення та оплата
1. Клієнт натискає «Оформити замовлення» в `CheckoutUI`.
2. `OrderService` перевіряє кошик через `CartRepository`, створює `Order` зі статусом «Очікує на оплату» та зберігає `OrderItem` із фіксацією `price_at_purchase`.
3. Клієнт вводить платіжні дані та підтверджує оплату.
4. Система відправляє запит на `PaymentGateway`, отримує «Успішно» і змінює статус `Order` на «Оплачено».
5. Користувач бачить підтвердження замовлення.

### Сценарій 2: Відхилення платежу через недостатню кількість коштів
1. Замовлення `Order` вже існує у статусі «Очікує на оплату».
2. Клієнт підтверджує оплату.
3. `PaymentGateway` повертає помилку «Недостатньо коштів».
4. `OrderService` залишає статус «Очікує на оплату» і повертає повідомлення про помилку в `CheckoutUI`.

### Сценарій 3: Тайм-аут платіжного шлюзу
1. Клієнт підтверджує оплату для існуючого `Order`.
2. `PaymentGateway` не відповідає протягом 30 секунд.
3. Система перериває запит, залишає статус «Очікує на оплату» та показує «Сервіс тимчасово недоступний».

### Сценарій 4: Спроба чекауту з порожнім кошиком
1. Клієнт намагається перейти до чекауту.
2. `OrderService` перевіряє кошик у `CartRepository` — він порожній.
3. `CheckoutUI` блокує кнопку «Оформити замовлення» та показує «Додайте товари до кошика».

## Критерії прийняття (для Завдання 3)
- На кожен із 4 сценаріїв створена окрема Sequence-діаграма у PlantUML (+ рендер PNG).
- У кожній діаграмі є хоча б один комбінований фрагмент (`alt`, `opt` або `loop`).
- Учасники та виклики узгоджені з ER-моделлю (LAB1) та Gherkin (LAB2).
- Використовуються реалістичні назви методів (наприклад, `createOrder()`, `processPayment()`).
- Activation bars коректні (не обриваються всередині гілок `alt`/`opt`).

**Повна відповідь AI:**
Твоя специфікація має чітку структуру і добре узгоджена з артефактами попередніх робіт (ER-модель, Gherkin). Однак під час переведення цього тексту в детальні Sequence-діаграми виникне кілька «білих плям» у логіці та послідовності викликів між об'єктами.

Ось 5 конкретних питань, які допоможуть усунути ці прогалини перед побудовою діаграм:

Отримання актуальних цін та перевірка наявності (Book / BookRepository):
У списку учасників немає BookRepository (хоча сутність Book згадана в контексті). Звідки OrderService бере поточну ціну книги, щоб зафіксувати її в price_at_purchase, та як розраховує total_amount? Ці дані вже зберігаються всередині об'єкта кошика (CartRepository), чи потрібен окремий запит до бази книг (а заодно й перевірка, чи є потрібна кількість примірників у наявності)?

Життєвий цикл кошика під час та після чекауту:
Коли саме очищається кошик у CartRepository — одразу після створення замовлення у статусі «Очікує на оплату» чи лише після отримання статусу «Оплачено» від PaymentGateway? Також варто уточнити, чи блокується кошик від змін, поки клієнт вводить платіжні дані (між кроками 2 і 3 у Сценарії 1), щоб уникнути розбіжності між товарами в кошику та вже створеним Order.

Деталізація збереження позицій замовлення (loop та транзакційність):
У Сценарії 1 вказано: «створює Order ... та зберігає OrderItem». Для діаграми важливо знати: OrderService викликає один пакетний метод у OrderRepository (наприклад, saveOrderWithItems(order, items)), чи це відбувається в циклі loop [для кожного товару в кошику] з окремими викликами addOrderItem()?

Повторна оплата в Сценаріях 2 та 3 (початковий стан системи):
Сценарії 2 і 3 починаються з того, що Order «вже існує у статусі Очікує на оплату». Чи означає це, що діаграми для цих сценаріїв мають починатися одразу з моменту введення/підтвердження платіжних даних у CheckoutUI (без етапу перевірки кошика), чи перед викликом PaymentGateway система все ж робить запит OrderRepository.getOrderById(), щоб валідувати суму та статус замовлення?

Обробка асинхронної відповіді та тайм-ауту (PaymentGateway):
У Сценарії 3 система перериває запит через 30 секунд. Як саме це відобразити у взаємодії: OrderService робить синхронний виклик до PaymentGateway, який повертає TimeoutException (або використовується фрагмент break / opt [час очікування > 30с]), чи між OrderService і PaymentGateway є проміжний рівень (наприклад, PaymentService)? Також чи записується факт невдалої спроби оплати в БД (наприклад, у таблицю транзакцій або лог замовлення), чи в OrderRepository взагалі не відбувається жодних викликів у Сценаріях 2 та 3?

Відповідай на ці пункти в довільному форматі — на основі твоїх рішень ми зможемо одразу перейти до генерації коректного PlantUML-коду з правильними активаціями та комбінованими фрагментами.

---

## Крок 2: Мої відповіді на питання AI та перша генерація діаграм
**Мій промпт (мої рішення, внесені в spec.md):**
Ось мої рішення які я вже вніс у spec.md
1. Додав BookRepository - OrderService бере з нього актуальну Book.price для фіксації price_at_purchase та розрахунку total_amount
2. Кошик блокується (lockCart()) на час обробки платежу (згідно з REQ-4 із LAB2)очищається (clearCart()) лише після статусу Оплачено а при помилці/тайм-ауті - розблоковується (unlockCart())
3. Для збереження позицій використовуємо loop [для кожної книги в кошику] із викликами BookRepository та OrderRepository.saveOrderItem()
4. У Сценаріях 2 і 3 починаємо з підтвердження оплати але спочатку викликаємо OrderRepository.getOrderById() щоб перевірити статус Очікує на оплату та суму
5. OrderService звертається до PaymentGateway напряму (без PaymentService). При помилці або тайм-ауті > 30 с статус у БД не змінюється.
Тепер згенеруй 4 окремі Sequence-діаграми у синтаксисі PlantUML (по одній на кожен сценарій):
- scenario1_success.puml
- scenario2_insufficient_funds.puml
- scenario3_timeout.puml
- scenario4_empty_cart.puml
У кожній діаграмі обов'язково використай реалістичні назви методів return-повідомлення activation bars (activate/deactivate) та мінімум один комбінований фрагмент (alt/opt/loop)

**Повна відповідь AI:**
Нижче наведено 4 окремі Sequence-діаграми у синтаксисі PlantUML, узгоджені з твоїми рішеннями. У всіх діаграмах витримано симетрію activate/deactivate (активації не перетинають межі комбінованих фрагментів) та використано єдину термінологію сутностей і методів.

1. scenario1_success.puml
@startuml
title Сценарій 1: Успішне оформлення замовлення та оплата

actor Customer
participant CheckoutUI
participant OrderService
database CartRepository
database BookRepository
database OrderRepository
participant PaymentGateway

Customer -> CheckoutUI: clickCheckout()
activate CheckoutUI

CheckoutUI -> OrderService: createOrder(customerId)
activate OrderService

OrderService -> CartRepository: getCartItems(customerId)
activate CartRepository
CartRepository --> OrderService: cartItems
deactivate CartRepository

OrderService -> OrderRepository: createOrder(customerId, "Очікує на оплату")
activate OrderRepository
OrderRepository --> OrderService: orderId
deactivate OrderRepository

loop для кожної книги в кошику
  OrderService -> BookRepository: getBookPrice(bookId)
  activate BookRepository
  BookRepository --> OrderService: price
  deactivate BookRepository

  OrderService -> OrderRepository: saveOrderItem(orderId, bookId, quantity, price_at_purchase)
  activate OrderRepository
  OrderRepository --> OrderService: itemSaved
  deactivate OrderRepository
end

OrderService -> OrderRepository: updateTotalAmount(orderId, total_amount)
activate OrderRepository
OrderRepository --> OrderService: updated
deactivate OrderRepository

OrderService --> CheckoutUI: orderSummary(orderId, total_amount)
deactivate OrderService

CheckoutUI --> Customer: showPaymentForm(total_amount)
deactivate CheckoutUI

Customer -> CheckoutUI: submitPayment(orderId, paymentDetails)
activate CheckoutUI

CheckoutUI -> OrderService: processPayment(orderId, paymentDetails)
activate OrderService

OrderService -> CartRepository: lockCart(customerId)
activate CartRepository
CartRepository --> OrderService: cartLocked
deactivate CartRepository

OrderService -> PaymentGateway: charge(orderId, total_amount, paymentDetails)
activate PaymentGateway
PaymentGateway --> OrderService: paymentSuccess(transactionId)
deactivate PaymentGateway

opt оплата успішна
  OrderService -> OrderRepository: updateOrderStatus(orderId, "Оплачено")
  activate OrderRepository
  OrderRepository --> OrderService: statusUpdated
  deactivate OrderRepository

  OrderService -> CartRepository: clearCart(customerId)
  activate CartRepository
  CartRepository --> OrderService: cartCleared
  deactivate CartRepository
end

OrderService --> CheckoutUI: paymentConfirmed(orderId)
deactivate OrderService

CheckoutUI --> Customer: showOrderConfirmation(orderId)
deactivate CheckoutUI
@enduml

2. scenario2_insufficient_funds.puml
@startuml
title Сценарій 2: Відхилення платежу через недостатню кількість коштів

actor Customer
participant CheckoutUI
participant OrderService
database OrderRepository
database CartRepository
participant PaymentGateway

Customer -> CheckoutUI: submitPayment(orderId, paymentDetails)
activate CheckoutUI

CheckoutUI -> OrderService: processPayment(orderId, paymentDetails)
activate OrderService

OrderService -> OrderRepository: getOrderById(orderId)
activate OrderRepository
OrderRepository --> OrderService: order(status="Очікує на оплату", total_amount)
deactivate OrderRepository

OrderService -> CartRepository: lockCart(customerId)
activate CartRepository
CartRepository --> OrderService: cartLocked
deactivate CartRepository

OrderService -> PaymentGateway: charge(orderId, total_amount, paymentDetails)
activate PaymentGateway
PaymentGateway --> OrderService: paymentDeclined("Недостатньо коштів")
deactivate PaymentGateway

alt платіж відхилено (недостатньо коштів)
  OrderService -> CartRepository: unlockCart(customerId)
  activate CartRepository
  CartRepository --> OrderService: cartUnlocked
  deactivate CartRepository

  note right of OrderService
    Статус замовлення в БД не змінюється
    ("Очікує на оплату")
  end note
end

OrderService --> CheckoutUI: paymentError("Недостатньо коштів")
deactivate OrderService

CheckoutUI --> Customer: showErrorMessage("Недостатньо коштів")
deactivate CheckoutUI
@enduml

3. scenario3_timeout.puml
@startuml
title Сценарій 3: Тайм-аут платіжного шлюзу

actor Customer
participant CheckoutUI
participant OrderService
database OrderRepository
database CartRepository
participant PaymentGateway

Customer -> CheckoutUI: submitPayment(orderId, paymentDetails)
activate CheckoutUI

CheckoutUI -> OrderService: processPayment(orderId, paymentDetails)
activate OrderService

OrderService -> OrderRepository: getOrderById(orderId)
activate OrderRepository
OrderRepository --> OrderService: order(status="Очікує на оплату", total_amount)
deactivate OrderRepository

OrderService -> CartRepository: lockCart(customerId)
activate CartRepository
CartRepository --> OrderService: cartLocked
deactivate CartRepository

OrderService -> PaymentGateway: charge(orderId, total_amount, paymentDetails)
activate PaymentGateway
... очікування відповіді > 30 секунд ...
PaymentGateway --x OrderService: TimeoutException
deactivate PaymentGateway

opt тайм-аут > 30 с
  OrderService -> CartRepository: unlockCart(customerId)
  activate CartRepository
  CartRepository --> OrderService: cartUnlocked
  deactivate CartRepository

  note right of OrderService
    Запит перервано. Статус у БД
    залишається "Очікує на оплату"
  end note
end

OrderService --> CheckoutUI: serviceUnavailableError("Сервіс тимчасово недоступний")
deactivate OrderService

CheckoutUI --> Customer: showErrorMessage("Сервіс тимчасово недоступний")
deactivate CheckoutUI
@enduml

4. scenario4_empty_cart.puml
@startuml
title Сценарій 4: Спроба чекауту з порожнім кошиком

actor Customer
participant CheckoutUI
participant OrderService
database CartRepository

Customer -> CheckoutUI: openCheckoutPage()
activate CheckoutUI

CheckoutUI -> OrderService: validateCart(customerId)
activate OrderService

OrderService -> CartRepository: getCartItems(customerId)
activate CartRepository
CartRepository --> OrderService: cartItems = []
deactivate CartRepository

OrderService --> CheckoutUI: cartValidationResult(isEmpty=true)
deactivate OrderService

alt кошик порожній
  CheckoutUI -> CheckoutUI: disableCheckoutButton()
  CheckoutUI --> Customer: showMessage("Додайте товари до кошика")
end

deactivate CheckoutUI
@enduml

---

## Крок 3: Аудит та виправлення топ-3 розбіжностей
**Мій промпт (виявлені розбіжності та вимоги до перегенерації):**
Я провів аудит згенерованих діаграм на відповідність ER-моделі (LAB1) сценаріям (LAB2) та стандарту UML і знайшов 3 розбіжності Я оновлюю ці правила у своєму spec.md:
1. Узгодженість з ER-моделлю (order_date) У scenario1_success.puml при створенні Order відсутнє поле order_date (воно є в сутності Order із Завдання 1) Додай фіксацію order_date у виклик createOrder(customerId, order_date, "Очікує на оплату").
2. Stateless-логіка OrderService у scenario1_success.puml Між кроками createOrder і submitPayment сервіс OrderService деактивуєтьсяТому при processPayment(orderId, paymentDetails) він не може знати total_amount без запиту до БД Додай виклик OrderRepository.getOrderById(orderId) перед lockCart() та charge(), як це зроблено у Сценаріях 2 і 3
3. Семантика alt у scenario2_insufficient_funds.puml та scenario4_empty_cart.puml Фрагмент alt використано з однією гілкою без else У Сценарії 2 заміни alt на opt [платіж відхилено (недостатньо коштів)]а у Сценарії 4 додай гілку else [кошик містить товари] (де CheckoutUI активує кнопку оформлення замовлення) зберігши коректні activate/deactivate
Перегенеруй лише ці 3 виправлені файли (scenario1_success.puml, scenario2_insufficient_funds.puml, scenario4_empty_cart.puml) scenario3_timeout.puml чіпати не потрібно

**Повна відповідь AI:**
Нижче наведено оновлені файли з урахуванням усіх трьох правок (узгодженість з ER-моделлю, stateless-виклик getOrderById() та коректна семантика opt/alt).

1. scenario1_success.puml
@startuml
title Сценарій 1: Успішне оформлення замовлення та оплата

actor Customer
participant CheckoutUI
participant OrderService
database CartRepository
database BookRepository
database OrderRepository
participant PaymentGateway

Customer -> CheckoutUI: clickCheckout()
activate CheckoutUI

CheckoutUI -> OrderService: createOrder(customerId)
activate OrderService

OrderService -> CartRepository: getCartItems(customerId)
activate CartRepository
CartRepository --> OrderService: cartItems
deactivate CartRepository

OrderService -> OrderRepository: createOrder(customerId, order_date, "Очікує на оплату")
activate OrderRepository
OrderRepository --> OrderService: orderId
deactivate OrderRepository

loop для кожної книги в кошику
  OrderService -> BookRepository: getBookPrice(bookId)
  activate BookRepository
  BookRepository --> OrderService: price
  deactivate BookRepository

  OrderService -> OrderRepository: saveOrderItem(orderId, bookId, quantity, price_at_purchase)
  activate OrderRepository
  OrderRepository --> OrderService: itemSaved
  deactivate OrderRepository
end

OrderService -> OrderRepository: updateTotalAmount(orderId, total_amount)
activate OrderRepository
OrderRepository --> OrderService: updated
deactivate OrderRepository

OrderService --> CheckoutUI: orderSummary(orderId, total_amount)
deactivate OrderService

CheckoutUI --> Customer: showPaymentForm(total_amount)
deactivate CheckoutUI

Customer -> CheckoutUI: submitPayment(orderId, paymentDetails)
activate CheckoutUI

CheckoutUI -> OrderService: processPayment(orderId, paymentDetails)
activate OrderService

OrderService -> OrderRepository: getOrderById(orderId)
activate OrderRepository
OrderRepository --> OrderService: order(status="Очікує на оплату", total_amount)
deactivate OrderRepository

OrderService -> CartRepository: lockCart(customerId)
activate CartRepository
CartRepository --> OrderService: cartLocked
deactivate CartRepository

OrderService -> PaymentGateway: charge(orderId, total_amount, paymentDetails)
activate PaymentGateway
PaymentGateway --> OrderService: paymentSuccess(transactionId)
deactivate PaymentGateway

opt оплата успішна
  OrderService -> OrderRepository: updateOrderStatus(orderId, "Оплачено")
  activate OrderRepository
  OrderRepository --> OrderService: statusUpdated
  deactivate OrderRepository

  OrderService -> CartRepository: clearCart(customerId)
  activate CartRepository
  CartRepository --> OrderService: cartCleared
  deactivate CartRepository
end

OrderService --> CheckoutUI: paymentConfirmed(orderId)
deactivate OrderService

CheckoutUI --> Customer: showOrderConfirmation(orderId)
deactivate CheckoutUI
@enduml

2. scenario2_insufficient_funds.puml
@startuml
title Сценарій 2: Відхилення платежу через недостатню кількість коштів

actor Customer
participant CheckoutUI
participant OrderService
database OrderRepository
database CartRepository
participant PaymentGateway

Customer -> CheckoutUI: submitPayment(orderId, paymentDetails)
activate CheckoutUI

CheckoutUI -> OrderService: processPayment(orderId, paymentDetails)
activate OrderService

OrderService -> OrderRepository: getOrderById(orderId)
activate OrderRepository
OrderRepository --> OrderService: order(status="Очікує на оплату", total_amount)
deactivate OrderRepository

OrderService -> CartRepository: lockCart(customerId)
activate CartRepository
CartRepository --> OrderService: cartLocked
deactivate CartRepository

OrderService -> PaymentGateway: charge(orderId, total_amount, paymentDetails)
activate PaymentGateway
PaymentGateway --> OrderService: paymentDeclined("Недостатньо коштів")
deactivate PaymentGateway

opt платіж відхилено (недостатньо коштів)
  OrderService -> CartRepository: unlockCart(customerId)
  activate CartRepository
  CartRepository --> OrderService: cartUnlocked
  deactivate CartRepository

  note right of OrderService
    Статус замовлення в БД не змінюється
    ("Очікує на оплату")
  end note
end

OrderService --> CheckoutUI: paymentError("Недостатньо коштів")
deactivate OrderService

CheckoutUI --> Customer: showErrorMessage("Недостатньо коштів")
deactivate CheckoutUI
@enduml

3. scenario4_empty_cart.puml
@startuml
title Сценарій 4: Спроба чекауту з порожнім кошиком

actor Customer
participant CheckoutUI
participant OrderService
database CartRepository

Customer -> CheckoutUI: openCheckoutPage()
activate CheckoutUI

CheckoutUI -> OrderService: validateCart(customerId)
activate OrderService

OrderService -> CartRepository: getCartItems(customerId)
activate CartRepository
CartRepository --> OrderService: cartItems
deactivate CartRepository

OrderService --> CheckoutUI: cartValidationResult(isEmpty)
deactivate OrderService

alt кошик порожній (isEmpty == true)
  CheckoutUI -> CheckoutUI: disableCheckoutButton()
  CheckoutUI --> Customer: showMessage("Додайте товари до кошика")
else кошик містить товари (isEmpty == false)
  CheckoutUI -> CheckoutUI: enableCheckoutButton()
  CheckoutUI --> Customer: showCheckoutForm()
end

deactivate CheckoutUI
@enduml
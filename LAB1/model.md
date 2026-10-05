erDiagram

&#x20;   Customer {

&#x20;       int id PK

&#x20;       string name

&#x20;       string email

&#x20;   }

&#x20;   

&#x20;   Book {

&#x20;       int id PK

&#x20;       string title

&#x20;       DECIMAL price

&#x20;   }

&#x20;   

&#x20;   Author {

&#x20;       int id PK

&#x20;       string name

&#x20;       string country

&#x20;   }

&#x20;   

&#x20;   Order {

&#x20;       int id PK

&#x20;       date order\_date

&#x20;       string status

&#x20;       DECIMAL total\_amount

&#x20;       int customer\_id FK

&#x20;   }

&#x20;   

&#x20;   OrderItem {

&#x20;       int order\_id PK, FK

&#x20;       int book\_id PK, FK

&#x20;       int quantity

&#x20;       DECIMAL price\_at\_purchase

&#x20;   }



&#x20;   %% Зв'язки

&#x20;   Customer ||--o{ Order : "має (1:N)"

&#x20;   Author }|--|{ Book : "написав (M:N)"

&#x20;   Order ||--|{ OrderItem : "містить (1:N до асоціативної)"

&#x20;   Book ||--o{ OrderItem : "включена в (1:N до асоціативної)"


erDiagram

&#x20;   CUSTOMER {

&#x20;       int id PK

&#x20;       string name

&#x20;       string email

&#x20;   }

&#x20;   

&#x20;   BOOK {

&#x20;       int id PK

&#x20;       string title

&#x20;       DECIMAL price

&#x20;   }

&#x20;   

&#x20;   AUTHOR {

&#x20;       int id PK

&#x20;       string name

&#x20;       string country

&#x20;   }

&#x20;   

&#x20;   ORDER {

&#x20;       int id PK

&#x20;       date order\_date

&#x20;       string status

&#x20;       DECIMAL total\_amount

&#x20;       int customer\_id FK

&#x20;   }

&#x20;   

&#x20;   ORDER\_ITEM {

&#x20;       int order\_id PK, FK

&#x20;       int book\_id PK, FK

&#x20;       int quantity

&#x20;       DECIMAL price\_at\_purchase

&#x20;   }



&#x20;   %% Зв'язки

&#x20;   CUSTOMER ||--o{ ORDER : "має (1:N)"

&#x20;   AUTHOR }|--|{ BOOK : "написав (M:N)"

&#x20;   ORDER ||--|{ ORDER\_ITEM : "містить (1:N до асоціативної)"

&#x20;   BOOK ||--o{ ORDER\_ITEM : "включена в (1:N до асоціативної)"


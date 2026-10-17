# Диаграммы проекта

Диаграммы описывают проектируемую структуру из `architecture.md` и процесс UC-01. Они подготовлены для этой версии документации. Блоки Mermaid можно просматривать в GitHub.

## Компоненты

```mermaid
flowchart TB
    Client[Клиент] --> Browser[Браузер и Bootstrap]
    Trainer[Кинолог] --> Browser
    Admin[Администратор] --> Browser
    Browser --> Django[Приложение Django]
    Django --> Accounts[Учётные записи]
    Django --> Pets[Питомцы]
    Django --> Schedule[Расписание]
    Django --> Booking[Бронирование]
    Accounts --> DB[(SQLite)]
    Pets --> DB
    Schedule --> DB
    Booking --> DB
```

## Связи сущностей

```mermaid
erDiagram
    USER ||--o{ PET : owns
    USER ||--o| TRAINER : profile
    TRAINER ||--o{ SESSION : teaches
    HALL ||--o{ SESSION : hosts
    PET ||--o{ BOOKING : has
    SESSION ||--o{ BOOKING : contains
```

Каждый питомец принадлежит одному пользователю. Каждое занятие проводится одним кинологом в одном зале. Каждая запись относится к одному питомцу и одному занятию.

## Подтверждение бронирования

```mermaid
sequenceDiagram
    actor Client as Клиент
    participant UI as Интерфейс
    participant App as Django
    participant DB as База данных
    Client->>UI: Выбрать занятие и своего питомца
    Client->>UI: Подтвердить запись
    Note over UI: «Сохранение», повторное нажатие заблокировано
    UI->>App: Запрос бронирования
    App->>App: Проверить права и время начала
    App->>DB: Проверить дубликат и место при сохранении
    alt Запись допустима
        DB-->>App: Запись сохранена
        App-->>UI: Номер и статус записи
        alt Подтверждение доставлено
            UI-->>Client: Запись подтверждена
        else Ответ потерян
            Note over UI: «Результат не подтверждён», ввод сохранён
            Client->>UI: После восстановления связи открыть «Мои записи»
            UI->>App: Запрос своих записей
            App->>DB: Прочитать запись по питомцу и занятию
            DB-->>App: Существующая запись
            App-->>UI: Номер и статус записи
            UI-->>Client: Показать сохранённую запись
        end
    else Нет места или есть дубликат
        DB-->>App: Запись не создана
        App-->>UI: Причина отказа
        UI-->>Client: Выбрать другое занятие или открыть запись
    end
```

## Диаграмма проектных классов

Диаграмма описывает модель документации, а не наличие реализованных классов Python. Классы соответствуют одноимённым сущностям из архитектуры; AGE_MONTHS означает возраст в полных месяцах, START_AT и END_AT — моменты начала и окончания.

```mermaid
classDiagram
    class User {
        ID id
        string name
        string contact
        Role role
    }
    class Pet {
        ID id
        ID owner_id
        string name
        string breed
        int age_months
        string features
    }
    class Trainer {
        ID id
        ID user_id
        string profile
    }
    class Hall {
        ID id
        string name
    }
    class Session {
        ID id
        string kind
        datetime start_at
        datetime end_at
        ID trainer_id
        ID hall_id
        int capacity
    }
    class Booking {
        ID id
        ID pet_id
        ID session_id
        Status status
        datetime created_at
    }
    User "1" --> "0..*" Pet : owns
    User "1" --> "0..1" Trainer : profile
    Trainer "1" --> "0..*" Session : teaches
    Hall "1" --> "0..*" Session : hosts
    Pet "1" --> "0..*" Booking : has
    Session "1" --> "0..*" Booking : contains
```

Pet.name: длина после удаления крайних пробелов 1–60; Pet.age_months: необязательное целое ≥ 0. Session.capacity: положительное целое; end_at > start_at. Booking.status принимает «Подтверждена», «Отменена», «Завершена». Role и Status — типы значений полей, а не дополнительные классы предметной области.

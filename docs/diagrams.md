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
    UI->>App: Запрос бронирования
    App->>App: Проверить права и время начала
    App->>DB: Проверить дубликат и место при сохранении
    alt Запись допустима
        DB-->>App: Запись сохранена
        App-->>UI: Номер и статус записи
        UI-->>Client: Запись подтверждена
    else Нет места или есть дубликат
        DB-->>App: Запись не создана
        App-->>UI: Причина отказа
        UI-->>Client: Выбрать другое занятие или открыть запись
    end
```

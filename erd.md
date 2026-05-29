```mermaid
erDiagram
    USERS {
        INTEGER id PK
        enum global_role
        VARCHAR username
        VARCHAR email
        VARCHAR password_hash
        VARCHAR bio
        TIMESTAMP last_seen
        BOOLEAN is_online
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    SESSIONS {
        INTEGER id PK
        INTEGER user_id FK
        VARCHAR refresh_token
        VARCHAR fingerprint
        TIMESTAMP expires_at
        TIMESTAMP created_at
    }

    CHATS {
        INTEGER id PK
        enum type
        VARCHAR title
        VARCHAR avatar_url
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    CHAT_PARTICIPANTS {
        INTEGER id PK
        INTEGER chat_id FK
        INTEGER user_id FK
        enum role
        TIMESTAMP joined_at
        TIMESTAMP last_read_message_time
    }

    MESSAGES {
        INTEGER id PK
        INTEGER chat_id FK
        INTEGER user_id FK
        TEXT text
        INTEGER reply_to_id FK
        BOOLEAN is_deleted
        TIMESTAMP created_at
        TIMESTAMP edited_at
    }

    ATTACHMENTS {
        INTEGER id PK
        INTEGER message_id FK
        TEXT file_url
        VARCHAR file_type
        VARCHAR file_name
        TIMESTAMP created_at
    }

    MESSAGE_REACTIONS {
        INTEGER id PK
        INTEGER message_id FK
        INTEGER user_id FK
        enum emoji
    }

    INVITE_LINKS {
        INTEGER id PK
        INTEGER chat_id FK
        VARCHAR token
        TIMESTAMP expires_at
        TIMESTAMP created_at
    }

    CHANNEL_SETTINGS {
        INTEGER id PK
        INTEGER chat_id FK
        VARCHAR description
        BOOLEAN is_private
    }

    REPORTS {
        INTEGER id PK
        INTEGER chat_id FK
        INTEGER user_id FK
        TEXT reason
        enum status
    }

    AVATARS {
        INTEGER id PK
        enum entity_type
        INTEGER entity_id FK
        VARCHAR avatar_url
        BOOLEAN is_primary
        TIMESTAMP created_at
    }

    USERS ||--o{ SESSIONS : ""
    USERS ||--o{ CHAT_PARTICIPANTS : ""
    USERS ||--o{ MESSAGES : ""
    USERS ||--o{ MESSAGE_REACTIONS : ""
    USERS ||--o{ REPORTS : ""

    CHATS ||--o{ CHAT_PARTICIPANTS : ""
    CHATS ||--o{ MESSAGES : ""
    CHATS ||--o{ INVITE_LINKS : ""
    CHATS ||--|| CHANNEL_SETTINGS : ""
    CHATS ||--o{ REPORTS : ""

    MESSAGES ||--o{ ATTACHMENTS : ""
    MESSAGES ||--o{ MESSAGE_REACTIONS : ""
    MESSAGES ||--o{ MESSAGES : ""

    USERS ||--o{ AVATARS : ""
    CHATS ||--o{ AVATARS : ""
```

Связь | Тип | Описание
| --- | --- | --- |
Users → Sessions | 1:N | Один пользователь может иметь несколько активных сессий (устройств/токенов)
Users → Chat_Participants | 1:N | Один пользователь может состоять во многих чатах
Users → Messages | 1:N | Один пользователь может отправлять множество сообщений
Users → Message_Reactions | 1:N | Один пользователь может ставить реакции на разные сообщения
Users → Reports | 1:N | Один пользователь может создавать несколько жалоб
Users → Avatars | 1:N | Один пользователь может иметь несколько аватаров 
Chats → Chat_Participants | 1:N | В одном чате может быть множество участников
Chats → Messages | 1:N | В одном чате хранится история из множества сообщений
Chats → Invite_Links | 1:N | У чата может быть несколько ссылок-приглашений (активных и истёкших)
Chats → Channel_Settings | 1:1 | У каждого канала свой один набор настроек
Chats → Reports | 1:N | На один чат могут поступать жалобы от разных пользователей
Chats → Avatars | 1:N | У чата может быть несколько аватаров (история смены обложки)
Messages → Attachments | 1:N | К одному сообщению можно прикрепить несколько файлов
Messages → Message_Reactions | 1:N | Одно сообщение может получать реакции от многих пользователей
Messages → Messages | 1:N | На одно сообщение может быть несколько ответов (хранится через reply_to_id) 
Users ↔ Chats | M:N | Через промежуточную таблицу Chat_Participants — пользователь вступает во многие чаты, в чате много пользователей


## Ответы на вопросы ДЗ 9

Чем связь 1:N отличается от M:N? Приведите пример каждой из вашего проекта.

Один ко многим значит что на одну запись в таблице, может ссылаться много записей из другой, например категория содержит много товаров. 
При записи многие ко многим создается дополнительная таблица которая хранит ключи с обоих таблиц, например Студент ходит на много курсов, и на курс ходит много студентов.

Почему связь M:N нельзя реализовать двумя таблицами? Зачем нужна промежуточная?

Потому что тогда надо в одной из таблиц хранить массив ключей из другой таблицы. И с этим почти невозможно работать когда очень много записей. БД не сможет гарантировать что все ключи из массива существуют. 

Что будет, если удалить запись, на которую ссылается FK? (Подумайте, мы разберём это на лекции)

То либо она удалиться вместе с ней CASCAD, либо в место неё будет null. Еще может просто запретить удаление если стоит настройка RESTRICT

Может ли FK быть NULL? Когда это полезно?

Да, если надо удалить на то что ссылается FK, но не надо удалять то что ссылается. Или если хранит необязательное поле то тоже может быть null в FK

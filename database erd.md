# Entity Relationship Diagram платформы

``` mermaid
erDiagram
    %% СХЕМА PERSONAL
    CREDENTIALS {
        uuid user_id PK
        varchar email UK
        varchar password_hash
        varchar role
        varchar refresh_token_hash
        timestamptz refresh_expires_at
        boolean is_blocked
        timestamptz created_at
        timestamptz updated_at
    }
    PROFILES {
        uuid user_id PK, FK
        varchar username UK
        text bio
        text avatar_url
        geography last_location "PostGIS Point (GiST)"
        timestamptz location_updated_at
        timestamptz created_at
        timestamptz updated_at
    }
    INTERESTS {
        serial id PK
        varchar slug UK
        varchar name UK
        varchar category
        timestamptz created_at
    }
    USER_INTERESTS {
        uuid user_id PK, FK
        int interest_id PK, FK
        timestamptz created_at
    }

    CREDENTIALS ||--|| PROFILES : "1:1"
    PROFILES ||--o{ USER_INTERESTS : "FK"
    INTERESTS ||--o{ USER_INTERESTS : "FK"

    %% СХЕМА EVENTS
    LOCATIONS {
        uuid id PK
        varchar name
        text address
        varchar fuzzy_district
        geography coordinates "PostGIS Point (GiST)"
        uuid created_by "logical -> PROFILES"
        varchar external_source_id UK
        timestamptz created_at
        timestamptz updated_at
    }
    EVENTS {
        uuid id PK
        uuid author_id "logical -> PROFILES"
        uuid location_id FK
        varchar title
        text description
        timestamptz starts_at
        timestamptz ends_at
        int max_participants
        varchar join_policy
        varchar status
        boolean is_crawled
        text external_url UK
        timestamptz created_at
        timestamptz updated_at
    }
    EVENT_TAGS {
        uuid event_id PK, FK
        int interest_id PK "logical -> INTERESTS"
        timestamptz created_at
    }
    EVENT_PARTICIPANTS {
        uuid id PK
        uuid event_id FK
        uuid user_id "logical -> PROFILES"
        varchar role
        varchar status
        timestamptz joined_at
        timestamptz updated_at
    }
    JOIN_REQUESTS {
        uuid id PK
        uuid event_id FK
        uuid user_id "logical -> PROFILES"
        varchar comment
        varchar status
        timestamptz created_at
        timestamptz updated_at
    }
    PINGS {
        uuid id PK
        uuid event_id FK
        uuid author_id "logical -> PROFILES"
        varchar message
        int radius_meters
        timestamptz expires_at
        timestamptz created_at
    }
    CHECK_INS {
        uuid id PK
        uuid event_id FK
        uuid user_id "logical -> PROFILES"
        geography coordinates "PostGIS Point"
        float distance_meters
        timestamptz created_at
    }

    LOCATIONS ||--o{ EVENTS : "FK"
    EVENTS ||--o{ EVENT_TAGS : "FK"
    EVENTS ||--o{ EVENT_PARTICIPANTS : "FK"
    EVENTS ||--o{ JOIN_REQUESTS : "FK"
    EVENTS ||--o{ PINGS : "FK"
    EVENTS ||--o{ CHECK_INS : "FK"

    %% СХЕМА CHAT
    EVENT_CHATS {
        uuid id PK
        uuid event_id UK "logical -> EVENTS"
        boolean is_active
        timestamptz last_message_at
        timestamptz created_at
        timestamptz updated_at
    }
    MESSAGES {
        uuid id PK
        uuid chat_id FK
        uuid user_id "logical -> PROFILES"
        varchar sender_username
        uuid reply_to_id FK "self-reference"
        varchar type "text | system"
        text body
        boolean is_deleted
        timestamptz created_at
        timestamptz edited_at
    }

    EVENT_CHATS ||--o{ MESSAGES : "FK"
    MESSAGES ||--o{ MESSAGES : "reply_to_id"

    %% СХЕМА NOTIFY
    NOTIFICATIONS {
        uuid id PK
        uuid user_id "logical -> PROFILES"
        varchar type
        varchar title
        text body
        jsonb payload
        boolean is_read
        timestamptz read_at
        timestamptz created_at
    }

    %% ЛОГИЧЕСКИЕ СВЯЗИ МЕЖДУ СХЕМАМИ
    PROFILES ||..o{ EVENTS : "author_id"
    PROFILES ||..o{ EVENT_PARTICIPANTS : "user_id"
    PROFILES ||..o{ MESSAGES : "user_id"
    PROFILES ||..o{ NOTIFICATIONS : "user_id"
    INTERESTS ||..o{ EVENT_TAGS : "interest_id"
    EVENTS ||..|| EVENT_CHATS : "event_id"
```
# 📊 Data Model: Private Family Communications App

## Overview
This schema supports secure messaging, family group management, user authentication, and media sharing. All sensitive fields are encrypted at rest and in transit.

---

## 🧑‍💻 Users

Stores user identity and authentication details.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| user_id       | UUID           | PK                               | Unique user identifier         |
| name          | VARCHAR(100)   | NOT NULL                         | Display name                   |
| email         | VARCHAR(255)   | UNIQUE, NOT NULL                 | Login credential               |
| phone         | VARCHAR(20)    | UNIQUE, NULLABLE                 | Optional secondary credential  |
| password_hash | TEXT           | NOT NULL                         | Hashed with bcrypt/scrypt      |
| created_at    | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | Account creation timestamp     |
| last_login    | TIMESTAMP      | NULLABLE                         | For session tracking           |

---

## 👨‍👩‍👧 Families

Defines family groups and ownership.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| family_id     | UUID           | PK                               | Unique family group ID         |
| name          | VARCHAR(100)   | NOT NULL                         | Family name                    |
| created_by    | UUID           | FK → Users.user_id               | Creator of the group           |
| created_at    | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | Timestamp of creation          |

---

## 👥 Memberships

Maps users to family groups with role-based access.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| membership_id | UUID           | PK                               | Unique membership ID           |
| user_id       | UUID           | FK → Users.user_id               | Member                         |
| family_id     | UUID           | FK → Families.family_id          | Associated family              |
| role          | ENUM           | CHECK ('admin', 'member')        | Access level                   |
| joined_at     | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | When user joined               |

---

## 💬 Messages

Stores encrypted messages and optional media.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| message_id    | UUID           | PK                               | Unique message ID              |
| family_id     | UUID           | FK → Families.family_id          | Group context                  |
| sender_id     | UUID           | FK → Users.user_id               | Message author                 |
| content       | TEXT           | ENCRYPTED                        | Message body                   |
| media_url     | TEXT           | ENCRYPTED, NULLABLE              | Optional media attachment      |
| created_at    | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | Sent time                      |
| read_by       | JSONB          | NULLABLE                         | Array of user_ids who read     |

---

## 📎 Media

Stores metadata for shared media files.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| media_id      | UUID           | PK                               | Unique media ID                |
| uploader_id   | UUID           | FK → Users.user_id               | Who uploaded                   |
| family_id     | UUID           | FK → Families.family_id          | Group context                  |
| file_type     | VARCHAR(50)    | NOT NULL                         | MIME type                      |
| file_size     | INT            | NOT NULL                         | In bytes                       |
| storage_url   | TEXT           | ENCRYPTED                        | S3 or CDN location             |
| uploaded_at   | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | Timestamp                      |

---

## 🔐 Encryption Keys (Optional for E2EE)

Stores public keys for secure message exchange.

| Field         | Type           | Constraints                     | Notes                          |
|---------------|----------------|----------------------------------|--------------------------------|
| key_id        | UUID           | PK                               | Unique key ID                  |
| user_id       | UUID           | FK → Users.user_id               | Owner of the key               |
| public_key    | TEXT           | NOT NULL                         | Used for encryption            |
| created_at    | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP        | Key generation time            |
| revoked_at    | TIMESTAMP      | NULLABLE                         | For key rotation               |

---

## 🔍 Indexing & Performance Notes
- Index `Users.email`, `Messages.created_at`, `Memberships.family_id` for fast lookup.
- Consider partitioning `Messages` by `family_id` for scalability.
- Use CDN for media delivery and cache headers for performance.

---

## 🧪 Audit & Compliance
- All tables include `created_at` for audit trails.
- Optional `updated_at` and `deleted_at` fields can be added for soft deletes and versioning.
- GDPR-compliant data deletion workflows should be implemented for `Users` and `Messages`.

---

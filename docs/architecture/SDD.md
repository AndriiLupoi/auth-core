# SDD — Software Design Document

**Проект:** AuthCore  
**Версія:** 1.0  
**Дата:** 2025-04-15  
**Статус:** Актуальний

---

## 1. Загальна архітектура

AuthCore реалізований як окремий мікросервіс зі **stateless-архітектурою**. Сервіс не зберігає стан у пам'яті процесу — всі дані зберігаються у PostgreSQL (основна БД) та Redis (кеш токенів та сесій).

**Архітектурний стиль:** Мікросервіс з REST API, гексагональна (ports & adapters) внутрішня архітектура.

### Стек технологій

| Категорія | Технологія |
|-----------|-----------|
| Runtime | Java 21 (LTS) |
| Фреймворк | Spring Boot 3.2 |
| HTTP | Spring Web MVC (вбудований Tomcat) |
| Основна БД | PostgreSQL 16 |
| Кеш / Blacklist | Redis 7 |
| ORM | Spring Data JPA / Hibernate 6 |
| Безпека | Spring Security 6 + OAuth2 Resource Server |
| JWT | nimbus-jose-jwt |
| Черга задач | Spring AMQP + RabbitMQ |
| Міграції БД | Flyway |
| Збірка | Maven |

---

## 2. Компоненти системи

| Компонент | Відповідальність |
|-----------|-----------------|
| Auth Controller | Приймає HTTP-запити, валідує вхідні дані, повертає відповідь |
| Auth Service | Бізнес-логіка автентифікації та авторизації |
| Token Service | Генерація, підпис та валідація JWT-токенів |
| User Repository | Взаємодія з PostgreSQL для операцій над обліковими записами |
| Session Repository | Управління сесіями через Redis |
| OAuth Adapter | Інтеграція із зовнішніми OAuth 2.0 провайдерами (Google, GitHub) |
| Notification Worker | Асинхронна відправка email та SMS через чергу |
| Audit Logger | Запис подій безпеки у БД |

---

## 3. Схема взаємодії компонентів

```
Client
  │
  ▼
Auth Controller
  │
  ├──► Auth Service ──► User Repository ──► PostgreSQL
  │         │
  │         ├──► Token Service ──► Redis (blacklist)
  │         │
  │         └──► OAuth Adapter ──► Google / GitHub
  │
  ├──► Notification Worker ──► RabbitMQ ──► Email / SMS
  │
  └──► Audit Logger ──► PostgreSQL (audit_log table)
```

---

## 4. REST API Ендпоінти

| Метод | Шлях | Опис | Авторизація |
|-------|------|------|-------------|
| POST | `/auth/register` | Реєстрація нового користувача | — |
| POST | `/auth/login` | Вхід у систему, видача access + refresh токенів | — |
| POST | `/auth/refresh` | Оновлення access-токена через refresh token | — |
| POST | `/auth/logout` | Вихід з поточної сесії (інвалідація refresh-токена) | Bearer Token |
| POST | `/auth/logout-all` | Завершення всіх активних сесій користувача | Bearer Token |
| POST | `/auth/2fa/enable` | Увімкнення двофакторної автентифікації (TOTP) | Bearer Token |
| POST | `/auth/2fa/verify` | Підтвердження коду 2FA при увімкненні | Bearer Token |
| POST | `/auth/password/forgot` | Запит на скидання пароля (відправка email) | — |
| POST | `/auth/password/reset` | Встановлення нового пароля за токеном | — (reset token) |
| GET | `/auth/sessions` | Отримання списку активних сесій користувача | Bearer Token |
| GET | `/oauth/:provider` | Редирект на сторінку OAuth провайдера | — |
| GET | `/oauth/:provider/callback` | Callback URL після авторизації через OAuth | — |

---

## 5. Структура бази даних

### Таблиця `users`

| Поле | Тип | Опис |
|------|-----|------|
| id | UUID | Первинний ключ |
| email | VARCHAR(255) | Унікальний email |
| password_hash | VARCHAR(255) | bcrypt-хеш пароля |
| is_email_verified | BOOLEAN | Статус верифікації |
| is_locked | BOOLEAN | Блокування акаунту |
| created_at | TIMESTAMP | Дата створення |

### Таблиця `sessions`

| Поле | Тип | Опис |
|------|-----|------|
| id | UUID | Первинний ключ |
| user_id | UUID | FK → users.id |
| refresh_token_hash | VARCHAR | Хеш refresh-токена |
| expires_at | TIMESTAMP | Термін дії |
| created_at | TIMESTAMP | Дата створення |

---

## 6. Безпека

- JWT підписуються алгоритмом **RS256** (асиметричні ключі)
- Access-токен: TTL **15 хвилин**
- Refresh-токен: TTL **30 днів**, зберігається у Redis
- Після logout refresh-токен додається до blacklist у Redis
- Brute-force захист: лічильник у Redis, блокування після 5 спроб

---

*Зміни до API повинні супроводжуватись оновленням цього документа та відповідних тест-кейсів у Traceability Matrix.*
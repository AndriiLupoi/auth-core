# Developer Onboarding

**Проект:** AuthCore  
**Версія:** 1.0  

---

## Вимоги до середовища

| Інструмент | Версія | Для чого |
|-----------|--------|----------|
| Java | 21 (LTS) | Runtime |
| Maven | 3.9+ | Збірка |
| Docker | 24+ | Контейнери |
| Docker Compose | 2.20+ | Локальне середовище |
| Git | будь-яка | Контроль версій |

---

## Швидкий старт

### 1. Клонувати репозиторій

```bash
git clone https://github.com/AndriiLupoi/auth-core.git
cd authcore
```

### 2. Запустити інфраструктуру (PostgreSQL + Redis + RabbitMQ)

```bash
docker compose up -d
```

### 3. Налаштувати змінні середовища

```bash
cp .env.example .env
# Відредагувати .env під локальне середовище
```

Мінімально необхідні змінні:

```env
DB_URL=jdbc:postgresql://localhost:5432/authcore
DB_USERNAME=authcore
DB_PASSWORD=secret
REDIS_URL=redis://localhost:6379
JWT_PRIVATE_KEY=...   # RSA приватний ключ (base64)
JWT_PUBLIC_KEY=...    # RSA публічний ключ (base64)
```

### 4. Запустити сервіс

```bash
mvn spring-boot:run
```

Сервіс доступний на `http://localhost:8080`.

---

## Структура проекту

```
src/
├── main/java/com/authcore/
│   ├── controller/     # Auth Controller, OAuth Controller
│   ├── service/        # Auth Service, Token Service
│   ├── repository/     # User Repository, Session Repository
│   ├── adapter/        # OAuth Adapter (Google, GitHub)
│   ├── worker/         # Notification Worker
│   ├── audit/          # Audit Logger
│   └── config/         # Spring Security, JWT config
└── test/
    ├── unit/
    └── integration/
```

---

## Запуск тестів

```bash
# Unit тести
mvn test

# Integration тести (потрібен запущений Docker)
mvn verify -P integration-tests

# Всі тести
mvn verify
```

---

## Корисні посилання

- [System Specification (SSD)](../architecture/SSD.md) — що система робить
- [Software Design (SDD)](../architecture/SDD.md) — як система влаштована
- [Infrastructure (ISD)](../architecture/ISD.md) — як система розгортається
- [Test Strategy](../quality/test-strategy.md) — як тестується

---

## Часті проблеми

**Проблема:** `Connection refused` до PostgreSQL  
**Рішення:** Переконайтесь що `docker compose up -d` виконано і контейнери запущені (`docker ps`)

**Проблема:** JWT ключі не знайдено  
**Рішення:** Згенеруйте RSA пару та додайте у `.env`:
```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```
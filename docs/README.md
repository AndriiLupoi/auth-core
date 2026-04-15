# AuthCore Documentation

> Централізований мікросервіс автентифікації та авторизації

## Про систему

**AuthCore** — мікросервіс автентифікації та авторизації користувачів для інтеграції у розподілені системи та SPA-застосунки.  

Сервіс реалізує протоколи **OAuth 2.0** та **OpenID Connect**, забезпечуючи єдиний вхід (SSO). Підтримує парольну автентифікацію, двофакторну автентифікацію (2FA), федеративний вхід через Google/GitHub та роботу з JWT-токенами.

**Роль API**: Internal  
**Цільова аудиторія**: Frontend-розробники, Backend-сервіси, QA та DevOps.

---

## Навігація по документації

| Розділ                        | Документ                                      | Призначення |
|------------------------------|-----------------------------------------------|-------------|
| **Архітектура**              | [System Specification (SSD)](architecture/SSD.md) | Функціональні та нефункціональні вимоги |
| **Архітектура**              | [Software Design (SDD)](architecture/SDD.md)     | Архітектура, компоненти, API |
| **Архітектура**              | [Infrastructure (ISD)](architecture/ISD.md)      | Розгортання та інфраструктура |
| **Якість**                   | [Test Strategy](quality/test-strategy.md)        | Підхід до тестування |
| **Якість**                   | [Traceability Matrix](quality/traceability-matrix.md) | Простежуваність вимог |
| **Для розробників**          | [Onboarding](developer/onboarding.md)            | Як запустити проект |
| **API Documentation**        | [Опис API](api/index.md)                         | Контракт API |
| **API Documentation**        | [OpenAPI Специфікація](api/openapi.yaml)         | Повна специфікація |
| **API Documentation**        | [Swagger UI](api/swagger.md)                     | Інтерактивне тестування API |
| **Візуальна документація**   | [Storybook](frontend/storybook.md)               | UI-компоненти |
| **CI/CD**                    | [Автоматизація документації](ci-cd-docs.md)      | Pipeline та публікація |

---

## Single Source of Truth

- Функціональні вимоги → `architecture/SSD.md`
- Архітектурні рішення → `architecture/SDD.md`
- API контракт → `api/openapi.yaml`
- Тестування → `quality/traceability-matrix.md`

---

## Версійність документації

| Версія | Дата       | Опис змін |
|--------|------------|-----------|
| 1.2.0  | 15.04.2026 | Додано OpenAPI, Swagger UI, CI/CD (ЛР-9) |
| 1.0.0  | 2025       | Базова документація (ЛР-7 + ЛР-8) |

**Публікація**: https://AndriiLupoi.github.io/authcore-docs/

---

*Документація генерується автоматично через GitHub Actions*
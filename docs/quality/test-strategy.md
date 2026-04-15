# Test Strategy

**Проект:** AuthCore  
**Версія:** 1.0  
**Статус:** Актуальний

---

## 1. Мета

Забезпечити коректність та безпечність усіх механізмів автентифікації та авторизації у системі AuthCore. Тестування охоплює всі функціональні вимоги, визначені у [SSD](../architecture/SSD.md).

---

## 2. Рівні тестування

| Рівень | Інструменти | Що тестується | Ціль покриття |
|--------|-------------|---------------|---------------|
| Unit-тести | JUnit 5, Mockito | Сервіси, утиліти, Token Service, репозиторії | 80%+ |
| Integration-тести | Spring Boot Test + Testcontainers | API ендпоінти з реальною БД та Redis | Повне покриття критичних шляхів |
| E2E-тести | Playwright | Реєстрація, вхід, 2FA, OAuth flows | Основні користувацькі сценарії |
| Security-тести | OWASP ZAP + ручні перевірки | SQL injection, brute-force, token manipulation, XSS, auth bypass | Всі критичні ендпоінти |
| Performance-тести | k6 | Навантажувальне тестування | До 1000 RPS |

---

## 3. Середовище тестування

Тести виконуються у окремому **Docker Compose stack** з:
- PostgreSQL (тестова БД, ізольована від dev/prod)
- Redis (окремий інстанс)
- RabbitMQ (mock для notification worker)

Тестові дані очищуються після кожного тест-рану (rollback транзакцій або truncate).

---

## 4. Підхід до регресії

- При кожному **Pull Request** автоматично запускаються unit + integration тести через GitHub Actions
- E2E тести запускаються при деплої на Staging
- Security тести — вручну перед кожним релізом
- Performance тести — вручну при значних змінах у логіці автентифікації

---

## 5. Критерії якості

| Критерій | Мінімальний поріг |
|----------|-------------------|
| Unit test coverage | ≥ 80% |
| Integration: критичні шляхи | 100% (FR-01, FR-02, FR-03, FR-06) |
| Час відповіді (P95) | ≤ 300 мс |
| Error rate у production | < 0.1% |

---

## 6. Типи тестів за вимогами

| Вимога | Unit | Integration | E2E | Security | Performance |
|--------|------|-------------|-----|----------|-------------|
| FR-01 (Реєстрація) | ✅ | ✅ | — | ✅ | — |
| FR-02 (Вхід) | ✅ | ✅ | — | ✅ | ✅ |
| FR-03 (Refresh token) | ✅ | ✅ | — | ✅ | — |
| FR-04 (2FA) | ✅ | ✅ | ✅ | ✅ | — |
| FR-05 (OAuth) | — | — | ✅ | ✅ | — |
| FR-06 (Reset password) | ✅ | ✅ | — | ✅ | — |
| FR-07 (Sessions) | ✅ | ✅ | — | — | — |
| FR-08 (Admin) | ✅ | ✅ | — | ✅ | — |

---

## 7. Відповідальність

| Роль | Відповідальність |
|------|-----------------|
| Backend-розробник | Unit та Integration тести |
| QA-інженер | E2E тести, Traceability Matrix |
| Security-інженер | Security тести, OWASP checklist |
| DevOps | Performance тести, CI/CD інтеграція |

---

*Детальні тест-кейси та їх відповідність вимогам — у [Traceability Matrix](traceability-matrix.md).*
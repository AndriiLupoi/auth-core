# ISD — Infrastructure Specification Document

**Проект:** AuthCore  
**Версія:** 1.0  
**Статус:** Актуальний

---

## 1. Середовище розгортання

Система розгортається у хмарному середовищі (AWS або GCP) з використанням контейнеризації. Передбачено три окремі середовища.

| Середовище | Призначення | Кількість реплік | Оркестрація |
|------------|-------------|------------------|-------------|
| Development | Розробка та локальне тестування | 1 | Docker Compose |
| Staging | Передвипускне тестування, QA | 2 | Kubernetes |
| Production | Живе навантаження | 3–10 (HPA) | Kubernetes + HPA |

---

## 2. Основні інфраструктурні компоненти

| Компонент | Технологія | Призначення |
|-----------|-----------|-------------|
| Оркестрація | Kubernetes (EKS / GKE) | Управління контейнерами, автоматичний рестарт |
| Load Balancer | AWS ALB / GCP CLB | Розподіл трафіку між репліками |
| База даних | PostgreSQL (RDS / Cloud SQL) | Основне сховище з реплікою читання |
| Кеш | Redis (ElastiCache / Memorystore) | Сесії та blacklist токенів |
| Docker Registry | ECR / Artifact Registry | Зберігання Docker-образів |
| Secrets | AWS Secrets Manager / GCP Secret Manager | Credentials та JWT-ключі |
| Ingress | NGINX Ingress Controller | TLS-термінація, маршрутизація |
| Моніторинг | Prometheus + Grafana | Метрики сервісу |
| Логування | ELK Stack | Централізовані логи |

---

## 3. Принципи масштабування

### Горизонтальне автомасштабування (HPA)

- При навантаженні CPU > **70%** Kubernetes автоматично додає нові поди
- Максимальна кількість реплік: **10**
- Мінімальна кількість реплік: **3** (production)

### Stateless-дизайн

Кожен под не зберігає стан, тому балансування без sticky sessions. Стан зберігається виключно у PostgreSQL та Redis.

### Database Connection Pooling

Використання **PgBouncer** між сервісом та PostgreSQL для ефективного управління з'єднаннями.

### Redis Cluster

При зростанні навантаження Redis масштабується горизонтально через **Redis Cluster mode**. Конфігурація: Master + 1 Replica.

---

## 4. CI/CD Pipeline

```
git push (main / feature branch)
        │
        ▼
  GitHub Actions
        │
        ├── 1. Lint + Code style check
        ├── 2. Unit tests
        ├── 3. Integration tests (Docker Compose)
        ├── 4. Build Docker image
        ├── 5. Push to Registry (ECR / Artifact Registry)
        ├── 6. Deploy to Staging (автоматично)
        └── 7. Deploy to Production (після ручного підтвердження)
```

### Умови деплою

| Середовище | Тригер | Умова |
|------------|--------|-------|
| Staging | Автоматично при push в `main` | Всі тести пройдені |
| Production | Вручну (approve у GitHub Actions) | Staging стабільний |

---

## 5. Моніторинг та алертинг

### Ключові метрики (Prometheus)

- `auth_requests_total` — загальна кількість запитів
- `auth_request_duration_seconds` — час відповіді
- `auth_failed_logins_total` — кількість невдалих спроб входу
- `auth_active_sessions` — кількість активних сесій

### Алерти (Grafana Alerting)

| Алерт | Умова | Пріоритет |
|-------|-------|-----------|
| HighErrorRate | Error rate > 5% за 5 хв | Critical |
| SlowResponse | P95 latency > 500ms | Warning |
| PodDown | Кількість подів < min replicas | Critical |
| HighCPU | CPU > 80% за 10 хв | Warning |

---

## 6. Безпека інфраструктури

- Усі credentials зберігаються у Secrets Manager (не в коді)
- TLS термінується на рівні Ingress Controller
- Network Policies обмежують трафік між подами
- Docker-образи скануються на вразливості перед деплоєм (Trivy)

---

*При зміні інфраструктури або додаванні нових середовищ — оновити цей документ та відповідний розділ CI/CD.*
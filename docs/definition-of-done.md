# Definition of Done (Documentation) — AuthCore

## Загальні принципи

Зміна вважається **завершеною** лише тоді, коли вся пов'язана документація оновлена,
перевірена та успішно опублікована. Документація є **частиною коду**, а не опційним
доповненням.

> **Правило:** Pull Request не може бути злитий у `main`, якщо CI-перевірки документації
> не пройшли успішно.

---

## 1. DoD за типом зміни

### 1.1 Додано новий API-ендпоінт

| # | Обов'язкова дія | Артефакт | Де перевіряється |
|---|-----------------|----------|------------------|
| 1 | Додати ендпоінт до `docs/openapi.yaml` | OpenAPI spec | CI: `spectral lint` |
| 2 | Описати всі параметри, requestBody, responses (200, 4xx, 5xx) | OpenAPI spec | CI: schema validation |
| 3 | Додати `operationId` та `summary` | OpenAPI spec | CI: spectral rule |
| 4 | Swagger UI збирається без помилок | Swagger UI | CI: build job |
| 5 | Якщо ендпоінт потребує аутентифікації — вказати `security` | OpenAPI spec | Code review |
| 6 | Оновити `CHANGELOG.md` розділ `[Unreleased]` | Changelog | Code review |

**Критерій готовності:** `spectral lint` повертає 0 errors, Swagger UI будується.

---

### 1.2 Змінено існуючий ендпоінт (response / request)

| # | Обов'язкова дія | Артефакт | Де перевіряється |
|---|-----------------|----------|------------------|
| 1 | Оновити відповідну схему в `components/schemas` | OpenAPI spec | CI: `spectral lint` |
| 2 | Якщо зміна зворотно-сумісна — minor-bump версії API (`1.1` → `1.2`) | OpenAPI `info.version` | Code review |
| 3 | Якщо breaking change — major-bump (`1.x` → `2.0`) + migration guide | OpenAPI + `docs/migration/` | Code review |
| 4 | Додати запис до `CHANGELOG.md` з міткою `Changed` або `Breaking` | Changelog | Code review |
| 5 | Перевірити, що приклади в OpenAPI (`example:`) відповідають новій схемі | OpenAPI spec | CI: schema validation |

**Критерій готовності:** версія API оновлена, breaking changes задокументовані та вказані у PR description.

---

### 1.3 Додано або змінено UI-компонент

| # | Обов'язкова дія | Артефакт | Де перевіряється |
|---|-----------------|----------|------------------|
| 1 | Створити або оновити Story у `src/stories/<ComponentName>.stories.tsx` | Storybook | CI: Storybook build |
| 2 | Story повинна охоплювати: Default, Disabled, Loading/Error (де застосовно) | Storybook | Code review |
| 3 | Додати Controls (ArgsTable) для всіх props компонента | Storybook | Code review |
| 4 | Пройти accessibility audit (a11y addon): 0 critical помилок | Storybook a11y | CI: storybook-addon-a11y |
| 5 | Storybook збирається без помилок (`build-storybook` exit 0) | Storybook | CI: build job |
| 6 | Якщо змінено публічне API компонента — оновити Props таблицю | Storybook | Code review |

**Критерій готовності:** `npm run build-storybook` завершується успішно, всі Stories рендеряться.

---

### 1.4 Breaking change в API

| # | Обов'язкова дія | Артефакт | Де перевіряється |
|---|-----------------|----------|------------------|
| 1 | Major-bump версії: `1.x.x` → `2.0.0` | OpenAPI `info.version` + git tag | CI + Code review |
| 2 | Створити `docs/migration/v1-to-v2.md` з описом: що змінилось, як мігрувати | Migration guide | Code review |
| 3 | Попередня версія документації зберігається під `/versions/v1/` | GitHub Pages | CI: mike deploy |
| 4 | У Swagger UI додати deprecation notice для старих ендпоінтів (якщо підтримуються) | OpenAPI `deprecated: true` | CI: spectral |
| 5 | Повідомити команду через `#authcore-alerts` Slack | Slack | Manual |
| 6 | Оновити README.md: поточна версія API | README | Code review |

**Критерій готовності:** нова версія документації опублікована, стара доступна під `/versions/v1/`.

---

### 1.5 Нефункціональна зміна (refactoring, performance, config)

| # | Обов'язкова дія | Артефакт | Де перевіряється |
|---|-----------------|----------|------------------|
| 1 | Якщо змінилась інфраструктура — оновити `docs/infrastructure.md` | ISD | Code review |
| 2 | Якщо змінився CI/CD — оновити `docs/ci-cd-docs.md` | CI/CD docs | Code review |
| 3 | Якщо змінились нефункціональні характеристики (ліміти, TTL, тощо) — оновити SSD | SSD | Code review |
| 4 | Загальний сайт документації будується без помилок | MkDocs | CI: mkdocs build --strict |

**Критерій готовності:** `mkdocs build --strict` завершується без помилок.

---

## 2. Чек-ліст для Pull Request

Кожен PR, що стосується документації, повинен мати у description наступний чек-ліст:

```markdown
## Documentation Checklist

### API Changes
- [ ] `docs/openapi.yaml` оновлено
- [ ] `spectral lint` проходить без errors
- [ ] Версія API оновлена (якщо потрібно)
- [ ] Breaking changes задокументовані

### UI Changes
- [ ] Storybook Stories оновлено / додано
- [ ] `npm run build-storybook` успішний
- [ ] Accessibility: 0 critical errors

### General
- [ ] `CHANGELOG.md` оновлено
- [ ] `mkdocs build --strict` успішний
- [ ] Code review documentation пройдено
```

---

## 3. Автоматичні CI-перевірки (обов'язкові)

Наступні перевірки **блокують merge** при невдачі:

| Перевірка | Команда | Блокує merge |
|-----------|---------|:------------:|
| OpenAPI validation | `spectral lint docs/openapi.yaml` | ✅ |
| Swagger UI build | `npm run build:swagger-ui` | ✅ |
| Storybook build | `npm run build-storybook` | ✅ |
| MkDocs build | `mkdocs build --strict` | ✅ |
| Breaking change check | `optic-ci diff --check` | ✅ |
| A11y audit | `storybook-a11y-audit` | ✅ |

---

## 4. Приклади: що є порушенням DoD

| Ситуація | Порушення | Дія |
|----------|-----------|-----|
| Додано `POST /auth/sessions/revoke`, але не оновлено OpenAPI | ❌ Ендпоінт не задокументований | Оновити `openapi.yaml` |
| Змінено поле `expiresIn: number` → `expiresAt: string`, версія не змінена | ❌ Breaking change без major-bump | Bump до `2.0.0` + migration guide |
| Доданий `<OtpInput>` без Story | ❌ Компонент не в Storybook | Додати Stories |
| PR злитий без оновлення `CHANGELOG.md` | ❌ Немає запису про зміну | Ретроактивно оновити |
| `mkdocs build` падає через непідтверджені посилання | ❌ Broken docs | Виправити перед merge |

---

## 5. Відповідальність

| Роль | Відповідальність |
|------|-----------------|
| **Розробник** | Оновлення OpenAPI, Stories, Changelog при кожній зміні |
| **Tech Lead / Reviewer** | Перевірка DoD під час code review |
| **DevOps** | Підтримка CI/CD пайплайну документації |
| **QA** | Перевірка відповідності документації реальній поведінці API |

---

*Документ: `docs/definition-of-done.md` | Версія: 1.0 | AuthCore Project*
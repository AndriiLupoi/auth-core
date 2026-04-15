# Documentation CI/CD Pipeline — AuthCore

## Огляд

Цей документ описує процес автоматичної генерації, валідації та публікації документації
для мікросервісу **AuthCore** у рамках CI/CD-пайплайну.

Документація є частиною **Definition of Done** і публікується автоматично при кожному
злитті у гілку `main`.

---

## 1. Артефакти документації

| Артефакт | Джерело | Інструмент | URL після публікації |
|----------|---------|------------|----------------------|
| API-документація | `docs/openapi.yaml` | Swagger UI | `/docs/api/` |
| UI-компоненти | `src/stories/` | Storybook | `/docs/storybook/` |
| Загальна документація | `docs/` | MkDocs | `/docs/` |
| Changelog | `CHANGELOG.md` | Автогенерація | `/docs/changelog/` |

---

## 2. Тригери генерації

```
push → feature/*     → Валідація та preview-збірка (без публікації)
push → develop       → Збірка + публікація на Staging Pages
merge → main         → Збірка + валідація + публікація на Production Pages
tag   → v*.*.*       → Фіксація версіонованої документації
```

---

## 3. Pipeline: Повна схема

### 3.1 Feature Branch (push до `feature/*`)

```yaml
name: Docs Validation

on:
  push:
    branches: ['feature/**']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate OpenAPI spec
        run: |
          npx @stoplight/spectral-cli lint docs/openapi.yaml \
            --ruleset .spectral.yaml

      - name: Check docs build (dry-run)
        run: |
          pip install mkdocs mkdocs-material
          mkdocs build --strict

      - name: Lint Storybook stories
        run: |
          npm ci
          npm run storybook:build -- --quiet
```

### 3.2 Main Branch (merge до `main`)

```yaml
name: Docs Build & Publish

on:
  push:
    branches: [main]

jobs:
  # JOB 1: Validate
  validate-openapi:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate OpenAPI 3.0 specification
        run: |
          npx @stoplight/spectral-cli lint docs/openapi.yaml
      - name: Check breaking changes
        run: |
          npx @optic/optic-ci diff docs/openapi.yaml \
            --from main --check

  # JOB 2: Build API Docs
  build-swagger:
    needs: validate-openapi
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate Swagger UI
        run: |
          npx swagger-ui-dist-package copy ./dist/api
          cp docs/openapi.yaml ./dist/api/openapi.yaml
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: swagger-ui
          path: dist/api/

  # JOB 3: Build Storybook
  build-storybook:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - name: Build Storybook
        run: npm run build-storybook -- --output-dir dist/storybook
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: storybook
          path: dist/storybook/

  # JOB 4: Build MkDocs site
  build-mkdocs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Python
        uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install mkdocs mkdocs-material mkdocstrings
      - name: Build documentation site
        run: mkdocs build --strict --site-dir dist/site
      - uses: actions/upload-artifact@v4
        with:
          name: mkdocs-site
          path: dist/site/

  # JOB 5: Publish (залежить від всіх попередніх)
  publish:
    needs: [build-swagger, build-storybook, build-mkdocs]
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4

      - name: Assemble final structure
        run: |
          mkdir -p public
          cp -r mkdocs-site/* public/
          cp -r swagger-ui/ public/api/
          cp -r storybook/ public/storybook/

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          cname: docs.authcore.example.com
```

---

## 4. Структура URL публікації

```
https://docs.authcore.example.com/
├── /                        ← Головна сторінка (MkDocs)
├── /api/                    ← Swagger UI (OpenAPI docs)
├── /storybook/              ← Storybook UI components
├── /changelog/              ← Журнал змін
└── /versions/
    ├── /v1.0/               ← Зафіксована версія документації
    └── /v0.9/               ← Попередня версія
```

### GitHub Pages налаштування

```yaml
# .github/workflows/pages.yml
environment:
  name: github-pages
  url: ${{ steps.deployment.outputs.page_url }}
```

---

## 5. Versioning документації

### 5.1 Принципи

| Тип зміни | Версія документації | Дія |
|-----------|---------------------|-----|
| Bugfix / патч | Без зміни версії | Перезапис поточної |
| Minor (нові ендпоінти, зворотно-сумісні) | Minor bump (1.1 → 1.2) | Перезапис + оновлення changelog |
| Major (breaking changes) | Major bump (1.x → 2.0) | Нова папка `/versions/v2/`, стара зберігається |

### 5.2 Автоматичне версіонування при тегу

```yaml
on:
  push:
    tags: ['v*.*.*']

jobs:
  version-docs:
    steps:
      - name: Extract version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV

      - name: Archive current docs as versioned
        run: |
          mkdir -p public/versions/${{ env.VERSION }}
          cp -r dist/site/* public/versions/${{ env.VERSION }}/

      - name: Update version selector in MkDocs
        run: |
          # mike — інструмент для versioned MkDocs
          pip install mike
          mike deploy ${{ env.VERSION }} latest --push --update-aliases
```

### 5.3 Правило breaking changes

- Breaking change в API → обов'язковий major-тег (`v2.0.0`)
- Попередня версія документації зберігається під `/versions/v1/`
- У Swagger UI відображається банер: `⚠ Breaking changes in v2.0. See migration guide.`

---

## 6. Якість та валідація

### OpenAPI Validation Rules (`.spectral.yaml`)

```yaml
rules:
  operation-summary: error         # Кожен endpoint має summary
  operation-operationId: error     # Унікальний operationId
  path-params-defined: error       # Path params задокументовані
  response-contains-description: warn
  schema-required: warn
  no-$ref-siblings: error
```

### Storybook перевірки

```bash
# Перевіряється автоматично у CI:
- Storybook build не падає (exit 0)
- Всі Stories мають назву та опис
- Accessibility audit (storybook-addon-a11y): 0 critical errors
```

---

## 7. Сповіщення

При помилці пайплайну — автоматичне сповіщення у Slack-канал `#authcore-alerts`:

```yaml
- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'authcore-alerts'
    slack-message: |
      ❌ *Docs pipeline failed*
      Branch: `${{ github.ref_name }}`
      Job: `${{ github.job }}`
      <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View run>
```

---

## 8. Локальна розробка документації

```bash
# Запуск Swagger UI локально
npx swagger-ui-serve docs/openapi.yaml --port 3001

# Запуск Storybook локально
npm run storybook

# Запуск MkDocs локально
mkdocs serve

# Валідація OpenAPI специфікації
npx @stoplight/spectral-cli lint docs/openapi.yaml
```

---
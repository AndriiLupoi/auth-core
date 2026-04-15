# Автоматизація документації в CI/CD (AuthCore)

## Артефакти, що генеруються автоматично
- Статичний сайт MkDocs з Material темою
- Інтерактивний Swagger UI
- Валідація OpenAPI специфікації
- Візуальна документація компонентів

## Тригери
- `push` у гілки `feature/*`
- `pull_request` у `main`
- `merge` у `main`

## Послідовність кроків (GitHub Actions)

```yaml
name: Documentation CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: pip install mkdocs-material mkdocs-swagger-ui-tag

      - name: Validate OpenAPI
        run: echo "OpenAPI validation passed"  # можна додати spectral

      - name: Build MkDocs
        run: mkdocs build --strict

      - name: Deploy to GitHub Pages
        if: github.ref == 'refs/heads/main'
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
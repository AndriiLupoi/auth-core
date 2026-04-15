# 5. Візуальна документація frontend (Storybook)

---

## Огляд

Для демонстрації Storybook-документації обрано **2 UI-компоненти**, які використовуються у клієнтських застосунках, що інтегруються з AuthCore API:

| № | Компонент | Опис |
|---|-----------|------|
| 1 | `Button` | Кнопка для відправки форм автентифікації (вхід, реєстрація, скидання пароля) |
| 2 | `TokenStatusBadge` | Бейдж для відображення стану JWT-токена або сесії користувача |

---

## Компонент 1: `Button`

### Опис

Кнопка загального призначення, що використовується у формах автентифікації (логін, реєстрація, скидання пароля, підтвердження 2FA). Підтримує стани: звичайний, вимкнений та завантаження.

### Props

| Prop | Тип | За замовчуванням | Опис |
|------|-----|-----------------|------|
| `label` | `string` | — | Текст кнопки |
| `variant` | `'primary' \| 'secondary' \| 'danger'` | `'primary'` | Візуальний варіант |
| `disabled` | `boolean` | `false` | Вимикає кнопку |
| `loading` | `boolean` | `false` | Показує спінер замість тексту |
| `onClick` | `() => void` | — | Обробник кліку |

---

### Stories

#### 1.1 Default (Базовий стан)

```tsx
// Button.stories.tsx

export const Default: Story = {
  args: {
    label: 'Увійти',
    variant: 'primary',
    disabled: false,
    loading: false,
  },
};
```

**Вигляд:**

<button style="background:#2563eb; color:white; border:none; padding:14px 32px; border-radius:8px; font-weight:600; font-size:16px; cursor:pointer;">
  Увійти
</button>

```tsx
<Button variant="primary">Увійти</Button>

**Очікувана поведінка:**
- Кнопка активна, реагує на наведення (`hover`) та натискання (`active`)
- Колір фону: `#1D9E75` (primary)
- Курсор: `pointer`
- При кліку викликається `onClick`

---

#### 1.2 Disabled (Вимкнений стан)

```tsx
export const Disabled: Story = {
  args: {
    label: 'Увійти',
    variant: 'primary',
    disabled: true,
    loading: false,
  },
};
```

**Вигляд:**

<button style="background:#64748b; color:white; border:none; padding:14px 32px; border-radius:8px; font-weight:600; font-size:16px; cursor:not-allowed; opacity:0.6;">
Увійти
</button>

**Очікувана поведінка:**
- Кнопка не реагує на клік
- Opacity: `0.4`
- Курсор: `not-allowed`
- `onClick` не викликається

**Коли використовується в AuthCore:**  
Кнопка «Підтвердити» у формі 2FA — вимкнена, поки користувач не ввів 6-значний TOTP-код.

---

#### 1.3 Loading (Стан завантаження)

```tsx
export const Loading: Story = {
  args: {
    label: 'Увійти',
    variant: 'primary',
    disabled: false,
    loading: true,
  },
};
```

**Вигляд:**

<button style="background:#2563eb; color:white; border:none; padding:14px 32px; border-radius:8px; font-weight:600; font-size:16px; cursor:wait; display:flex; align-items:center; gap:10px;">
<span style="animation: spin 1s linear infinite;">⟳</span>
Завантаження...
</button>

**Очікувана поведінка:**
- Відображається анімований спінер замість або поруч із текстом
- Кнопка заблокована під час запиту (не допускає повторного кліку)
- Курсор: `wait`

**Коли використовується в AuthCore:**  
Після натискання «Увійти» — поки виконується `POST /auth/login`, кнопка переходить у стан `loading` до отримання відповіді або помилки.

---

#### 1.4 Danger (Небезпечна дія)

```tsx
export const Danger: Story = {
  args: {
    label: 'Завершити всі сесії',
    variant: 'danger',
    disabled: false,
    loading: false,
  },
};
```

**Вигляд:**

<button style="background:#ef4444; color:white; border:none; padding:14px 32px; border-radius:8px; font-weight:600; font-size:16px; cursor:pointer;">
Завершити всі сесії
</button>

**Очікувана поведінка:**
- Колір фону: `#E24B4A` (danger)
- Використовується для деструктивних дій

**Коли використовується в AuthCore:**  
Кнопка «Завершити всі сесії» (`POST /auth/logout-all`) в адміністративній панелі управління сесіями.

---

### Повна таблиця Stories для `Button`

| Story | `variant` | `disabled` | `loading` | Опис |
|-------|-----------|------------|-----------|------|
| `Default` | `primary` | `false` | `false` | Звичайний активний стан |
| `Disabled` | `primary` | `true` | `false` | Вимкнена кнопка |
| `Loading` | `primary` | `false` | `true` | Очікування відповіді API |
| `Danger` | `danger` | `false` | `false` | Деструктивна дія |
| `Secondary` | `secondary` | `false` | `false` | Другорядна дія |

---

---

## Компонент 2: `TokenStatusBadge`

### Опис

Бейдж для відображення поточного стану JWT-токена або сесії користувача. Використовується в адміністративній панелі AuthCore для відображення статусу сесій (`GET /auth/sessions`).

### Props

| Prop | Тип | За замовчуванням | Опис |
|------|-----|-----------------|------|
| `status` | `'active' \| 'expired' \| 'revoked' \| 'pending'` | — | Поточний стан токена/сесії |
| `label` | `string` | автоматично | Текст бейджа (можна перевизначити) |
| `showIcon` | `boolean` | `true` | Показувати/приховати іконку статусу |

---

### Stories

#### 2.1 Active (Базовий стан — активна сесія)

```tsx
// TokenStatusBadge.stories.tsx

export const Active: Story = {
  args: {
    status: 'active',
    showIcon: true,
  },
};
```

**Вигляд:**


<button style="background:#EAF3DE; color:#3B6D11; border:none; padding:10px 18px; border-radius:8px; font-weight:600; font-size:14px; display:inline-flex; align-items:center; gap:6px;">
  ● Активна
</button>

**Очікувана поведінка:**
- Колір фону: зелений (`#EAF3DE`)
- Колір тексту та іконки: `#3B6D11`
- Іконка: заповнене коло (`●`)
- Відображається, коли сесія валідна і access-токен не прострочений

---

#### 2.2 Expired (Прострочений токен)

```tsx
export const Expired: Story = {
  args: {
    status: 'expired',
    showIcon: true,
  },
};
```

**Вигляд:**

<button style="background:#FAEEDA; color:#854F0B; border:none; padding:10px 18px; border-radius:8px; font-weight:600; font-size:14px; display:inline-flex; align-items:center; gap:6px;">
  ⏱ Прострочено
</button>

**Очікувана поведінка:**
- Колір фону: жовтий/amber (`#FAEEDA`)
- Колір тексту та іконки: `#854F0B`
- Іконка: годинник (`⏱`)
- Відображається, коли TTL access-токена (15 хв) вичерпано, але refresh-токен ще дійсний

---

#### 2.3 Revoked (Відкликана сесія)

```tsx
export const Revoked: Story = {
  args: {
    status: 'revoked',
    showIcon: true,
  },
};
```

**Вигляд:**

<button style="background:#FCEBEB; color:#A32D2D; border:none; padding:10px 18px; border-radius:8px; font-weight:600; font-size:14px; display:inline-flex; align-items:center; gap:6px;">
  ✕ Відкликано
</button>

**Очікувана поведінка:**
- Колір фону: червоний (`#FCEBEB`)
- Колір тексту та іконки: `#A32D2D`
- Іконка: хрестик (`✕`)
- Відображається після примусового виходу (`POST /auth/logout` або `/auth/logout-all`) або після блокування адміністратором

---

#### 2.4 Pending (Очікування верифікації)

```tsx
export const Pending: Story = {
  args: {
    status: 'pending',
    showIcon: true,
  },
};
```

**Вигляд:**

<button style="background:#F1F5F9; color:#475569; border:none; padding:10px 18px; border-radius:8px; font-weight:600; font-size:14px; display:inline-flex; align-items:center; gap:6px;">
  ◌ Очікує верифікації
</button>

**Очікувана поведінка:**
- Колір фону: сірий (`#F1EFE8`)
- Колір тексту та іконки: `#5F5E5A`
- Іконка: порожнє коло (`◌`)
- Відображається для акаунтів, що зареєстровані, але email ще не підтверджено (FR-01)

---

### Повна таблиця Stories для `TokenStatusBadge`

| Story | `status` | Колір | Іконка | Коли використовується |
|-------|----------|-------|--------|-----------------------|
| `Active` | `active` | Зелений | `●` | Валідна активна сесія |
| `Expired` | `expired` | Жовтий | `⏱` | TTL токена вичерпано |
| `Revoked` | `revoked` | Червоний | `✕` | Сесія примусово завершена |
| `Pending` | `pending` | Сірий | `◌` | Email не підтверджено |

---

## Зв'язок компонентів із вимогами SSD

| Компонент | Story | Вимога SSD | Ендпоінт API |
|-----------|-------|------------|--------------|
| `Button` — `Loading` | Стан під час запиту | FR-02 (вхід) | `POST /auth/login` |
| `Button` — `Disabled` | Заблокована кнопка 2FA | FR-04 (2FA) | `POST /auth/2fa/verify` |
| `Button` — `Danger` | Завершення всіх сесій | FR-07 (сесії) | `POST /auth/logout-all` |
| `TokenStatusBadge` — `Active` | Активна сесія | FR-07 (сесії) | `GET /auth/sessions` |
| `TokenStatusBadge` — `Revoked` | Відкликана сесія | FR-07 (сесії) | `POST /auth/logout` |
| `TokenStatusBadge` — `Pending` | Email не підтверджено | FR-01 (реєстрація) | `POST /auth/register` |

---

## Структура файлів Storybook

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.module.css
│   │   └── Button.stories.tsx       ← Stories: Default, Disabled, Loading, Danger
│   └── TokenStatusBadge/
│       ├── TokenStatusBadge.tsx
│       ├── TokenStatusBadge.module.css
│       └── TokenStatusBadge.stories.tsx  ← Stories: Active, Expired, Revoked, Pending
└── .storybook/
    ├── main.ts
    └── preview.ts
```

---
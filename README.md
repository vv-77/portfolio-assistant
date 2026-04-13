# 💼 Портфолио AI Agent Team

Сайт-портфолио с каталогом проектов и AI-ассистентов.

## 🚀 Технологии

- **HTML5** — семантическая верстка
- **CSS3** — CSS Variables, Grid, Flexbox, адаптивность
- **Vanilla JavaScript** — без фреймворков
- **GitHub Pages** — хостинг

## 📁 Структура проекта

```
portfolio-assistant/
├── index.html              # Главная страница
├── projects.html           # Каталог проектов
├── favicon.svg             # Фавикон
├── styles.css              # Общие стили
├── scripts.js              # Общие скрипты
├── .nojekyll               # Для GitHub Pages (отключает Jekyll)
├── README.md               # Этот файл
│
├── recipe_bot_llm/         # Проект 1
│   └── index.html
│
├── saas/                   # Проект 2
│   └── index.html
│
├── ostrov-okon/            # Проект 3
│   ├── index.html
│   └── screenshots/
│       ├── photo1.jpg
│       └── savvi-chat.html
│
├── mortgage-calculator/    # Проект 4
│   └── index.html
│
├── yamabushi-game/         # Японские кроссворды (игра)`n└── new-project/            # Новый проект
    ├── index.html
    └── screenshots/
```

## 📝 Как добавить новый проект

### Шаг 1: Создай папку проекта

```
portfolio-assistant/new-project-name/
```

### Шаг 2: Создай `index.html` внутри папки

**Шаблон для страницы проекта:**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Название проекта</title>
  <style>
    body { margin: 0; font-family: Inter, system-ui, sans-serif; background: #eef9ff; color: #0f172a; }
    .page { max-width: 920px; margin: auto; padding: 34px 24px; }
    .breadcrumbs { margin-bottom: 20px; font-size: .95rem; color: #475569; }
    .breadcrumbs a { color: #2563eb; text-decoration: none; }
    .badge { display: inline-flex; align-items: center; gap: .5rem; padding: 8px 14px; background: #dbeafe; color: #1e40af; border-radius: 999px; font-weight: 700; }
    h1 { margin: 10px 0 12px; font-size: clamp(2rem, 4vw, 2.9rem); }
    p { color: #475569; line-height: 1.8; margin: 0; }
    section { margin-top: 28px; }
    section h2 { margin-bottom: 12px; font-size: 1.4rem; }
    ul { padding-left: 20px; color: #475569; }
    ul li { margin-bottom: 10px; }
    .links { display: flex; flex-wrap: wrap; gap: 12px; margin-top: 24px; }
    .button { display: inline-flex; align-items: center; justify-content: center; padding: 12px 18px; border-radius: 12px; background: #2563eb; color: #fff; text-decoration: none; font-weight: 600; }
  </style>
</head>
<body>
  <main class="page">
    <div class="breadcrumbs"><a href="../index.html">Главная</a> → <a href="../projects.html">Каталог</a> → Название проекта</div>
    <span class="badge">Категория</span>
    <h1>Название проекта</h1>
    <p>Описание проекта.</p>
    
    <section>
      <h2>Что реализовано</h2>
      <ul>
        <li>Функция 1</li>
        <li>Функция 2</li>
      </ul>
    </section>
    
    <section>
      <h2>Технологии</h2>
      <ul>
        <li>Технология 1</li>
        <li>Технология 2</li>
      </ul>
    </section>
    
    <div class="links">
      <a class="button" href="https://github.com/..." target="_blank">GitHub</a>
      <a class="button" href="../projects.html">Вернуться в каталог</a>
    </div>
  </main>
</body>
</html>
```

### Шаг 3: Добавь скриншоты (если нужны)

```
portfolio-assistant/new-project-name/screenshots/
├── screenshot1.jpg
└── screenshot2.png
```

### Шаг 4: Добавь карточку в каталог

**Сообщи мне:**
- Название проекта
- Описание (1-2 предложения)
- Технологии (теги)
- Иконку/эмодзи (если хочешь)

**Я добавлю карточку в:**
- `projects.html` (каталог проектов)
- `index.html` (главная страница)

## 🎨 Пример запроса на добавление проекта

```
Добавь новый проект:
- Название: Telegram Shop Bot
- Описание: Бот для интернет-магазина с корзиной и оплатой
- Технологии: Python, aiogram, Stripe
- Иконка: 🛒
```

## 🌐 Доступ к проектам

После пуша на GitHub, проекты доступны по адресам:

- Главная: `https://твой-юзер.github.io/portfolio-assistant/`
- Каталог: `https://твой-юзер.github.io/portfolio-assistant/projects.html`
- Проект: `https://твой-юзер.github.io/portfolio-assistant/new-project-name/`

## 🔄 Обновление сайта

1. Сохрани изменения локально
2. Открой **GitHub Desktop**
3. Нажми **Refresh** (Ctrl+R)
4. Напиши комментарий к коммиту
5. Нажми **Commit to main**
6. Нажми **Push origin**
7. GitHub Pages обновится через 1-2 минуты

## 🎯 Особенности

- **Адаптивный дизайн** — работает на всех устройствах
- **Тёмная тема** — переключается через `styles.css`
- **Без фреймворков** — чистый HTML/CSS/JS
- **Быстрая загрузка** — статические файлы
- **Лёгкое добавление** — просто создай папку с `index.html`

---

© 2026 AI Agent Team. Сделано на HTML5 · CSS3 · Vanilla JS
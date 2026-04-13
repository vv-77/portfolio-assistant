# ⚠️ ИНСТРУКЦИЯ ПО ВОССТАНОВЛЕНИЮ ПРОЕКТА

## Если сайт сломан / отображается некорректно:

### Вариант 1: Откат через Git (самый быстрый)
```powershell
cd "C:\Users\Валерий\Documents\GitHub\portfolio-assistant"
git reset --hard HEAD
```

### Вариант 2: Восстановление из архива
1. Найди архив `portfolio-assistant-backup-*.zip` в папке `C:\Users\Валерий\Documents\GitHub\`
2. Распакуй его **вместо** текущей папки `portfolio-assistant`
```powershell
Expand-Archive "portfolio-assistant-backup-20260413-162617.zip" -DestinationPath "C:\Users\Валерий\Documents\GitHub\portfolio-assistant" -Force
```

---

## ЧТО НЕ ТРОГАТЬ БЕЗ НЕОБХОДИМОСТИ:

| Файл | Почему |
|------|--------|
| `index.html` | Главная страница — легко сломать кодировку |
| `projects.html` | Каталог проектов |
| `styles.css` | Стили всего сайта |

---

## ПЕРЕД ЛЮБЫМИ ПРАВКАМИ:

1. **Сделай коммит рабочей версии:**
   ```powershell
   git add .
   git commit -m "WORKING VERSION перед правками"
   ```

2. **Или скопируй файлы вручную:**
   ```powershell
   Copy-Item "index.html" "index.html.backup.$(Get-Date -Format 'yyyyMMdd-HHmmss')"
   ```

---

## ТЕКУЩАЯ ВЕРСИЯ:

- **Дата последнего бэкапа:** 13.04.2026 16:26
- **Статус:** Рабочая версия
- **Проекты на главной:** 4 проекта + карточка портфолио
- **Год в футере:** 2026

---

## КОНТАКТЫ ПО ВОПРОСАМ:

Если что-то сломалось — откатывайся к последнему рабочему коммиту или архиву.

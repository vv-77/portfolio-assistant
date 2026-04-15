# Sales Kit (единый файл) — HR AI Assistant B2B

Этот файл содержит **полную склейку** содержимого папки `sales/` для удобного офлайн‑просмотра и поиска по одному документу.

## Оглавление

- `TARIFFS.md`
- `UPGRADE-GUIDE.md`
- `support/client-faq.md`
- `support/troubleshooting.md`
- `support/escalation-flow.md`
- `billing/invoice-template.md`
- `billing/renewal-reminder.md`
- `billing/license-key-gen.py`
- `tariffs/cloud-onboard.py`
- `tariffs/hybrid-onboard.py`
- `tariffs/boxed-onboard.py`
- `tariffs/upgrade-path.py`
- `scripts/health-check.py`
- `scripts/migrate-tariff.py`
- `scripts/activate-client.sh`

---

## `TARIFFS.md`

```md
# Тарифы — HR AI Assistant B2B

| Тариф | Режим | Для кого | Что входит | Когда апгрейд |
|---|---|---|---|---|
| Cloud | `LLM_MODE=cloud` | Быстрый старт без локальной инфраструктуры | RAG + облачный LLM, web+telegram, обновления базы, поддержка | Нужно офлайн / требования ИБ / нет интернета |
| Hybrid | `LLM_MODE=hybrid` | Компромисс по ИБ (локальная база) | Локальный RAG, анонимизация запроса, облачный LLM | Нужно полностью офлайн или строгий контур |
| Boxed | `LLM_MODE=local` | Полный офлайн (закрытый контур) | Ollama + локальная модель, локальная база, лицензия, регламент обновлений | Нужна централизация/масштабирование/кластер |

## Как понять, что клиенту нужно больше

- **Cloud → Hybrid**, если:
  - нельзя отправлять текст в облако без анонимизации
  - нужно локально хранить документы и историю

- **Hybrid → Boxed**, если:
  - интернет нестабилен/запрещён
  - требуется полный офлайн (закрытый контур)

## Что важно проговорить заранее
- Где хранится база знаний (`rag.db_path`)
- Кто отвечает за обновление документов (`kb/raw/` и пересборка)
- Как ротировать ключи/токены (только `.env`)
```

---

## `UPGRADE-GUIDE.md`

```md
# UPGRADE-GUIDE — Cloud → Hybrid → Boxed

## Cloud → Boxed (5 шагов)

1) **Бэкап**
- Скопируйте `.env` и `config.yaml`
- Скопируйте папку базы знаний (по `rag.db_path`)

2) **Перевод режима**
- Используйте скрипт:
  - `sales/tariffs/upgrade-path.py --from cloud --to boxed`

3) **Проверка Ollama**
- Убедитесь, что Ollama установлена и доступна
- Проверьте локальную модель (по требованиям клиента)

4) **Проверка базы знаний**
- В `rag.db_path` должен быть `chroma.sqlite3`
- Прогоните `python scripts/test_query.py`

5) **Запуск и smoke-тест**
- `python main.py`
- Telegram: `/start` → вопрос текстом
- Web: `POST /api/v1/chat`

## Cloud → Hybrid

- `sales/tariffs/upgrade-path.py --from cloud --to hybrid`
- Убедитесь, что локальная база собрана (`python scripts/build_kb.py`)
- Проверьте анонимизацию (`sales/tariffs/hybrid-onboard.py`)

## Откат (если что-то пошло не так)
- Бэкапы лежат в `sales/backups/<timestamp>/`
- Восстановите `.env` и `config.yaml`, затем повторите запуск
```

---

## `support/client-faq.md`

```md
# FAQ для клиентов — HR AI Assistant B2B

## 1) Как задать вопрос?
- В Telegram: нажмите `/start`, затем задайте вопрос обычным текстом.
- В Web API: используйте `POST /api/v1/chat`.

## 2) Что поддерживается?
- ТК РФ, отпуска, больничные, справки, шаблоны документов (по загруженной базе знаний).

## 3) Почему бот отвечает «В базе нет данных»?
Варианты причин:
- В базе знаний нет нужного документа/раздела.
- База не собрана или собрана не по тому пути.
- Вопрос слишком общий или сформулирован не по теме документов.

Что делать:
- Добавьте документы в `kb/raw/` и запустите сборку базы.
- Повторите вопрос, добавив контекст (должность, ситуация, период).

## 4) Где хранится база знаний?
Путь указан в `config.yaml` → `rag.db_path` (по умолчанию `C:\HR-Local-Data\vector_db`).

## 5) Как сделать резервную копию?
Скопируйте папку базы целиком (см. путь выше).

## 6) Как ротировать ключи/токены?
- Замените значения в `.env`.
- Перезапустите приложение.
- Старые ключи отзовите в личном кабинете провайдера (если требуется).

## 7) Какие данные отправляются в облако?
В cloud/hybrid режимах вопрос отправляется провайдеру LLM. В hybrid режиме запрос анонимизируется (ФИО/телефон/email/ИНН).
```

---

## `support/troubleshooting.md`

```md
# Troubleshooting — HR AI Assistant B2B

## Чек-лист: «Бот не отвечает»

1) Проверить, что процесс запущен:
- в консоли видно: `🚀 HR-бот запущен...`

2) Проверить Web API:
- `POST http://127.0.0.1:8000/api/v1/chat`
- если порт занят → ошибка `winerror 10048` → завершить процесс на порту 8000

3) Проверить Telegram:
- если `TelegramConflictError` → запущен второй polling-инстанс → остановить все экземпляры бота

## Чек-лист: «Ошибка API / Ошибка LLM»

1) Проверить `.env`:
- `LLM_MODE`
- `CLOUD_API_KEY` (или `GEMINI_API_KEY`)
- `CLOUD_BASE_URL` / `GOOGLE_GEMINI_BASE_URL`

2) Проверить доступ в интернет.

3) Быстрый автотест:
- `python scripts/debug_rag.py`

## Чек-лист: «В базе нет данных»

1) Проверить, что база собрана:
- в `rag.db_path` существует `chroma.sqlite3`

2) Быстрая проверка поиска:
- `python scripts/test_query.py`

3) Если в `kb/raw/` добавили новые документы:
- запустить пересборку: `python scripts/build_kb.py`
```

---

## `support/escalation-flow.md`

```md
# Эскалация в техподдержку — HR AI Assistant B2B

## Когда эскалировать (в течение 15 минут)
- Приложение не запускается / падает сразу после запуска.
- Telegram не принимает сообщения / конфликты polling.
- Web API не поднимается (порт занят, 5xx).
- Cloud LLM возвращает 401/403/429/5xx.
- База знаний «пустая», хотя `scripts/test_query.py` показывает документы.

## Что собрать перед эскалацией (минимум)
- Режим: `LLM_MODE` (cloud/local/hybrid)
- Версия архива/папки поставки
- Путь `rag.db_path` из `config.yaml`
- Снимок вывода консоли при старте `python main.py` (последние 50–100 строк)
- Результат запуска:
  - `python scripts/debug_rag.py` (первые строки + итог)

## Каналы эскалации (заглушка)
- Email: {{support_email}}
- Telegram/чат: {{support_chat}}
- SLA: {{sla}}

## Классификация инцидента
- P1: прод недоступен / массовое падение
- P2: деградация качества / частичные отказы
- P3: консультации / вопросы по документам
```

---

## `billing/invoice-template.md`

```md
# Шаблон счёта (пример) — HR AI Assistant B2B

> Заполните реквизиты и суммы. Цены — в рублях. НДС — по вашему режиму налогообложения.

## Счёт № {{invoice_number}} от {{date}}

### Поставщик
- **Организация**: {{vendor_name}}
- **ИНН/КПП**: {{vendor_inn}} / {{vendor_kpp}}
- **ОГРН**: {{vendor_ogrn}}
- **Адрес**: {{vendor_address}}
- **Р/с**: {{vendor_rs}}
- **Банк**: {{vendor_bank}}
- **К/с**: {{vendor_ks}}
- **БИК**: {{vendor_bik}}
- **Email**: {{vendor_email}}

### Покупатель
- **Организация**: {{client_name}}
- **ИНН/КПП**: {{client_inn}} / {{client_kpp}}
- **Адрес**: {{client_address}}
- **Контакт**: {{client_contact}}
- **Email**: {{client_email}}

## Позиции

| № | Наименование | Кол-во | Ед. | Цена, ₽ | Сумма, ₽ |
|---:|---|---:|:--:|---:|---:|
| 1 | Лицензия HR AI Assistant B2B ({{tariff}}) | 1 | усл. | {{price_rub}} | {{price_rub}} |
| 2 | Техническая поддержка ({{support_term}}) | 1 | усл. | {{support_price_rub}} | {{support_price_rub}} |

**Итого**: {{total_rub}} ₽  
**НДС**: {{vat_rub}} ₽  
**Итого к оплате**: {{total_due_rub}} ₽

## Условия
- Срок поставки/активации: {{delivery_terms}}
- Срок оплаты: {{payment_terms}}
- Основание: Договор/оферта № {{contract_number}} от {{contract_date}}

## Примечания
- Доступы/ключи передаются через защищённый канал связи.
- Секреты клиента (токены/ключи) не принимаются на хранение — только в `.env` клиента.
```

---

## `billing/renewal-reminder.md`

```md
# Шаблон письма о продлении поддержки — HR AI Assistant B2B

Тема: Продление поддержки HR AI Assistant B2B ({{client_name}})

Здравствуйте, {{client_contact}}!

Напоминаем, что срок технической поддержки по продукту **HR AI Assistant B2B** заканчивается **{{support_end_date}}**.

### Что входит в поддержку
- консультации по настройке (cloud/local/hybrid)
- помощь с диагностикой ошибок API/Telegram/Web
- рекомендации по обновлению базы знаний и качеству документов
- плановые обновления (по договорённости)

### Варианты продления
- **12 месяцев**: {{price_12m}} ₽
- **6 месяцев**: {{price_6m}} ₽

Чтобы продлить поддержку, ответьте на это письмо и укажите желаемый срок (6/12 месяцев). Мы отправим счёт и инструкцию по продлению.

С уважением,  
{{vendor_name}}  
{{vendor_email}} | {{vendor_phone}}
```

---

## `billing/license-key-gen.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Генератор лицензий для Boxed.
# Формат: {client_id}-{timestamp}-{tariff}-{exp}-{checksum}
# checksum: SHA256 от payload + секрет (LICENSE_SECRET) → первые 12 символов.
# Секреты не хардкодим: берём из env/CLI.
import argparse
import hashlib
import os
import re
from dataclasses import dataclass
from datetime import date, datetime, timedelta, timezone

_KEY_RE = re.compile(
    r"^(?P<client>[^-]+)-(?P<ts>\d{14})-(?P<tariff>[^-]+)-(?P<exp>\d{4}-\d{2}-\d{2})-(?P<chk>[A-F0-9]{12})$"
)


@dataclass(frozen=True)
class LicenseInfo:
    client_id: str
    timestamp: str
    tariff: str
    expires: str
    checksum: str


def _checksum(payload: str, secret: str) -> str:
    h = hashlib.sha256((payload + "|" + secret).encode("utf-8")).hexdigest()
    return h[:12].upper()


def generate(client_id: str, tariff: str, days: int, secret: str) -> str:
    ts = datetime.now(timezone.utc).strftime("%Y%m%d%H%M%S")
    exp = (date.today() + timedelta(days=days)).isoformat()
    payload = f"{client_id}|{ts}|{tariff}|{exp}"
    chk = _checksum(payload, secret)
    return f"{client_id}-{ts}-{tariff}-{exp}-{chk}"


def parse(key: str) -> LicenseInfo | None:
    m = _KEY_RE.match((key or "").strip())
    if not m:
        return None
    return LicenseInfo(
        client_id=m.group("client"),
        timestamp=m.group("ts"),
        tariff=m.group("tariff"),
        expires=m.group("exp"),
        checksum=m.group("chk"),
    )


def verify(key: str, secret: str, *, allow_expired: bool = False) -> tuple[bool, str]:
    info = parse(key)
    if not info:
        return False, "Неверный формат ключа"

    payload = f"{info.client_id}|{info.timestamp}|{info.tariff}|{info.expires}"
    expected = _checksum(payload, secret)
    if expected != info.checksum:
        return False, "Контрольная сумма не совпадает (ключ повреждён или неверный секрет)"

    try:
        exp = date.fromisoformat(info.expires)
        if (date.today() > exp) and not allow_expired:
            return False, f"Лицензия истекла: {info.expires}"
    except Exception:
        return False, "Не удалось прочитать дату истечения"

    return True, "ОК"


def main() -> None:
    ap = argparse.ArgumentParser(description="Генерация/проверка лицензий (Boxed).")
    ap.add_argument("--generate", action="store_true", help="Сгенерировать ключ")
    ap.add_argument("--verify", metavar="KEY", help="Проверить ключ")
    ap.add_argument("--client-id", default="CLIENT", help="Идентификатор клиента")
    ap.add_argument("--tariff", default="BOXED", help="Тариф (например BOXED)")
    ap.add_argument("--days", type=int, default=365, help="Срок в днях")
    ap.add_argument("--secret", default=os.environ.get("LICENSE_SECRET", ""), help="Секрет (LICENSE_SECRET)")
    ap.add_argument("--allow-expired", action="store_true", help="Разрешить истёкшие при проверке")
    args = ap.parse_args()

    secret = (args.secret or "").strip()
    if not secret:
        print("⚠️ Не задан секрет LICENSE_SECRET. Это снижает защиту ключей.")
        print("Что делать дальше: задайте LICENSE_SECRET в окружении/CLI и повторите.")
        secret = "UNSAFE"

    if args.generate:
        key = generate(args.client_id, args.tariff, args.days, secret)
        print("✅ Ключ сгенерирован:")
        print(key)
        return

    if args.verify:
        ok, msg = verify(args.verify, secret, allow_expired=args.allow_expired)
        print(("✅" if ok else "⚠️") + f" Проверка: {msg}")
        return

    print("⚠️ Не выбрано действие. Используйте --generate или --verify KEY.")


if __name__ == "__main__":
    main()
```

---

## `tariffs/cloud-onboard.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Скрипт подключения клиента в Cloud-режиме (проверки + тестовый запрос).
# Секреты не хардкодим: читаем из .env / переменных окружения.
import asyncio
import os
from pathlib import Path


PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def _load_env() -> None:
    """Мягко подгружает .env в os.environ (без падения)."""
    p = PROJECT_ROOT / ".env"
    if not p.exists():
        print("⚠️ Файл .env не найден. Создайте его из .env.example и заполните ключи.")
        return
    try:
        for line in p.read_text(encoding="utf-8").splitlines():
            s = line.strip()
            if not s or s.startswith("#") or "=" not in s:
                continue
            k, v = s.split("=", 1)
            os.environ.setdefault(k.strip(), v.strip().strip('"'))
    except Exception as e:
        print(f"⚠️ Не удалось прочитать .env: {e}")


async def _test_cloud() -> str:
    """Тестовый запрос через штатный роутер LLM."""
    from core.llm_router import generate_answer  # noqa: E402
    system = (
        "Отвечай ТОЛЬКО по контексту. Если информации нет, пиши: "
        "'В базе нет данных. Обратитесь к HR-отделу.' "
        "Никогда не выдумывай статьи законов."
    )
    q = "Сколько дней отпуска положено за первый год?"
    ctx = (
        "Статья 115. Ежегодный основной оплачиваемый отпуск — 28 календарных дней.\n"
        "Статья 122. Право на отпуск за первый год — после 6 месяцев непрерывной работы."
    )
    return await generate_answer(system, q, ctx)


def main() -> None:
    _load_env()
    os.environ.setdefault("LLM_MODE", "cloud")

    tg = (os.environ.get("TELEGRAM_TOKEN") or "").strip()
    cloud_key = (os.environ.get("CLOUD_API_KEY") or "").strip()
    base = (os.environ.get("CLOUD_BASE_URL") or "").strip()

    if not tg:
        print("⚠️ TELEGRAM_TOKEN пустой. Telegram-канал не запустится (это не критично для Cloud).")
    if not cloud_key:
        print("⚠️ CLOUD_API_KEY пустой. Заполните .env и повторите запуск.")
        print("Что делать дальше: откройте .env → задайте CLOUD_API_KEY → повторите cloud-onboard.py.")
        return
    if not base:
        print("⚠️ CLOUD_BASE_URL пустой. Рекомендуется ProxyAPI (Gemini): https://api.proxyapi.ru/google")
        print("Что делать дальше: откройте .env → задайте CLOUD_BASE_URL → повторите cloud-onboard.py.")
        return

    try:
        ans = asyncio.run(_test_cloud())
        if str(ans).strip().startswith("⚠️ Ошибка LLM"):
            print(str(ans).strip())
            print("Что делать дальше: проверьте ключи/URL провайдера и доступ в интернет.")
            return
        print("✅ Готово. Бот отвечает в облачном режиме")
    except Exception as e:
        print(f"⚠️ Ошибка проверки Cloud: {e}")
        print("Что делать дальше: проверьте .env, доступ в интернет и повторите запуск.")


if __name__ == "__main__":
    main()
```

---

## `tariffs/hybrid-onboard.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Скрипт подключения клиента в Hybrid (RAG локально + LLM облако).
# Проверяет наличие локальной векторной БД и доступность облачного API, тестирует анонимизацию.
import asyncio
import os
from pathlib import Path

PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def _load_env() -> None:
    p = PROJECT_ROOT / ".env"
    if not p.exists():
        print("⚠️ .env не найден. Создайте из .env.example и заполните ключи.")
        return
    try:
        for line in p.read_text(encoding="utf-8").splitlines():
            s = line.strip()
            if not s or s.startswith("#") or "=" not in s:
                continue
            k, v = s.split("=", 1)
            os.environ.setdefault(k.strip(), v.strip().strip('"'))
    except Exception as e:
        print(f"⚠️ Не удалось прочитать .env: {e}")


def _check_chroma(db_path: str) -> bool:
    try:
        p = Path(db_path)
        if not p.exists():
            return False
        # Минимальная проверка характерных артефактов.
        sqlite_ok = (p / "chroma.sqlite3").exists()
        return sqlite_ok
    except Exception:
        return False


async def _test_hybrid() -> tuple[str, str]:
    from core.llm_router import generate_answer  # noqa: E402
    from core.llm_router import _anonymize as anonymize  # noqa: E402

    system = (
        "Отвечай ТОЛЬКО по контексту. Если информации нет, пиши: "
        "'В базе нет данных. Обратитесь к HR-отделу.' "
        "Никогда не выдумывай статьи законов."
    )
    # Здесь намеренно есть персональные данные — проверяем, что анонимайзер их режет.
    q = "Иванов Иван +79001234567: сколько дней отпуска положено за первый год?"
    ctx = "Ежегодный основной оплачиваемый отпуск — 28 календарных дней."
    red = anonymize(q)
    ans = await generate_answer(system, q, ctx)
    return red, ans


def main() -> None:
    _load_env()
    os.environ["LLM_MODE"] = "hybrid"

    from core.config import config  # noqa: E402

    cloud_key = (os.environ.get("CLOUD_API_KEY") or "").strip() or (os.environ.get("GEMINI_API_KEY") or "").strip()
    base = (os.environ.get("CLOUD_BASE_URL") or "").strip() or (os.environ.get("GOOGLE_GEMINI_BASE_URL") or "").strip()

    if not cloud_key:
        print("⚠️ Не задан ключ для облака (CLOUD_API_KEY / GEMINI_API_KEY). Заполните .env.")
        print("Что делать дальше: откройте .env → задайте ключ → повторите hybrid-onboard.py.")
        return
    if not base and not str(getattr(config, "cloud", None) and config.cloud.base_url or "").strip():
        print("⚠️ Не задан base_url провайдера (CLOUD_BASE_URL / GOOGLE_GEMINI_BASE_URL). Заполните .env.")
        return

    if not _check_chroma(config.rag.db_path):
        print(f"⚠️ Локальная векторная БД не найдена или неполная: {config.rag.db_path}")
        print("Что делать дальше: положите документы в kb/raw/ → запустите python scripts/build_kb.py.")
        return

    try:
        red, ans = asyncio.run(_test_hybrid())
        if "[РЕДАКТИРОВАНО]" not in red:
            print("⚠️ Анонимизация не сработала. Проверьте регулярные выражения/входные данные.")
            print(f"Факт: {red}")
            return
        if str(ans).strip().startswith("⚠️ Ошибка LLM"):
            print(str(ans).strip())
            print("Что делать дальше: проверьте ключ/URL провайдера и доступ в интернет.")
            return
        print("✅ Гибридный режим активен")
    except Exception as e:
        print(f"⚠️ Ошибка проверки Hybrid: {e}")
        print("Что делать дальше: проверьте .env и повторите запуск.")


if __name__ == "__main__":
    main()
```

---

## `tariffs/boxed-onboard.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Скрипт активации Boxed (полный офлайн): проверка Ollama + локальная БД + генерация лицензии.
# Секреты не хардкодим. Лицензия генерируется с checksum на основании LICENSE_SECRET (env/CLI).
import argparse
import asyncio
import hashlib
import os
import uuid
from datetime import datetime, timedelta, timezone
from pathlib import Path

import requests

PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def _load_env() -> None:
    p = PROJECT_ROOT / ".env"
    if not p.exists():
        return
    try:
        for line in p.read_text(encoding="utf-8").splitlines():
            s = line.strip()
            if not s or s.startswith("#") or "=" not in s:
                continue
            k, v = s.split("=", 1)
            os.environ.setdefault(k.strip(), v.strip().strip('"'))
    except Exception:
        pass


def _check_chroma(db_path: str) -> bool:
    try:
        p = Path(db_path)
        return p.exists() and (p / "chroma.sqlite3").exists()
    except Exception:
        return False


def _check_ollama(base_url: str) -> bool:
    try:
        url = base_url.rstrip("/") + "/api/tags"
        r = requests.get(url, timeout=10)
        return r.status_code == 200
    except Exception:
        return False


async def _offline_answer() -> str:
    from core.llm_router import generate_answer  # noqa: E402
    system = (
        "Отвечай ТОЛЬКО по контексту. Если информации нет, пиши: "
        "'В базе нет данных. Обратитесь к HR-отделу.' "
        "Никогда не выдумывай статьи законов."
    )
    q = "Сколько дней отпуска положено за первый год?"
    ctx = "Ежегодный основной оплачиваемый отпуск — 28 календарных дней."
    return await generate_answer(system, q, ctx)


def _make_license(client_id: str, days: int, secret: str) -> tuple[str, str]:
    exp = (datetime.now(timezone.utc) + timedelta(days=days)).date().isoformat()
    ts = datetime.now(timezone.utc).strftime("%Y%m%d%H%M%S")
    raw = f"{client_id}|{ts}|{exp}"
    chk = hashlib.sha256((raw + "|" + secret).encode("utf-8")).hexdigest()[:12].upper()
    key = f"{client_id}-{ts}-{exp}-{chk}"
    return key, exp


def main() -> None:
    _load_env()
    parser = argparse.ArgumentParser(description="Boxed onboarding (офлайн).")
    parser.add_argument("--client-id", default=str(uuid.uuid4()), help="ID клиента (по умолчанию UUID)")
    parser.add_argument("--days", type=int, default=365, help="Срок лицензии в днях (по умолчанию 365)")
    parser.add_argument("--license-secret", default=os.environ.get("LICENSE_SECRET", ""), help="Секрет для checksum (не хранить в коде)")
    args = parser.parse_args()

    os.environ["LLM_MODE"] = "local"

    from core.config import config  # noqa: E402

    if not _check_chroma(config.rag.db_path):
        print(f"⚠️ Векторная БД не найдена: {config.rag.db_path}")
        print("Что делать дальше: положите документы в kb/raw/ → запустите python scripts/build_kb.py.")
        return

    if not _check_ollama(config.llm.base_url):
        print(f"⚠️ Ollama недоступна по адресу: {config.llm.base_url}")
        print("Что делать дальше: установите/запустите Ollama и проверьте, что API доступен.")
        return

    try:
        ans = asyncio.run(_offline_answer())
        if str(ans).strip().startswith("⚠️ Ошибка LLM"):
            print(str(ans).strip())
            print("Что делать дальше: проверьте локальную модель Ollama и ресурсы (RAM).")
            return
    except Exception as e:
        print(f"⚠️ Ошибка офлайн-теста: {e}")
        return

    secret = (args.license_secret or "").strip()
    if not secret:
        print("⚠️ Не задан license-secret (LICENSE_SECRET). Лицензия будет создана БЕЗ защиты checksum.")
        secret = "UNSAFE"

    key, _exp = _make_license(args.client_id, args.days, secret)
    try:
        lic_path = PROJECT_ROOT / "license.key"
        lic_path.write_text(key, encoding="utf-8")
    except Exception as e:
        print(f"⚠️ Не удалось записать license.key: {e}")
        print(f"Лицензия (скопируйте вручную): {key}")
        return

    masked = key[:4] + "-XXXX"
    print(f"✅ Коробка активирована. Лицензия: {masked}")


if __name__ == "__main__":
    main()
```

---

## `tariffs/upgrade-path.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Миграция тарифа/режима: Cloud → Hybrid → Boxed.
# Делает бэкап конфигов, точечно обновляет .env и config.yaml, затем прогоняет базовые тесты.
import argparse
import os
import shutil
import subprocess
from datetime import datetime
from pathlib import Path
from typing import Any

import yaml

PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def _backup(path: Path, backup_dir: Path) -> None:
    try:
        backup_dir.mkdir(parents=True, exist_ok=True)
        if path.exists():
            shutil.copy2(path, backup_dir / path.name)
    except Exception as e:
        print(f"⚠️ Не удалось сделать бэкап {path.name}: {e}")


def _read_env(path: Path) -> dict[str, str]:
    out: dict[str, str] = {}
    if not path.exists():
        return out
    try:
        for line in path.read_text(encoding="utf-8").splitlines():
            s = line.strip()
            if not s or s.startswith("#") or "=" not in s:
                continue
            k, v = s.split("=", 1)
            out[k.strip()] = v.strip().strip('"')
    except Exception as e:
        print(f"⚠️ Не удалось прочитать .env: {e}")
    return out


def _write_env_pointwise(path: Path, updates: dict[str, str]) -> None:
    """Точечная запись: сохраняет существующие ключи, добавляет отсутствующие."""
    current = _read_env(path)
    current.update({k: v for k, v in updates.items() if v is not None})
    try:
        lines = []
        for k, v in current.items():
            vv = (v or "")
            lines.append(f'{k}="{vv}"')
        path.write_text("\n".join(lines) + "\n", encoding="utf-8")
    except Exception as e:
        print(f"⚠️ Не удалось обновить .env: {e}")


def _update_config_yaml(path: Path, to_mode: str) -> None:
    try:
        data: dict[str, Any] = {}
        if path.exists():
            with open(path, encoding="utf-8") as f:
                data = yaml.safe_load(f) or {}
        dep = data.get("deployment") or {}
        if not isinstance(dep, dict):
            dep = {}
        dep["mode"] = to_mode
        data["deployment"] = dep
        with open(path, "w", encoding="utf-8") as f:
            yaml.dump(data, f, sort_keys=False, allow_unicode=True)
    except Exception as e:
        print(f"⚠️ Не удалось обновить config.yaml: {e}")


def _run_tests() -> bool:
    try:
        r = subprocess.run(
            [sys.executable, "-m", "pytest", "-q"],
            cwd=str(PROJECT_ROOT),
            capture_output=True,
            text=True,
            timeout=600,
        )
        if r.returncode != 0:
            print("⚠️ Тесты не прошли.")
            if r.stdout:
                print(r.stdout.strip()[:2000])
            if r.stderr:
                print(r.stderr.strip()[:2000])
            return False
        return True
    except Exception as e:
        print(f"⚠️ Не удалось запустить тесты: {e}")
        return False


def main() -> None:
    parser = argparse.ArgumentParser(description="Upgrade path: cloud/hybrid/boxed.")
    parser.add_argument("--from", dest="src", required=True, choices=["cloud", "hybrid", "boxed"])
    parser.add_argument("--to", dest="dst", required=True, choices=["cloud", "hybrid", "boxed"])
    args = parser.parse_args()

    ts = datetime.now().strftime("%Y%m%d-%H%M%S")
    backup_dir = PROJECT_ROOT / "sales" / "backups" / ts
    env_path = PROJECT_ROOT / ".env"
    cfg_path = PROJECT_ROOT / "config.yaml"

    _backup(env_path, backup_dir)
    _backup(cfg_path, backup_dir)

    # Точечно обновляем режим.
    updates = {"LLM_MODE": "cloud" if args.dst in ("cloud", "hybrid") else "local"}
    _write_env_pointwise(env_path, updates)
    _update_config_yaml(cfg_path, "cloud" if args.dst == "cloud" else ("hybrid" if args.dst == "hybrid" else "local"))

    ok = _run_tests()

    print("\nЧек-лист после апгрейда:")
    print("- Проверьте запуск: python main.py")
    print("- Проверьте Web API: POST http://127.0.0.1:8000/api/v1/chat")
    print("- Проверьте Telegram: /start и вопрос текстом")
    if args.dst in ("hybrid", "boxed"):
        print("- Проверьте наличие базы: C:\\HR-Local-Data\\vector_db (или rag.db_path в config.yaml)")
    if args.dst == "boxed":
        print("- Проверьте Ollama и локальную модель")

    if ok:
        print("✅ Миграция выполнена")
    else:
        print("⚠️ Миграция выполнена, но тесты не прошли. Смотрите вывод выше и откатитесь из sales/backups при необходимости.")


if __name__ == "__main__":
    main()
```

---

## `scripts/health-check.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Диагностика: "Почему бот тормозит?" / "Почему не отвечает?"
# Скрипт безопасный: ничего не меняет, только проверяет окружение и доступность компонентов.
import asyncio
import os
import time
from pathlib import Path


PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def _load_env() -> None:
    p = PROJECT_ROOT / ".env"
    if not p.exists():
        print("⚠️ .env не найден. Создайте из .env.example и заполните.")
        return
    try:
        for line in p.read_text(encoding="utf-8").splitlines():
            s = line.strip()
            if not s or s.startswith("#") or "=" not in s:
                continue
            k, v = s.split("=", 1)
            os.environ.setdefault(k.strip(), v.strip().strip('"'))
    except Exception as e:
        print(f"⚠️ Не удалось прочитать .env: {e}")


def _check_db(db_path: str) -> bool:
    p = Path(db_path)
    ok = p.exists() and (p / "chroma.sqlite3").exists()
    print(("✅" if ok else "⚠️") + f" Векторная БД: {db_path}")
    return ok


async def _probe_llm() -> bool:
    from core.llm_router import generate_answer  # noqa: E402
    system = (
        "Отвечай ТОЛЬКО по контексту. Если информации нет, пиши: "
        "'В базе нет данных. Обратитесь к HR-отделу.' "
        "Никогда не выдумывай статьи законов."
    )
    q = "Тест: скажи ОК"
    ctx = "Если вопрос тестовый — ответь: ОК."
    t0 = time.time()
    ans = await generate_answer(system, q, ctx)
    dt = time.time() - t0
    if str(ans).strip().startswith("⚠️ Ошибка LLM"):
        print(f"⚠️ LLM: {ans}")
        return False
    print(f"✅ LLM отвечает за {dt:.2f} сек")
    return True


def main() -> None:
    _load_env()
    from core.config import config  # noqa: E402

    mode = (os.environ.get("LLM_MODE") or config.deployment.mode or "local").strip().lower()
    print(f"Режим: {mode}")

    db_ok = _check_db(config.rag.db_path)
    try:
        llm_ok = asyncio.run(_probe_llm())
    except Exception as e:
        print(f"⚠️ LLM проверка не выполнена: {e}")
        llm_ok = False

    if not db_ok:
        print("Что делать дальше: добавьте документы в kb/raw/ → запустите python scripts/build_kb.py.")
    if mode in ("cloud", "hybrid") and not llm_ok:
        print("Что делать дальше: проверьте CLOUD_API_KEY / CLOUD_BASE_URL и доступ в интернет.")
    if mode == "local" and not llm_ok:
        print("Что делать дальше: проверьте Ollama, модель и ресурсы (RAM).")

    if db_ok and llm_ok:
        print("✅ Health-check пройден")


if __name__ == "__main__":
    main()
```

---

## `scripts/migrate-tariff.py`

```python
# Version: 1.0
import sys
if sys.platform == "win32":
    try: sys.stdout.reconfigure(encoding="utf-8")
    except: pass

# Перевод клиента на другой тариф: бэкап → миграция → тест.
# Этот скрипт не хардкодит секреты и не удаляет данные.
import argparse
import subprocess
from pathlib import Path


PROJECT_ROOT = Path(__file__).resolve().parents[2]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))


def main() -> None:
    ap = argparse.ArgumentParser(description="Миграция тарифа (cloud/hybrid/boxed).")
    ap.add_argument("--from", dest="src", required=True, choices=["cloud", "hybrid", "boxed"])
    ap.add_argument("--to", dest="dst", required=True, choices=["cloud", "hybrid", "boxed"])
    args = ap.parse_args()

    tool = PROJECT_ROOT / "sales" / "tariffs" / "upgrade-path.py"
    if not tool.exists():
        print("⚠️ Не найден sales/tariffs/upgrade-path.py")
        return

    try:
        r = subprocess.run(
            [sys.executable, str(tool), "--from", args.src, "--to", args.dst],
            cwd=str(PROJECT_ROOT),
            text=True,
        )
        if r.returncode == 0:
            print("✅ Миграция тарифа завершена")
        else:
            print("⚠️ Миграция завершилась с ошибкой. Смотрите вывод выше.")
    except Exception as e:
        print(f"⚠️ Не удалось выполнить миграцию: {e}")
        print("Что делать дальше: запустите sales/tariffs/upgrade-path.py вручную и проверьте бэкапы.")


if __name__ == "__main__":
    main()
```

---

## `scripts/activate-client.sh`

```bash
#!/usr/bin/env bash
# Version: 1.0
# Активация нового клиента (для Linux/macOS окружений).
# На Windows рекомендуется выполнять аналогичные шаги в PowerShell вручную.
#
# Скрипт:
# - создаёт .env из .env.example (если нет)
# - проверяет наличие ключевых переменных
# - выполняет простой health-check (если доступен python)

set -euo pipefail

ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/../.." && pwd)"

cd "$ROOT_DIR"

if [[ ! -f ".env" ]]; then
  if [[ -f ".env.example" ]]; then
    cp ".env.example" ".env"
    echo "✅ Создан .env из .env.example"
  else
    echo "⚠️ .env.example не найден. Нечего копировать."
    exit 0
  fi
fi

missing=0
grep -q '^LLM_MODE=' .env || { echo "⚠️ Нет LLM_MODE в .env"; missing=1; }
grep -q '^TELEGRAM_TOKEN=' .env || { echo "⚠️ Нет TELEGRAM_TOKEN в .env"; }
grep -q '^CLOUD_API_KEY=' .env || { echo "⚠️ Нет CLOUD_API_KEY в .env"; }
grep -q '^CLOUD_BASE_URL=' .env || { echo "⚠️ Нет CLOUD_BASE_URL в .env"; }

if [[ "$missing" -eq 1 ]]; then
  echo "Что делать дальше: откройте .env и заполните переменные."
  exit 0
fi

if command -v python >/dev/null 2>&1; then
  python sales/scripts/health-check.py || true
else
  echo "⚠️ python не найден в PATH. Пропускаю health-check."
fi

echo "✅ Клиент активирован"
```


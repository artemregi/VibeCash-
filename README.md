# VibeCash Quiz — Документация

## Что это

Квиз "Какой шанс, что ИИ заменит тебя в 2027?" — лид-генератор для проекта VibeCash.
Пользователь проходит анкету, получает результат (типаж + процент замены ИИ), оставляет контакты.
Данные приходят в Telegram-бота красивым сообщением + сохраняются для выгрузки в CSV/Excel.

---

## Архитектура

```
[Квиз (index.html)]
        |
        |-- Telegram Bot API (sendMessage) --> красивое сообщение в чат
        |-- Cloudflare Worker (/save)      --> сохранение лида в KV Storage

[Telegram бот]
        |
        |-- Кнопка "📊 Лиды"  --> CSV файл
        |-- Кнопка "📋 Excel" --> .xls файл с оформлением
        |
        |-- Cloudflare Worker (webhook) обрабатывает команды
```

---

## Сервисы и доступы

### 1. GitHub — репозиторий с кодом квиза
- **Репо:** https://github.com/artemregi/VibeCash-
- **Файл:** `index.html` — весь квиз в одном файле
- **Аккаунт:** artemregi

### 2. Telegram Bot
- **Имя бота:** @VibeCash_leads_bot
- **Bot Token:** `8747524672:AAEv7-IJv_uLBVueeiGiSv_q3dpnSSQOqJY`
- **Chat ID (группа для лидов):** `-4996740635`
- **BotFather:** https://t.me/BotFather — управление ботом

### 3. Cloudflare Workers — бэкенд бота
- **Аккаунт:** Higidra1946@gmail.com
- **Дашборд:** https://dash.cloudflare.com
- **Worker URL:** `https://broken-forest-9451.higidra1946.workers.dev`
- **Эндпоинты:**
  - `/save` — POST, сохраняет лид (вызывается из квиза)
  - `/webhook` — POST, принимает Telegram updates
  - `/register` — GET, регистрирует вебхук в Telegram
- **KV Namespace:** `LEADS` (привязан к Worker как binding `LEADS`)
- **Данные хранятся:** KV → ключ `all` → JSON массив лидов

---

## Как развернуть с нуля

### Шаг 1: Telegram бот
1. Открой https://t.me/BotFather
2. `/newbot` → дай имя → получи токен
3. Создай группу/канал для лидов, добавь бота
4. Узнай Chat ID (отправь сообщение, проверь через `https://api.telegram.org/bot<TOKEN>/getUpdates`)

### Шаг 2: Cloudflare Worker
1. Зарегистрируйся на https://dash.cloudflare.com (бесплатно)
2. **Compute → Workers & Pages → Create → Start with Hello World → Deploy**
3. **Edit Code** → вставь код Worker (см. раздел ниже) → **Deploy**
4. **Storage & databases → KV → Create namespace** → назови `LEADS`
5. **Вернись к Worker → Bindings → Add → KV Namespace** → Variable: `LEADS`, namespace: `LEADS`
6. Открой в браузере: `https://<worker-url>/register` — зарегистрирует вебхук
7. В боте напиши `/start` — появятся кнопки

### Шаг 3: Обнови index.html
В файле `index.html` замени 3 константы:
```javascript
const TG_BOT = "<BOT_TOKEN>";
const TG_CHAT = "<CHAT_ID>";
const WORKER_URL = "https://<worker-url>.workers.dev";
```

### Шаг 4: Задеплой квиз
```bash
git clone https://github.com/artemregi/VibeCash-.git
# отредактируй index.html если нужно
git add index.html
git commit -m "Update config"
git push origin main
```

---

## Код Cloudflare Worker

Полный код Worker хранится в Cloudflare UI (Edit Code).
Если потерял — возьми из этого README.

Ключевые части:
- `BOT_TOKEN` и `CHAT_ID` — захардкожены в Worker
- `/save` — принимает JSON лида, пишет в KV
- `/webhook` — обрабатывает Telegram команды (/start, 📊 Лиды, 📋 Excel)
- `buildCsv()` — генерирует CSV
- `buildXls()` — генерирует HTML-таблицу в формате .xls с оформлением
- `exportFile()` — отправляет файл в Telegram через sendDocument

---

## Секретный тест

В поле Telegram на первом экране квиза введи `///go` и нажми "Далее".
Все ответы заполнятся рандомно и отправятся — для тестирования бота и таблицы.

---

## Структура данных лида (JSON)

```json
{
  "timestamp": "2026-05-25T12:00:00Z",
  "contact": {
    "name": "Имя",
    "telegram": "@username",
    "wants_diagnostic_call": true
  },
  "result": {
    "typazh": "АРХИТЕКТОР",
    "knowledge_percent": 85,
    "replacement_risk_percent": 12,
    "raw_knowledge_points": 170
  },
  "answers": {
    "age": "25-30",
    "occupation": "dev",
    "income": "70-150",
    "tools": ["chatgpt", "claude"],
    "frequency": "daily_pro",
    "purpose": ["work", "earn"],
    "skills": ["agent", "vibecode"],
    "tried_to_earn": "текст",
    "city": "moscow",
    "blocker": "no_clients",
    "budget": "30-100"
  }
}
```

---

## Типажи

| Ключ | Название | Знания % | Цвет |
|------|----------|----------|------|
| ghost | ПРИЗРАК | 0–19 | red |
| dinosaur | ДИНОЗАВР | 20–39 | red |
| user | ЮЗЕР | 40–59 | amber |
| practitioner | ПРАКТИК | 60–79 | green |
| architect | АРХИТЕКТОР | 80–100 | green |

---

## Важные заметки

- **Google Apps Script НЕ работает** с Telegram вебхуками (302 редирект). Используем Cloudflare Workers.
- **Университетские Google аккаунты** (sdu.edu.kz и т.п.) блокируют внешний доступ к Apps Script. Используй личный Gmail.
- Worker бесплатный — 100,000 запросов/день.
- KV бесплатный — 100,000 чтений/день, 1,000 записей/день.

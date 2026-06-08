---
name: wnw-outreach-check
description: >
  Проверка результативности WNW-рассылки и работы ИИ-ассистента (Татьяна) на ответы.
  Логинится в КЛИЕНТСКИЙ аккаунт Railway (Womennetworking club), проверяет, что листенер
  жив и реально отвечает, затем считает воронку и метрики из Supabase. READ-ONLY.
  Триггеры: проверь рассылку, как работает бот, результативность аутрича, отвечает ли
  ассистент, метрики WNW, жив ли листенер, reply-rate, проверка outreach.
metadata:
  domain: wnw-outreach
  scope: review
  output-format: report
  account: railway-client
---

# WNW Outreach — проверка результативности (Railway клиентки + Supabase)

Скилл отвечает на два вопроса разом:

1. **Операционка:** ИИ-ассистент (Татьяна) реально жив и отвечает участницам? Листенер на
   Railway не упал, сессия Telegram не аннулирована, рассылка и follow-up отработали?
2. **Результативность:** какова воронка по когорте — reply-rate, link-rate, скорость ответа
   ассистента (TTFR), эскалации к менеджеру, утечки автоответчиков, отказы, отыгрыш
   мягких возражений?

Первый слой берётся из **Railway-логов клиентского аккаунта**, второй — из **Supabase**
(`scripts/outreach_metrics.py`, read-only). Итог — отчёт в `clients/wnw/tasks/`.

---

## ⛔ Жёсткие правила (читать до запуска)

1. **Скилл строго READ-ONLY.** Никаких `railway up` / `redeploy` / `down` /
   `variables --set`, никаких записей в БД. Только `status`, `logs`, `variables` (чтение),
   `ssh ... "ls/cat"`, `SELECT`. Если возникает соблазн что-то «починить» — это другой
   скилл/сессия, здесь только диагностика.
2. **Сначала подтвердить аккаунт.** CLI по умолчанию залогинен в ЛИЧНЫЙ
   `gnilosiranton`, прод WNW — на КЛИЕНТСКОМ `clubwomennetworking@gmail.com`
   (проект `wnw-api` = `a81373a6`). Перепутать аккаунты тривиально. Пока шаг 0 не
   подтвердил клиентский проект — дальше не идти.
3. **Не запускать листенер локально.** Любой локальный `tg_outreach.py listen` при живом
   Railway-листенере → `AuthKeyDuplicatedError` и аннулирование сессии (нужна
   перегенерация: телефон + SMS + 2FA). Диагностика идёт только через `railway logs`,
   `railway ssh` и read-only SQL.
4. **Метрики не путать с операционкой.** Пустой output, «нет данных», тишина в логах ≠
   «всё ок». Низкий reply-rate может означать мёртвого листенера, а не плохой текст —
   всегда сверять оба слоя.

---

## Шаг 0 — Гейт аккаунта Railway (КРИТИЧНО)

### 0.1 Достать клиентский project-token

Прод WNW деплоится/читается **project-токеном** клиентского воркспейса (формат UUID),
а не через `railway login`. Порядок поиска токена:

1. Переменная окружения сессии: `echo "${RAILWAY_TOKEN_WNW_CLIENT:+set}"`.
2. Локальный файл (эфемерный, в `/tmp`, часто отсутствует после перезагрузки):
   `cat /tmp/railway-migration/tokens.env 2>/dev/null` — ищи токен для проекта `wnw-api`.
3. Если нигде нет — **спросить Антона** (или взять в дашборде клиентки:
   Railway → Womennetworking club → проект `wnw-api` → Settings → Tokens). НЕ полагаться
   на `/tmp`.

Дальше во всех командах токен подставляется так:

```bash
RAILWAY_TOKEN=<client_project_token> railway <команда>
```

### 0.2 Подтвердить, что это именно клиентский `wnw-api`

```bash
RAILWAY_TOKEN=<client_project_token> railway status --json
```

Проверки (все три должны сойтись), иначе **СТОП**:

- project id начинается с `a81373a6` (`a81373a6-5bd9-496e-9810-8fc6733552ec`);
- имя проекта `wnw-api`, воркспейс `Womennetworking club`;
- в списке сервисов есть `tg-outreach-listener` и `tg-outreach-cron`.

> Project-токен `railway whoami` не проходит (это не user-токен) — это нормально.
> Идентичность подтверждается именно по `railway status --json`, а НЕ по `whoami`.

⚠️ Если видишь проект `cd7c8bfe` или воркспейс `Antthonyyy's Projects` — это ЛИЧНЫЙ
аккаунт, **не прод клиентки**. Останавливайся и бери правильный токен.

---

## Шаг 1 — Операционное здоровье (Railway, клиентский аккаунт)

Цель — доказать, что ассистент реально жив и отвечает, прежде чем верить метрикам.

### 1.1 Статус сервисов

```bash
RAILWAY_TOKEN=<token> railway status --json
```

Оба сервиса (`tg-outreach-listener`, `tg-outreach-cron`) должны быть в рабочем
состоянии (не `CRASHED`/`REMOVED`). Листенер — это 24/7-демон; он и отвечает участницам.

### 1.2 Логи листенера — отвечает ли ассистент

`railway logs` тейлит вывод, поэтому снимаем срез через `head` (SIGPIPE завершит стрим):

```bash
RAILWAY_TOKEN=<token> railway logs --service tg-outreach-listener 2>&1 | head -400
```

Грепать по сигнатурам из `scripts/tg_outreach.py` (источник истины по строкам логов):

| Сигнатура (что искать) | Где в коде | Что значит |
|---|---|---|
| `✅ … Ответ отправлен (часть …) → <имя> (@user)` | `tg_outreach.py:507` | **Ассистент реально ответил.** Считай число за сутки — это пульс. |
| `🚨 manual_hold — отложенный ответ отменён` | `tg_outreach.py:484` | Эскалация: бот замолчал, ждёт менеджера. |
| `👤 Менеджер уже ответил — бот молчит` | `tg_outreach.py:490` | Окно приоритета менеджера сработало (норма). |
| `⚠️ DB … failed` (save_message/add_alert/set_stage/get_stage) | `tg_outreach.py:280/290/299/309` | Сбой записи в БД — метрики/контекст могут плыть. |
| `❌ Ошибка отправки` | `tg_outreach.py:531` | Telegram отверг отправку (флуд-лимит / блок / пир). |
| `AuthKeyDuplicated` / `❌ Сессия не авторизована` | `tg_outreach.py:805,1270` | **Авария: сессия мертва, бот не отвечает вообще.** |

**Красные флаги:** ноль `✅ Ответ отправлен` за сутки при наличии входящих; любой
`AuthKeyDuplicated`; серия `❌ Ошибка отправки`; повторяющиеся `⚠️ DB … failed`.

### 1.3 Логи cron — рассылка и follow-up

```bash
RAILWAY_TOKEN=<token> railway logs --service tg-outreach-cron 2>&1 | head -200
```

Сверить, что отработали джобы non-TG (newcomers sync / regs / reconcile / export).
Сами TG-джобы (send_daily 11:00 Kyiv, follow_ups, nw_reminder, post_networking_check)
крутятся `outreach_scheduler_tg.py` **внутри listener-контейнера** — их следы искать в
логах листенера, не cron.

### 1.4 Дедуп-файл рассылки на volume (хрупкое место)

Холодная рассылка дедупится ТОЛЬКО JSON-файлом на volume (в БД дедупа нет —
`tg_outreach.py` ~1152). Потеря файла = риск повторной отправки уже отконтаченным +
PeerFlood:

```bash
RAILWAY_TOKEN=<token> railway ssh --service tg-outreach-listener \
  'ls -la /data/outreach_logs/*_sent.json'
```

Файл(ы) должны существовать и быть непустыми. Пусто/нет файла → флажок «риск дублей
при следующей авто-рассылке», эскалировать Антону (но в этом скилле НЕ чинить).

> `railway ssh` пробрасывает команду только аргументом (stdin не идёт) — поэтому `ls`/`cat`
> как строка-аргумент, без heredoc.

---

## Шаг 2 — Результативность (Supabase, read-only)

Прод-данные живут в Supabase (схема `outreach.*`). Основной инструмент —
`scripts/outreach_metrics.py`: только `SELECT`, `set_session(readonly=True)`, листенер/cron
не трогает.

### 2.1 Сверить, что считаем тот же DB, что и прод

`outreach_metrics.py` берёт DSN из локального `.env` (`SUPABASE_DB_URL`). Supabase общий
для обоих аккаунтов, поэтому обычно это и есть прод. Для надёжности сверь хост с тем,
что реально стоит у листенера клиентки:

```bash
RAILWAY_TOKEN=<token> railway variables --service tg-outreach-listener --json \
  | python3 -c "import json,sys; v=json.load(sys.stdin); u=v.get('SUPABASE_DB_URL',''); print('host:', u.split('@')[-1].split('/')[0] if u else '—')"
```

Хост должен совпасть с хостом в локальном `.env::SUPABASE_DB_URL`. Если расходятся —
метрики считаются по чужой БД, дальше нельзя (исправить DSN или взять прод-DSN из
railway variables и прокинуть через `SUPABASE_DB_URL=... python scripts/outreach_metrics.py`).

### 2.2 Запустить метрики

```bash
# окно по умолчанию — последние 7 дней
python scripts/outreach_metrics.py

# произвольное окно (даты включительно, UTC)
python scripts/outreach_metrics.py --since 2026-05-20 --until 2026-05-28
```

Когорта = контакты с первым касанием (out + template) в окне. Скрипт выдаёт:

- **reply-rate** — % когорты, кто ответил хоть раз;
- **автоответчики** — распространённость + **P4 KPI**: сколько просочилось в активные
  стадии (цель 0 — детектор автоответов должен ловить их до диалога);
- **стадии воронки** в когорте (`sent_first_touch`…`manual_hold`);
- **link-rate** — % дошедших до `got_link/registered/attended/qualified` (из когорты и из
  ответивших);
- **declined-rate** — % отказов;
- **escalations (manual_hold)** + сколько из них без последующего исходящего;
- **soft-objection → got_link** — отыгрыш «подумаю/не сейчас/неактуально»;
- **TTFR** — медиана/max времени `входящее → первый ответ` (скорость ассистента;
  считается по парам сообщений, не по контактам — не сравнивать напрямую с cohort-%).

### 2.3 Сводка по воронке/шаблонам (опционально)

Быстрые счётчики и аналитические view:

```bash
python scripts/outreach_db.py stats        # счётчики по таблицам
```

View (доступны и в Supabase, и в SQLite; `scripts/outreach_db.py:319+`):
`v_funnel_overall`, `v_template_performance` (A/B шаблонов: sent→reply→attend→bought),
`v_alerts_summary`, `v_daily_activity`, `v_contacts_full`.

> `scripts/generate_dashboard.py` собирает HTML, но завязан на **локальный SQLite** — для
> прод-Postgres он не годится без подмены backend. Для прод-картины опирайся на
> `outreach_metrics.py` + view через psql/`outreach_db.py`.

---

## Шаг 3 — Качество ответов ассистента (углублённо, по необходимости)

Если нужен разбор не «сколько», а «как» отвечает бот:

### 3.1 Метрики по логам листенера

Слить срез логов в файл и прогнать парсер (cache hit, экономия токенов, длина ответов,
подозрительные ответы — эмодзи/штампы/стены текста):

```bash
RAILWAY_TOKEN=<token> railway logs --service tg-outreach-listener 2>&1 | head -2000 > /tmp/listen_out.txt
python scripts/outreach_log_stats.py /tmp/listen_out.txt
python scripts/outreach_log_stats.py --tail 100        # последние 100 AI-запросов
```

### 3.2 Точечная проверка диалогов

Выбрать в `v_contacts_full` несколько свежих контактов с `in_count>0` и прочитать
их реальные `in→out` пары в `outreach.messages` (read-only SELECT). Сверить ответы бота
с правилами диалогов WNW (только русский; тон на «вы»; никаких обещаний персонального SLA;
цены 370/600/750/1100€; direct invite в первом касании). Нарушения — в отчёт как примеры.

---

## Шаг 4 — Отчёт

Сохранить в `clients/wnw/tasks/outreach_check_<YYYY-MM-DD>.md`. Структура:

```
# WNW Outreach — проверка <дата>, окно <since…until>

## Вердикт
ЗЕЛЁНЫЙ / ЖЁЛТЫЙ / КРАСНЫЙ — одной строкой (жив ли ассистент + здорова ли воронка)

## Операционка (Railway, аккаунт клиентки)
- Аккаунт/проект: подтверждён a81373a6 ✔
- listener / cron: статус
- Ответов за сутки (✅ Ответ отправлен): N
- Сессия: OK / AuthKeyDuplicated
- Ошибки отправки / DB-сбои: …
- Дедуп-файл на volume: есть/нет

## Результативность (Supabase, окно …)
- Когорта: N | reply-rate | link-rate | declined
- TTFR медиана/max
- Эскалации manual_hold (+ без ответа)
- Автоответчики: распространённость / P4-утечка
- Soft-objection → got_link
- A/B шаблонов (v_template_performance)

## Аномалии и примеры
(красные флаги из логов; 2-3 примера слабых ответов бота, если шаг 3)

## Рекомендации
(конкретные действия; что чинить — отдельной задачей, здесь не трогаем прод)
```

Делить факты на **verified** (видел в логах/SQL) и **believed** (предположение) —
не выдавать догадки за подтверждённое.

---

## Пороги интерпретации (ориентиры, не догма)

| Метрика | 🟢 норма | 🟡 внимание | 🔴 тревога |
|---|---|---|---|
| Ответов листенера/сутки при входящих | >0, соразмерно входящим | резко ниже обычного | **0 при наличии входящих** |
| Сессия Telegram | авторизована | — | любой `AuthKeyDuplicated` |
| reply-rate когорты | ≥ привычной базы | заметное падение | обвал + тишина в логах |
| TTFR медиана | минуты–десятки минут | часы | сутки+ (бот не отвечает) |
| P4-утечка автоответчиков | 0 | единичные | системные (детектор сломан) |
| escalations без ответа | около 0 | несколько | растущая очередь брошенных |
| дедуп-файл на volume | есть, непустой | — | отсутствует (риск дублей) |

База сравнения — прошлые `outreach_check_*` в `clients/wnw/tasks/` и история отправок.
Абсолютные «хорошо/плохо» зависят от объёма рассылки в окне — всегда смотреть в динамике.

---

## Справочник (инфраструктура прода)

**Аккаунт Railway:** `clubwomennetworking@gmail.com`, воркспейс «Womennetworking club».
**Проект:** `wnw-api` = `a81373a6-5bd9-496e-9810-8fc6733552ec`,
URL `https://wnw-api-production-ae17.up.railway.app`.

| Сервис | Service ID | Роль |
|---|---|---|
| `tg-outreach-listener` | `1405d447-1219-4a66-8aa1-a018cd2aa9dd` | listen + TG-джобы (send 11:00 Kyiv, follow_ups, nw_reminder, post_check) — отвечает участницам |
| `tg-outreach-cron` | `10f22df5-7833-4272-a438-4c1f347fbf5d` | non-TG (newcomers/regs/reconcile/export) |

**Инструменты (этот репо `/Users/mac/targetolog-ai-assistant`):**
- `scripts/outreach_metrics.py` — метрики воронки, read-only, DSN из `.env::SUPABASE_DB_URL`.
- `scripts/outreach_db.py stats` + view (`:319+`) — счётчики и аналитика.
- `scripts/outreach_log_stats.py` — разбор логов листенера (cache/токены/качество).
- `scripts/tg_outreach.py` — источник сигнатур логов (см. таблицу в 1.2).

**Подключённые памяти (контекст):** Railway-миграция на клиентку, split listener/cron,
дедуп volume-only, CRM dual-write, изоляция TG-аккаунтов, правила диалогов WNW.

**Чего НЕ делать здесь:** деплой/redeploy, правка env, запись в БД, локальный listen,
любые действия в `/Users/mac/Downloads/wnw crm/` (отдельный репо/продукт).

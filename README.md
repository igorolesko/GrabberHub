# GrabberHub

Персональний менеджер завантажень для Unraid.
Gopeed + yt-dlp + browserless-v2 в одному Web UI на порту 3030.

## Структура проекту

```
grabberhub/
├── backend/
│   ├── app.py              ← Flask, blueprints, запуск
│   ├── config.py           ← всі змінні оточення
│   ├── database.py         ← SQLite CRUD
│   ├── routes/
│   │   ├── download.py     ← POST /api/resolve, /api/download
│   │   ├── tasks.py        ← GET/DELETE/PUT /api/tasks
│   │   ├── cookies.py      ← /api/cookies
│   │   └── status.py       ← GET /api/status
│   └── engines/
│       ├── gopeed.py       ← Gopeed API клієнт
│       ├── ytdlp.py        ← yt-dlp завантаження
│       └── browserless.py  ← HDrezka/UAKino/Filmix авто-парсинг
├── frontend/
│   ├── index.html
│   ├── css/style.css
│   └── js/
│       ├── app.js          ← головна логіка, роутинг
│       ├── api.js          ← fetch запити
│       ├── ui.js           ← рендер, toast, форматування
│       └── tasks.js        ← список задач, polling
├── tampermonkey/
│   ├── hdrezka.js
│   ├── uakino.js
│   └── filmix.js
├── Dockerfile
└── docker-compose.yml
```

## Запуск

```bash
# 1. Клонуй проект
cp -r grabberhub /mnt/user/appdata/grabberhub/src
cd /mnt/user/appdata/grabberhub/src

# 2. Встанови BROWSERLESS_TOKEN в docker-compose.yml

# 3. Збери і запусти
docker compose up -d --build

# 4. Відкрий http://192.168.2.222:3030
```

## Движки

| Движок | Коли використовується |
|--------|----------------------|
| **Gopeed** | Прямі mp4/mkv, M3U8, торренти, HDrezka, UAKino, Filmix (через Tampermonkey) |
| **yt-dlp** | YouTube, Vimeo, TikTok, Instagram, 1800+ сайтів |
| **browserless** | HDrezka, UAKino, Filmix (автоматично, без Tampermonkey) |
| **auto** | Сервіс визначає сам по домену |

## Папки

```
/mnt/disk1/download/
├── film/       ← фільми
├── serial/     ← серіали
├── multik/     ← мультики
├── dok/        ← документальні
├── tvshows/    ← TV шоу
├── youtube/    ← YouTube/Vimeo тощо
└── incomplete/ ← тимчасові файли
```

## Tampermonkey скрипти

Встанови розширення Tampermonkey і додай скрипти з папки `tampermonkey/`.

| Скрипт | Сайт | Що робить |
|--------|------|-----------|
| `hdrezka.js` | HDrezka | Кнопка на плеєрі → відправляє mp4 URL |
| `uakino.js`  | UAKino  | Перехоплює M3U8 → кнопки 1080p/720p/480p |
| `filmix.js`  | Filmix  | Перехоплює mp4 → кнопки по якості |

Змінна `HUB_URL` в кожному скрипті — адреса твого GrabberHub.

## API

```
POST /api/resolve          ← аналіз URL
POST /api/download         ← запуск завантаження
GET  /api/tasks            ← список задач
DELETE /api/tasks/:id      ← видалити задачу
DELETE /api/tasks/completed ← очистити завершені
PUT  /api/tasks/:id/pause  ← пауза (Gopeed)
PUT  /api/tasks/:id/resume ← продовжити (Gopeed)
GET  /api/cookies          ← список cookie файлів
POST /api/cookies          ← завантажити cookie файл
DELETE /api/cookies/:name  ← видалити cookie файл
GET  /api/status           ← стан системи
```

## Оновлення yt-dlp

```bash
docker exec grabberhub yt-dlp --update
# або повний rebuild:
docker compose build --no-cache && docker compose up -d
```

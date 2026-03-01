# ⚡ AegisX

**Централизирана система за управление на банове за CS 1.6 / Half-Life / GoldSrc**

> Модерна замяна на AMXBans – AMX Mod X плъгини + REST API + React Web панел

[![Версия](https://img.shields.io/badge/версия-1.0.0-blue)]()
[![Лиценз](https://img.shields.io/badge/лиценз-GPL%20v2-green)]()
[![Платформа](https://img.shields.io/badge/платформа-CS%201.6%20%7C%20GoldSrc-orange)]()

---

## 📋 Съдържание

- [Функции](#функции)
- [Архитектура](#архитектура)
- [Изисквания](#изисквания)
- [Инсталация](#инсталация)
- [Конфигурация](#конфигурация)
- [AMXX Команди](#amxx-команди)
- [API](#api)

---

## ✨ Функции

### AMXX Плъгини
| Плъгин | Функция |
|--------|---------|
| `aegisx_core.amxx`   | SQL връзка, REST API, heartbeat, forwards, suspicious check |
| `aegisx_bans.amxx`   | Ban/Unban система, kick при connect |
| `aegisx_comms.amxx`  | Mute / Gag / Silence |
| `aegisx_report.amxx` | `/report` команда, скрийншот, admin нотификации |
| `aegisx_sync.amxx`   | Multi-сървър sync (RCON Push + MySQL Polling fallback) |

### Web панел
- 🌙 Dark / ☀️ Light тема с превключвател
- 🇧🇬 Български / 🇬🇧 English (i18n)
- 📊 Dashboard – статистики, сървъри, последни банове
- 🔨 Банове – списък, търсене, unban
- 🔇 Комс – mute/gag/silence управление
- 🚩 Репорти – преглед и действия
- 🖥️ Сървъри – online статус, играчи, карта
- 👤 Играчи – профил, бан история, IP/ник алиаси
- ⚠️ Suspicious – маркиране за наблюдение (без ban права)
- 👑 Админи – управление, флагове, изтичане

### Интеграции
- 🔔 **Discord webhooks** – нотификации за нов бан, репорт, suspicious
- 🔄 **RCON Push sync** – почти real-time синхронизация между сървъри
- 🔐 **JWT + API Key** – двойна автентикация

---

## 🏗️ Архитектура

```
CS 1.6 Сървъри (Pterodactyl / Linux)
  aegisx_*.amxx
      │ HTTP (API Key)        │ RCON Push
      ▼                       ▼
┌─────────────────────────────────────┐
│           VPS (Docker)              │
│  MySQL ←→ Fastify API ←→ React UI   │
│              ↕ nginx (SSL)          │
└─────────────────────────────────────┘
              ↕ Discord Webhook
```

**Sync механизъм:**
```
Нов бан на Сървър A
  → MySQL INSERT
  → REST API → RCON "aeg_forcesync bans" → Сървър B (< 1 сек)
  → MySQL polling на 30 сек (fallback ако RCON fail)
```

---

## 📦 Изисквания

### Сървър (AMXX)
- AMX Mod X 1.10.x
- MySQL X модул
- easy_http 0.4+ (за REST API)
- ezjson (за JSON парсване)

### VPS (API + панел)
- Docker Engine 20.10+
- Docker Compose v2.0+
- Минимум 1GB RAM
- Домейн с SSL сертификат (Let's Encrypt)

---

## 🚀 Инсталация

### 1. VPS – Docker Compose

```bash
# Клонирай проекта
git clone https://github.com/yourname/aegisx.git
cd aegisx

# Конфигурирай .env
cp .env.example .env
nano .env
# Задай: MYSQL_ROOT_PASSWORD, DB_PASS, JWT_SECRET, API_KEY_SALT

# SSL (Let's Encrypt)
mkdir -p ssl
docker run --rm -p 80:80 \
  -v $(pwd)/ssl:/etc/letsencrypt \
  certbot/certbot certonly --standalone \
  -d yourdomain.com --agree-tos -m you@email.com

# Копирай сертификата
cp ssl/live/yourdomain.com/fullchain.pem ssl/cert.pem
cp ssl/live/yourdomain.com/privkey.pem   ssl/key.pem

# Стартирай
docker compose up -d --build

# Провери
docker compose ps
docker compose logs api --tail=50
```

**Достъп:** `https://yourdomain.com`

---

### 2. Сървър – AMXX плъгини

```bash
# Копирай в addons/amxmodx/
plugins/    ← aegisx_*.amxx
configs/AegisX/   ← core.cfg, bans.cfg, comms.cfg, report.cfg, sync.cfg
lang/       ← aegisx_*.txt
```

**`configs/AegisX/core.cfg`:**
```
aegisx_sql_host    "IP_на_MySQL"
aegisx_sql_user    "aegisx"
aegisx_sql_pass    "паролата"
aegisx_sql_db      "aegisx"
aegisx_api_url     "https://yourdomain.com/api"
aegisx_api_token   "твоя_api_key"
aegisx_srv_name    "My CS 1.6 Server"
```

**`configs/plugins-aegisx.ini`:**
```
aegisx_core.amxx
aegisx_bans.amxx
aegisx_comms.amxx
aegisx_report.amxx
aegisx_sync.amxx
```

**`configs/plugins.ini`** – добави:
```
plugins-aegisx.ini
```

### 3. Генериране на API Key за сървъра

```bash
# Влез в панела → Admins → (не е реализирано) или директно в MySQL:
# Генерирай ключ:
openssl rand -hex 32

# INSERT в MySQL:
INSERT INTO aeg_api_tokens (name, token_hash, type, server_id, permissions, is_active, created_at)
VALUES (
  'Server 1',
  SHA2(CONCAT('ТВОЯ_КЛЮЧ', 'API_KEY_SALT_ОТ_ENV'), 256),
  0, 1, '["*"]', 1, UNIX_TIMESTAMP()
);
```

Задай ключа в `core.cfg`:
```
aegisx_api_token "ТВОЯ_КЛЮЧ"
```

---

## ⚙️ Конфигурация

### core.cfg (пълни CVARs)
| CVAR | По подразбиране | Описание |
|------|-----------------|----------|
| `aegisx_sql_host` | `127.0.0.1` | MySQL хост |
| `aegisx_sql_port` | `3306` | MySQL порт |
| `aegisx_sql_user` | `aegisx` | MySQL потребител |
| `aegisx_sql_pass` | — | MySQL парола |
| `aegisx_sql_db`   | `aegisx` | MySQL база |
| `aegisx_sql_prefix` | `aeg` | Префикс на таблиците |
| `aegisx_api_url`  | — | URL на REST API |
| `aegisx_api_token`| — | API Key (FCVAR_PROTECTED) |
| `aegisx_api_enabled` | `1` | Включи REST API |
| `aegisx_srv_name` | hostname | Показвано име на сървъра |
| `aegisx_log_level` | `1` | 0=debug 1=info 2=warn 3=error |

### sync.cfg
| CVAR | По подразбиране | Описание |
|------|-----------------|----------|
| `aegisx_sync_interval` | `30` | Polling fallback (сек) |
| `aegisx_sync_bans`     | `1`  | Синхронизирай банове |
| `aegisx_sync_comms`    | `1`  | Синхронизирай комс |
| `aegisx_sync_admins`   | `1`  | Синхронизирай админи |
| `aegisx_sync_status`   | `1`  | Heartbeat към БД |

---

## 🎮 AMXX Команди

### Бан команди
| Команда | Достъп | Описание |
|---------|--------|----------|
| `aeg_ban <играч> <минути> <причина>` | ADMIN_BAN | Бани играч |
| `aeg_unban <steamid>` | ADMIN_BAN | Отбани играч |
| `aeg_banmenu` | ADMIN_BAN | Менюто за бан |
| `aeg_banlist` | ADMIN_BAN | Списък с банове |

### Комс команди
| Команда | Достъп | Описание |
|---------|--------|----------|
| `aeg_mute <играч> <минути> [причина]` | ADMIN_BAN | Mute |
| `aeg_gag <играч> <минути> [причина]`  | ADMIN_BAN | Gag |
| `aeg_silence <играч> <минути> [причина]` | ADMIN_BAN | Silence |
| `aeg_unmute <играч>` | ADMIN_BAN | Unmute |
| `aeg_ungag <играч>` | ADMIN_BAN | Ungag |

### Sync команди
| Команда | Достъп | Описание |
|---------|--------|----------|
| `aeg_forcesync <bans\|comms\|admins\|all>` | ADMIN_RCON | Force sync (RCON push) |
| `aeg_sync` | ADMIN_RCON | Manual пълен sync |
| `aeg_syncstatus` | ADMIN_BAN | Статус на sync |

### Чат команди (играчи)
| Команда | Описание |
|---------|----------|
| `/report` или `!report` | Репортвай играч |

---

## 🔌 API

Базов URL: `https://yourdomain.com/api`

### Автентикация
```
# За Web панела (JWT):
Authorization: Bearer <jwt_token>

# За AMXX плъгините (API Key):
X-Api-Key: <api_key>
```

### Основни endpoints
| Метод | Endpoint | Описание |
|-------|----------|----------|
| POST | `/auth/login` | Вход, получи JWT |
| GET  | `/bans` | Списък с банове |
| POST | `/bans` | Нов бан |
| DELETE | `/bans/:id` | Unban |
| GET  | `/comms` | Комс наказания |
| POST | `/comms` | Ново наказание |
| GET  | `/reports` | Репорти |
| POST | `/reports` | Нов репорт |
| PATCH | `/reports/:id` | Смени статус |
| GET  | `/servers` | Сървъри |
| GET  | `/players/:steamid` | Профил на играч |
| GET  | `/suspicious/check/:steamid` | Проверка |
| POST | `/suspicious` | Маркирай играч |

---

## 📜 Лиценз

AegisX е издаден под **GNU General Public License v2.0**.

Базиран на концепцията на AMXBans. Цялостно пренаписан и модернизиран.

---

*AegisX – Модерно управление на банове за Counter Strike 1.6 · 2026*

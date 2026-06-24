# 🏴‍☠️ Luffy Panel

A lightweight VLESS-over-WebSocket proxy panel built with FastAPI, deployable on [Render](https://render.com) or [Railway](https://railway.app).

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)
&nbsp;&nbsp;
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

---

## ✨ Features

- **VLESS over WebSocket (TLS)** tunneling
- Multi-inbound management with per-user traffic quotas
- Connection limits per inbound (max IPs), enforced live on the proxy WebSocket handler
- Expiry date support per inbound
- Subscription link (`/sub/<uid>`) compatible with v2rayNG, Hiddify, etc.
- Clean IP / alternative address management (single or bulk delete)
- Real-time dashboard: CPU, memory, hourly traffic chart
- **Live log streaming** over WebSocket (`/ws/live-logs`)
- **Telegram bot** with full command set, inline menus, and auto alerts (quota/expiry)
- Telegram bot configurable directly from the panel UI (no env vars required)
- Bilingual UI (English / Persian) — panel and bot both
- Dark & Light mode
- Session-based authentication with password change
- **Persistent storage** to a local JSON file (`panel_db.json`)
- Keep-alive mechanism for free-tier hosting

---

## 🗂️ Project Structure

```
.
├── main.py             # FastAPI application (gateway + panel UI + Telegram bot)
├── panel_db.json        # Auto-generated persistent storage (created at runtime)
├── requirements.txt    # Python dependencies
├── render.yaml         # Render deployment config
└── Procfile             # Process entry point
```

---

## 🚀 Deploy on Render

### One-click via `render.yaml`

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [render.com](https://render.com) → **New Web Service** → connect your repo.
3. Render will auto-detect `render.yaml` and configure everything.
4. Set your `ADMIN_PASSWORD` environment variable (default: `admin`).

> 💡 **Tip:** For better speed, set the **Region** to **Frankfurt (EU)** in Render settings.

### Manual Setup

| Field | Value |
|---|---|
| **Environment** | Python |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `python main.py` |

### 🌐 Render & Cloudflare Clean IPs

> **This panel on Render routes through Cloudflare's clean IPs exclusively.**
>
> Render's infrastructure sits behind Cloudflare's network, so all VLESS+WS configs will automatically use **Cloudflare clean IP ranges** — which are generally unblocked and stable in restricted regions.
>
> ✅ Use the panel URL directly — Cloudflare CDN handles routing automatically.
>
> If configs don't connect, try manually entering a known Cloudflare clean IP (e.g. `104.21.x.x` or `172.67.x.x`) in your client instead of the hostname.

> ⚠️ **Storage note:** Render's free tier filesystem is ephemeral across redeploys. `panel_db.json` will survive simple restarts of the same running instance, but a fresh deploy may reset it unless you attach a persistent disk.

---

## 🚂 Deploy on Railway

### One-click deploy

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → select your repo.
3. Wait for the deployment to finish. You'll be given a URL — that's your service domain. To access the panel, just add `/login` to the end of your domain.

### ⚠️ Railway IP Addresses

> **Railway does NOT use Cloudflare. It uses its own dedicated IP ranges.**
>
> Railway's outbound IPs typically fall in the range **`69.46.46.x`**, so your configs will use Railway's own IPs — not Cloudflare's. These may or may not be accessible depending on your network restrictions.
>
> **If configs don't work on Railway:**
> 1. Check whether the `69.46.46.x` range is reachable from your network.
> 2. Enable **Fragment Mode** in your v2ray / v2rayNG client (see section below).
> 3. Switch to Render for Cloudflare clean IP routing.

---

## 🔧 Fragment Mode (v2rayNG / v2ray)

If your configurations are not connecting — especially on Railway — enable **Fragment Mode** in your client:

**v2rayNG (Android):**
1. Go to **Settings → Fragment**
2. Enable Fragment and set: Packets `tlshello`, Length `10-30`, Interval `10-20`
3. Reconnect

**v2ray (Desktop):** Add to your `outbound` → `streamSettings`:

```json
"sockopt": {
  "dialerProxy": "fragment",
  "tcpKeepAliveIdle": 100
}
```

Fragment mode splits the TLS ClientHello packet to bypass deep packet inspection (DPI) firewalls.

---

## ▶️ Run Locally

```bash
pip install -r requirements.txt
python main.py
```

Panel will be available at: `http://localhost:8000/login`

> After deploying on Render or Railway, access your panel at: `https://yourdomain/login`

---

## ⚙️ Environment Variables

| Variable | Description | Default |
|---|---|---|
| `ADMIN_PASSWORD` | Panel login password | `admin` |
| `SECRET_KEY` | Session & hash secret (auto-generated) | random |
| `PORT` | Server port | `8000` |

> ⚠️ **Change `ADMIN_PASSWORD` before deploying to production.**
>
> 💡 Telegram bot token and admin ID are **no longer required as env vars** — configure them directly from the panel's Settings section (see [Telegram Bot](#-telegram-bot) below). They're saved into `panel_db.json` and the bot restarts automatically when updated.

---

## 📦 Dependencies

```
fastapi==0.104.1
uvicorn==0.24.0
websockets==12.0
httpx==0.25.1
psutil==5.9.6
pyTelegramBotAPI
```

> `pyTelegramBotAPI` is optional — if it's not installed, the app runs fine without the Telegram bot feature and logs a warning on startup.

---

## 📌 Static IPs

| Platform | Static IP? | Notes |
|---|---|---|
| **Render** (Free) | ❌ No | Shared Cloudflare IPs; clean and stable |
| **Render** (Paid) | ✅ Yes | Available on Starter plan and above |
| **Railway** | ✅ Optional | Enable via Settings → Networking → Static IP (paid feature) |

---

## 🔌 API Endpoints

### Auth
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/login` | Login with password |
| `POST` | `/api/logout` | Logout |
| `GET` | `/api/me` | Check session status |
| `POST` | `/api/change-password` | Change admin password |

### Panel Settings (Telegram bot config)
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/settings` | Get current Telegram bot token & admin ID |
| `POST` | `/api/settings` | Update Telegram bot token/admin ID — bot restarts automatically |

### Inbounds
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/links` | List all inbounds |
| `POST` | `/api/links` | Create new inbound |
| `PATCH` | `/api/links/{uid}` | Edit inbound (active state, quota, label, max connections, expiry, usage reset) |
| `DELETE` | `/api/links/{uid}` | Delete inbound (also force-closes its active connections) |

### Subscription
| Method | Path | Description |
|---|---|---|
| `GET` | `/sub/{uid}` | Base64 subscription (v2ray/Hiddify compatible) |

### Clean IPs
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/addresses` | List alternative addresses |
| `POST` | `/api/addresses` | Add address |
| `DELETE` | `/api/addresses/{index}` | Remove a single address by index |
| `DELETE` | `/api/addresses` | Remove **all** addresses at once |

### System
| Method | Path | Description |
|---|---|---|
| `GET` | `/stats` | Server stats (auth required) |
| `GET` | `/health` | Health check |
| `WS` | `/ws/live-logs` | Real-time log stream (last 150 lines, then live updates) |

---

## 🌐 VLESS Config Format

```
vless://<uuid>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/ws/<uuid>&sni=<domain>&fp=chrome&alpn=http/1.1#Luffy-<name>
```

---

## 🖥️ Panel Pages

| Page | Description |
|---|---|
| **Dashboard** | Traffic, uptime, CPU/memory, hourly chart, live log viewer |
| **Inbounds** | Create/edit/delete users, copy config, QR code, per-user connection limit |
| **Traffic** | Total stats |
| **Clean IP** | Manage alternative subscription addresses (single or bulk delete) |
| **Settings** | Configure Telegram bot token & admin ID |
| **Security** | Change password |

---

## 🤖 Telegram Bot — Full Integration Details

The Telegram bot is built into `main.py` using `pyTelegramBotAPI` (`AsyncTeleBot`) and is **fully wired to the live panel state** — it reads/writes the exact same `LINKS` dict and `CUSTOM_ADDRESSES` list the web dashboard uses, so any change made from Telegram instantly shows up in the panel and vice versa.

### How it connects to the panel

1. Open the panel → **Settings** page.
2. Enter your **bot token** (from [@BotFather](https://t.me/BotFather)) and your **numeric Telegram admin ID** (e.g. from [@userinfobot](https://t.me/userinfobot)) into the `tg-token` / `tg-admin-id` fields.
3. Click save → the frontend calls `saveSettings()`, which sends `POST /api/settings`.
4. The backend (`update_settings`):
   - Saves the token/admin ID into `CONFIG["telegram_token"]` / `CONFIG["telegram_admin_id"]`.
   - Persists them to `panel_db.json` via `save_db()`.
   - Immediately calls `restart_telegram_bot()` — **no server restart or redeploy required**, the bot goes live within seconds.
5. On every startup, `load_db()` restores the saved token/admin ID and `restart_telegram_bot()` is called automatically so the bot reconnects without re-entering credentials.

### Token validation & safe restart logic

`restart_telegram_bot()` doesn't just blindly start polling — it does several safety steps:

- First calls `_stop_telegram_bot()` to cleanly cancel any previous polling task. This matters because **Telegram only allows one active `getUpdates` long-polling connection per bot token** — without this cleanup, saving settings twice would spawn duplicate pollers that fight each other (`409 Conflict`), and commands could silently stop being delivered.
- Calls Telegram's `deleteWebhook?drop_pending_updates=true` to clear any stale webhook/update queue.
- Calls `getMe` to **verify the token is actually valid** before starting the bot. If Telegram rejects it, the bot is not started and an error is logged — it won't crash the panel.
- Only after the token is confirmed does it instantiate `AsyncTeleBot(token)` and start the polling loop as a background `asyncio.Task` (`bot_polling_task`).
- If `pyTelegramBotAPI` isn't installed, this whole flow is skipped gracefully and just logs a warning — the rest of the panel keeps working.

### Admin-only access control

Every single command handler calls `_is_admin_chat(message.chat.id, admin_id)` first. If the chat ID sending the command doesn't match the configured admin ID exactly, the message is **silently ignored** (and logged as a warning server-side) — the bot will not respond to anyone except the configured admin, even if they have the bot token.

### Bilingual interface

All bot text comes from the `BOT_I18N` dictionary (`en` / `fa`), the same i18n pattern used by the web panel. Language is stored in `CONFIG["bot_lang"]` and persisted to `panel_db.json`. A 🇮🇷/🇬🇧 inline button on the main menu calls the `tg_lang_toggle` callback to flip it instantly, without restarting the bot.

### Commands

| Command | Description |
|---|---|
| `/start` | Welcome message + inline main menu (`build_main_keyboard()`): Stats, Users, Top Users, Create User guide, Add Clean IP guide, Language toggle |
| `/stats` | Live server dashboard: domain, CPU %, memory %, uptime, active connections, total traffic (MB), inbound count — pulled from the same data as `/stats` HTTP endpoint |
| `/users` | Full list of all inbounds with usage, limit, expiry date, and 🟢/🔴 on-off status |
| `/top` | Top 5 inbounds ranked by `used_bytes`, descending |
| `/create [name] [limit_GB] [days]` | Creates a new inbound directly in `LINKS` (same as the web "Create" form). Validates name (English letters/numbers only), numeric limit, integer days. `0`/`0` = unlimited quota, no expiry. Example: `/create Ali 15 30`. Replies with the full VLESS link + subscription URL. |
| `/addaddr [ip_or_domain]` | Appends to `CUSTOM_ADDRESSES` — same list the panel's Clean IP page manages. Rejects duplicates and invalid characters. |
| `/disable [username]` | Sets `LINKS[name]["active"] = False` — instantly drops the inbound's ability to accept new WebSocket connections |
| `/enable [username]` | Sets `LINKS[name]["active"] = True` |
| `/reset [username]` | Sets `used_bytes` back to `0` for that inbound |

Inline buttons on `/start`/menu also expose **guided help text** for `/create` and `/addaddr` (`create_guide` / `addip_guide`) so the admin doesn't need to remember the exact syntax.

Every command that changes state (`/create`, `/disable`, `/enable`, `/reset`, `/addaddr`) calls `save_db()` immediately afterward, so changes made via Telegram are persisted to `panel_db.json` exactly like changes made via the web UI.

### Automatic background alerts

A separate background task, `telegram_notifier_cron()`, runs continuously (started at app startup via `asyncio.create_task`) and checks **every 60 seconds**:

- **⚠️ Quota alert** — for every *active* inbound, if `used_bytes >= limit_bytes` (and `limit_bytes > 0`), it sends `quota_alert` to the admin chat. 
- **⏰ Expiry alert** — if an inbound's `expires_at` timestamp is in the past, it sends `expiry_alert` to the admin chat.

To avoid spamming the admin every minute, each alert is only sent **once per inbound per condition**, tracked via an in-memory `notified_uids` set (keys like `quota_<uid>` / `expiry_<uid>`). If the cron finds no Telegram token/admin ID configured, it simply sleeps and rechecks every 60 seconds without doing anything.

### Quota & expiry enforcement is also live on the proxy itself

This isn't just a notification feature — `check_quota(uid, extra_bytes)` is called on **every single packet** flowing through the `/ws/{uuid}` proxy tunnel (both `ws_to_tcp` and `tcp_to_ws` directions). If a user is disabled, expired, or over quota, the WebSocket connection is immediately closed with code `1008`. So disabling a user from Telegram, or a quota alert firing, reflects an actual connection cutoff — not just a dashboard number.

### Per-inbound connection limit

`max_connections` (settable from the web panel's edit form, field `ec`) is enforced in the `/ws/{uuid}` handler **before** the WebSocket handshake completes: if `count_connections_for_link(uuid) >= max_connections`, the new connection is rejected pre-handshake (so it looks like a normal closed/forbidden endpoint to DPI probes, not a live VLESS server — see the in-code comment on this design choice). Deleting an inbound (via panel or could be extended to Telegram) calls `close_connections_for_link(uid)` to force-drop all of that inbound's currently active sessions right away.

---

## 📱 Client Setup (v2rayNG / Hiddify)

1. Open the panel and go to **Inbounds**.
2. Click **Sub** to copy the subscription URL.
3. In your client app, add a new subscription with that URL.
4. Update subscription — configs will appear automatically.

---

## ⚠️ Notes

- Inbounds, custom addresses, and panel/bot settings are persisted to a local file (`panel_db.json`) and reloaded automatically on startup.
- On ephemeral hosting (e.g. Render free tier across redeploys), this file may not survive a full redeploy — for guaranteed persistence across redeploys, attach a persistent disk or switch to an external database (e.g. SQLite on a mounted volume).
- The keep-alive task pings `/health` every 10 minutes to prevent Render free-tier spin-down.
- Per-inbound `max_connections` is enforced live: once the limit is reached, new connection attempts for that UID are rejected until an existing session disconnects.

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to your branch: `git push origin feature/amazing-feature`
5. Open a **Pull Request**

---

## 📄 License

MIT — use freely, modify as needed.

---

[My Telegram channel](https://t.me/Luffy_sh_op)

---
---
---

# 🏴‍☠️ لوفی پنل

یک پنل پراکسی سبک VLESS-over-WebSocket ساخته‌شده با FastAPI، قابل استقرار روی [Render](https://render.com) یا [Railway](https://railway.app).

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)
&nbsp;&nbsp;
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

---

## ✨ امکانات

- تانلینگ **VLESS روی WebSocket (TLS)**
- مدیریت چند اینباند با محدودیت ترافیک برای هر کاربر
- محدودیت تعداد اتصال (IP) برای هر اینباند، به‌صورت زنده روی هندلر پراکسی اعمال می‌شود
- پشتیبانی از تاریخ انقضا برای هر اینباند
- لینک اشتراک (`/sub/<uid>`) سازگار با v2rayNG، Hiddify و غیره
- مدیریت آی‌پی تمیز / آدرس‌های جایگزین (حذف تکی یا دسته‌جمعی)
- داشبورد لحظه‌ای: CPU، حافظه، نمودار ترافیک ساعتی
- **استریم لاگ زنده** روی WebSocket (`/ws/live-logs`)
- **ربات تلگرام** با مجموعه کامل دستورات، منوهای inline و هشدار خودکار (اتمام حجم/انقضا)
- قابلیت تنظیم ربات تلگرام مستقیم از داخل پنل (بدون نیاز به env var)
- رابط کاربری دو زبانه (فارسی / انگلیسی) — هم پنل، هم ربات
- حالت تاریک و روشن
- احراز هویت مبتنی بر session با امکان تغییر رمز
- **ذخیره‌سازی دائمی** در یک فایل JSON محلی (`panel_db.json`)
- مکانیزم keep-alive برای هاستینگ رایگان

---

## 🗂️ ساختار پروژه

```
.
├── main.py              # اپلیکیشن FastAPI (گیت‌وی + رابط پنل + ربات تلگرام)
├── panel_db.json         # ذخیره‌سازی دائمی (خودکار ساخته می‌شود)
├── requirements.txt     # وابستگی‌های پایتون
├── render.yaml          # تنظیمات استقرار Render
└── Procfile              # نقطه ورود پروسه
```

---

## 🚀 استقرار روی Render

### یک‌کلیکی با `render.yaml`

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. ریپو را fork کنید یا روی GitHub آپلود کنید.
2. به [render.com](https://render.com) بروید ← **New Web Service** ← ریپو را متصل کنید.
3. Render به‌صورت خودکار `render.yaml` را شناسایی و همه چیز را تنظیم می‌کند.
4. متغیر `ADMIN_PASSWORD` را تنظیم کنید (پیش‌فرض: `admin`).

> 💡 **نکته:** برای سرعت بهتر، **Region** را روی **Frankfurt (EU)** تنظیم کنید.

### تنظیم دستی

| فیلد | مقدار |
|---|---|
| **محیط** | Python |
| **دستور Build** | `pip install -r requirements.txt` |
| **دستور Start** | `python main.py` |

### 🌐 Render و آی‌پی‌های تمیز Cloudflare

> **⭐ این پنل روی Render فقط از آی‌پی‌های تمیز Cloudflare استفاده می‌کند.**
>
> زیرساخت Render پشت شبکه Cloudflare قرار دارد، بنابراین تمام کانفیگ‌های VLESS+WS به‌صورت خودکار از **آی‌پی‌های تمیز Cloudflare** عبور می‌کنند — که معمولاً آنبلاک و پایدار هستند.
>
> ✅ URL پنل را مستقیم استفاده کنید — Cloudflare CDN مسیریابی را خودکار انجام می‌دهد.
>
> اگر کانفیگ‌ها وصل نشدند، یک آی‌پی تمیز شناخته‌شده Cloudflare (مثل `104.21.x.x` یا `172.67.x.x`) را در کلاینت خود به جای hostname وارد کنید.

> ⚠️ **نکته ذخیره‌سازی:** فایل‌سیستم پلن رایگان Render بین دیپلوی‌های جدید پایدار نیست. `panel_db.json` بین ریستارت‌های ساده همون اینستنس می‌مونه، ولی با یک دیپلوی جدید ممکنه پاک بشه؛ برای پایداری کامل بین دیپلوی‌ها، یک دیسک دائمی وصل کنید.

---

## 🚂 استقرار روی Railway

### استقرار یک‌کلیکی

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. ریپو را fork کنید یا روی GitHub آپلود کنید.
2. به [railway.app](https://railway.app) بروید ← **New Project** ← **Deploy from GitHub repo** ← ریپو را انتخاب کنید.
3. صبر کنید تا deploy شود. بعد از deploy یک URL به شما داده می‌شود که آن دامنه سرویس شماست. برای ورود به پنل کافیست به آخر دامنه‌تان `/login` اضافه کنید.

### ⚠️ آی‌پی‌های Railway

> **⭐ Railway از Cloudflare استفاده نمی‌کند و از آی‌پی‌های اختصاصی خودش استفاده می‌کند.**
>
> آی‌پی‌های خروجی Railway معمولاً در رنج **`69.46.46.x`** هستند، بنابراین کانفیگ‌های شما از آی‌پی‌های خود Railway عبور می‌کنند — نه از Cloudflare. این آی‌پی‌ها ممکن است بسته به محدودیت‌های شبکه شما در دسترس باشند یا نباشند.
>
> **اگر کانفیگ‌ها روی Railway کار نکرد:**
> 1. بررسی کنید که رنج `69.46.46.x` از شبکه شما در دسترس است.
> 2. **حالت Fragment را در کلاینت v2ray / v2rayNG فعال کنید** (بخش زیر را ببینید).
> 3. برای استفاده از آی‌پی‌های تمیز Cloudflare، به Render بروید.

---

## 🔧 فعال‌کردن Fragment Mode (در v2rayNG / v2ray)

اگر کانفیگ‌ها وصل نمی‌شوند — به‌خصوص روی Railway — **حالت Fragment را فعال کنید:**

**v2rayNG (اندروید):**
1. به **Settings → Fragment** بروید
2. Fragment را فعال کنید و تنظیم کنید: Packets روی `tlshello`، Length روی `10-30`، Interval روی `10-20`
3. مجدداً وصل شوید

**v2ray (دسکتاپ):** به `outbound` → `streamSettings` اضافه کنید:

```json
"sockopt": {
  "dialerProxy": "fragment",
  "tcpKeepAliveIdle": 100
}
```

حالت Fragment بسته TLS ClientHello را تقسیم می‌کند تا از فایروال‌های DPI عبور کند.

---

## ▶️ اجرای محلی

```bash
pip install -r requirements.txt
python main.py
```

پنل در این آدرس در دسترس است: `http://localhost:8000/login`

> بعد از استقرار روی Render یا Railway، از این آدرس وارد پنل شوید: `https://yourdomain/login`

---

## ⚙️ متغیرهای محیطی

| متغیر | توضیح | پیش‌فرض |
|---|---|---|
| `ADMIN_PASSWORD` | رمز ورود به پنل | `admin` |
| `SECRET_KEY` | مخفی session و هش (خودکار تولید می‌شود) | تصادفی |
| `PORT` | پورت سرور | `8000` |

> ⚠️ **بعد از استقرار در محیط عمومی، `ADMIN_PASSWORD` را تغییر دهید.**
>
> 💡 توکن و آیدی ادمین ربات تلگرام دیگه **نیازی به env var نیست** — مستقیم از بخش تنظیمات پنل (به بخش [ربات تلگرام](#-ربات-تلگرام) مراجعه کنید) قابل تنظیمه. این مقادیر در `panel_db.json` ذخیره می‌شن و با هر تغییر، ربات خودکار ری‌استارت می‌شه.

---

## 📦 وابستگی‌ها

```
fastapi==0.104.1
uvicorn==0.24.0
websockets==12.0
httpx==0.25.1
psutil==5.9.6
pyTelegramBotAPI
```

> نصب `pyTelegramBotAPI` اختیاریه — اگر نصب نباشه، برنامه بدون قابلیت ربات تلگرام به‌درستی اجرا می‌شه و فقط یک هشدار موقع استارت نمایش می‌ده.

---

## 📌 آی‌پی استاتیک

| پلتفرم | آی‌پی استاتیک؟ | توضیحات |
|---|---|---|
| **Render** (رایگان) | ❌ خیر | آی‌پی‌های مشترک Cloudflare؛ تمیز و پایدار |
| **Render** (پولی) | ✅ بله | از پلان Starter به بالا در دسترس |
| **Railway** | ✅ اختیاری | از طریق Settings → Networking → Static IP فعال شود (ویژگی پولی) |

---

## 🔌 مسیرهای API

### احراز هویت
| متد | مسیر | توضیح |
|---|---|---|
| `POST` | `/api/login` | ورود با رمز |
| `POST` | `/api/logout` | خروج |
| `GET` | `/api/me` | بررسی وضعیت session |
| `POST` | `/api/change-password` | تغییر رمز ادمین |

### تنظیمات پنل (کانفیگ ربات تلگرام)
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/settings` | گرفتن توکن و آیدی ادمین تلگرام فعلی |
| `POST` | `/api/settings` | به‌روزرسانی توکن/آیدی ادمین — ربات خودکار ری‌استارت می‌شود |

### اینباندها
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/links` | لیست همه اینباندها |
| `POST` | `/api/links` | ایجاد اینباند جدید |
| `PATCH` | `/api/links/{uid}` | ویرایش اینباند (وضعیت فعال، حجم، نام، محدودیت اتصال، انقضا، صفر کردن مصرف) |
| `DELETE` | `/api/links/{uid}` | حذف اینباند (اتصالات فعال آن نیز فوراً قطع می‌شود) |

### اشتراک
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/sub/{uid}` | اشتراک Base64 (سازگار با v2ray/Hiddify) |

### آی‌پی تمیز
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/addresses` | لیست آدرس‌های جایگزین |
| `POST` | `/api/addresses` | افزودن آدرس |
| `DELETE` | `/api/addresses/{index}` | حذف یک آدرس بر اساس اندیس |
| `DELETE` | `/api/addresses` | حذف **همه** آدرس‌ها با یک درخواست |

### سیستم
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/stats` | آمار سرور (نیاز به احراز هویت) |
| `GET` | `/health` | بررسی سلامت سرور |
| `WS` | `/ws/live-logs` | استریم لاگ زنده (۱۵۰ خط آخر، سپس به‌روزرسانی لحظه‌ای) |

---

## 🌐 فرمت کانفیگ VLESS

```
vless://<uuid>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/ws/<uuid>&sni=<domain>&fp=chrome&alpn=http/1.1#Luffy-<name>
```

---

## 🖥️ صفحات پنل

| صفحه | توضیح |
|---|---|
| **داشبورد** | ترافیک، آپتایم، CPU/حافظه، نمودار ساعتی، نمایش لاگ زنده |
| **اینباندها** | ایجاد/ویرایش/حذف کاربر، کپی کانفیگ، کد QR، محدودیت اتصال هر کاربر |
| **ترافیک** | آمار کلی |
| **آی‌پی تمیز** | مدیریت آدرس‌های جایگزین اشتراک (حذف تکی یا دسته‌جمعی) |
| **تنظیمات** | کانفیگ توکن و آیدی ادمین ربات تلگرام |
| **امنیت** | تغییر رمز |

---

## 🤖 ربات تلگرام — جزئیات کامل یکپارچه‌سازی

ربات تلگرام مستقیم داخل `main.py` با کتابخانه `pyTelegramBotAPI` (کلاس `AsyncTeleBot`) ساخته شده و **کاملاً به وضعیت زنده پنل وصله** — یعنی دقیقاً همون دیکشنری `LINKS` و لیست `CUSTOM_ADDRESSES` که داشبورد وب استفاده می‌کنه رو می‌خونه و می‌نویسه؛ پس هر تغییری که از تلگرام انجام بدی، فوراً توی پنل هم دیده می‌شه و برعکس.

### نحوه اتصال ربات به پنل

1. پنل رو باز کن ← صفحه **تنظیمات**.
2. **توکن ربات** (از [@BotFather](https://t.me/BotFather)) و **آیدی عددی تلگرام** خودت به‌عنوان ادمین (مثلاً از [@userinfobot](https://t.me/userinfobot)) رو در فیلدهای `tg-token` / `tg-admin-id` وارد کن.
3. روی ذخیره بزن → فرانت‌اند تابع `saveSettings()` رو صدا می‌زنه که درخواست `POST /api/settings` می‌فرسته.
4. بک‌اند (تابع `update_settings`):
   - توکن/آیدی ادمین رو در `CONFIG["telegram_token"]` / `CONFIG["telegram_admin_id"]` ذخیره می‌کنه.
   - با `save_db()` در فایل `panel_db.json` پایدارش می‌کنه.
   - فوراً `restart_telegram_bot()` رو صدا می‌زنه — **نیازی به ریستارت سرور یا دیپلوی مجدد نیست**، ربات ظرف چند ثانیه فعال می‌شه.
5. در هر استارت برنامه، `load_db()` توکن/آیدی ذخیره‌شده رو بازیابی می‌کنه و `restart_telegram_bot()` به‌صورت خودکار صدا زده می‌شه تا ربات بدون نیاز به وارد کردن مجدد اطلاعات وصل بشه.

### اعتبارسنجی توکن و منطق ری‌استارت ایمن

تابع `restart_telegram_bot()` فقط کورکورانه polling رو شروع نمی‌کنه، بلکه چند مرحله ایمنی داره:

- اول `_stop_telegram_bot()` رو صدا می‌زنه تا هر تسک polling قبلی به‌طور تمیز کنسل بشه. این موضوع مهمه چون **تلگرام فقط اجازه یک اتصال long-polling فعال (`getUpdates`) برای هر توکن ربات** رو می‌ده — بدون این مرحله، ذخیره دوباره تنظیمات باعث می‌شد چند poller هم‌زمان با هم تداخل کنن (خطای `409 Conflict`) و دستورات مثل `/start` به‌طور خاموش از کار بیفتن.
- با فراخوانی `deleteWebhook?drop_pending_updates=true` هر webhook یا صف آپدیت قدیمی رو پاک می‌کنه.
- با فراخوانی `getMe` **اعتبار واقعی توکن** رو قبل از استارت ربات بررسی می‌کنه. اگه تلگرام توکن رو رد کنه، ربات استارت نمی‌خوره و فقط یک خطا لاگ می‌شه — پنل کرش نمی‌کنه.
- فقط بعد از تایید توکن، شیء `AsyncTeleBot(token)` ساخته می‌شه و حلقه polling به‌عنوان یک تسک پس‌زمینه `asyncio.Task` (`bot_polling_task`) استارت می‌خوره.
- اگه `pyTelegramBotAPI` نصب نباشه، کل این روند به‌آرامی رد می‌شه و فقط یک هشدار لاگ می‌شه — بقیه پنل به‌درستی کار می‌کنه.

### کنترل دسترسی فقط برای ادمین

هر هندلر دستوری اول تابع `_is_admin_chat(message.chat.id, admin_id)` رو صدا می‌زنه. اگه آیدی چت فرستنده دستور دقیقاً با آیدی ادمین تنظیم‌شده مطابقت نداشته باشه، پیام **به‌طور کامل و خاموش نادیده گرفته می‌شه** (و سمت سرور به‌عنوان هشدار لاگ می‌شه) — ربات به هیچ‌کس جز ادمین تنظیم‌شده پاسخ نمی‌ده، حتی اگه توکن ربات رو هم داشته باشه.

### رابط دو زبانه

همه متن‌های ربات از دیکشنری `BOT_I18N` (`en` / `fa`) میان، همون الگوی i18n که پنل وب استفاده می‌کنه. زبان در `CONFIG["bot_lang"]` ذخیره و در `panel_db.json` پایدار می‌شه. یک دکمه inline 🇮🇷/🇬🇧 روی منوی اصلی، callback به نام `tg_lang_toggle` رو صدا می‌زنه تا زبان فوراً بدون ری‌استارت ربات عوض بشه.

### دستورات

| دستور | توضیح |
|---|---|
| `/start` | پیام خوش‌آمد + منوی اصلی inline (`build_main_keyboard()`): آمار، کاربران، پرمصرف‌ترین‌ها، راهنمای ساخت کاربر، راهنمای افزودن آی‌پی تمیز، تغییر زبان |
| `/stats` | داشبورد زنده سرور: دامنه، درصد CPU، درصد رم، آپ‌تایم، اتصالات فعال، ترافیک کل (MB)، تعداد اینباند — از همون داده‌ای که endpoint وب `/stats` استفاده می‌کنه |
| `/users` | لیست کامل همه اینباندها با مصرف، سقف، تاریخ انقضا و وضعیت 🟢/🔴 |
| `/top` | ۵ اینباند برتر بر اساس `used_bytes`، نزولی |
| `/create [نام] [حجم_GB] [روز]` | ساخت مستقیم اینباند جدید در `LINKS` (دقیقاً مثل فرم «ساخت» در پنل وب). نام (فقط حروف/عدد انگلیسی)، حجم عددی، روز عدد صحیح رو اعتبارسنجی می‌کنه. `0`/`0` یعنی حجم نامحدود و بدون انقضا. مثال: `/create Ali 15 30`. در پاسخ، لینک کامل VLESS + آدرس اشتراک رو می‌فرسته. |
| `/addaddr [آی‌پی_یا_دامنه]` | به `CUSTOM_ADDRESSES` اضافه می‌کنه — دقیقاً همون لیستی که صفحه آی‌پی تمیز پنل مدیریت می‌کنه. آدرس تکراری و کاراکتر نامعتبر رد می‌شه. |
| `/disable [نام‌کاربری]` | `LINKS[name]["active"]` رو `False` می‌کنه — فوراً امکان پذیرفتن اتصال WebSocket جدید برای اون اینباند قطع می‌شه |
| `/enable [نام‌کاربری]` | `LINKS[name]["active"]` رو `True` می‌کنه |
| `/reset [نام‌کاربری]` | `used_bytes` اون اینباند رو به `0` برمی‌گردونه |

دکمه‌های inline روی منوی `/start` همچنین **متن راهنمای گام‌به‌گام** برای `/create` و `/addaddr` (`create_guide` / `addip_guide`) رو نمایش می‌دن تا ادمین نیازی به حفظ کردن سینتکس دقیق نداشته باشه.

هر دستوری که وضعیت رو تغییر می‌ده (`/create`، `/disable`، `/enable`، `/reset`، `/addaddr`) بلافاصله بعدش `save_db()` رو صدا می‌زنه، پس تغییرات انجام‌شده از تلگرام دقیقاً مثل تغییرات پنل وب در `panel_db.json` ذخیره می‌شن.

### هشدارهای خودکار پس‌زمینه

یک تسک پس‌زمینه جدا به نام `telegram_notifier_cron()` به‌صورت مداوم اجرا می‌شه (موقع استارت برنامه با `asyncio.create_task` راه‌اندازی می‌شه) و **هر ۶۰ ثانیه** این موارد رو چک می‌کنه:

- **⚠️ هشدار اتمام حجم** — برای هر اینباند *فعال*، اگه `used_bytes >= limit_bytes` باشه (و `limit_bytes > 0`)، پیام `quota_alert` به چت ادمین فرستاده می‌شه.
- **⏰ هشدار انقضا** — اگه زمان `expires_at` یک اینباند گذشته باشه، پیام `expiry_alert` به چت ادمین فرستاده می‌شه.

برای اینکه ادمین هر دقیقه اسپم نشه، هر هشدار فقط **یک‌بار به‌ازای هر اینباند و هر شرط** فرستاده می‌شه، که با یک `set` در حافظه به نام `notified_uids` ردیابی می‌شه (کلیدهایی مثل `quota_<uid>` / `expiry_<uid>`). اگه توکن یا آیدی ادمین تنظیم نشده باشه، این تسک فقط می‌خوابه و هر ۶۰ ثانیه دوباره چک می‌کنه بدون اینکه کاری انجام بده.

### اعمال محدودیت حجم و انقضا روی خود پراکسی هم زنده است

این فقط یک قابلیت اعلان نیست — تابع `check_quota(uid, extra_bytes)` روی **هر پکت** که از تونل پراکسی `/ws/{uuid}` رد می‌شه صدا زده می‌شه (هم در مسیر `ws_to_tcp` هم `tcp_to_ws`). اگه کاربری غیرفعال، منقضی یا بیش از سقف مصرفش شده باشه، اتصال WebSocket فوراً با کد `1008` بسته می‌شه. پس غیرفعال کردن کاربر از تلگرام، یا فعال شدن هشدار اتمام حجم، یک قطع اتصال واقعی رو نشون می‌ده — نه فقط یک عدد روی داشبورد.

### محدودیت تعداد اتصال هر اینباند

`max_connections` (که از فرم ویرایش پنل وب، فیلد `ec`، قابل تنظیمه) داخل هندلر `/ws/{uuid}` **قبل از تکمیل handshake وب‌سوکت** اعمال می‌شه: اگه `count_connections_for_link(uuid) >= max_connections` باشه، اتصال جدید قبل از handshake رد می‌شه (طوری که از دید پروب‌های DPI شبیه یک endpoint بسته/ممنوع عادی به نظر برسه، نه یک سرور VLESS زنده — به کامنت داخل کد در مورد این تصمیم طراحی مراجعه کن). حذف یک اینباند (از پنل، و قابل توسعه برای تلگرام هم) تابع `close_connections_for_link(uid)` رو صدا می‌زنه تا همه سشن‌های فعال اون اینباند فوراً قطع بشن.

---

## 📱 راه‌اندازی کلاینت (v2rayNG / Hiddify)

1. پنل را باز کنید و به **اینباندها** بروید.
2. روی **Sub** کلیک کنید تا لینک اشتراک کپی شود.
3. در اپ کلاینت، یک اشتراک جدید با آن لینک اضافه کنید.
4. اشتراک را آپدیت کنید — کانفیگ‌ها به‌صورت خودکار نمایش داده می‌شوند.

---

## ⚠️ نکات مهم

- اینباندها، آدرس‌های تمیز و تنظیمات پنل/ربات در یک فایل محلی (`panel_db.json`) ذخیره می‌شوند و موقع استارت به‌صورت خودکار بارگذاری می‌شوند.
- روی هاستینگ‌های موقتی (مثلاً پلن رایگان Render بین دیپلوی‌های جدید)، ممکن است این فایل بین دیپلوی‌ها از بین برود — برای پایداری تضمینی بین دیپلوی‌ها، یک دیسک دائمی وصل کنید یا از یک دیتابیس خارجی (مثلاً SQLite روی یک volume) استفاده کنید.
- تسک keep-alive هر ۱۰ دقیقه به `/health` پینگ می‌زند تا از خواب رفتن سرویس رایگان Render جلوگیری کند.
- محدودیت `max_connections` هر اینباند به‌صورت زنده اعمال می‌شود: وقتی به سقف برسد، تلاش‌های اتصال جدید برای آن UID رد می‌شوند تا یکی از سشن‌های فعال قطع شود.

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to your branch: `git push origin feature/amazing-feature`
5. Open a **Pull Request**

---

## 📄 لایسنس

MIT — آزادانه استفاده و ویرایش کنید.

---

[چنل تلگراممون](https://t.me/Luffy_sh_op)

# yazbunu

**Featherweight structured logging with a killer web viewer.**

*Zero dependencies. Zero build tools. Zero RAM overhead. Maximum power.*

[English](#english) | [Turkce](#turkce)

---

<a id="english"></a>

## What is yazbunu?

yazbunu is a structured JSONL logging library for Python with a built-in web viewer that punches way above its weight. The server is a thin file reader (~0 extra RAM). All the intelligence — virtual scrolling, full-text search, pattern detection, analytics — runs in your browser via Web Workers.

```python
from yazbunu import get_logger, init_logging

init_logging(log_dir="./logs", project="myapp")
logger = get_logger("api.auth")

logger.info("user logged in", user="alice", duration_ms=42)
logger.error("token expired", user="bob", task="refresh-session")
```

Then open `http://localhost:9880` and see everything.

## Why yazbunu?

| | yazbunu | Loki + Grafana | Logfire | `tail -f \| jq` |
|---|---|---|---|---|
| **Setup** | `pip install yazbunu` | Docker, configs, dashboards | Account, SDK, cloud | Already there |
| **Dependencies** | 0 (core) / 1 (server) | 50+ containers | Cloud service | None |
| **Server RAM** | ~15MB baseline | 500MB+ | N/A | N/A |
| **Query language** | `level:error src:api task:42` | LogQL | SQL | jq filters |
| **Keyboard-driven** | vim bindings, `:` commands | Mouse-heavy | Mouse-heavy | Terminal only |
| **Offline** | PWA, works offline | No | No | Yes |
| **Themes** | 11 built-in + custom | Dashboard themes | Fixed | Terminal theme |
| **Price** | Free | Free / $$$  | Freemium | Free |

## Features

### Core Logging
- **Zero-dependency** Python structured logger
- Context binding: `logger.bind(task="42")` — all subsequent calls carry the context
- JSONL output with rotating file handler (50MB x 5 by default)
- Automatic function/line info on WARNING+

### Web Viewer
- **Virtual scrolling** — 100 recycled DOM nodes, handles 100k+ lines smoothly
- **Web Worker** — all parsing, indexing, filtering off the main thread
- **Query language** — `level:error AND src:api`, `duration_ms:>1000`, `task:42 OR task:43`
- **11 themes** — dark, light, monokai, solarized, nord, dracula, gruvbox, catppuccin, tokyo-night, high-contrast + custom
- **Vim keybindings** — `j/k`, `g/G`, `10j`, `/` search, `:` command palette, `?` help
- **Sparkline bar** — clickable activity/error timeline (canvas, zero DOM)
- **Scrollbar minimap** — error/warning density markers
- **Pattern detection** — auto-groups similar messages ("task dispatched" x847)
- **Correlation** — click a pill to filter, stackable breadcrumbs, related logs within time window
- **Detail panel** — syntax-highlighted JSON, copy fields, related logs
- **Bookmarks** — annotate lines, export as markdown
- **Export** — JSONL, CSV, markdown table, plain text, clipboard, share links
- **Browser notifications** — ERROR/CRITICAL alerts when tab is backgrounded
- **WebSocket tail** — real-time streaming with HTTP fallback
- **Mobile** — responsive layout, touch gestures (swipe to bookmark/copy)
- **PWA** — installable, offline app shell
- **Accessibility** — ARIA labels, screen reader support, reduced motion

### Server
- **aiohttp** — single optional dependency
- **Streaming reads** — ring buffer, never loads full files into memory
- **Token auth** — auto-generated, stored in `~/.yazbunu/auth.json`
- **`/health`** — liveness + auth URL for process manager integration
- **`/api/stats`** — streaming stats without JSON parsing
- **`/api/context`** — field-value grep without JSON parsing
- **QR page** — scan to connect from mobile

## Install

```bash
pip install yazbunu
```

For the web viewer:

```bash
pip install yazbunu[server]
```

## Quick Start

```python
# app.py
from yazbunu import get_logger, init_logging

init_logging(log_dir="./logs", project="demo")
log = get_logger("main")

log.info("started")
log.info("processing", task="42", items=150)
log.warning("slow query", duration_ms=3200, table="users")
log.error("connection failed", host="db.local", retries=3)
```

```bash
# Start the viewer
yazbunu-server --log-dir ./logs
# Open http://localhost:9880
```

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `j` / `k` | Move down / up |
| `5j` | Jump 5 lines |
| `g` / `G` | First / last line |
| `Ctrl-d` / `Ctrl-u` | Page down / up |
| `/` | Focus search |
| `n` / `N` | Next / prev match |
| `Enter` | Expand line details |
| `e` | Cycle level filter |
| `f` | Toggle live tail |
| `b` | Bookmark line |
| `B` | Show bookmarks |
| `:` | Command palette |
| `?` | Show all shortcuts |
| `Esc` | Close / reset |

## Query Language

```
level:error                           # by level
src:orchestrator                      # by source
task:42                               # by any context field
duration_ms:>1000                     # numeric comparison
level:error OR level:warning          # boolean
src:api AND NOT msg:healthcheck       # exclusion
(task:42 OR task:43) AND agent:coder  # grouping
after:"5m ago"                        # time filters
```

## Integration with Process Managers

yazbunu's `/health` endpoint returns both liveness status and auth info:

```bash
curl http://localhost:9880/health
# {"status":"ok","version":"0.2.0","uptime_sec":3600,"token":"abc...","url":"http://...?token=abc..."}
```

Your process manager can:
1. Health-check yazbunu via `GET /health`
2. Extract the `url` field
3. Send it to users (Telegram, Slack, email — whatever)
4. Users click the link, instantly authenticated

## Architecture

```
  Python App          yazbunu server          Browser
  ┌─────────┐        ┌─────────────┐        ┌──────────────────┐
  │ get_log  │───────>│ .jsonl file │<───────│  Web Worker      │
  │ ger()    │ write  │             │  read  │  - JSON parse    │
  │          │        │ aiohttp     │───────>│  - Inverted index│
  └─────────┘        │ ~15MB RAM   │  WS/   │  - Query engine  │
                      │ 0 cache     │  HTTP  │  - Aggregation   │
                      └─────────────┘        │  - Patterns      │
                                             ├──────────────────┤
                                             │  Main Thread     │
                                             │  - Virtual scroll│
                                             │  - Canvas charts │
                                             │  - Vim keybinds  │
                                             └──────────────────┘
```

**The server is dumb on purpose.** It reads files and streams bytes. That's it. All intelligence lives in the browser — Web Workers for CPU-heavy tasks, Canvas for zero-DOM charts, virtual scrolling for unlimited log lines. This is why yazbunu can show 100k lines with 11 themes, pattern detection, and a query engine while the server uses ~15MB of RAM.

## License

MIT

---

<a id="turkce"></a>

## yazbunu nedir?

yazbunu, Python icin yapilandirilmis JSONL loglama kutuphanesi ve guclu bir web goruntuleyicisidir. Sunucu tarafinda neredeyse sifir RAM kullanimi, tum zeka tarayicida calisir.

```python
from yazbunu import get_logger, init_logging

init_logging(log_dir="./logs", project="uygulamam")
logger = get_logger("api.yetki")

logger.info("kullanici girisi", kullanici="ayse", sure_ms=42)
logger.error("token suresi doldu", kullanici="mehmet", gorev="yenileme")
```

Sonra `http://localhost:9880` adresini acin.

## Neden yazbunu?

- **Sifir bagimlilik** — cekirdek kutuphane hicbir sey gerektirmez
- **Tuy gibi hafif** — sunucu ~15MB RAM, dosyalari akis olarak okur, bellekte hicbir sey tutmaz
- **Guclu sorgu dili** — `level:error AND src:api`, `sure_ms:>1000`, `gorev:42 OR gorev:43`
- **Vim tuslari** — `j/k/g/G`, `:` komut paleti, `?` yardim
- **11 tema** — karanlik, aydinlik, monokai, solarized, nord, dracula ve daha fazlasi
- **Gercek zamanli** — WebSocket ile canli log takibi
- **Mobil uyumlu** — dokunmatik hareketler, PWA olarak yuklenebilir
- **Desen tespiti** — benzer mesajlari otomatik gruplar
- **Korelasyon** — pill'e tikla, filtrele, iliskili loglari gor
- **Yer imleri** — satirlari isaretleyip not ekleyin
- **Disa aktarma** — JSONL, CSV, markdown, duz metin

## Kurulum

```bash
pip install yazbunu

# Web goruntuleyici icin:
pip install yazbunu[server]
```

## Hizli Baslangic

```python
from yazbunu import get_logger, init_logging

init_logging(log_dir="./logs", project="demo")
log = get_logger("ana")

log.info("basladi")
log.info("isleniyor", gorev="42", adet=150)
log.warning("yavas sorgu", sure_ms=3200, tablo="kullanicilar")
```

```bash
yazbunu-server --log-dir ./logs
# http://localhost:9880 adresini acin
```

## Surecler ile Entegrasyon

yazbunu'nun `/health` endpointi hem canlilik durumunu hem de baglanti bilgisini dondurur:

```bash
curl http://localhost:9880/health
# {"status":"ok","version":"0.2.0","token":"abc...","url":"http://...?token=abc..."}
```

Surec yoneticiniz (ornegin Yasar Usta):
1. `GET /health` ile yazbunu'yu kontrol eder
2. `url` alanini alir
3. Telegram/Slack/e-posta ile kullaniciya gonderir
4. Kullanici linke tiklar, aninda baglanir

## Lisans

MIT

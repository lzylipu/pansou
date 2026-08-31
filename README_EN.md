# 🔍 PanSou — Cloud Drive Search API

**High-performance cloud-drive resource search API with Telegram channels and a pluggable async plugin system.**

**English | [简体中文](./README.md)**

---

> ⚠️ **Note**: this repository is a fork of [fish2018/pansou](https://github.com/fish2018/pansou). The upstream project remains the primary reference — see its [original README](https://github.com/fish2018/pansou) for the canonical docs. All credit for the core design belongs to the original author.
>
> PanSou is a high-performance cloud-drive resource search API service supporting TG channel search and custom plugin search. The system is designed around performance and extensibility: concurrent search, intelligent result ranking, and classification by cloud-drive type.

---

## ✨ Features (details in the [System Design Doc](docs/系统开发设计文档.md))

- **High-performance search** — concurrently executes multiple TG channel and async plugin searches; worker-pool design manages concurrent tasks efficiently
- **Cloud-drive type classification** — auto-detects multiple cloud-drive link types and groups results accordingly
- **Intelligent ranking** — multi-dimensional ranking based on plugin level, time freshness and priority keywords
- **Async plugin system** — extend search sources via plugins, with a "respond fast, keep processing" async mode that solves slow-source latency. See the [Plugin Development Guide](docs/插件开发指南.md); for AI-assisted development see the [Plugin Skill Guide](docs/AI客户端使用PanSou插件开发Skill指南.md)
- **Two-tier cache** — sharded in-memory + sharded on-disk cache, dramatically improving repeated-query speed and concurrency

## 📚 Developer Docs

- [Plugin Development Guide](docs/插件开发指南.md) — plugin interfaces, async mechanism, priorities, filter strategies and workflow
- [Plugin Skill Usage Guide](docs/AI客户端使用PanSou插件开发Skill指南.md) — reuse the plugin development rules in Codex, Claude, Cursor, Windsurf, Copilot Chat, Cline/Roo Code and other AI clients
- [Plugin Skill Source](docs/pansou-plugin-developer-SKILL.md) — installable directly into Skill-capable clients, or referenced as project rules

## 🌐 Supported Cloud Drive Types

Baidu (`baidu`), Aliyun (`aliyun`), Quark (`quark`), Guangya (`guangya`), Tianyi (`tianyi`), UC (`uc`), China Mobile (`mobile`), 115 (`115`), PikPak (`pikpak`), Xunlei (`xunlei`), 123 (`123`), magnet (`magnet`), eD2K (`ed2k`), others (`others`)

## 🚀 Quick Start

### Docker deployment

**1. All-in-one (frontend + backend)**

```bash
docker run -d --name pansou -p 80:80 ghcr.io/fish2018/pansou-web
```

**2. Backend API only**

```bash
docker run -d --name pansou -p 8888:8888 ghcr.io/fish2018/pansou:latest
```

**Docker Compose (recommended)**

```bash
# Download the config file
curl -o docker-compose.yml https://raw.githubusercontent.com/fish2018/pansou/refs/heads/main/docker-compose.yml

# Start the service
docker-compose up -d

# View logs
docker-compose logs -f
```

The service is available at `http://localhost:8888`.

### Install from source

**Requirements**: Go 1.18+; optional SOCKS5 proxy for restricted-region Telegram access.

```bash
git clone https://github.com/fish2018/pansou.git
cd pansou
```

#### Basic configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Service port | `8888` |
| `PROXY` | SOCKS5 proxy, e.g. `socks5://127.0.0.1:1080` | none |
| `HTTPS_PROXY` / `HTTP_PROXY` | HTTPS/HTTP proxies | none |
| `CHANNELS` | Default TG channels to search, comma-separated | `tgsearchers3` |
| `ENABLED_PLUGINS` | Plugins to enable, comma-separated | none (must be explicit) |

#### Authentication (optional)

PanSou ships an optional JWT-based auth module, disabled by default. When enabled, all API endpoints (except login) require a valid JWT token.

| Variable | Description | Default |
|----------|-------------|---------|
| `AUTH_ENABLED` | Enable authentication | `false` |
| `AUTH_USERS` | User accounts, format `user1:pass1,user2:pass2` | none |
| `AUTH_TOKEN_EXPIRY` | Token validity (hours) | `24` |
| `AUTH_JWT_SECRET` | JWT signing secret | auto-generated (set manually recommended) |

```bash
# Enable auth with a single user
docker run -d --name pansou -p 8888:8888 \
  -e AUTH_ENABLED=true \
  -e AUTH_USERS=admin:admin123 \
  -e AUTH_TOKEN_EXPIRY=24 \
  ghcr.io/fish2018/pansou:latest
```

**Auth API:**

- `POST /api/auth/login` — log in and obtain a token
- `POST /api/auth/verify` — verify a token
- `POST /api/auth/logout` — log out (client discards the token)

```bash
# 1. Log in to get a token
curl -X POST http://localhost:8888/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'

# 2. Call the search API with the token
curl -X POST http://localhost:8888/api/search \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"kw":"your keyword"}'
```

#### Advanced configuration (defaults are fine)

<details>
<summary>Expand advanced options (usually no changes needed)</summary>

| Variable | Description | Default |
|----------|-------------|---------|
| `CONCURRENCY` | Concurrent search count | auto-computed |
| `CACHE_TTL` | Cache validity (minutes) | `60` |
| `CACHE_MAX_SIZE` | Max cache size (MB) | `100` |
| `PLUGIN_TIMEOUT` | Plugin timeout (seconds) | `30` |
| `ASYNC_RESPONSE_TIMEOUT` | Fast-response timeout (seconds) | `4` |
| `ASYNC_LOG_ENABLED` | Verbose async plugin logging | `true` |
| `CACHE_PATH` | Cache file path | `./cache` |
| `SHARD_COUNT` | Cache shard count | `8` |
| `CACHE_WRITE_STRATEGY` | Cache write strategy (immediate/hybrid) | `hybrid` |
| `ENABLE_COMPRESSION` | Enable compression | `false` |
| `MIN_SIZE_TO_COMPRESS` | Compression threshold (bytes) | `1024` |
| `GC_PERCENT` | Go GC trigger percentage | `50` |
| `ASYNC_MAX_BACKGROUND_WORKERS` | Max background workers | CPU cores × 5 |
| `ASYNC_MAX_BACKGROUND_TASKS` | Max background tasks | workers × 5 |
| `ASYNC_CACHE_TTL_HOURS` | Async cache validity (hours) | `1` |
| `ASYNC_PLUGIN_ENABLED` | Enable async plugins | `true` |
| `HTTP_READ_TIMEOUT` | HTTP read timeout (seconds) | auto-computed |
| `HTTP_WRITE_TIMEOUT` | HTTP write timeout (seconds) | auto-computed |

</details>

## 🤝 Acknowledgments

This fork exists thanks to the original [fish2018/pansou](https://github.com/fish2018/pansou) project and its author. All core architecture, plugin system and caching design come from upstream.

## 📄 License

MIT — see [LICENSE](./LICENSE).

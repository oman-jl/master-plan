# Master Plan — The Big Blueprint

A curated overview of the AI-agent infrastructure and side-projects built on this host, and where to find each piece.
This is the "gift" repository: it ties everything together. Each project has its own dedicated repo (links below).

---

## 1. The stack at a glance

```
                 ┌────────────────────────────────────────────────────┐
   public        │  Cloudflare quick tunnel (cloudflared)             │
   internet ───► │  https://<name>.trycloudflare.com                  │
                 │        │                                          │
                 │        ▼                                          │
                 │  nginx (local listener, Basic Auth)                │
                 │        │                                          │
                 │        ▼                                          │
                 │  upstream service (see per-project repos)          │
                 └────────────────────────────────────────────────────┘
```

Common pattern: **cloudflared → nginx (Basic Auth) → service**. No systemd; services run via `nohup`/`setsid` with file logging. Auth via `.htpasswd` (generated locally, never committed).

---

## 2. Projects & repos

| Project | Repo | What it is |
|---|---|---|
| my-callme | `oman-jl/my-callme` | Self-hosted /call-me clone (FastAPI + Twilio/mock + MCP + CLI) |
| Orgo control | `oman-jl/orgo-control` | Control stack for Orgo.ai Cloud PC (bash/screenshot/SSH/demo) |
| Cursor web chat | `oman-jl/cursor-web-chat` | Web chat UI backed by `cursor-agent` CLI |
| Tunnel stack | `oman-jl/nanobot-tunnels` | Reusable nginx + cloudflared + noVNC reference setup |
| AppuBot / Abacus | (config/setup, covered under tunnel stack) | Electron desktop apps exposed via noVNC |

---

## 3. What each project does

### 3.1 my-callme — self-hosted phone bridge
- Backend mirrors the `callmemcp.com` API: `/register /identity /pair /ring /text /calls /say /hangup /title /events /listen`.
- Providers: **mock** (demo/test, no setup) and **twilio** (real calls/SMS, set `TWILIO_*` env).
- Includes a `callme`-compatible CLI, a stdio MCP server, and a run script with isolated state.
- Build: fetch `my-callme-master.zip` from the repo's **v1.0.0 release**.

### 3.2 orgo-control
- `orgo_client.py` — Orgo API client (Bearer token from env, no secrets).
- `control.py` — status / provision / screenshot / bash / ssh-enable / deploy-demo / report.
- `demo/` — sample Python HTTP app (port 8000).

### 3.3 cursor-web-chat
- Thin stdlib HTTP server; `POST /api/chat` shells out to `cursor-agent --yolo --print --output-format json --mode ask`.
- Paths configurable via env (`CURSOR_AGENT`, `CURSOR_WS`, `INDEX_HTML`).

### 3.4 nanobot-tunnels
- Copy-paste nginx site configs for the tunnel fronts + guardian launcher scripts for Electron apps under a containerized supervisor.

---

## 4. Reproducing the whole thing

Each repo is self-contained with its own README. General recipe per service:

1. Run the service (see its repo README).
2. Put an nginx listener in front with Basic Auth (`nginx/*.conf` templates).
3. Start a Cloudflare quick tunnel: `cloudflared tunnel --url http://127.0.0.1:<listener> &> tunnel.log &`.
4. Read the public URL from `tunnel.log` (changes on restart).
5. Protect everything with `.htpasswd` (owner-created).

---

## 5. Security notes (important)
- **No secrets in any repo.** Keys, passwords, and live URLs are environment/owner-supplied.
- `.htpasswd` files, `.env`, logs, pids are gitignored.
- Orgo session cookies are domain-bound — in-proxy login to orgo.ai may not persist.
- Treat any tunnel that fronts an unauthenticated local gateway as sensitive; always put Basic Auth in front.

---

## 6. Status
- my-callme: ✅ built + pushed + release archive.
- orgo-control: ✅ live + this repo.
- cursor-web-chat: ✅ live + this repo.
- nanobot-tunnels: ✅ live (4+ tunnels) + this repo.
- This master-plan: ✅ the gift.

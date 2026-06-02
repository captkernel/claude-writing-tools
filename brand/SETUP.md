# SETUP — Self-hosting Postiz + wiring the pipeline

This is the infrastructure half of the project: stand up Postiz (the publisher),
then connect the brand layer and a content generator to it. The brand and
generation layers are plain prompt files (`brand-prompts.md`, plus your existing
writing-and-content prompt pack); Postiz is the only piece that needs a server.

## How the pieces fit

```
  brand-prompts.md            writing/content prompt pack         Postiz (self-hosted)
 ┌──────────────────┐        ┌───────────────────────┐          ┌────────────────────┐
 │ positioning      │        │ turns a brief + brand  │          │ schedule + publish │
 │ voice            │ ─────▶ │ profile into captions, │ ───────▶ │ across 30+ channels │
 │ visual direction │ brand  │ threads, image prompts │  drafts  │ (X, LI, IG, TikTok)│
 └──────────────────┘ profile└───────────────────────┘          └────────────────────┘
        (1) brand layer            (2) generation layer               (3) scheduling layer
```

- **(1) Brand layer** — run the guided prompts in `brand-prompts.md` once. Output is
  a reusable *brand profile* (positioning, voice rules, visual direction). It is an
  input artifact, not a running service.
- **(2) Generation layer** — your existing writing-and-content prompt pack, given the
  brand profile + a brief, produces post copy and image prompts. No new code needed;
  see `publish.md`.
- **(3) Scheduling layer** — Postiz takes the generated posts via its API / agent CLI
  and schedules/publishes them. This is the only component you self-host.

## Standing up Postiz (Docker)

Postiz ships an official all-in-one Docker image (app + Postgres + Redis can be run
together via compose). Minimal single-container quickstart:

```bash
docker run -d \
  --name postiz \
  -p 5000:5000 \
  -v postiz-config:/config \
  -v postiz-uploads:/uploads \
  -e MAIN_URL="http://localhost:5000" \
  -e FRONTEND_URL="http://localhost:5000" \
  -e NEXT_PUBLIC_BACKEND_URL="http://localhost:5000/api" \
  -e JWT_SECRET="$(openssl rand -hex 32)" \
  -e DATABASE_URL="postgresql://postiz:postiz@postiz-postgres:5432/postiz" \
  -e REDIS_URL="redis://postiz-redis:6379" \
  ghcr.io/gitroomhq/postiz-app:latest
```

For anything beyond a local trial, use `docker-compose` so Postgres and Redis are
managed alongside the app and persisted. Sketch:

```yaml
# docker-compose.yml (skeleton — pin versions and set real secrets before use)
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    ports: ["5000:5000"]
    environment:
      MAIN_URL: "https://postiz.example.com"
      FRONTEND_URL: "https://postiz.example.com"
      NEXT_PUBLIC_BACKEND_URL: "https://postiz.example.com/api"
      JWT_SECRET: "<32+ byte random hex>"
      DATABASE_URL: "postgresql://postiz:postiz@postgres:5432/postiz"
      REDIS_URL: "redis://redis:6379"
    depends_on: [postgres, redis]
    volumes:
      - postiz-config:/config
      - postiz-uploads:/uploads
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: postiz
      POSTGRES_PASSWORD: postiz
      POSTGRES_DB: postiz
    volumes: [postgres-data:/var/lib/postgresql/data]
  redis:
    image: redis:7-alpine
    volumes: [redis-data:/data]
volumes:
  postiz-config: {}
  postiz-uploads: {}
  postgres-data: {}
  redis-data: {}
```

Then:

```bash
docker compose up -d
# open http://localhost:5000 and create the first admin account
```

> Always check the current Postiz docs for the canonical compose file and the full
> env-var list before deploying — env-var names and the recommended image change
> over time. The block above is a working shape, not a guaranteed-current config.

## Connecting social channels

1. Open the Postiz web UI (port 5000) and create your account / workspace.
2. Add channels under **Settings → Channels**. Each platform (X, LinkedIn,
   Instagram, TikTok, Threads, Reddit, YouTube, Discord, Bluesky, ...) is an OAuth
   connection — you authorize Postiz against each account once.
3. Some platforms (X, etc.) require you to register your own developer app and paste
   client keys into Postiz's env/config; the UI flags which need this.

## Getting an API key / CLI access for automation

- **Public API** — generate an API key in the Postiz settings; pass it as a bearer
  token on REST calls (see `publish.md`).
- **Agent CLI** — Postiz publishes an agent CLI intended for driving it from AI
  agents (the source post calls out "Postiz agent CLI ... perfect for OpenClaw and
  other agents"). Install it and authenticate with the same API key / instance URL.
- **n8n / Make.com** — there is a custom n8n node and Make.com integration if you'd
  rather orchestrate visually instead of from code.

## Honest caveats

- **Only worth it if you're running an actual content operation.** Self-hosting means
  you own a Postgres, a Redis, OAuth app registrations per platform, token refreshes,
  upgrades, and uptime. For one account posting occasionally, native schedulers or a
  hosted tool are less work. The payoff appears at multi-channel, multi-post-per-day,
  agent-driven volume.
- **Platform API access is the real friction, not Postiz.** Instagram/TikTok/X
  approval, app review, and rate limits are external constraints Postiz can't remove.
- **"BrandingOS" is just guided prompts.** There is no product to install for the
  brand layer — `brand-prompts.md` is the whole thing. Treat the source's
  "replicates a full brand team" and "evolves weekly with your data" framing as
  marketing. What's genuinely useful is the structured Q&A that yields a reusable
  brand profile; that's what we kept.
- **AGPL-3.0.** Postiz is AGPL-licensed. Fine for internal self-hosting; if you expose
  a modified version as a network service to others, the license obligations apply.
- **Self-hosting != free.** "$0/month" ignores the VPS, storage, and your time.

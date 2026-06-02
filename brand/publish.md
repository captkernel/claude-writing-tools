# publish.md — Automation sketch (generate → schedule via Postiz)

How the publish loop works end to end. The generation layer is **your existing
writing-and-content prompt pack** — we don't rebuild it here; we feed it the brand
profile and a brief, then hand the result to Postiz. This is a sketch of the shapes,
not a runnable script (you don't need Postiz live to follow it).

## The loop

```
brand-profile.md ─┐
                  ├─▶ [generation layer: writing/content prompt pack] ─▶ posts.json
brief (topic) ────┘                                                        │
                                                                           ▼
                                                       [Postiz API / agent CLI] ─▶ scheduled across channels
```

1. **Generate.** Give the writing-and-content prompt pack: the brand profile (voice +
   positioning + visual rules) and a brief (topic, platform, goal). It returns post
   copy per platform plus an image prompt.
2. **(Optional) render images.** Pass the image prompt to your image tool (GPT Image /
   Higgsfield / Seedance, per the brand profile's template). Upload the asset to Postiz
   or pass a URL.
3. **Schedule.** Send each post to Postiz via REST or the agent CLI with a target
   channel + publish time. Postiz handles per-platform formatting and delivery.

### Intermediate format the generator should emit

Have the prompt pack emit a predictable JSON array so the publish step is mechanical:

```json
[
  {
    "platform": "x",
    "content": "Hook line.\n\nBody.\n\nCTA.",
    "image_prompt": "..., palette #0E1116/#3B82F6, bold minimal, no text",
    "publish_at": "2026-06-05T14:00:00Z"
  },
  {
    "platform": "linkedin",
    "content": "Longer on-voice variant of the same idea.",
    "image_prompt": "...",
    "publish_at": "2026-06-05T15:00:00Z"
  }
]
```

## Posting via the Public API (example request shape)

> Illustrative shape only. Confirm the exact endpoint path, field names, and how
> channel IDs / media are referenced against the current Postiz API docs before
> wiring this up — Postiz's schema evolves.

```bash
# 1. Get your channel/integration IDs (so you know what to target)
curl -s https://postiz.example.com/api/integrations \
  -H "Authorization: Bearer $POSTIZ_API_KEY"

# 2. Create a scheduled post
curl -s -X POST https://postiz.example.com/api/posts \
  -H "Authorization: Bearer $POSTIZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "scheduled",
    "date": "2026-06-05T14:00:00Z",
    "posts": [
      {
        "integrationId": "<x-channel-id>",
        "value": [{ "content": "Hook line.\n\nBody.\n\nCTA." }],
        "media": []
      }
    ]
  }'
```

A driver script just loops the generator's `posts.json` and fires one request per
entry, mapping `platform → integrationId`:

```bash
for row in $(jq -c '.[]' posts.json); do
  platform=$(jq -r '.platform' <<<"$row")
  content=$(jq -r '.content'  <<<"$row")
  date=$(jq -r '.publish_at'  <<<"$row")
  integration=$(jq -r --arg p "$platform" '.[$p]' channel-map.json)
  curl -s -X POST https://postiz.example.com/api/posts \
    -H "Authorization: Bearer $POSTIZ_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$(jq -n --arg id "$integration" --arg c "$content" --arg d "$date" \
        '{type:"scheduled", date:$d, posts:[{integrationId:$id, value:[{content:$c}]}]}')"
done
```

## Posting via the agent CLI

The source calls out a Postiz **agent CLI** built for AI agents ("perfect for OpenClaw
and other agents"). The shape that matters: it's a command you can call from a Claude
Code skill / subagent, so an agent can both generate and schedule in one run.

```bash
# install + authenticate once
postiz login --url https://postiz.example.com --token "$POSTIZ_API_KEY"

# schedule a post (flags illustrative — see `postiz --help` for the real surface)
postiz post create \
  --channel x \
  --content "Hook line.\n\nBody.\n\nCTA." \
  --at "2026-06-05T14:00:00Z" \
  --media ./out/cover.png
```

Because it's a plain CLI, the natural integration is a small Claude Code skill that:
runs the brand profile + brief through the writing-and-content prompt pack, writes
`posts.json`, then shells out to `postiz post create` (or the REST loop above) per row.

## Where each layer lives (recap)

- **Brand layer** → `brand-prompts.md` → produces `brand-profile.md` (run once).
- **Generation layer** → your existing writing-and-content prompt pack (reused, not
  rebuilt) → produces `posts.json`.
- **Scheduling layer** → Postiz API or agent CLI → publishes across channels.

## Caveats

- Field names, endpoint paths, and CLI flags above are **illustrative** — verify
  against current Postiz docs.
- Rate limits and platform approvals (IG/TikTok/X) are the real bottleneck, not the
  Postiz call itself.
- Keep `POSTIZ_API_KEY` out of the repo (env var / secrets manager).
- Don't fully auto-publish on day one — schedule into a review/approval state first and
  watch a few cycles before trusting the loop unattended.

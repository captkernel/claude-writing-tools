# brand-prompts.md — Guided brand workflow (parameterised)

A concrete, repeatable brand-definition workflow distilled from the BrandingOS
source. The source's value is its structure: answer a short set of questions across
three stages and you get a reusable **brand profile** that feeds the content
generator. We dropped the "replicates a full brand team" and "evolves weekly with
your data" claims — those are framing, not mechanism.

**How to use:** run this once per brand. Paste each stage's prompt into Claude,
answer the `{{...}}` slots, and collect the outputs into a single `brand-profile.md`.
That profile is the input to the generation layer in `publish.md`.

The three things founders skip (and this covers): where you're going and why, how you
sound and feel, how you look.

---

## Inputs you provide once

```yaml
business:        "{{one-line description of the business/product}}"
audience:        "{{who it is for — be specific: role, situation, pain}}"
offer:           "{{what you sell / the core value exchange}}"
goal:            "{{the outcome that matters in the next 90 days}}"
competitors:     "{{2-4 names or 'the default option people use instead'}}"
constraints:     "{{budget, channels you actually post on, hard no-gos}}"
```

---

## Stage 1 — Positioning (where you're going and why)

> You are acting as a brand strategist. Using the inputs below, produce a tight
> positioning that makes this brand a deliberate choice rather than "just another
> option in the feed." Be concrete; reject generic claims that any competitor could
> also make.
>
> Inputs:
> - Business: `{{business}}`
> - Audience: `{{audience}}`
> - Offer: `{{offer}}`
> - 90-day goal: `{{goal}}`
> - Alternatives people use instead: `{{competitors}}`
>
> Produce:
> 1. **One-sentence positioning statement** — `For {{audience}} who {{pain}}, {{brand}}
>    is the {{category}} that {{differentiator}}, unlike {{alternative}}.`
> 2. **3 differentiators** — each phrased so a competitor genuinely could NOT say it.
> 3. **Audience snapshot** — their trigger moment, what they fear, what they want to
>    be seen as.
> 4. **What we are NOT** — 3 anti-positions to keep us off-brand-safe.
> 5. **Proof obligations** — what we must show/prove for the positioning to be believed.

---

## Stage 2 — Voice (how you sound and feel)

> You are acting as a brand voice lead. Define a voice that is recognizable across
> a feed of competitors. Ground every choice in the positioning from Stage 1.
>
> Inputs:
> - Positioning statement: `{{paste Stage 1 output}}`
> - Audience snapshot: `{{paste Stage 1 output}}`
> - Personality dials (rate each 1-5): formal↔casual `{{n}}`, serious↔playful
>   `{{n}}`, expert↔peer `{{n}}`, reserved↔bold `{{n}}`.
>
> Produce:
> 1. **Voice in 3 adjectives** + one sentence each on what it means in practice.
> 2. **Do / Don't table** — 6 rows (e.g. "Do: short declaratives. Don't: hedge with
>    'maybe/kind of'.").
> 3. **Lexicon** — 8-12 on-brand words/phrases + 6 banned words/clichés.
> 4. **Sentence-level rules** — opener style, sentence length target, emoji policy,
>    CTA style.
> 5. **Three sample lines** — same idea written on-voice for: a hook, a body line, a CTA.

---

## Stage 3 — Visual direction (how you look)

> You are acting as a creative director. Define a visual direction that matches the
> positioning and voice — not just aesthetics. Output must be usable two ways: as a
> human style guide AND as text-to-image prompt scaffolding for tools like GPT Image,
> Higgsfield, or Seedance (the source's tools), so visuals can be generated even
> without a designer.
>
> Inputs:
> - Positioning + voice summary: `{{paste Stages 1-2}}`
> - Hard constraints: `{{brand colors already in use, logo if any, platform formats}}`
>
> Produce:
> 1. **Mood in 3 words** + 1 reference adjective for lighting, composition, palette.
> 2. **Palette** — 3-5 hex values with a usage note each (primary/accent/neutral).
> 3. **Typographic feel** — 1 display + 1 body pairing characteristics (not necessarily
>    exact fonts).
> 4. **Imagery rules** — subject, framing, what to avoid (e.g. "no generic stock smiles").
> 5. **Reusable image-prompt template** — a fill-in prompt string that bakes in the
>    palette/mood, e.g.:
>    `"{{subject}}, {{composition}}, {{lighting}}, palette {{hexes}}, {{mood words}},
>    on-brand for {{brand}}, high detail, no text"`.
> 6. **Per-format notes** — feed post, story/reel cover, profile/banner.

---

## Output: the brand profile

Collect the three stage outputs into `brand-profile.md` with this shape. This file is
the single source of truth the generation layer reads.

```markdown
# Brand Profile — {{brand}}

## Positioning
<Stage 1 output>

## Voice
<Stage 2 output>

## Visual direction
<Stage 3 output>
```

> Optional, honest version of "evolves weekly": once a month (not on a hype cadence),
> paste your top- and bottom-performing posts back in and ask Claude which voice/visual
> rules to tighten. It's a manual review, not an automatic learning loop — treat it as
> editing the profile, not magic.

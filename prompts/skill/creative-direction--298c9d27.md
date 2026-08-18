---
title: "creative-direction"
type: "skill"
tags: ["kit","skill","create brand guide","lock voice","brand voice","creative direction","visual style guide","brand consistency"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "298c9d27-320c-4e5f-8d76-9be0667c43c0"
---

> Brand voice + visual style guidelines for ad creatives. Use when locking brand voice across multiple ad variants, defining consistent visual style, or creating brand bible content. Triggers — "create brand guide", "lock voice", "brand voice", "creative direction", "visual style guide", "brand consistency".

# Creative Direction — Brand Voice & Visual Style Lock

**Purpose:** Keep ad copy, image, and video creatives consistent with a single brand voice across hundreds of variants. Avoids the "AI slop" feel of generic models.

## When to use this skill

- Generating multiple variants for A/B testing — they must all sound like the *same brand*.
- Building a brand bible from scratch (new client, new product line).
- Auditing existing creatives for voice drift.
- Onboarding a new model (Gemini → Claude → Flux): ensure voice survives the model switch.

## Core principle: voice = (rules) + (examples) + (constraints)

A brand voice that survives 100 variants needs three layers:

### 1. Rules (top-level system prompt)
- Tone (e.g., "warm, professional, never sarcastic")
- Persona (e.g., "expert mentor, not a salesperson")
- Banned phrases ("best ever", "you", "buy now"-style spam)
- Required phrases (signature CTA, brand tagline)

### 2. Examples (few-shot, retrieved by similarity)
- 5-20 past creatives that performed well (CTR > median, conversions > target)
- Stored as embeddings (text-embedding-3-small)
- Retrieved at gen-time via cosine similarity to the new brief
- Inject as `## Past winning examples` block before the prompt

### 3. Constraints (per-channel)
- Meta: 90 chars headline, 125 chars body, 30 chars CTA
- Google Search: 30 chars headlines (× 15), 90 chars descriptions (× 4)
- Google Display: 30 chars short headline, 90 chars long
- LinkedIn: 70 chars headline, 150 chars description
- TikTok: 100 chars caption (vertical-first)

## Visual style lock

Same three layers apply to image/video:

### Visual Rules
- Color palette (3-5 hex codes)
- Typography (Hebrew + Latin fonts)
- Logo placement zone (corner + safe area)
- Photography style ("editorial", "lifestyle", "studio", never "stock")
- Aspect ratios per placement (1:1, 9:16, 16:9, 4:5)

### Visual Examples
- Mood board: 5-10 reference images
- For Ideogram/Recraft, encode as part of the prompt (style reference URLs)

### Visual Constraints
- Hebrew text rendering: Ideogram 3.0 only (Flux/Imagen broken)
- Vector logos: Recraft V3 only
- Background removal before placement: PhotoRoom

## Brand bible structure (output template)

```markdown
# Brand Bible — {brand}

## Voice
- Tone: {warm/clinical/playful/authoritative}
- Persona: {who is the brand "speaking as"}
- We say: {3 phrases}
- We don't say: {3 phrases}

## Visuals
- Palette: {primary} {secondary} {accent}
- Typography: {Hebrew font} + {Latin font}
- Style: {1-paragraph description}
- Reference URLs: {3-5 URLs to past winning creatives}

## Constraints
- Per channel: {table of char limits}
- Hebrew rendering: Ideogram only
- Logo zone: {corner + margin}

## Past winners
{Top 10 creatives by CTR with their copy + visual}
```

## Integration

The brand bible lives in `src/lib/ai/prompts/brand-voice.ts` and is loaded as a static system prompt addon. The "past winners" section is dynamic — populated at runtime via pgvector retrieval from `ppc.brand_voice_vectors`.

## Anti-patterns

- ❌ Different voice per variant ("we tried 5 styles") → A/B testing is for *creative*, not *voice*
- ❌ "All variants sound generic AI" → missing few-shot examples
- ❌ Voice drift across models → re-anchor with examples in every prompt
- ❌ "Hebrew text in image looks broken" → using Flux/Imagen instead of Ideogram

## Quick checklist

Before generating a creative, confirm:
- [ ] Brand bible loaded as system prompt
- [ ] Top-5 past winners retrieved (if pgvector populated)
- [ ] Channel-specific char limits enforced
- [ ] Hebrew text → Ideogram routing
- [ ] Logo zone respected in visual prompt

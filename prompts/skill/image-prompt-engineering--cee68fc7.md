---
title: "image-prompt-engineering"
type: "skill"
tags: ["kit","skill","image prompt","ideogram prompt","flux prompt","ad image prompt","creative prompt","image"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:03:35.307177+00:00"
id: "cee68fc7-2fd3-4314-9bba-c8e1baf4460e"
---

> Per-model image prompt templates. Each model (Ideogram, Flux, Imagen, Recraft, SD) has different syntax. Use when building image prompts for ads — especially Hebrew text-in-image. Triggers — "image prompt", "ideogram prompt", "flux prompt", "ad image prompt", "hebrew text in image", "creative prompt".

# Image Prompt Engineering — Per-Model Templates

**Purpose:** Image generation models are *not interchangeable* at the prompt level. A prompt that works on Flux fails on Ideogram and vice versa. This skill documents what each model needs.

## Model selection matrix

| Need | Model | Why |
|------|-------|-----|
| Hebrew text rendered in image | **Ideogram 3.0** | Only model that renders Hebrew correctly |
| English/Latin text on image | Ideogram 3.0 or Recraft V3 | Both excellent |
| Vector logo or icon | **Recraft V3** | Native vector output |
| Photorealistic, no text | **Flux 1.1 Pro Ultra** (fal.ai) | Fastest, cheapest, best quality for photo |
| Brand-consistent variations | **Bannerbear** template | Programmatic, RTL-aware |
| Generative fill / inpaint | **Adobe Firefly** | Best inpaint model |
| Background removal | **PhotoRoom** | Cleanest cutouts |
| 4× upscale | **Magnific** | Best upscaler |

## Ideogram 3.0 prompt template (Hebrew text-in-image)

Ideogram is unique: it understands "render the text X exactly". Use this:

```
{{Brief description of the scene}},
text on image: "{{Hebrew text exactly}}",
style: {{REALISTIC | DESIGN | GENERAL}},
aspect_ratio: {{1:1 | 9:16 | 16:9 | 4:5}},
color palette: {{2-3 dominant colors}},
mood: {{1-2 mood words}}
```

**Example (good):**
```
Modern Hebrew-language ad for a teacher training course,
text on image: "הירשמו עכשיו",
style: DESIGN,
aspect_ratio: 1:1,
color palette: deep blue and warm yellow,
mood: confident, inviting
```

**Example (bad — wrong model):**
```
A teacher with Hebrew text "הירשמו"
→ Flux/Imagen will render gibberish "הצרשמ" or random Latin chars
```

## Flux 1.1 Pro Ultra prompt template (photoreal, no text)

Flux loves rich, descriptive prompts. Length helps. Avoid camera-tech jargon (it ignores it).

```
{{Subject in detail}},
{{action or state}},
{{environment}},
{{lighting}},
{{mood}},
{{style: editorial / lifestyle / studio}},
shot on {{camera, optional — Flux uses for hint}},
{{quality boosters: ultra-detailed, 4K, sharp focus}}
```

**Example:**
```
Three young Israeli teachers laughing together in a sunlit classroom,
gentle warm light from a tall window, books and notebooks on a wooden table,
editorial style, candid composition, vibrant but natural colors, 4K, sharp focus
```

**Avoid:**
- Hebrew text in image (broken)
- Negative prompts (Flux ignores them)
- Camera tech jargon (waste of tokens)

## Recraft V3 prompt template (vector + logos)

Recraft outputs SVG. Best for logos, icons, branded shapes. Use clean, structural language.

```
{{Subject as a vector illustration}},
{{Style: flat / minimal / geometric}},
{{Color palette}},
{{Composition: centered / symmetrical}},
{{Output: vector / svg / illustration}}
```

**Example:**
```
Logo for "Pil Project" educational platform,
flat geometric vector,
warm yellow and navy blue,
centered, balanced composition,
modern educational mark, vector illustration
```

## Imagen 4 prompt template (Google, weak Hebrew)

Imagen is strong on photorealism but weak with text. Use only when you need Google ecosystem (Vertex AI, Gemini API native).

Same template as Flux but shorter — Imagen prefers concise.

## Stable Diffusion 3.5 prompt template (open-source, self-host)

Use only if you have GPU infra. Add quality boosters. Heavy on negative prompt.

```
Positive: {{description}}, masterpiece, best quality, ultra-detailed
Negative: low quality, blurry, deformed, watermark, signature, text
```

## Aspect ratio map (per ad placement)

| Placement | Ratio | Pixels |
|-----------|-------|--------|
| Meta Feed | 1:1 | 1080×1080 |
| Meta Stories / Reels | 9:16 | 1080×1920 |
| Meta Right Column | 1.91:1 | 1200×628 |
| Google Display Square | 1:1 | 1080×1080 |
| Google Display Banner | 1.91:1 | 1200×628 |
| Google Display Skyscraper | 9:16 | 600×1200 |
| LinkedIn Single Image | 1.91:1 | 1200×628 |
| TikTok / Reel | 9:16 | 1080×1920 |
| Pinterest | 2:3 | 1000×1500 |

## Hebrew RTL specific tips

- ✅ **Ideogram**: write the Hebrew exactly as you want it. It handles RTL.
- ✅ Specify `style: DESIGN` for Hebrew typography on a clean background.
- ❌ Don't mix Hebrew + Latin in the same `text on image:` string — confuses the model. Generate two passes if needed.
- ❌ Don't use Hebrew **alongside** complex scenes — Ideogram works best with text on simple backgrounds.

## Quick decision tree

```
Need text in image?
├── Hebrew? → Ideogram 3.0
├── English? → Ideogram 3.0 or Recraft V3
└── No text? → Flux 1.1 Pro Ultra

Need vector / logo? → Recraft V3
Need brand-locked variations? → Bannerbear template
Need 4× upscale on existing? → Magnific
Need background removed? → PhotoRoom
Need to fill/inpaint a region? → Adobe Firefly
```

## Anti-patterns

- ❌ "Generic" prompts ("a nice ad") → models default to AI-generic look
- ❌ Hebrew in Flux/Imagen → broken letters
- ❌ Long camera jargon for Flux → wasted tokens
- ❌ Skipping aspect_ratio → wrong placement size, ad rejected
- ❌ Reusing same prompt across 5 models → each needs different syntax

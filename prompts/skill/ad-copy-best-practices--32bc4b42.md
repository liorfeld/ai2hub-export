---
title: "ad-copy-best-practices"
type: "skill"
tags: ["kit","skill","ad copy","headline gen","meta copy","google ads copy","ad text","responsive search ad"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:51:03.524192+00:00"
id: "32bc4b42-cd0c-46a3-a171-85a61ff1a7ec"
---

> Meta + Google + LinkedIn ad copy guardrails — char limits, policy compliance, banned phrases, conversion-tested structures. Use when generating headlines, descriptions, body, and CTAs for any paid ad. Triggers — "ad copy", "headline gen", "meta copy", "google ads copy", "ad text", "responsive search ad", "PMax copy".

# Ad Copy Best Practices — Meta, Google, LinkedIn, TikTok

**Purpose:** Generate ad copy that *passes platform review* and *converts*. Each platform has hard char limits, banned phrases, and content policies — violations get ads rejected or accounts flagged.

## Char limits (hard, platform-enforced)

### Meta (Facebook + Instagram)
| Field | Limit | Notes |
|-------|-------|-------|
| Primary Text (body) | 125 chars (recommended) | up to 30K but truncated |
| Headline | 27 chars (recommended) | up to 40 |
| Description | 27 chars (recommended) | up to 30 |
| CTA | Pre-set list only | "Sign Up", "Learn More", etc. |

### Google Ads — Responsive Search Ad (RSA)
| Field | Count | Each |
|-------|-------|------|
| Headline | 3-15 | 30 chars |
| Description | 2-4 | 90 chars |
| Path 1, Path 2 | 1 each | 15 chars each |

### Google Ads — Performance Max (PMax)
| Asset | Count | Limit |
|-------|-------|-------|
| Short headline | up to 5 | 30 chars |
| Long headline | up to 5 | 90 chars |
| Description | up to 5 | 90 chars |
| Logo | 1 | image |
| Image | up to 20 | image |
| Video | up to 5 | YouTube |

### LinkedIn Single Image Ad
| Field | Limit |
|-------|-------|
| Introductory text | 600 chars |
| Headline | 70 chars |
| Description | 100 chars |

### TikTok In-Feed
| Field | Limit |
|-------|-------|
| Caption | 100 chars |
| Display name | 40 chars |

## Banned phrases (rejection triggers)

### Meta — these will get ads rejected
- "You" (personal attribute claims) — except in clearly-permitted contexts
- "Click here" / "Click below"
- Discriminatory ("women only" / "for [protected class]")
- Health claims ("lose weight fast", "guaranteed cure")
- Emojis indicating personal attributes (👶 implying user has baby)
- Before/after pictures (some categories)
- Drug/alcohol references (without restriction)
- Misleading "sale" / "limited time" without actual deadline
- ALL CAPS for full sentences

### Google — these will get ads rejected or flagged
- "BEST" / "#1" / superlatives without proof
- "FREE" without context
- Excessive punctuation ("!!!" "???")
- Trademark of competitors in copy
- Phone numbers in ad text (use call extension)
- Misleading promises ("get rich quick")
- Capital letter abuse

### LinkedIn — additional restrictions
- Personal pronouns ("Are you a CEO?")
- Inferred professional info ("As a manager, you...")
- Age, gender, family status references

## Structures that convert (tested)

### Meta Body — "PAS" structure (125 chars)
```
[Pain in 25 chars] [Agitate in 50 chars] [Solution in 50 chars]
```
**Example:** קשה למצוא מדריכים? המורים מתישים, התלמידים בחוץ. הגיעה ההכשרה שתחזיר אותך לכיתה.

### Google Search Headline — Variants per intent
- **Brand**: "{{Brand name}} | {{Tagline}}"
- **Product feature**: "{{Specific feature}} for {{Audience}}"
- **Outcome**: "Get {{Outcome}} in {{Timeframe}}"
- **Question**: "Looking for {{Need}}?"
- **Number / proof**: "{{N}} {{Audience}} chose us"

### Google Description — "Benefit + Proof + CTA"
```
[Benefit in 30 chars]. [Proof or differentiator in 30 chars]. [CTA in 30 chars]
```

### LinkedIn — B2B value-first
```
Headline: [Outcome that resonates with role]
Body: [3 sentences: pain → solution → proof]
CTA: "Learn More" or "Download"
```

## Hebrew copy specific guidelines

### Punctuation
- Use Hebrew punctuation: `?` (not Latin `?`), `!`, `,`, `;`, `:`
- Quotation marks: `"..."` or `«...»` (not `"..."`)
- Apostrophe: `׳` for abbreviations (`גב׳`, `ד״ר`)
- Geresh / Gershayim — use proper Unicode chars for acronyms

### Tone
- ❌ Avoid imperative: "תקנו עכשיו!" reads aggressive
- ✅ Prefer invitation: "מוזמנים להירשם", "ההזמנה שלכם פתוחה"
- ✅ "יחד" / "אנחנו" creates community feel
- ❌ Direct "אתה" can feel intrusive in B2C — prefer plural "אתם"

### Common mistakes
- ❌ "הכי טוב!" → Meta likely rejects (superlative without proof)
- ❌ "100% הצלחה" → guaranteed result claim, rejected
- ❌ Long sentences (>15 words) → truncated on mobile
- ❌ Tagline-only headline → no value prop
- ✅ Clear value + clear CTA + clear next step

## Quality checklist (before submission)

- [ ] Char limits respected (all variants)
- [ ] No banned phrases per platform
- [ ] CTA is specific (not generic "Learn More")
- [ ] Brand voice consistent with brand bible
- [ ] Past-winner cosine similarity > 0.7 (if pgvector available)
- [ ] Hebrew punctuation correct (proper geresh/quotes)
- [ ] No double-spacing, no emoji-bombing, no ALL CAPS
- [ ] Each variant is *materially different* (not paraphrased — test different angles)

## Variant generation strategy (for A/B testing)

When asked for N variants, vary by **angle**, not by **wording**:
1. **Pain-led** — start with the problem
2. **Outcome-led** — start with the benefit
3. **Social proof** — "{{N}} customers chose us"
4. **Question** — "Looking for {{X}}?"
5. **Story / mini-case** — "When {{Person}} needed {{X}}, they..."
6. **Authority** — "{{Expert / Source}} recommends"
7. **Urgency** — "{{Limited time}} {{Offer}}"
8. **Curiosity** — "What if you could {{X}}?"

Each variant should target a different angle, not paraphrase the same one.

## Anti-patterns

- ❌ All variants from same angle → not actually testing anything
- ❌ "Best ever" / "#1" without proof → rejected
- ❌ Click-bait ("You won't believe...") → low quality score on Google
- ❌ Mixing 3+ CTAs in one ad → confusion, low conversion
- ❌ Different headline tone in each variant of *same* campaign → brand drift

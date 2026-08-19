---
title: "video-script-writing"
type: "skill"
tags: ["kit","skill","video script","reel script","video ad","8s ad","15s ad","tiktok script"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:50:09.406941+00:00"
id: "05702782-9c22-4986-abdf-84bb38772db1"
---

> Short-form ad video script structure (8s/15s/30s) for Veo, Runway, Luma, HeyGen. Hook → Problem → Solution → CTA. Use when writing video ad briefs or scripts. Triggers — "video script", "reel script", "video ad", "8s ad", "15s ad", "tiktok script", "veo prompt video".

# Video Script Writing — 8s / 15s / 30s Ad Structure

**Purpose:** Short-form video ads (Reels, TikTok, YouTube Shorts, FB Stories) need a *specific* structure to convert. Generic "show product, say thing, end" gets ignored. This skill encodes the proven structures.

## The universal structure

Every effective short-form ad follows the same 4-act pattern:

| Section | 8s ad | 15s ad | 30s ad |
|---------|-------|--------|--------|
| **Hook** | 0-1s | 0-2s | 0-3s |
| **Problem** | 1-3s | 2-5s | 3-8s |
| **Solution** | 3-6s | 5-12s | 8-22s |
| **CTA** | 6-8s | 12-15s | 22-30s |

The **first 1.5 seconds determine retention.** If you don't hook, viewers swipe. So the hook is the most important second of the entire ad.

## Hook patterns (proven)

### 1. Pattern interrupt
"What you're doing wrong with X"
"Stop scrolling — this is for {{Audience}}"
"I was today years old when I learned..."

### 2. Bold claim
"The reason {{Audience}} fail at {{X}}"
"Why you'll never need {{Y}} again"

### 3. Question
"Have you ever felt {{Pain}}?"
"What if I told you {{Surprising}}?"

### 4. Visual disruption
- Hard cut from black
- Unexpected motion
- Close-up on face
- Text overlay with single bold word

### 5. POV / character entry
"Day 1 as a {{Role}}..."
"Me trying to {{Activity}}..."

## Problem section (1-3s)

Make the viewer say "yes, that's me." Concrete, specific, *visual*.

- ❌ "Many people struggle with productivity"
- ✅ "It's 3pm, your inbox is at 47, and you haven't started the actual work"

For Hebrew specifically:
- Show the moment, not the abstraction
- Use familiar Israeli context (afternoon traffic, parents pickup, אוטובוס איחור)

## Solution section (3-22s, longest)

Three sub-beats:
1. **Reveal** — what is it
2. **Why it works** — one mechanism / proof
3. **Show, don't tell** — UI footage, real result, testimonial

Avoid:
- ❌ Listing 5 features
- ❌ Founder talking-head explaining (boring)
- ❌ Abstract benefits ("amazing experience")

Prefer:
- ✅ One concrete moment of value
- ✅ Real screen/product footage (not mock)
- ✅ Single specific outcome ("from 47 emails to 3 in 5 minutes")

## CTA section (last 1-3s)

Single CTA. Specific. Visible.

- ✅ "הירשמו עכשיו — קישור בביו"
- ✅ "הקליקו על השלט"
- ✅ "תנסו חינם 7 ימים"
- ❌ "Learn more" (too vague)
- ❌ "Visit our website" (too friction)
- ❌ Two CTAs ("Sign up OR call us")

## Veo 3 prompt template (8s ad)

Veo needs visual + audio direction. Format:

```
8-second ad video.

Hook (0-1s): {{visual + audio}}
Problem (1-3s): {{visual + audio}}
Solution (3-6s): {{visual + audio}}
CTA (6-8s): {{visual + on-screen text + audio}}

Style: {{cinematic / handheld / studio}}
Aspect ratio: {{16:9 / 9:16}}
Voiceover: {{language, tone}}
On-screen text: {{specific words, only at CTA}}
```

**Example (8s teacher training ad, vertical):**
```
8-second ad video, vertical 9:16.

Hook (0-1s): Close-up on a frustrated teacher's face, classroom blurred behind. Sound: kids talking loudly.
Problem (1-3s): Wide shot — a chaotic classroom, teacher trying to control. Hebrew voiceover: "כיתה חדשה, ואין לך כלים?"
Solution (3-6s): Same teacher 3 weeks later, calm classroom, students engaged. Voiceover: "קורס מדריכים שמשנה את הכיתה."
CTA (6-8s): On-screen Hebrew text: "הירשמו עכשיו" with brand logo. Voiceover: "קישור בתיאור."

Style: cinematic, warm color grade, gentle motion blur on transitions
```

## Runway Gen-4 prompt template (vertical reels)

Runway loves *camera direction*. Be specific.

```
{{Subject in frame}}, {{action}}, {{camera move}}, {{environment}}
```

**Example:**
```
Young woman teacher in classroom, smiling, slow zoom in to her face,
warm sunlight from window, blue uniform, vertical 9:16, 720p
```

## Captions / on-screen text

- **Burn in via Captions.ai** — automatic timing + Hebrew support
- Keep on-screen text **3-5 words max per cut**
- Bold sans-serif (Heebo, Assistant) for Hebrew
- Contrast: white text + dark stroke (or vice versa)
- Position: lower-third (avoid mouth area on talking-head)

## Multiple variant strategy (A/B)

For testing, vary the **hook** mainly:
- Same problem/solution/CTA
- 5 different hooks
- Same length, same voice

Then test the **best hook** against:
- Different problem framing
- Different solution proof
- Different CTA wording

## Quality checklist

- [ ] Hook in first 1.5 seconds
- [ ] Single, specific CTA at end
- [ ] Problem is concrete (not abstract)
- [ ] Solution is shown, not just told
- [ ] No 2nd CTA mid-ad
- [ ] Captions burned in (mobile autoplay-mute)
- [ ] Aspect ratio matches placement
- [ ] Audio is intentional (not generic music bed)
- [ ] Hebrew voiceover natural (not robotic)

## Anti-patterns

- ❌ Slow start ("Hello, today we'll talk about...") → instant swipe
- ❌ Founder face for 30s → 90% drop-off by 5s
- ❌ Generic stock footage → no emotional pull
- ❌ Multiple CTAs / multiple offers in one ad → diluted conversion
- ❌ No captions → autoplay-mute kills retention
- ❌ Audio-dependent humor → fails when muted

---
title: "fullstack-il"
type: "skill"
tags: ["kit","skill"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:12:16.024355+00:00"
id: "8ae9398d-2609-4050-acd0-4c051d26a6ca"
---

> Israeli Fullstack Guidelines - Next.js 15, Tailwind v4, RTL, Hebrew. Use for any web development task.

# Fullstack IL - Hebrew Web Development Guidelines

## כיצד להשתמש

Skill זה מכיל הנחיות לפיתוח web בעברית. קרא את הקבצים הרלוונטיים לפי הצורך:

## קבצים זמינים

| קובץ | תוכן |
|------|------|
| **DESIGN.md** | מערכת עיצוב - Spacing, Typography, Colors, RTL, Palettes |
| **CHARTS.md** | גרפים - recharts patterns, RTL dashboards |
| **COMPONENTS.md** | קומפוננטות מורכבות - Toasts, Pagination, Alerts |
| **API.md** | Backend - Server Actions, Supabase, Validation |
| **SECURITY.md** | אבטחה - Auth, RLS, Headers |
| **MOBILE.md** | Expo & React Native |
| **CONTENT.md** | כתיבת תוכן בעברית |
| **WORKFLOWS.md** | n8n אוטומציות |
| **OPTIMIZATION.md** | ביצועים - Web Vitals, Caching |
| **FEATURES.md** | פיצ'רים נפוצים |
| **DEVTOOLS.md** | כלי פיתוח |
| **UI-UX-PRO-MAX.md** | שכבת UI/UX מתקדמת למשימות מורכבות |

## כללי ברזל

1. **RTL First** - כל עיצוב מתחיל מימין לשמאל
2. **Mobile First** - responsive design תמיד
3. **TypeScript Strict** - אין `any`, אין `console.log`
4. **Gap Over Margin** - Parent שולט על ריווח

## Stack

- Next.js 15 App Router
- Tailwind CSS v4
- Supabase (Auth + Database)
- TypeScript 5
- Shadcn/ui + Radix (⚠️ חייב DirectionProvider ל-RTL! ראה DESIGN.md 7.5)

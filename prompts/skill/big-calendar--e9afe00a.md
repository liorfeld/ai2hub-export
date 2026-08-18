---
title: "big-calendar"
type: "skill"
tags: ["kit","skill","big calendar","react-big-calendar","לוח שנה","calendar component","scheduling ui","event calendar"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "e9afe00a-f745-4c7c-82f0-ae1f4fedb0fa"
---

> React Big Calendar patterns for Hebrew RTL scheduling UIs - לוח שנה, אירועים, drag-and-drop, resource scheduling ב-Next.js 15. Triggers: "big calendar", "react-big-calendar", "לוח שנה", "calendar component", "scheduling UI", "event calendar".

# BIG-CALENDAR.md - React Big Calendar for Hebrew RTL Scheduling

ספרייה: [bigcalendar/react-big-calendar](https://github.com/bigcalendar/react-big-calendar) (לשעבר jquense) — v1.20.0+, תומכת React 16.14–19.

**מתי להשתמש:** לוח שנה מלא עם תצוגות חודש/שבוע/יום/סדר-יום, גרירת אירועים, תזמון משאבים.
**מתי לא:** Date picker בודד → `/components` (shadcn Calendar). אינטגרציה עם Google Calendar → `/gws`.

## Setup

```bash
pnpm add react-big-calendar date-fns
pnpm add -D @types/react-big-calendar
```

✅ אומת: `react-big-calendar@1.20.0` + `@types/react-big-calendar@1.16.3` + React 19 — `tsc --noEmit` נקי ב-strict.

---

# 1. BASIC SETUP

## 1.1 Localizer עברי (חובה מחוץ לקומפוננטה!)

```tsx
'use client'
import { Calendar, dateFnsLocalizer, Views, type View } from 'react-big-calendar'
import { format, parse, startOfWeek, getDay } from 'date-fns'
import { he } from 'date-fns/locale'
import 'react-big-calendar/lib/css/react-big-calendar.css'

// ✅ מחוץ לקומפוננטה — StrictMode מרנדר פעמיים, יצירה בפנים = re-mount של כל הלוח
const localizer = dateFnsLocalizer({
  format,
  parse,
  // שבוע ישראלי מתחיל ביום ראשון (weekStartsOn: 0)
  startOfWeek: () => startOfWeek(new Date(), { weekStartsOn: 0 }),
  getDay,
  locales: { he },
})
```

```tsx
// ❌ אסור — localizer בתוך הקומפוננטה
function BadCalendar() {
  const localizer = dateFnsLocalizer({ ... }) // נוצר מחדש בכל רנדר!
}
```

## 1.2 Hebrew Messages (תרגום מלא ל-UI)

הספרייה לא כוללת עברית — חייבים לספק `messages`:

```tsx
import type { Messages } from 'react-big-calendar'

export const hebrewMessages: Messages = {
  allDay: 'כל היום',
  previous: 'הקודם',
  next: 'הבא',
  today: 'היום',
  month: 'חודש',
  week: 'שבוע',
  work_week: 'שבוע עבודה',
  day: 'יום',
  agenda: 'סדר יום',
  date: 'תאריך',
  time: 'שעה',
  event: 'אירוע',
  noEventsInRange: 'אין אירועים בטווח זה',
  showMore: (total) => `+${total} נוספים`,
}
```

## 1.3 קומפוננטה בסיסית — RTL + גובה

```tsx
interface CalEvent {
  id: string
  title: string
  start: Date
  end: Date
  allDay?: boolean
  resource?: string
}

export function HebrewCalendar({ events }: { events: CalEvent[] }) {
  return (
    // ✅ גובה מפורש חובה! בלי גובה — הלוח לא נראה (default: height 100%)
    <div dir="rtl" className="h-[600px] p-4">
      <Calendar<CalEvent>
        localizer={localizer}
        events={events}
        culture="he"
        rtl                       // ✅ הופך layout מימין לשמאל
        messages={hebrewMessages}
        startAccessor="start"
        endAccessor="end"
        defaultView={Views.MONTH}
      />
    </div>
  )
}
```

**3 דברים שאסור לשכוח:**

| חובה | למה |
|------|-----|
| `rtl` | הופך את כיוון הימים/עמודות |
| `culture="he"` | שמות ימים/חודשים בעברית מה-locale |
| גובה מפורש על העטיפה | בלי זה הלוח קורס ל-0px |

---

# 2. NEXT.JS 15 INTEGRATION

## 2.1 Client Component בלבד

```tsx
'use client'  // ✅ חובה — הספרייה ניגשת ל-DOM + CSS import

import 'react-big-calendar/lib/css/react-big-calendar.css'
```

ה-CSS import חייב להיות בקומפוננטת client, לא ב-layout.tsx.

## 2.2 Dynamic Import (אופציונלי — code splitting)

```tsx
// app/schedule/page.tsx (Server Component)
import dynamic from 'next/dynamic'

const ScheduleCalendar = dynamic(() => import('@/components/ScheduleCalendar'), {
  ssr: false,
  loading: () => <div className="h-[600px] animate-pulse rounded-lg bg-muted" />,
})

export default async function SchedulePage() {
  const events = await getEvents() // fetch בשרת
  return <ScheduleCalendar events={events} />
}
```

⚠️ אירועים מהשרת מגיעים כ-string — להמיר ל-`Date` לפני העברה ללוח:

```tsx
const parsed = raw.map((e) => ({ ...e, start: new Date(e.start), end: new Date(e.end) }))
```

## 2.3 Controlled State (view + date)

```tsx
const [view, setView] = useState<View>(Views.MONTH)
const [date, setDate] = useState(new Date())

<Calendar
  view={view}
  onView={setView}
  date={date}
  onNavigate={setDate}
  ...
/>
```

✅ תמיד controlled — אחרת ניווט "הקודם/הבא" לא ישרוד re-render מה-parent.

---

# 3. TAILWIND V4 THEMING

## 3.1 קובץ CSS ייעודי (כלל #4 — globals.css = תוכן עניינים)

```css
/* app/styles/calendar.css */

.rbc-calendar {
  font-family: inherit;
  --rbc-event-bg: var(--primary);
  --rbc-event-fg: var(--primary-foreground);
  --rbc-today-bg: var(--accent);
  --rbc-border: var(--border);
}

/* אירוע — padding מינימלי + radius */
.rbc-event {
  background-color: var(--rbc-event-bg);
  color: var(--rbc-event-fg);
  border: none;
  border-radius: calc(var(--radius) - 2px);
  padding: 2px 8px; /* תוכן לא נוגע בבורדר */
}

.rbc-today {
  background-color: var(--rbc-today-bg);
}

.rbc-month-view,
.rbc-time-view,
.rbc-agenda-view {
  border-color: var(--rbc-border);
  border-radius: var(--radius);
}

.rbc-header {
  padding: 8px 4px;
  font-weight: 600;
}

/* Toolbar — כפתורים עם touch target תקין */
.rbc-toolbar button {
  min-height: 44px;
  padding: 8px 16px;
  border-color: var(--rbc-border);
  border-radius: calc(var(--radius) - 2px);
}

.rbc-toolbar button.rbc-active {
  background-color: var(--primary);
  color: var(--primary-foreground);
}
```

```css
/* app/globals.css */
@import './styles/calendar.css';
```

❌ אסור צבעים hard-coded ב-`eventPropGetter` — רק CSS vars מה-theme.

---

# 4. INTERACTIONS

## 4.1 בחירת אירוע + בחירת slot (יצירת אירוע)

```tsx
<Calendar<CalEvent>
  selectable
  onSelectEvent={(event) => openEventPanel(event)}        // קליק על אירוע קיים
  onSelectSlot={({ start, end }) => openCreatePanel(start, end)} // גרירה על זמן ריק
  ...
/>
```

💡 פתיחת עריכה/יצירה → Side Panel (`/side-panel`), לא Modal.

## 4.2 צביעת אירועים לפי קטגוריה

```tsx
const eventPropGetter = useCallback((event: CalEvent) => ({
  className: `rbc-event--${event.resource ?? 'default'}`, // ✅ class, לא style inline
}), [])

<Calendar eventPropGetter={eventPropGetter} ... />
```

```css
/* app/styles/calendar.css */
.rbc-event--meeting { background-color: var(--chart-1); }
.rbc-event--task    { background-color: var(--chart-2); }
.rbc-event--holiday { background-color: var(--chart-3); }
```

## 4.3 שעות עבודה + גלילה אוטומטית

```tsx
<Calendar
  min={new Date(0, 0, 0, 7, 0)}        // תצוגת שבוע/יום מתחילה ב-07:00
  max={new Date(0, 0, 0, 20, 0)}       // ונגמרת ב-20:00
  scrollToTime={new Date(0, 0, 0, 8, 0)} // גלילה ל-08:00 ב-mount (פעם אחת בלבד!)
  step={30}                             // רזולוציית slot: 30 דק'
  timeslots={2}                         // 2 slots לשעה
  ...
/>
```

⚠️ `scrollToTime` עובד רק ב-mount הראשון — שינוי prop לא יגלול מחדש.

---

# 5. DRAG & DROP

```tsx
'use client'
import withDragAndDrop from 'react-big-calendar/lib/addons/dragAndDrop'
import 'react-big-calendar/lib/css/react-big-calendar.css'
import 'react-big-calendar/lib/addons/dragAndDrop/styles.css' // ✅ CSS נוסף חובה

const DnDCalendar = withDragAndDrop<CalEvent>(Calendar)

export function DraggableCalendar({ events, onUpdate }: Props) {
  return (
    <div dir="rtl" className="h-[600px] p-4">
      <DnDCalendar
        localizer={localizer}
        events={events}
        culture="he"
        rtl
        messages={hebrewMessages}
        resizable
        onEventDrop={({ event, start, end }) => onUpdate(event.id, start, end)}
        onEventResize={({ event, start, end }) => onUpdate(event.id, start, end)}
        draggableAccessor={(event) => !event.allDay} // אילו אירועים ניתנים לגרירה
      />
    </div>
  )
}
```

עדכון אופטימי + שרת:

```tsx
const onUpdate = async (id: string, start: Date, end: Date) => {
  setEvents((prev) => prev.map((e) => (e.id === id ? { ...e, start, end } : e))) // אופטימי
  const ok = await updateEventAction(id, start, end) // Server Action
  if (!ok) revertAndToast()
}
```

---

# 6. RESOURCE SCHEDULING (חדרים / אנשים)

```tsx
const resources = [
  { id: 'room-a', title: 'חדר ישיבות א׳' },
  { id: 'room-b', title: 'חדר ישיבות ב׳' },
]

<Calendar
  defaultView={Views.DAY} // resources עובד בתצוגות day/week
  resources={resources}
  resourceIdAccessor="id"
  resourceTitleAccessor="title"
  resourceAccessor="resource"  // השדה באירוע שמצביע על המשאב
  ...
/>
```

---

# 7. MOBILE / RESPONSIVE

תצוגת חודש צפופה מדי במובייל — ברירת מחדל לפי רוחב מסך:

```tsx
'use client'
import { useEffect, useState } from 'react'

export function ResponsiveCalendar({ events }: Props) {
  const [view, setView] = useState<View>(Views.MONTH)

  useEffect(() => {
    const mq = window.matchMedia('(max-width: 640px)')
    const apply = () => setView(mq.matches ? Views.AGENDA : Views.MONTH)
    apply()
    mq.addEventListener('change', apply)
    return () => mq.removeEventListener('change', apply)
  }, [])

  return (
    <div dir="rtl" className="h-[70dvh] sm:h-[600px] p-2 sm:p-4">
      <Calendar view={view} onView={setView} ... />
    </div>
  )
}
```

| מסך | תצוגה מומלצת |
|-----|---------------|
| < 640px | `agenda` או `day` |
| 640–1024px | `week` |
| > 1024px | `month` / `week` |

- Toolbar buttons: מינימום 44x44px (מטופל ב-CSS למעלה)
- גובה במובייל: `h-[70dvh]` (לא px קבוע)

---

# 8. DO'S ✅ / DON'TS ❌

```tsx
// ✅ localizer מחוץ לקומפוננטה
// ✅ rtl + culture="he" + messages — שלושתם ביחד
// ✅ גובה מפורש על העטיפה (h-[600px] / h-[70dvh])
// ✅ controlled view + date
// ✅ צבעים דרך className + CSS vars
// ✅ 'use client' + CSS imports בקומפוננטה

// ❌ localizer/messages כ-object literal בתוך JSX (re-render אינסופי)
// ❌ moment.js — date-fns קטן ב-~30% ועם he locale מובנה
// ❌ style inline עם hex ב-eventPropGetter
// ❌ שכחת dragAndDrop/styles.css כשמשתמשים ב-DnD
// ❌ העברת dates כ-string מהשרת בלי new Date()
```

## Troubleshooting

| בעיה | פתרון |
|------|-------|
| הלוח לא נראה בכלל | אין גובה על הקונטיינר — הוסף `h-[600px]` |
| ימים באנגלית | חסר `culture="he"` או `locales: { he }` ב-localizer |
| כפתורי toolbar באנגלית | חסר `messages={hebrewMessages}` |
| שבוע מתחיל ביום שני | `startOfWeek` בלי `weekStartsOn: 0` |
| גרירה לא עובדת | חסר `withDragAndDrop` או ה-CSS של ה-addon |
| הלוח קופץ ל-defaultView אחרי עדכון | עבור ל-controlled: `view` + `onView` |
| `Invalid Date` באירועים | dates הגיעו כ-string מה-API — המר ל-`Date` |

## Related Skills

- `/design` — spacing, tokens, RTL foundation
- `/components` — Side Panel לעריכת אירוע, date pickers
- `/mobile` — breakpoints מלאים
- `/gws` — סנכרון מול Google Calendar (MCP)
- `/api` — Server Actions לשמירת אירועים

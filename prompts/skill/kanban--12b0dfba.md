---
title: "kanban"
type: "skill"
tags: ["kit","skill","kanban","קנבן","לוח שיבוץ","לוח משימות","dnd-kit","גרירת כרטיסים"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:57:24.223099+00:00"
id: "12b0dfba-0dcd-4a84-a5c4-a67db40c248c"
---

> Kanban dispatch board patterns — multi-container drag-and-drop עם @dnd-kit, שיבוץ בגרירה, סדר מתמיד (order_index), עמודות לפי עובד/סטטוס, RTL עברית. מבוסס על לוח הניקיון של PMS (לוח ניקיון בפרודקשן). Triggers: "kanban", "קנבן", "לוח שיבוץ", "לוח משימות", "drag and drop board", "dnd-kit", "גרירת כרטיסים", "dispatch board", "לוח ניקיון".

# KANBAN.md — לוח קנבן/שיבוץ עם גרירה (dnd-kit)

**מקור אמת:** לוח הניקיון של PMS — `pms/app/(dashboard)/housekeeping/page.tsx` (~1100 שורות, production).
זהו מתכון לשחזור אותה יכולת בכל פרויקט, עם התאמת נראות לפרויקט הנוכחי.

**מתי להשתמש:** לוח שיבוץ/משימות עם גרירה בין עמודות (עובדים, סטטוסים, שלבים) + סדר ידני מתמיד.
**מתי לא:** רשימה ממוינת בלי גרירה → טבלה רגילה. לוח זמנים על ציר זמן → `/big-calendar`.

## 0. התאמה לפרויקט הנוכחי (חובה לפני כתיבת קוד!)

הסקיל הזה מתאר **מבנה ולוגיקה** — הנראות תמיד של הפרויקט שבו אתה רץ:

```bash
grep -roh "bg-\[#[0-9a-fA-F]*\]" app/ components/ src/ 2>/dev/null | sort | uniq -c | sort -rn | head   # פלטת צבעים אמיתית
ls components/shared/ components/ui/ src/components/ 2>/dev/null   # קומפוננטות קיימות
cat DESIGN.md CLAUDE.md 2>/dev/null | head -50                     # שפה עיצובית
```

- יש `SidePanel`/`Icon`/`DateInput` או מקבילות? → להשתמש בהן, לא להמציא.
- צבע primary של הפרויקט מחליף את ה-`#1e40af` שבדוגמאות. טוקנים (`bg-card`, `text-muted-foreground`, `bg-accent`) נשארים.
- מיפוי ישויות: "עובד ניקיון" → הישות המקבצת של הפרויקט (נציג/שלב/סטטוס), "חדר" → הכרטיס, "checkout_time" → שדה המיון הטבעי.

## 1. Stack

```bash
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

עמוד `"use client"` יחיד + Server Actions. אין ספריית kanban — dnd-kit בלבד (Tier 3 בקיט).

## 2. מודל הנתונים

```ts
// סטטוסים + טריגר מקור — להתאים לדומיין
type Status = "pending" | "in_progress" | "done" | "skipped"

interface Task {
  id: string
  assigned_to: string | null        // null = לא משויך
  status: Status
  order_index: number               // הסדר הידני המתמיד בתוך עמודה
  // שדות מיון טבעי + תצוגה לפי הדומיין (תאריך/שעה/מספר חדר/שם אורח...)
}

interface Board {
  groups: GroupSummary[]                 // "עובדים" — העמודות
  byGroup: Record<string, Task[]>        // ממוין לפי order_index מהשרת
  unassigned: Task[]
}
```

**עקרון order_index:** השרת הוא מקור האמת לסדר. כל עמודה ממוינת `ORDER BY assigned_to NULLS FIRST, order_index`.

**מיון טבעי (ברירת מחדל לפני גרירה ידנית):** תאריך → שעה → מזהה עם `localeCompare(..., { numeric: true })`:

```ts
function compareBySort(a: Task, b: Task): number {
  const ad = a.date || "9999-12-31"; const bd = b.date || "9999-12-31"
  if (ad !== bd) return ad.localeCompare(bd)
  const at = a.time || "99:99:99"; const bt = b.time || "99:99:99"
  if (at !== bt) return at.localeCompare(bt)
  return (a.label ?? "").localeCompare(b.label ?? "", undefined, { numeric: true })
}
```

## 3. חוזה Server Actions (6 פעולות)

| פעולה | חתימה | הערות |
|-------|-------|-------|
| `getBoard` | `(tenantId, date) → Board` | שאילתה אחת עם JOIN, קיבוץ בצד השרת |
| `assign` | `(tenantId, taskId, groupId \| null)` | אחרי שיוך — **חישוב order_index מחדש לכל דלי היעד** לפי מיון טבעי |
| `reorder` | `(tenantId, groupId \| null, orderedIds[])` | `SET order_index = i+1` לכל id לפי הסדר החדש |
| `setStatus` | `(tenantId, taskId, status)` | מעדכן גם `started_at`/`completed_at` |
| `updateNotes` / `delete` / `createManual` | — | פאנל עריכה + יצירה ידנית |

כל פעולה מסוננת `tenant_id` (multi-tenant) ובודקת הרשאה בצד שרת.

## 4. מנוע הגרירה — 7 הכללים שנלמדו בדם

זה הלב של הסקיל. multi-container sortable ב-dnd-kit שביר — אלה הפתרונות שעובדים ב-production:

**(1) שלושה סנסורים — עכבר, מגע, מקלדת (חובה!):** `PointerSensor` לבד שובר מובייל (סוויפ גורר במקום לגלול) ונגישות (אין גרירת מקלדת):
```ts
const sensors = useSensors(
  useSensor(MouseSensor,    { activationConstraint: { distance: 8 } }),        // קליק ≠ גרירה
  useSensor(TouchSensor,    { activationConstraint: { delay: 250, tolerance: 5 } }), // long-press גורר, סוויפ גולל
  useSensor(KeyboardSensor, { coordinateGetter: sortableKeyboardCoordinates }),
)
```
ולצידם `accessibility` על ה-DndContext — הוראות + הכרזות בעברית לקורא מסך:
```ts
accessibility={{
  screenReaderInstructions: { draggable: "לחץ רווח להרמה, חצים להזזה, רווח לשחרור, Escape לביטול." },
  announcements: {
    onDragStart: ({ active }) => `הרמת את ${taskLabel(active.id)}`,
    onDragEnd:   ({ active, over }) => over ? `${taskLabel(active.id)} שובץ אל ${containerLabel(over.id)}` : undefined,
    // onDragOver / onDragCancel באותה תבנית
  },
}}
```

**(2) Collision detection יציב** — `pointerWithin` (הצבע פיזית בתוך ה-droppable) עם fallback ל-`closestCenter` כשהצבע ברווחים:
```ts
const stableCollision: CollisionDetection = useCallback((args) => {
  const within = pointerWithin(args)
  return within.length > 0 ? within : closestCenter(args)
}, [])
```

**(3) Refs למקור וליעד — לא state!** עד ש-`handleDragEnd` רץ, React אולי לא commit-ה את ה-setBoard האופטימי מ-`handleDragOver`, ולכן חיפוש container לפי state מחזיר תשובה ישנה. שני refs פותרים את הבעיה:
```ts
const dragSourceRef = useRef<string | null>(null)  // נקבע ב-dragStart
const dragDestRef   = useRef<string | null>(null)  // נקבע בכל dragOver חוצה-עמודות
```

**(4) מהלך אופטימי ב-onDragOver** (התבנית הקנונית של dnd-kit): מזיזים את הכרטיס ב-state כבר בריחוף — כך אנימציית השחרור מחליקה במקום לקפוץ. מוסיפים **anti-bounce**: אם היעד הוא עמודת המקור בזמן שהכרטיס כבר הוזז ממנה — לחסום, אחרת הכרטיס מפמפם הלוך-חזור:
```ts
if (overContainer === dragSourceRef.current && activeContainer !== dragSourceRef.current) return
```

**(5) השהיית polling בזמן גרירה** — הלוח מתרענן כל 5 שניות; רענון באמצע גרירה דורס את ה-state האופטימי:
```ts
const dragInFlight = useRef(false)   // true ב-dragStart, false ב-finally של dragEnd/cancel
const loadBoardSafe = async () => { if (dragInFlight.current) return; await loadBoard() }
useEffect(() => { loadBoard(); const t = setInterval(loadBoardSafe, 5000); return () => clearInterval(t) }, [...])
```

**(6) Commit ב-dragEnd לפי ה-refs:**
```ts
// חוצה-עמודות: היעד האחרון מה-ref. כל commit עטוף — כשל בלי משוב = אובדן עבודה שקט!
if (lastDest && lastDest !== sourceContainer) {
  try {
    const result = await assign(tenantId, activeId, lastDest === UNASSIGNED_ID ? null : lastDest)
    if (!result.success) toast.error(`השיבוץ לא נשמר: ${result.error ?? "שגיאה"}`)
  } catch { toast.error("השיבוץ לא נשמר — בעיית תקשורת. נסה שוב.") }
  await loadBoard()   // ground truth — גם rollback אם נכשל
  return
}
// אותה עמודה: arrayMove אופטימי + persist של כל הסדר
const newList = arrayMove(list, oldIndex, overIndex)
setBoard(prev => setContainerList(prev, sourceContainer, newList))
await reorder(tenantId, sourceContainer === UNASSIGNED_ID ? null : sourceContainer, newList.map(t => t.id))
```
תמיד `loadBoard()` בסוף (ground truth), ו-`dragInFlight.current = false` ב-`finally`.

**(7) DragCancel → שחזור אמת מהשרת:** לאפס refs + `loadBoard()`.

**DragOverlay** — ease-out חלק (לא bounce — מיושן ונכשל ב-design review), ומכבד reduced-motion:
```tsx
const prefersReducedMotion = typeof window !== "undefined" &&
  window.matchMedia("(prefers-reduced-motion: reduce)").matches

<DragOverlay dropAnimation={prefersReducedMotion ? null : { duration: 220, easing: "cubic-bezier(0.22, 1, 0.36, 1)" }}>
  {activeTask ? <MiniCard task={activeTask} className={`shadow-2xl border-2 border-primary ${prefersReducedMotion ? "" : "rotate-2"}`} /> : null}
</DragOverlay>
```

## 4.5 לקחים מסבב ביקורת 5-סוכנים (חובה!)

1. **תאריכים — לעולם לא `toISOString().slice(0,10)`** לתאריך מקומי: זה UTC. בישראל "יום הבא" הופך no-op ו"היום" שגוי בין 00:00–03:00. תמיד פורמט מקומי:
   ```ts
   const toLocalIso = (d: Date) => `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,"0")}-${String(d.getDate()).padStart(2,"0")}`
   ```
2. **דיאלוג בתוך SidePanel → `createPortal(…, document.body)`**: `backdrop-blur`/transform על אב יוצרים containing block ש"כולא" `position:fixed` בתוך הפאנל. וגם: Escape ב-capture + `stopPropagation` כדי לא לסגור את הפאנל המארח; resolve ל-promise גם ב-overwrite וגם ב-unmount — אחרת ה-await נתקע לנצח.
3. **ה-container בזמן השחרור גובר על ה-hover-ref**: ה-anti-bounce מקפיא את dragDestRef בחזרה לעמודת המקור — גרירת מקלדת מבצעת commit לעמודה הלא-נכונה. ב-dragEnd: `dest = findContainer(e.over.id) ?? lastDest` (אלא אם over הוא הכרטיס הנגרר עצמו).
4. **שומר-רצף לטעינות**: `loadSeq.current++` בכל load וב-dragStart; תגובה נוחתת רק אם `seq === loadSeq.current` — אחרת החלפת-יום מהירה או poll שרץ לתוך גרירה מציירים לוח ישן.
5. **מקלדת לא יכולה לפתוח כרטיס**: dnd-kit עושה preventDefault ל-Enter/רווח (מתחיל גרירה). פתרון: כפתור `sr-only focus:not-sr-only` "פתח פרטים" בתוך הכרטיס עם stopPropagation.
6. **שגיאות שרת ל-toast — רק הודעות אצורות**: fallback של `err.message` מדליף שמות constraint/סכמה למשתמש. בשרת: `console.error` מלא + החזרת עברית גנרית.

## 5. מבנה הלייאאוט

```
┌─ Header: כותרת + DateInput + פילטרים-KPI + כפתור "משימה חדשה" ─┐
├─ UnassignedBanner (רוחב מלא, grid רספונסיבי 1→6 עמודות) ───────┤
├─ שורת עמודות (עובד לעובד, overflow-x-auto) ────────────────────┤
└─ SidePanel עריכה (נפתח בלחיצה על כרטיס) ───────────────────────┘
```

- **UnassignedBanner** — droppable רוחב-מלא עם `rectSortingStrategy`, grid `grid-cols-1 sm:2 md:3 lg:4 xl:5 2xl:6`. כולל "רצועת שחרור" מקווקוות בתחתית כדי שתמיד יהיה יעד גרירה גם כשה-grid מלא, ו-empty-state מקווקו גדול ("שחרר כאן להחזרה לשיבוץ").
- **Column** — `min-w-[280px] w-[280px] shrink-0`, `verticalListSortingStrategy`, header עם אייקון + כותרת + מונה, גוף `min-h-[140px]` עם `ring-2 ring-primary/40` בזמן `isOver`, empty-state מקווקו ("גרור חדר לכאן").
- **סטטיסטיקות עמודה** (בכותרת): מונה דחופים (אדום), מונה בעבודה (ירוק), שעת היציאה הקרובה, תג "עמוס" כש-`tasks.length >= 8`.
- כל droppable משנה מראה ב-`isOver` — פידבק ויזואלי חובה.

## 6. כרטיס (TaskCard)

`useSortable` + שורה עליונה: **נקודת דחיפות** (עבר=אדום, היום=ענבר, מחר=כחול, עתידי=אפור, בעבודה=ירוק) → ריבוע מזהה (מספר חדר) → שם/תוויות → שעת יעד `dir="ltr" tabular-nums`. שורה תחתונה: pill סטטוס צבעוני + תג "לא משויך"/"מראש".

```ts
const STATUS_VISUAL: Record<Status, { label: string; bg: string; text: string; border: string }> = {
  pending:     { label: "ממתין",  bg: "bg-amber-50 dark:bg-amber-950/20",     text: "text-amber-700 dark:text-amber-400",     border: "border-amber-200" },
  in_progress: { label: "בעבודה", bg: "bg-blue-50 dark:bg-blue-950/20",       text: "text-blue-700 dark:text-blue-400",       border: "border-blue-200" },
  done:        { label: "הושלם",  bg: "bg-emerald-50 dark:bg-emerald-950/20", text: "text-emerald-700 dark:text-emerald-400", border: "border-emerald-200" },
  skipped:     { label: "דולג",   bg: "bg-slate-100 dark:bg-slate-900/40",    text: "text-slate-600 dark:text-slate-400",     border: "border-slate-200" },
}
```

פרטים: `opacity: 0.3` בזמן גרירה, `cursor-grab active:cursor-grabbing`, `select-none`, `onClick` עם `stopPropagation` שנחסם אם `isDragging`.

## 7. Header: KPI + פילטרים מהירים + ניווט ימים

ה-KPI הם **כפתורי סינון-toggle** (לחיצה שנייה מבטלת) בתוך container מסוג segmented:
`הכל (n)` · `לא משויכים (n)` ענבר · `יציאות היום (n)` primary · `דחופים (n)` אדום.
הסינון מסנן את `unassigned` ואת כל `byGroup` יחד. `aria-pressed` על כל כפתור.

**ניווט ימים:** סדרן חי על אתמול/היום/מחר — לצד ה-`DateInput` תמיד: חץ יום-קודם (`chevron_right` ב-RTL), חץ יום-הבא (`chevron_left`), וכפתור "היום" שמופיע רק כשלא-היום. כפתורים 44x44.

**פעולה מהירה מהכרטיס:** הפעולה השכיחה (סימון "הושלם") חייבת כפתור ✓ על הכרטיס עצמו — 44x44, עם `stopPropagation` על pointerDown/touchStart/keyDown כדי לא להפעיל גרירה. 3 קליקים → קליק אחד.

## 8. פאנל עריכה + יצירה

- לחיצה על כרטיס → `SidePanel` (RTL, מצד שמאל): פרטי שהות (grid תאריכים) → select שיבוץ → בורר סטטוס (3 pills) → textarea הערות → footer עם מחק/ביטול/שמור.
- **אישורים: לעולם לא `confirm()` נייטיבי** (דיאלוג דפדפן LTR באנגלית). קומפוננטת `ConfirmDialog` משותפת + hook `useConfirm()` שמחזיר Promise<boolean> — ראה `pms/components/shared/ConfirmDialog.tsx` כרפרנס.
- שמירה בפאנל: בדוק `result.success` של כל פעולה + `toast.error` על כשל — לא שמירה עיוורת.
- שמירה חכמה: קוראים רק לפעולות שהשתנו (`status !== task.status → setStatus`, וכו').
- כפתור "משימה חדשה" → פאנל יצירה ידנית (`source_trigger: "manager_manual"`, מסומן על הכרטיס "מראש" + שם היוצר).

## 9. Guards

- **הרשאות:** `can("module", "edit")` — בלי הרשאה מציגים מסך נעילה, לא לוח לקריאה בלבד.
- **אין עמודות:** באנר ענבר "אין עובדי ניקיון מוגדרים" עם הסבר איך מוסיפים.
- **Loading:** ספינר מרכזי עד ה-fetch הראשון.
- **Error + Retry:** `getBoard` נכשל בטעינה ראשונה → מסך "טעינת הלוח נכשלה" עם כפתור נסה-שוב. נכשל ב-polling כשיש כבר לוח → באנר "אין תקשורת — מוצגים נתונים אחרונים" ושמירת ה-state הקיים.
- אסור `console.log` — בקוד המקור נשארו `console.warn("[DND]...")` לדיבוג; להסיר במימוש חדש.

## 10. צ'קליסט מימוש

1. [ ] סעיף 0 — צבעים/קומפוננטות/ישויות של הפרויקט הנוכחי
2. [ ] טבלת DB עם `assigned_to`, `status`, `order_index` + מיגרציה
3. [ ] 6 Server Actions (סעיף 3) עם סינון tenant
4. [ ] עמוד לוח: sensors + stableCollision + שלושת ה-refs (סעיף 4)
5. [ ] UnassignedBanner + Columns + TaskCard (סעיפים 5–6)
6. [ ] KPI-פילטרים + DateInput (סעיף 7)
7. [ ] SidePanel עריכה + פאנל יצירה (סעיף 8)
8. [ ] Guards (סעיף 9) + polling עם השהיה בגרירה
9. [ ] בדיקה: גרירה בין עמודות, סדר נשמר אחרי refresh, גרירה חזרה ל"לא משויך", ביטול (Esc)
10. [ ] מובייל: סוויפ גולל, long-press גורר; מקלדת: רווח+חצים מזיזים; קורא מסך מכריז בעברית
11. [ ] כשל שרת (נתק רשת) → toast שגיאה + הלוח חוזר ל-ground truth; dark mode תקין (טוקנים, לא hex)

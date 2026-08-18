---
title: "session-readiness"
type: "skill"
tags: ["kit","skill","permissions modal","first-run setup","device readiness","ask for camera/mic/gps/notifications","pwa install prompt","enable notifications"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "6e49f313-d02d-42e5-836b-6aaf84e9b3c0"
---

> >- First-session device-readiness gate for field PWAs — once per browser session it checks PWA install + GPS / notifications (with push-subscribe) / microphone / camera and prompts the user to enable whatever is missing. RTL-first, hydration-safe, dismissable. Use when a web app needs the user's device permissions granted before field work, or to add a "device & permissions" panel. Triggers: "permissions modal", "first-run setup", "device readiness", "ask for camera/mic/gps/notifications", "PWA install prompt", "enable notifications", "no devices registered".

# Session Readiness — first-run device & permission gate

A one-time-per-session modal that makes sure a field PWA actually has what it
needs: installed to the home screen and granted **location, notifications,
microphone, camera**. Built for CADPS (`apps/admin`) and reused as a persistent
Settings panel. RTL-first, Tailwind v4, no extra deps.

## Why it exists
Field agents open the app on a phone and expect GPS routing, push alerts, and
camera/mic to "just work". Browsers only grant those after an explicit user
gesture, and **granting notification permission does NOT register a push
device** — you must also create + persist a `PushSubscription`. This gate
surfaces everything that's off on first entry and walks the user through
enabling it, then gets out of the way.

## Pieces (CADPS reference implementation)
- `components/session-readiness.tsx` — the modal. Mount once inside the authed
  shell (`app-shell.tsx`, just before the provider closes).
- `lib/device-readiness.ts` — the shared brain: `queryPermission(cap)`,
  `requestCapability(cap)`, and the `CapKey`/`CapState`/`PermissionCap` types.
- `lib/use-install-prompt.ts` — a module singleton that captures the
  `beforeinstallprompt` event and exposes `useInstallPrompt()` + `isStandalone()`.
- `components/system-readiness-card.tsx` — the same checks as a persistent
  Settings card (PWA/GPS/mic/camera); notifications live in `PushToggle`.
- `lib/push.ts` — `subscribeToPush()` (permission → SW register → PushManager
  subscribe → POST to the API). Called from the notifications branch.

## How it works
1. **Once per session.** A `sessionStorage` flag (`cadps_readiness_v1`) is set on
   dismiss; the modal is skipped while it's present. Survives route changes,
   re-checks on a new session.
2. **Evaluate.** On mount it `Promise.all`s `queryPermission("location" |
   "notifications" | "microphone" | "camera")` (the Permissions API; wrapped in
   try/catch because Chrome accepts `"microphone"`/`"camera"` names that aren't in
   TS's `PermissionName` union — cast `as PermissionName`). PWA install state =
   `isStandalone()`. If everything is `"ok"`, it dismisses silently; otherwise it
   opens.
3. **Request.** Each row's button calls:
   - `install` → `promptInstall()` from the `beforeinstallprompt` singleton.
   - everything else → `requestCapability(cap)`, which fires the native prompt
     (`geolocation.getCurrentPosition` / `Notification.requestPermission` /
     `getUserMedia({audio|video})`) and returns the resulting `CapState`.
   - **Notifications also subscribe to push**: after permission is granted,
     `requestCapability("notifications")` calls `subscribeToPush()` so a device is
     actually registered. (Skipping this is the classic "no devices registered"
     bug — permission granted, but `pushManager.subscribe()` + the server POST
     never happened.)
4. **Per-capability state.** `Record<CapKey, "ok"|"missing"|"pending">`; buttons
   disable while `pending`, granted rows show a ✓.

## Gotchas baked in
- **Hydration safety.** Anything reading client-only state (e.g. a mounted flag,
  `Notification.permission`, `display-mode`) is read via `useSyncExternalStore`
  or after mount — never with a synchronous `setState` inside an effect (which
  also trips `react-hooks/set-state-in-effect`). Server and first client render
  must match.
- **`beforeinstallprompt` fires once.** Capture it in a module singleton and
  subscribe components to it; don't add per-component listeners (the event won't
  re-fire). `isStandalone()` = `matchMedia("(display-mode: standalone)")` OR iOS
  `navigator.standalone`.
- **Install needs a SW + manifest + fetch handler** to even be offered (see the
  PWA manifest + `public/sw.js`). The button only shows when `canInstall`.
- **`NEXT_PUBLIC_VAPID_PUBLIC_KEY` must be a build arg** (baked into the client
  bundle) or `subscribeToPush()` throws `push_misconfigured` and no device
  registers — symptom identical to the missing-subscribe bug.
- **RTL + i18n.** Labels come from the `readiness.*` / `pwa.install` keys in
  `lib/i18n.tsx` (en/he). The overlay is `dir`-agnostic (centered); rows use
  logical flex so they mirror correctly.
- **"speaker" isn't a permission.** Audio output needs no grant; only mic/camera/
  geolocation/notifications do.

## Reuse / extend
- **Add a capability:** add it to `CapKey`/`PermissionCap` + `PERMISSION_NAME` in
  `device-readiness.ts`, give it a `requestCapability` branch, and a `CAP_META`
  row (icon + label key) in the modal/card. Add the i18n label.
- **Persistent panel:** drop `<SystemReadinessCard/>` into a settings page; it
  shares `device-readiness.ts`, so behavior stays consistent with the modal.
- **Different gate cadence:** swap the `sessionStorage` flag for `localStorage`
  (once ever) or a versioned key (re-prompt after you add a capability).
- **Other stacks:** the logic is framework-agnostic — `device-readiness.ts` is
  plain browser APIs; only the modal/card shells are React.

## Verify
- First load in a fresh session → modal lists the off items → enabling flips each
  to ✓ → dismiss sets the session flag → no re-prompt that session.
- Notifications: enable, then a server "send test" delivers (a row exists in the
  push-subscriptions table). If it says "no devices", the subscribe step or the
  build-time VAPID key is missing.

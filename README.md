<div align="center">

# 🔔 Alertify

**Keyword based alert tones for Android notifications: pick the app, pick the keyword, get a sound you cannot miss.**

Alertify sits quietly in the background, watches every notification your phone receives, and plays a
distinctive alert the moment one matches what you asked it to watch. Your name in a group chat, an
email about an expected subject, a delivery update: if the words are there, Alertify makes sure you hear it.

[![Release](https://img.shields.io/badge/release-v3.0.0_(100)-2f6fdb?style=flat-square)](#-whats-new-in-v300)
[![Platform](https://img.shields.io/badge/Android-9.0%2B_(API_28%2B)-3ddc84?style=flat-square&logo=android&logoColor=white)](https://android.com)
[![Language](https://img.shields.io/badge/Kotlin-100%25-7f52ff?style=flat-square&logo=kotlin&logoColor=white)](https://kotlinlang.org)
[![UI](https://img.shields.io/badge/Material-Components-6750a4?style=flat-square&logo=material-design&logoColor=white)](https://material.io)
[![Billing](https://img.shields.io/badge/Google%20Play-Billing-34a853?style=flat-square&logo=google-play&logoColor=white)](#-premium--billing)
[![Size](https://img.shields.io/badge/APK-~3.0_MB-orange?style=flat-square)]()

[What's new in v3.0.0](#-whats-new-in-v300) ·
[Getting started](#-getting-started)

</div>

---

## 📦 What's new in v3.0.0

This release is a ground up refresh, both under the hood and on screen:

- 🔊 **Notification sound fix**: alerts now play reliably. In earlier builds the alert tone could silently fail to fire when a notification matched; the playback path has been reworked so a match always produces sound.
- 🎨 **Huge UI / UX overhaul**: every screen has been redesigned with Material Components, from the four slide onboarding to the drag to reorder keyword editor, range slider day scheduler, and light / dark / auto theming.

---

## ✨ Features

### 🎯 Keyword alerts

| Feature | Details |
| --- | --- |
| Per app rules | Pick any installed app, then add the keywords or key phrases Alertify should respond to |
| Match anything | Leave the keyword list empty and Alertify alerts on **every** notification from that app |
| Keyword priority | Drag to reorder keywords; the highest ranked match controls the alert, and a lower ranked keyword never interrupts one already playing |
| Auto stop | The alert stops the moment the notification is dismissed, by opening the app or swiping it away |
| Snooze | Configurable wait between consecutive replays: 15s, 30s, 1m, 2m, 5m, or 15m while the notification is still up |
| Search | Filter the full app list by name while selecting which apps to monitor |

### 🔊 Alert playback

| Feature | Details |
| --- | --- |
| Per keyword tones | Assign a different sound to every keyword within an app, not just per app |
| Per app tone | Choose any system ringtone, or a custom file (storage read is only requested when you pick a non system sound) |
| Alert type | Play on the **Notification** channel, or the **Alarm** channel: audible even when notifications are silenced |
| Volume | Per app volume slider |
| Looping | Keep the alert repeating for up to two minutes at a time so nothing is missed |

### 📅 Scheduling *(premium)*

| Feature | Details |
| --- | --- |
| Active days | Enable Alertify per weekday: Monday through Sunday |
| Active hours | Per day time range sliders, or toggle an all day switch |
| Outside schedule | Outside active windows Alertify stays silent and lets normal notification sounds through |

### 🎨 Theming

| Feature | Details |
| --- | --- |
| Light & Dark | Full dark theme with night qualified resources |
| Follow system | Automatic mode switches with the device setting |

### 🛡 Reliability

| Feature | Details |
| --- | --- |
| Foreground service | The listener runs as a foreground service so Android keeps it alive |
| Self healing | Detects a disconnected listener and offers a one tap toggle or rebind without a reboot |
| Battery aware | Prompts you to remove Alertify from battery optimization; ships OEM component permissions for Oppo and Huawei devices |

### 💳 Premium & billing

| Feature | Details |
| --- | --- |
| Dev support pack | One time in app purchase that unlocks scheduling and future exclusive features |
| Google Play Billing | Real time purchase flow with server style signature verification of purchases |

### 🧭 Onboarding & extras

| Feature | Details |
| --- | --- |
| Welcome flow | Four slide onboarding explaining how it works, with a Notification Access check before you enter the app |
| Permission light | Needs only Notification Access to function; no contacts, no location, no account |
| Rate & share | In app rating prompt and an About screen with social links |

---

## 🏗 Architecture

Alertify is a classic single activity stack built around one long lived service. The UI writes rules
into SharedPreferences; the listener reads them on every notification.

```text
                ┌─────────────────────────────────────────────┐
                │   Any app on the device posting a notice    │
                └──────────────────────┬──────────────────────┘
                                       │ onNotificationPosted()
                                       ▼
┌────────────────── NotificationListener (foreground service) ──────────────────┐
│                                                                              │
│   1. Find the AppEntity rule for the posting package (or none)               │
│   2. Match the notice text against that app's keywords,                      │
│      highest ranked keyword wins, never interrupted by a lower one           │
│   3. Consult the schedule: is today active and is "now" inside its range?    │
│   4. Play the per keyword tone, else the per app tone, on the right          │
│      channel: NOTIFICATION, or ALARM (audible even when silenced)            │
│   5. Snooze and loop while the notice is up; stop instantly on removal       │
└──────────────────────────────────────────────────────────────────────────────┘
                                       ▲
                                       │ rules, schedule, premium flag
       ┌───────────┬───────────────────┴──┬────────────────────┬─────────────┐
       │  Welcome  │       Main           │  AppSelector  →    │   Global    │
       │  4 slide  │  home: rule list,    │  AppSettings:      │  Settings:  │      ┌────────┐
       │  onboard  │  nav drawer, theme,  │  per app keywords, │  alert type,│─────▶│ About  │
       │  + access │  listener status,    │  tones, volume,    │  snooze,    │      │ rate + │
       │  check    │  battery prompt      │  loop, reorder     │  schedule,  │      │ socials│
       └───────────┴──────────────────────┴────────────────────┘  IAP       └────────┘
```

A few details worth calling out:

- **Keyword ranking is playback aware.** The service checks whether a higher ranked keyword is already sounding before a lower ranked one may start (`currentlyNotPlayingHigherRankKeyword`), so priority order actually governs what you hear, not just which entry matched first.
- **The alarm channel is the superpower.** Because alerts can route through the system alarm stream, Alertify can reach you even when your notifications are silenced or volume is zero, without ever holding the *Modify System Settings* permission.
- **The UI is dumb, the service is smart.** Screens persist plain preference entries (`AppEntity` models serialized as JSON); the listener re reads them, so no binding, broadcast, or sync layer is needed between them.
- **Failure is handled, not assumed away.** `onListenerDisconnected` triggers a rebind request, and the home screen surfaces the listener state with a manual toggle as a last resort.

---

## 📁 Project structure

This repository currently ships the compiled release build; application source is not part of the repo.

```text
Alertify/
└── Alertify.apk        # Release APK, v3.0.0 (versionCode 100), ~3.0 MB
```

Inside the APK (extracted from `classes.dex`, `AndroidManifest.xml`, and `resources.arsc`):

```text
com.nebz.alertify
├── ThemeApp                # Application class: applies the saved theme at launch
├── NotificationListener    # THE core: NotificationListenerService, playback, snooze,
│   │                       # schedule checks, channels, foreground lifetime, rebind
│   └── AlarmType           # enum: ALARM, NOTIFICATION
├── WelcomeActivity         # 4 slide onboarding (ViewPager + dots) + access check
├── MainActivity            # home: rule list, nav drawer, listener status, theme toggle
├── AppSelector             # searchable picker of installed apps (+ default SMS handling)
├── AppSettings             # per app: keywords, tone, volume, loop, alarm switch
├── KeywordReorderActivity  # drag handle based priority reordering
├── KeywordSelection*       # keyword add / edit dialogs
├── GlobalSettings          # alert type, snooze, weekday schedule sliders, IAP entry
├── AboutActivity           # socials (Instagram, Twitter, WhatsApp), share, rate
├── BillingManager          # Play Billing: SKU query, purchase, consume, verify
├── Security                # purchase signature verification
├── AppRater                # launch count based rating prompt
├── ThemeHelper             # light / dark / auto theme switching
└── AppEntity, DayEntity    # models: per app rule; per weekday schedule window
```

---

## 🧰 Tech stack

| Layer | Technology |
| --- | --- |
| Language | Kotlin (with a small Java interop surface) |
| Minimum / target SDK | API 28 (Android 9.0) / API 29 (Android 10) |
| UI | AndroidX AppCompat, RecyclerView, ViewPager, Material Components |
| Core | `NotificationListenerService`, foreground service, MediaPlayer, RingtoneManager |
| Storage | SharedPreferences (rules, schedule, premium state) |
| Monetization | Google Play Billing Library (one time consumable IAP) |
| Distribution | Direct APK via this repository |

---

## 🚀 Getting started

### Prerequisites

- An Android 9.0 (API 28) or newer device. There is nothing to compile: the repository ships a ready to install APK.

### Install

```bash
adb install Alertify.apk
# or copy Alertify.apk to the device and open it
```

> ⚠️ **Note:** the APK is signed with a debug key, so builds from different sources install as
> separate apps and will not update over each other.

### First run

1. **Walk through the four welcome slides**; the last one checks Notification Access.
2. **Grant Notification Access** when prompted. This is the only permission Alertify needs to function.
3. **Remove battery restrictions** when asked; aggressive OEM battery managers are the number one killer of notification listeners.
4. **Add your first rule**: pick an app from the home screen, type a keyword, choose a tone.

### Managing the listener

If alerts ever stop (after a reboot on some OEMs, for example), the home screen shows the listener
state. Use the in app toggle to rebind it; the app will tell you if a reboot is the only remaining option.

---

## 📲 Releases

| Release | Artifact |
| --- | --- |
| v3.0.0 (versionCode 100) | [`Alertify.apk`](Alertify.apk) in this repository |

There is no CI or build pipeline in this repository: releases here are the exported APK artifacts
themselves. To rebuild from source you would need the original project (package `com.nebz.alertify`,
compileSdk 28, Kotlin, AndroidX, Material Components, Play Billing), which is not included here.

---

## 🧭 Screens & components reference

| Screen | Purpose |
| --- | --- |
| Welcome | Onboarding slides, how it works, Notification Access gate |
| Main | List of alert rules, navigation, listener health, theme toggle |
| App Selector | Search and pick apps to monitor |
| App Settings | Keywords, per keyword tones, volume, looping, alarm switch |
| Keyword Reorder | Drag to prioritize keywords |
| Global Settings | Alert type, snooze duration, weekday schedule, premium purchase |
| About | Social links, share, rate |

<details>
<summary><strong>Full component inventory (extracted from the manifest and dex)</strong></summary>

| Component | Type | Role |
| --- | --- | --- |
| `com.nebz.alertify.ThemeApp` | Application | Theme bootstrap |
| `com.nebz.alertify.WelcomeActivity` | Activity | Launcher entry, onboarding |
| `com.nebz.alertify.MainActivity` | Activity | Home, navigation, diagnostics |
| `com.nebz.alertify.AppSelector` | Activity | App picker |
| `com.nebz.alertify.AppSettings` | Activity | Per app rule editor |
| `com.nebz.alertify.KeywordSelectionDialogActivity` | Activity | Quick keyword dialog |
| `com.nebz.alertify.KeywordReorderActivity` | Activity | Keyword priority drag list |
| `com.nebz.alertify.GlobalSettings` | Activity | Global options + scheduling + IAP |
| `com.nebz.alertify.AboutActivity` | Activity | About, socials, rate, share |
| `com.nebz.alertify.NotificationListener` | Service | Core listener, requires `BIND_NOTIFICATION_LISTENER_SERVICE` |
| `com.android.billingclient.api.ProxyBillingActivity` | Activity | Billing library aid |
| `com.nebz.alertify.BillingManager` / `Security` | Classes | Purchase flow + signature verification |
| `com.nebz.alertify.AppEntity` / `DayEntity` | Models | Per app rule / weekday window |
| `com.nebz.alertify.ThemeHelper`, `AppRater`, `AddManually` | Helpers | Theming, rating prompt, manual rule entry |

**Declared permissions:** Notification Listener binding, `FOREGROUND_SERVICE`,
`READ_EXTERNAL_STORAGE` (only for custom sound files), `com.android.vending.BILLING`,
plus OEM keep alive permissions for Oppo (`OPPO_COMPONENT_SAFE`) and Huawei (`USE_COMPONENT`).

</details>

---

## 💡 Why Alertify exists

Modern phones give every app an equal right to buzz. A message from your most important contact lands
with the same weight as a promotional ping, and Do Not Disturb silences both. Alertify restores the
asymmetry: you name the words that matter, and only those break through, with a sound of your
choosing, on a schedule of your choosing, audible even when everything else is muted. It is
notification management that runs entirely on device, needs a single permission, and collects nothing.

<div align="center">

<sub>Alertify: hear the notifications that matter, ignore the rest.</sub>

</div>

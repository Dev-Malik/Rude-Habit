# RudeHabit

**RudeHabit** is an iOS habit tracker built with SwiftUI. It helps users track daily habits with a deliberately irreverent tone: sarcastic motivation, “rude” reality checks, and accountability when habits slip.

This repository contains the **RudeHabit** iOS app, a **Widget Extension** for the Home Screen, and supporting services (Firebase, subscriptions, notifications, and optional AI-generated copy).

---

## Features

- **Habit tracking** — Create habits with schedules (weekdays), multiple habit types (**Simple**, **Time-based**, **Frequency**), targets, reminders, and completion history stored in the cloud.
- **Sarcastic daily quotes** — AI-generated short quotes (OpenAI) tuned for a sarcastic, motivational-but-roasty voice on the home screen.
- **Subscriptions** — RevenueCat powers trials, annual plans, and in-app purchase flows; free tier limits how many habits users can create (Pro unlocks unlimited habits).
- **Sign-in** — Firebase Authentication with **Sign in with Apple** and **Google Sign-In**.
- **Notifications** — Local reminders per habit plus a separate backend service that schedules follow-up notifications for missed habits (distinct from on-device reminder scheduling).
- **Support chat** — Heapchat SDK for in-app support, tied to the signed-in user when available.
- **Home Screen widgets** — Small / medium / large widgets read shared data from an App Group and refresh on a timeline policy.
- **Optional / dev UI** — Extra screens (e.g. streak stats, calendar-style views, tone slider experiments) reachable via navigation for development or future product use.

---

## Architecture

- **Pattern:** MVVM — views render state; **ViewModels** are `ObservableObject` types with `@Published` state and async work for networking and persistence.
- **Dependency injection:** Core services and view models are provided via `environmentObject` from the app entry point (`RudeHabitApp`).
- **UI:** SwiftUI with a shared **design system** (`DesignSystem`) for colors, typography, and spacing; the app defaults to **dark mode** at launch.
- **Persistence & sync:** **Firebase Firestore** stores users and habits under `users/{userId}/habits/{habitId}`; habit completions live in subcollections for scalable queries. **Firebase Auth** identifies the user.
- **Local preferences:** User defaults are accessed with `@AppStorage`, keyed by a **`UserDefaultsKeys`** enum for consistency.

---

## Repository layout

Key groups inside `RudeHabit/`:

- **`Views/`** — `ContentView` (onboarding / paywall / auth / home routing), `HomeView`, habit creation (`CreateHabit/`), onboarding, login, paywall screens, reusable components.
- **`ViewModels/`** — e.g. `DailyHabitViewModel` (habits, completions, widget export), `AuthenticationViewModel`.
- **`Models/`** — `Habit`, `HabitCompletion`, `User`, `Quote`, enums such as `HabitType` and `CompletionStatus`.
- **`Services/`** — Firebase habit/user APIs, RevenueCat, notifications (local + remote scheduling helper), OpenAI client, subscription limits.
- **`Utils/`** — Router (`Navigation`), design system, fonts, app storage keys, auth helpers.
- **`Paywall/`** — Multi-step paywall and product presentation.

---

## User flow (high level)

1. **Onboarding** — First launch can guide the user through onboarding stored in `UserDefaults`.
2. **Paywall** — Shown when onboarding is done until the user completes purchase or an active subscription is detected (RevenueCat can skip the paywall for subscribers).
3. **Authentication** — Login with Apple or Google; `UserDefaults` tracks authentication state for routing.
4. **Home** — Lists habits for the selected period, shows the daily quote card, and links to settings, habit creation/editing, and other destinations via a typed `Router` and `NavigationStack`.

---

## Data model (habits)

- **Habit types:** Simple (binary / count), time-based (multiple reminder times), frequency-based (`FrequencyConfig` with intervals and windows).
- **Scheduling:** `selectedDays` uses weekday components; empty means every day (see `shouldShowToday` in the habit model).
- **Completions:** Tracked as `HabitCompletion` records with status and optional completion time; the view model caches completions for responsiveness and uses optimistic updates where appropriate.

---

## Widget extension

- The main app writes **`widgetData.json`** into the shared App Group container **`group.com.inspiredevstudio.rudehabit`** and calls `WidgetCenter.shared.reloadAllTimelines()` after updates.
- The widget decodes that JSON into `WidgetHabitSummary` and renders **small**, **medium**, and **large** layouts.

---

## External services & configuration

To build and run, you typically need:

| Service | Used for |
|---------|----------|
| **Firebase** | Auth, Firestore, Cloud Messaging (FCM); `GoogleService-Info.plist` in the app target. |
| **RevenueCat** | Subscriptions and entitlement checks; API key configured in code (see `Constants` / app delegate). |
| **OpenAI** | Chat completions for daily sarcastic quotes; API key in `Constants` (should be secured for production — e.g. proxy, build settings, or secrets not committed). |
| **Heapchat** | Support / chat; configured in `AppDelegate`. |
| **Backend (HTTP)** | Missed-habit notification scheduling — base URL and API key live in `NotificationSchedulingService` (treat as sensitive). |

**Security note:** Rotate any keys that were ever committed to source control, move secrets to Xcode configuration or a secure backend, and never ship production apps with long-lived client-side keys in the repository.




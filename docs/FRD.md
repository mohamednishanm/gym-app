# Functional Requirements Document (FRD)

## Gym & Diet Tracker — Production Build Specification

| Field | Value |
| --- | --- |
| Document version | 2.0 (supersedes the 1.0 FRD dated 2026-11-02) |
| Date | 2026-11-02 |
| Product owner | Mohamed (single user) |
| Status | Ready for implementation |
| Target device | Samsung Galaxy M53 5G (primary), any Android 8.0+ phone (secondary) |
| Platform | Android native via React Native (Expo SDK 57, React Native 0.86) |
| Architecture | Local-first, offline, no backend in v1 |

---

## 1. Purpose and scope

### 1.1 Purpose

A personal Android app that helps one user:

1. Track body weight daily, including a carry-forward rule for missed days and a weight-loss journey.
2. Keep height and profile data used for calorie targets.
3. Log gym workouts with starting and last set weights, in pounds and kilograms.
4. See gym attendance on a monthly calendar.
5. Get Push / Pull / Legs / Cardio workout recommendations.
6. Receive a daily calorie and macro target, plan meals for each slot, and get meal suggestions that fit the remaining budget.

### 1.2 In scope (v1)

- Single local profile, no sign-in.
- All features listed in Section 5, offline.
- Android release build installable on the Samsung Galaxy M53 5G.
- Local notifications for the daily weigh-in.
- JSON export and import of all data.

### 1.3 Out of scope (v1)

- Cloud sync, accounts, multi-user, social features.
- Wearable or health-platform integration (Health Connect, Samsung Health).
- Barcode scanning and an online food database.
- iOS release (code stays cross-platform, but iOS is not tested or shipped in v1).
- Medical advice. All targets are estimates.

### 1.4 Glossary

| Term | Meaning |
| --- | --- |
| Logged weight | A weight the user entered for a specific date. |
| Carried weight | The weight shown for a date with no logged entry, taken from the most recent earlier logged date (FR-W4). |
| Daily plan | The calorie and macro targets for one local date, computed once and stored (FR-D2). |
| Split day | One of Push, Pull, Legs, Cardio, or Custom. |
| Training day | A day with at least one logged workout, or a day the user marks as a training day. |
| Meal slot | Breakfast, lunch, dinner, or snack. |
| Budget | The share of the daily calorie target assigned to one meal slot. |

---

## 2. Target device and platform constraints

### 2.1 Samsung Galaxy M53 5G reference profile

| Attribute | Value | Design implication |
| --- | --- | --- |
| SoC | MediaTek Dimensity 900 (6 nm), octa-core (2×A78 + 6×A55), Mali-G68 MC4 GPU | Avoid heavy GPU effects. Keep animations on the UI thread only where Reanimated is used. |
| RAM | 6 GB or 8 GB LPDDR4X | Keep the app's working memory under 250 MB. |
| Storage | 128 GB, expandable by microSD up to 1 TB | Export files must be writable to shared storage through the system share sheet. |
| Display | 6.7-inch FHD+ (1080 × 2400) Super AMOLED Plus, 120 Hz, punch-hole camera | Reference layout 412 × 915 dp. Respect the top cutout with safe-area insets. Dark theme saves power on AMOLED. |
| OS at launch | Android 12, One UI 4.1; upgradable to Android 13 / One UI 5.1 | Support Android 8.0 (API 26) and newer. Test on Android 12 and 13. |
| Battery | 5000 mAh, 25 W charging | Avoid background work. Use scheduled local notifications, not polling. |
| Biometrics | Side-mounted fingerprint sensor | Optional app lock in a later release. Not required in v1. |

### 2.2 Platform rules

- **minSdkVersion:** 26 (Android 8.0). **targetSdkVersion:** the value required by Google Play at release time (currently API 35 or newer). Set in `app.config.ts`, never hard-coded in native folders.
- **Architecture:** React Native New Architecture only (Fabric and TurboModules). Expo SDK 57 requires it, so no legacy-architecture code is allowed.
- **JS engine:** Hermes (default).
- **Orientation:** portrait only.
- **Density:** size everything in `dp` (React Native's density-independent units). Do not use fixed pixel sizes.
- **Font scaling:** support system font scale up to 200% without clipped text.

### 2.3 Performance budgets (measured on the M53, release build)

| Metric | Budget |
| --- | --- |
| Cold start to first interactive screen | 2.0 s or less |
| Warm start | 0.8 s or less |
| List scroll (history, calendar) | No dropped frames at 120 Hz in normal use |
| Tap-to-response | 100 ms or less |
| Local database query for any screen | 50 ms or less |
| APK size (universal, release) | 35 MB or less |
| Memory in use after 30 min of normal use | 250 MB or less |
| Battery | No measurable drain from the app when closed (no background services) |

---

## 3. Users and roles

- **Owner (only user):** full read and write access to all data on the device.
- There is no login. The app asks for a profile once at first launch (FR-O1).
- No roles, permissions, or sharing in v1.

---

## 4. Global behaviour

- **G1 Local-first:** all data is stored on the device. The app works fully in airplane mode.
- **G2 Units:** weight is stored in kilograms. Display unit (kg or lb) is a user setting. Height is stored in centimetres.
- **G3 Dates:** all dates use the device's local time zone and are stored as `YYYY-MM-DD`. A "day" starts at local midnight.
- **G4 Undo:** deletes of workouts, sets, and meal items show a 5-second undo snackbar. Hard deletion happens after the snackbar expires.
- **G5 Confirmation:** destructive actions that cannot be undone (delete all data, import with replace) require a confirmation dialog.
- **G6 Errors:** every failed write shows a message and keeps the user's input. No silent failure.
- **G7 Theme:** Light, Dark, or System (default). The Appearance setting applies immediately.

---

## 5. Functional requirements

Requirement IDs are stable. Use them in commits, tests, and pull requests.

### 5.1 Onboarding and profile (FR-O, FR-P)

**FR-O1 First launch.** If no profile exists, the app opens the First login screen (step 1 of 2). Required fields: name, age (13–100), sex (male or female), height, current weight, goal weight, activity level, goal. Continue is disabled until all required fields are valid.

**FR-O2 Activity levels.** Sedentary, Light, Moderate, Very active. These map to activity factors in Section 6.2.

**FR-O3 Goal.** Lose, Maintain, or Gain. Lose and Gain use a deficit or surplus from the Calorie setting (FR-D3).

**FR-O4 Completion.** Saving step 2 writes the profile and the first weight entry for today, then opens Home.

**FR-P1 View profile.** The Profile screen shows name, age, sex, height (cm and ft/in), goal weight (kg and lb), activity level, goal, units, appearance, and reminders.

**FR-P2 Edit profile.** The Edit button on Profile opens the Edit profile screen, pre-filled with current values. Save validates all fields. Cancel discards changes and returns to Profile.

**FR-P3 Effect of edits.** Edits to height, age, sex, activity, or goal take effect in the daily plan from the next local day (FR-D2). Edits to goal weight apply immediately to the journey progress.

**FR-P4 Units preference.** Changing kg or lb updates every display immediately. Stored values do not change.

**FR-P5 Appearance.** Light, Dark, or System. The choice is saved in settings.

### 5.2 Body weight (FR-W)

**FR-W1 Home display.** Home shows the resolved weight for today at the top in large type, with a small label that says "logged" or "carried from <date>".

**FR-W2 Edit by long press.** Long-pressing the weight value (about 400 ms) opens the Edit weight sheet for the selected date. An "Edit" text button on the same card opens the same sheet, so the action is also available to screen-reader users through an accessibility action named "Edit weight". A haptic pulse confirms the long press.

**FR-W3 Daily log.** One logged entry per date. The history screen lists dates in reverse order with weight, change from the previous resolved day, and a marker for logged or carried entries.

**FR-W4 Carry-forward rule.** When a date has no logged entry, its weight is the most recent earlier logged weight within the last 365 days. Carried values are not stored; they are derived when displayed. Entering a value for that date replaces the carried value immediately.

**FR-W5 No earlier entry.** If no logged weight exists for the date or any of the previous 365 days, the app shows "No weight yet — tap to add" and the daily plan uses the profile's current weight field.

**FR-W6 Validation.** Accepted range 20.0–400.0 kg (44.1–881.8 lb). One decimal place. Out-of-range input shows an inline error and disables Save.

**FR-W7 Weight-loss journey.** The Journey screen shows the start weight (earliest logged entry), current resolved weight, total change in kg and lb, percentage of the way from start to goal weight, a line chart of resolved weight by day for the selected range (7 days, 30 days, 90 days, all), and BMI from profile height and current weight.

**FR-W8 Progress percentage.** `progress = (start − current) / (start − goal) × 100`, clamped to 0–100. Hidden when goal weight is missing or equal to start weight.

**FR-W9 Reminder.** An optional daily local notification at a user-set time (default 07:00) asks for today's weigh-in. The notification is dismissed automatically when today has a logged entry.

### 5.3 Height (FR-H)

**FR-H1 Stored height.** Height is stored once in centimetres with one decimal place.

**FR-H2 Input.** The user enters cm, or ft and in. Range 100–250 cm (3 ft 3 in to 8 ft 2 in). Conversion: 1 in = 2.54 cm.

**FR-H3 Uses.** Height feeds BMR (FR-D1) and BMI (FR-W7).

### 5.4 Daily calorie and diet plan (FR-D)

**FR-D1 Energy target.** Calculated with the Mifflin–St Jeor equation (Section 6.1). The result is rounded to the nearest 10 kcal.

**FR-D2 Daily snapshot.** On the first app open of each local day, the app computes and stores one daily plan row for that date (BMR, TDEE, target, macros, training flag) using the profile and the resolved weight for that date. The row does not change during the day. A "Recalculate today" action in the Diet menu recomputes it on request.

**FR-D3 Deficit and surplus.** Default weight-loss deficit is 500 kcal/day (about 0.5 kg per week). The user may choose 250, 500, or 750 kcal/day for Lose. The user may choose a surplus of 250 kcal/day for Gain. Maintain has no adjustment.

**FR-D4 Floors.** The target never drops below 1,500 kcal (male) or 1,200 kcal (female). If the formula result is lower, the floor is used and the screen shows "Limited to the minimum for your sex".

**FR-D5 Macros.** Protein = 1.6 g per kg of resolved weight for Lose, 1.8 g per kg for Maintain and Gain. Fat = 25% of target kcal ÷ 9. Carbohydrate = remaining kcal ÷ 4. Rounded to 1 g. The Diet screen shows all three.

**FR-D6 Training-day adjustment.** On a training day (FR-D7), carbohydrate increases by 20 g (+80 kcal) and total target increases by the same 80 kcal. Rest days have no adjustment.

**FR-D7 Training-day rule.** A date is a training day if at least one workout is logged for it, or the user has tapped "Training day" on that date's plan. Because the daily plan is a snapshot, the training flag is set when the plan is first viewed and updated by logging a workout on the same day.

**FR-D8 Meal slots and budgets.** The target is split into four budgets by default: breakfast 25%, lunch 30%, dinner 30%, snack 15%. The user may change the split in Diet settings, provided the four shares add up to 100%.

**FR-D9 Planned meals.** Each meal slot lists planned items. An item has a name, time, calories, protein, carbohydrate, and fat. Items can be added (Add meal sheet), edited, and deleted (with undo, G4).

**FR-D10 Eaten marker.** Each planned item has an eaten toggle. The header shows two totals: **Planned** (sum of all items) and **Eaten** (sum of items marked eaten), each against the target.

**FR-D11 Over-budget warning.** If a slot's planned calories exceed its budget, the slot shows a warning. The warning does not block saving.

**FR-D12 Suggestions.** The app suggests foods from a bundled dataset (Section 18.2). A suggestion is shown only if it fits the remaining budget of at least one slot and, where possible, helps reach the remaining protein target. Each suggestion shows calories, protein, and the slot it fits. Tapping "+ Add" adds it as a planned item in that slot.

**FR-D13 Suggestion ranking.** Among fitting suggestions, rank first by protein per kcal (highest first), then by variety (do not repeat a suggestion already planned today). Show at most three.

**FR-D14 Disclaimer.** The Diet screen footer states: "Targets are estimates, not medical advice."

**FR-D15 Add-meal sheet.** Fields: meal type (breakfast, lunch, dinner, snack chips), name, calories, protein, time (default now, rounded to 5 min), quick picks from the suggestions dataset. Save is disabled until name and calories are valid (calories 1–3,000).

### 5.5 Workout tracking (FR-K)

**FR-K1 Create workout.** A workout has a date (default today; past dates allowed), name, split day (Push, Pull, Legs, Cardio, Custom), start time, end time, and optional notes. Duration is derived from start and end.

**FR-K2 Add exercise.** Exercises come from the bundled catalogue (Section 18.3) or the user's own names. The user may add, rename, reorder, or delete exercises within a workout.

**FR-K3 Sets.** Each exercise has ordered sets. A set has a weight (kg or lb entry, stored in kg), reps (0–100), and a set number. Weight accepts 0.0–500.0 kg in 0.5 kg steps (or the lb equivalent).

**FR-K4 Starting and last weight.** For each exercise, the app records the weight of the first set (starting weight) and the weight of the last set (last weight) in the most recent session. The difference is shown in kg and lb.

**FR-K5 Dual units.** Every weight and difference appears in both kg and lb. The user's preferred unit is shown first.

**FR-K6 Exercise history.** Tapping an exercise shows every past session with its sets and a line chart of starting and last weight over time. Range: 30 days, 90 days, all.

**FR-K7 Edit and delete.** Workouts, exercises, and sets can be edited and deleted. Deletes use undo (G4).

**FR-K8 Gym time.** Each workout's duration is shown. Weekly total gym time is shown on the Workouts header.

**FR-K9 Long-press to edit a set.** Long-pressing a set row (about 400 ms) opens the edit sheet for that set. The same action is available from a visible "Edit" control and as an accessibility action.

**FR-K10 Recommended next weight.** When a new session starts for an exercise, the suggested starting weight is the last weight used, shown with the note "Last time: 62.5 kg". The user chooses the weight.

### 5.6 Attendance calendar (FR-C)

**FR-C1 Month view.** Shows one month, with previous and next controls, and swipe between months.

**FR-C2 Gym day marks.** A date with at least one logged workout shows a dot in the split colour. A date with a workout whose split day is Cardio shows the cardio colour. Dates with no workout show nothing.

**FR-C3 Day detail.** Tapping a date opens its workouts with duration and exercise summaries. Tapping an empty past date offers "Log a workout for this day".

**FR-C4 Summary.** The header shows gym days this month and the current streak (FR-C5).

**FR-C5 Streak.** A streak is the number of consecutive ISO weeks (Monday to Sunday) with at least the planned number of gym days (default 3, set in Settings). The current week counts only once it ends or when the target is already met.

**FR-C6 Calendar data.** The calendar reads only indexed date ranges (Section 7.3). It must not load the whole history.

### 5.7 Workout recommendations (FR-R)

**FR-R1 Split cycle.** The recommended cycle is Push → Pull → Legs → Cardio, repeating.

**FR-R2 Advancement.** The cycle advances only when a workout is logged. The next recommended split is the one after the most recent logged split day. Rest days are not part of the cycle.

**FR-R3 Rest suggestion.** If the most recent workout was logged less than 20 hours ago, the Home screen shows "Rest or light cardio today" and keeps the same recommendation for tomorrow.

**FR-R4 Skipped days.** A skipped day does not move the cycle. Recommendation stays on the next split after the last logged one.

**FR-R5 Exercises.** The recommendation shows the split's exercises from the catalogue (Section 18.3). For each exercise with history, the suggested starting weight is the last used weight, with a note to add 2.5 kg (5 lb) when every set in the last session reached the target reps.

**FR-R6 Manual override.** The user may pick any split for today. The manual choice does not change the cycle.

**FR-R7 Explanation.** Every recommendation shows the reason in one line, for example "Legs is next after Pull".

### 5.8 Home dashboard (FR-H-HOME)

**FR-HM1 Content.** Greeting with the user's first name, today's date, large weight with its label (FR-W1), today's calorie target with planned and eaten totals, journey summary (change and percent to goal), the recommended workout card (FR-R), and a Start workout action.

**FR-HM2 Refresh.** Home refreshes whenever it gains focus and after any write.

### 5.9 Settings, data, and export (FR-S)

**FR-S1 Settings.** Units (kg, lb), appearance (System, Light, Dark), weigh-in reminder (on or off, time), workout reminder (on or off, time), planned gym days per week (1–7), calorie deficit choice (FR-D3), meal budget split (FR-D8).

**FR-S2 Export.** A single JSON file containing all data and a `schemaVersion` field, saved through the Android share sheet. The file name includes the date.

**FR-S3 Import.** Import reads a JSON file, validates it with the same schema (Section 7.4), and shows a summary (counts per table). The user chooses Replace all or Cancel. Merge is not in v1.

**FR-S4 Delete all data.** Available in Settings behind a confirmation dialog that requires typing "DELETE".

**FR-S5 About.** Shows app version, build number, and the disclaimer (FR-D14).

---

## 6. Calculation specifications

### 6.1 Energy and macros

```
BMR (male)   = 10 × weight_kg + 6.25 × height_cm − 5 × age + 5
BMR (female) = 10 × weight_kg + 6.25 × height_cm − 5 × age − 161
TDEE         = BMR × activity_factor
target       = TDEE + adjustment          (adjustment: −250/−500/−750 Lose, +250 Gain, 0 Maintain)
target       = max(target, floor)          (floor: 1500 male, 1200 female)
target       = round(target / 10) × 10
protein_g    = round(weight_kg × protein_per_kg)           (1.6 Lose, 1.8 otherwise)
fat_g        = round(target × 0.25 / 9)
carb_g       = round((target − protein_g × 4 − fat_g × 9) / 4)
```

Activity factors: Sedentary 1.2, Light 1.375, Moderate 1.55, Very active 1.725.

Training day: `target = target + 80`, `carb_g = carb_g + 20`.

### 6.2 Worked example

Input: male, 28 years, 177.8 cm, 79.4 kg, Moderate, Lose with 500 kcal deficit, goal weight 74.0 kg, not a training day.

```
BMR    = 10×79.4 + 6.25×177.8 − 5×28 + 5 = 1770.25
TDEE   = 1770.25 × 1.55 = 2744.0
target = 2744.0 − 500 = 2244.0 → round to 2240 kcal
protein = round(79.4 × 1.6) = 127 g
fat     = round(2240 × 0.25 / 9) = 62 g
carb    = round((2240 − 508 − 558) / 4) = 294 g
```

The design mockups use illustrative numbers (2,250 kcal, 128 g protein, 190 g carbohydrate). The app must compute these values, not copy them.

### 6.3 Carry-forward resolution

```
function resolveWeight(date):
  for d in [date, date−1, …, date−365]:
    if entry(d) exists: return { kg: entry(d).kg, source: d == date ? 'logged' : 'carried', fromDate: d }
  return null
```

Implementation must load the entries in one indexed range query, then walk in memory.

### 6.4 Journey and progress

```
start   = weight of earliest logged entry
current = resolveWeight(today).kg
change  = current − start
progress = clamp((start − current) / (start − goal) × 100, 0, 100)   if goal and start differ
bmi     = current / (height_m)²
```

### 6.5 Streak

Group gym days by ISO week. A week counts if its gym-day count is at least the planned days per week. The streak is the number of consecutive counted weeks ending at the last completed week, plus the current week if it already meets the target.

### 6.6 Units

```
lb = kg × 2.20462        kg = lb ÷ 2.20462
ft/in from cm: total_in = cm ÷ 2.54; ft = floor(total_in / 12); in = round(total_in mod 12)
```

Store kg and cm. Convert only for display. Round for display to 0.1 kg/lb or whole cm/in.

---

## 7. Data model

### 7.1 Storage

- **Engine:** SQLite via `expo-sqlite` (New Architecture build).
- **Access:** Drizzle ORM (`drizzle-orm/expo-sqlite`) with generated migrations committed to the repo.
- **Location:** app-private database file. Not exposed to other apps.
- **Transactions:** every multi-row write (for example, a workout with its exercises and sets) runs in one transaction.

### 7.2 Tables

```sql
CREATE TABLE profile (
  id              INTEGER PRIMARY KEY CHECK (id = 1),
  name            TEXT    NOT NULL CHECK (length(name) BETWEEN 1 AND 40),
  age             INTEGER NOT NULL CHECK (age BETWEEN 13 AND 100),
  sex             TEXT    NOT NULL CHECK (sex IN ('male','female')),
  height_cm       REAL    NOT NULL CHECK (height_cm BETWEEN 100 AND 250),
  activity_level TEXT    NOT NULL CHECK (activity_level IN ('sedentary','light','moderate','very_active')),
  goal            TEXT    NOT NULL CHECK (goal IN ('lose','maintain','gain')),
  goal_weight_kg  REAL    CHECK (goal_weight_kg IS NULL OR goal_weight_kg BETWEEN 20 AND 400),
  deficit_kcal    INTEGER NOT NULL DEFAULT 500 CHECK (deficit_kcal IN (250,500,750)),
  created_at      TEXT    NOT NULL,
  updated_at      TEXT    NOT NULL
);

CREATE TABLE weight_entries (
  date        TEXT PRIMARY KEY,                 -- YYYY-MM-DD, local
  weight_kg   REAL NOT NULL CHECK (weight_kg BETWEEN 20 AND 400),
  note        TEXT,
  updated_at  TEXT NOT NULL
);

CREATE TABLE daily_plan (
  date            TEXT PRIMARY KEY,
  bmr             REAL    NOT NULL,
  tdee            REAL    NOT NULL,
  target_kcal     INTEGER NOT NULL,
  protein_g       INTEGER NOT NULL,
  carb_g          INTEGER NOT NULL,
  fat_g           INTEGER NOT NULL,
  weight_kg       REAL    NOT NULL,            -- weight used for this snapshot
  is_training_day INTEGER NOT NULL DEFAULT 0 CHECK (is_training_day IN (0,1)),
  computed_at     TEXT    NOT NULL
);

CREATE TABLE workouts (
  id         TEXT PRIMARY KEY,                 -- ULID
  date       TEXT NOT NULL,
  name       TEXT NOT NULL,
  split_day  TEXT NOT NULL CHECK (split_day IN ('push','pull','legs','cardio','custom')),
  start_at   TEXT,
  end_at     TEXT,
  notes      TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);
CREATE INDEX idx_workouts_date ON workouts(date);

CREATE TABLE exercises (
  id          TEXT PRIMARY KEY,
  workout_id  TEXT NOT NULL REFERENCES workouts(id) ON DELETE CASCADE,
  catalogue_id TEXT,
  name        TEXT NOT NULL,
  position    INTEGER NOT NULL
);
CREATE INDEX idx_exercises_workout ON exercises(workout_id);
CREATE INDEX idx_exercises_name ON exercises(name);

CREATE TABLE sets (
  id           TEXT PRIMARY KEY,
  exercise_id  TEXT NOT NULL REFERENCES exercises(id) ON DELETE CASCADE,
  set_number   INTEGER NOT NULL CHECK (set_number >= 1),
  weight_kg    REAL    NOT NULL CHECK (weight_kg BETWEEN 0 AND 500),
  reps         INTEGER NOT NULL CHECK (reps BETWEEN 0 AND 100),
  UNIQUE (exercise_id, set_number)
);

CREATE TABLE meal_items (
  id          TEXT PRIMARY KEY,
  date        TEXT NOT NULL,
  slot        TEXT NOT NULL CHECK (slot IN ('breakfast','lunch','dinner','snack')),
  name        TEXT NOT NULL,
  kcal        INTEGER NOT NULL CHECK (kcal BETWEEN 1 AND 3000),
  protein_g   REAL NOT NULL DEFAULT 0,
  carb_g      REAL NOT NULL DEFAULT 0,
  fat_g       REAL NOT NULL DEFAULT 0,
  planned_time TEXT,                           -- HH:MM
  eaten       INTEGER NOT NULL DEFAULT 0 CHECK (eaten IN (0,1)),
  suggestion_id TEXT,
  created_at  TEXT NOT NULL,
  updated_at  TEXT NOT NULL
);
CREATE INDEX idx_meal_items_date ON meal_items(date);

CREATE TABLE settings (
  key   TEXT PRIMARY KEY,
  value TEXT NOT NULL                          -- JSON-encoded
);

CREATE TABLE schema_meta (                      -- written by migrations
  version INTEGER NOT NULL
);
```

Static reference data (catalogue, food suggestions) is bundled as TypeScript constants, not stored in tables, so it is versioned with the code.

### 7.3 Query rules

- Every screen reads a bounded range: calendar by month, history by 90-day pages, journey by the chosen range.
- Writes return the affected row IDs so only the affected queries refresh.
- No query may use `SELECT *` on workouts with unbounded sets joins. Load sets per exercise id in one batched `IN (...)` query.

### 7.4 Export format (v1)

```json
{
  "schemaVersion": 1,
  "exportedAt": "2026-11-02T09:00:00Z",
  "profile": { },
  "weightEntries": [ { "date": "2026-11-02", "weightKg": 79.4, "note": null } ],
  "workouts": [ { "id": "...", "date": "...", "name": "...", "splitDay": "push",
                  "exercises": [ { "name": "Bench press", "sets": [ { "setNumber": 1, "weightKg": 60.0, "reps": 8 } ] } ] } ],
  "mealItems": [ ],
  "settings": { }
}
```

Import validates every field against a Zod schema generated from the same constraints as Section 7.2 and rejects the whole file on the first invalid row.

---

## 8. Architecture

### 8.1 Layers

```
UI (screens, components)            React Native, Expo Router
   ↓ hooks only
Application hooks & view models     useToday(), useWeightJourney(), useDailyPlan() …
   ↓
Domain (pure TypeScript, no I/O)    calories.ts, weightResolution.ts, progress.ts,
                                     splitRecommendation.ts, streak.ts, units.ts,
                                     suggestions.ts
   ↓
Repositories (I/O boundary)         weightRepo, workoutRepo, mealRepo, profileRepo …
   ↓
Persistence                          Drizzle + expo-sqlite
```

Rules:

- Domain code imports nothing from React, React Native, or the database.
- Screens never call the database directly. They use hooks.
- Every domain function is a pure function with unit tests.

### 8.2 Project structure (feature-based)

```
gym-diet-tracker/
  app/                      Expo Router routes (file-based)
    (onboarding)/           first-login flow
    (tabs)/                 Home, Workouts, Calendar, Diet, Profile
    modal/                  edit-weight, add-meal, edit-profile, edit-set
    _layout.tsx
  src/
    features/
      weight/               components, hooks, screens-helpers
      workouts/
      calendar/
      diet/
      recommendations/
      profile/
      settings/
    domain/                 pure logic (calories, weight, split, units, streak)
    data/
      db/                   client, schema.ts, migrations/
      repositories/
      seed/                 catalogue.ts, suggestions.ts
    shared/
      ui/                   Button, Card, Chip, Sheet, Field, Text …
      theme/                tokens (color, type, spacing, radius), light & dark
      hooks/                useLongPress, useHaptics …
      lib/                  dates, formatting, errors
  test/                     unit, integration, e2e (Maestro flows)
  app.config.ts
  eas.json
```

### 8.3 Design tokens

Tokens come from the Claude Design canvas (see `design/`). Implement them as one theme object used by both themes.

| Token | Light | Dark |
| --- | --- | --- |
| background | `#F7F7F5` | `#121316` |
| surface | `#FFFFFF` | `#1C1E23` |
| surfaceAlt | `#EFEFEB` | `#2A2D35` |
| border | `#E6E6E2` | `#2A2D35` |
| textPrimary | `#1A1A1A` | `#F2F4F7` |
| textSecondary | `#8A8A85` | `#9AA0AC` |
| accent | `#2BAE84` | `#A7E8D2` |
| accentOn | `#FFFFFF` | `#0F1412` |
| danger | `#D64545` | `#FF8A8A` |

Spacing scale (dp): 4, 8, 12, 16, 20, 24, 32. Radius: 12 (inputs), 16 (rows), 20 (cards), 24 (sheets). Type scale: display 72/300, title 24/600, section 15/600, body 15/400, caption 12/500. Font: system default (Roboto on the M53).

---

## 9. Technology stack and rationale

The stack is chosen to be the most efficient option for a single-device, offline, data-heavy personal app.

| Concern | Choice | Why |
| --- | --- | --- |
| Framework | **Expo SDK 57** with **React Native 0.86** | Current release. Expo SDK 55 onward runs only on the New Architecture, which gives faster native calls and smaller overhead. Expo handles native builds, so no Android Studio project edits are needed. |
| Language | **TypeScript** in `strict` mode | Catches unit and data-shape mistakes at build time. |
| Navigation | **Expo Router** (file-based) | Standard in current Expo. Deep links and typed routes. |
| Local data | **expo-sqlite** with **Drizzle ORM** | Real relational queries and indexes. Fast on the M53. Typed schema and migrations. No backend needed. |
| Validation | **Zod** | Same schema for forms, export, and import. |
| UI state | **Zustand** | Small store for transient UI state (theme, open sheets). Not used for database data. |
| Server-style data hooks | **live-query hooks** from Drizzle, with `useMemo` on derived values | Keeps screens in sync with writes without a second cache layer. |
| Lists | **@shopify/flash-list** | Recycles cells. Smooth history and calendar scrolling on the M53. |
| Charts | A New-Architecture-compatible chart library (for example Victory Native XL on react-native-skia, or react-native-gifted-charts) | Line charts for weight and set history. Pick the one that passes the performance budget in Section 2.3. |
| Animation | **react-native-reanimated** | Runs on the UI thread. Used for sheet and long-press feedback. |
| Gestures | **react-native-gesture-handler** | Reliable long press and swipe between months. |
| Haptics | **expo-haptics** | Confirms long press and saves. |
| Notifications | **expo-notifications** (local, scheduled) | No server. Battery-friendly. |
| Export and import | **expo-file-system**, **expo-sharing**, **expo-document-picker** | Standard Android storage access without extra permissions. |
| Dates | **date-fns** (tree-shaken) | Small, immutable, local-time safe. |
| Icons | **lucide-react-native** or Expo vector icons | Consistent set, tree-shakable. |
| Language | TypeScript, ESLint (flat config), Prettier | Industry standard. |
| Unit tests | **Jest** with **jest-expo**, **React Native Testing Library** | Official Expo setup. Domain logic tested without a device. |
| Property tests | **fast-check** | Carry-forward and calorie rules tested across many inputs. |
| Database tests | In-memory SQLite through the same Drizzle schema | Real SQL, no mocks for persistence. |
| End-to-end tests | **Maestro** flows on an emulator with an M53-sized profile, plus manual test on a real M53 | Simple YAML flows. Works with Android. |
| Build and release | **EAS Build** and **EAS Submit**, **EAS Update** (JS-only fixes) | Cloud builds. No local Android SDK needed for release builds. |
| CI | **GitHub Actions**: lint, type-check, unit tests, EAS preview build on pull request | Catches regressions before they reach the phone. |
| Crash reporting | Optional, off by default. If enabled: **Sentry** with no personal data | Privacy first. |

### 9.1 Libraries to avoid

- Redux or MobX (too heavy for this app).
- AsyncStorage for structured data (use SQLite).
- Any library that requires the legacy architecture or is not New-Architecture compatible at SDK 57.
- Any SDK that sends analytics or health data off the device.

---

## 10. Non-functional requirements

| ID | Requirement |
| --- | --- |
| NFR-1 | Offline: 100% of features work with airplane mode on. |
| NFR-2 | Performance: budgets in Section 2.3 are release gates. |
| NFR-3 | Privacy: no network calls in v1. No analytics. No third-party SDK that collects data. Android auto-backup disabled (`allowBackup="false"`). |
| NFR-4 | Security: data stays in the app sandbox. Export is the user's explicit action. No secrets in source. |
| NFR-5 | Accessibility: TalkBack labels on all controls. Long-press actions also exposed as accessibility actions and visible buttons. Touch targets at least 48 × 48 dp. Text scales to 200%. Colour contrast at least 4.5:1 for body text in both themes. |
| NFR-6 | Reliability: no data loss on process kill mid-edit (writes are transactional). Migrations are forward-only and tested. |
| NFR-7 | Localisation-ready: all strings in one resource file. English in v1. Dates and numbers formatted with `Intl`. |
| NFR-8 | Maintainability: TypeScript strict, no `any` outside tests, ESLint clean, domain coverage at least 90%, overall at least 75%. |
| NFR-9 | Compatibility: Android 8.0 (API 26) to latest. Tested on Android 12 and 13 on the M53. |
| NFR-10 | Battery: no background services, no wake locks, no location. |

---

## 11. Navigation and screens

| Route | Screen | Entry point |
| --- | --- | --- |
| `/(onboarding)/welcome` | First login, steps 1 and 2 | First launch only |
| `/(tabs)/home` | Home | Tab |
| `/(tabs)/workouts` | Workout list | Tab |
| `/(tabs)/calendar` | Calendar | Tab |
| `/(tabs)/diet` | Diet | Tab |
| `/(tabs)/profile` | Profile | Tab |
| `/modal/edit-weight?date=` | Edit weight sheet | Long press on weight, or Edit weight |
| `/modal/add-meal?slot=` | Add meal sheet | "+ Add meal" |
| `/modal/edit-profile` | Edit profile | Profile → Edit |
| `/workouts/[id]` | Workout detail | Workout list |
| `/workouts/[id]/set/[setId]` | Edit set sheet | Long press on a set |
| `/journey` | Weight journey | Home → Journey |
| `/history` | Weight history | Home → History |
| `/exercise/[catalogueId]` | Exercise progress | Workout detail → exercise |
| `/settings` | Settings | Profile → Settings |

Tab order: Home, Workouts, Calendar, Diet, Profile. The tab bar is hidden only on onboarding and modals.

---

## 12. Acceptance criteria

Each criterion is a test. Automated tests are noted; anything without a note is a manual check on the M53.

| ID | Criterion |
| --- | --- |
| AC-W1 | Logging 79.4 kg for today updates Home, History, and Journey on the same save. (Integration test) |
| AC-W2 | With 80.0 kg logged on Monday and nothing on Tuesday, Tuesday shows 80.0 kg labelled "carried from Monday". Logging 79.4 kg on Tuesday replaces it. (Unit test, property test) |
| AC-W3 | 80.0 kg displays as 176.4 lb in lb mode and 80.0 kg in kg mode. Converting back and forth changes the value by no more than 0.1. (Unit test) |
| AC-W4 | Entering 19.9 kg or 400.1 kg is rejected with an inline error. (Unit test) |
| AC-W5 | Long-pressing the weight value for 400 ms opens the Edit weight sheet and produces a haptic pulse. The "Edit weight" accessibility action also opens it. (Manual, Maestro) |
| AC-P1 | Edit profile shows current values. Saving changes the profile. Today's daily plan is unchanged until the next day, and "Recalculate today" updates it. (Integration test) |
| AC-D1 | For the worked example in Section 6.2, the app computes 2,240 kcal, 127 g protein, 62 g fat, 294 g carbohydrate. (Unit test) |
| AC-D2 | A female profile with a target below 1,200 kcal is raised to 1,200 kcal and the "Limited" note appears. (Unit test) |
| AC-D3 | A training day adds 80 kcal and 20 g carbohydrate. A rest day adds nothing. (Unit test) |
| AC-D4 | Marking a meal as eaten updates the Eaten total and not the Planned total. (Integration test) |
| AC-D5 | Suggestions shown never exceed the remaining budget of their slot. (Property test) |
| AC-D6 | Adding a suggestion creates a meal item in the chosen slot with the suggestion's values. (Integration test) |
| AC-K1 | A workout with three exercises and two sets each saves in one transaction. Killing the app mid-save leaves no partial workout. (Integration test, manual) |
| AC-K2 | Exercise progress shows starting weight, last weight, and their difference in both kg and lb. (Unit test) |
| AC-C1 | A date with a logged workout shows a dot. Deleting that workout (after undo expires) removes the dot. (Integration test) |
| AC-C2 | The streak counts consecutive ISO weeks that meet the planned gym days. (Unit test) |
| AC-R1 | After a Push workout, Pull is recommended. After a skipped day, the recommendation stays on Pull. (Unit test) |
| AC-R2 | After a workout logged less than 20 hours ago, Home shows the rest or light cardio note. (Unit test) |
| AC-S1 | Export produces a valid v1 JSON file. Importing it into a fresh install reproduces counts for every table. (Integration test) |
| AC-S2 | Import of an invalid file is rejected and leaves existing data unchanged. (Integration test) |
| AC-N1 | With airplane mode on, every screen in Section 11 opens and every write succeeds. (Manual, Maestro) |
| AC-A1 | With system font scale at 200%, no text is clipped on any screen. (Manual) |
| AC-A2 | With TalkBack on, every control announces a meaningful label. (Manual) |
| AC-PERF1 | Cold start on the M53 release build is 2.0 s or less. Scrolling the 365-day history shows no dropped frames in normal use. (Manual, profiler) |

---

## 13. Testing strategy

| Level | Scope | Tool | Gate |
| --- | --- | --- | --- |
| Unit | Domain functions (calories, resolution, progress, split, streak, units, suggestions) | Jest | Pull request must pass |
| Property | Carry-forward, budget rules, unit round-trip | fast-check | Pull request must pass |
| Integration | Repositories against in-memory SQLite, transactions, export and import | Jest + Drizzle | Pull request must pass |
| Component | Forms, sheets, long-press behaviour | React Native Testing Library | Pull request must pass |
| End-to-end | Onboarding → log weight → log workout → plan a meal → export | Maestro on emulator (M53 profile 1080×2400, 412×915 dp) | Release candidate |
| Device | Full acceptance list on a real M53 (6 GB and 8 GB if available), Android 12 and 13 | Manual checklist | Release candidate |

---

## 14. Release and delivery

### 14.1 Build profiles (`eas.json`)

| Profile | Output | Use |
| --- | --- | --- |
| `development` | Development client APK | Daily coding with hot reload on the M53 |
| `preview` | Installable APK | Install directly on the M53 for testing |
| `production` | Android App Bundle (AAB) | Google Play upload |

### 14.2 Versioning

- Semantic version in `app.config.ts` (`1.0.0` for the first release).
- `android.versionCode` increments automatically on each production build (`autoIncrement: true` in EAS).
- Git tags match the release version.

### 14.3 Update channels

- JS-only fixes ship with **EAS Update** to the `production` channel.
- Changes to native modules, permissions, or Expo SDK ship as a new store build.

### 14.4 Store listing

- Package name: `com.yourname.gymdiet` (set by the owner before first build; cannot change later).
- Permissions: `POST_NOTIFICATIONS` only. No internet, location, storage, or camera permissions in v1.
- Privacy statement: "All data stays on your device. No account and no network use."

---

## 15. Engineering standards

- **Branching:** trunk-based. Short-lived feature branches, squash merge into `main`.
- **Commits:** Conventional Commits (`feat(weight): add carry-forward`, `fix(diet): clamp suggestions to budget`). Each commit references a requirement ID where relevant.
- **Pull requests:** must pass lint, type check, unit and integration tests, and the EAS preview build. Include screenshots for UI changes, both themes.
- **Code style:** Prettier defaults. ESLint with `typescript-eslint`, `react-hooks`, `react-native` rules, and `import/order`.
- **Architecture decisions:** recorded as short ADRs in `docs/adr/` (for example ADR-001 SQLite over AsyncStorage, ADR-002 snapshot daily plan).
- **Naming:** components PascalCase, hooks `useX`, domain functions verbs (`resolveWeight`, `computeDailyPlan`), test files `*.test.ts(x)` next to the code.
- **Dependencies:** pin the SDK version. Upgrade the Expo SDK once per release cycle, following the Expo upgrade guide.
- **Documentation:** this FRD is the source of truth. Change it first, then the code.

---

## 16. Build plan (phases for implementation)

Each phase must end with passing tests and a runnable preview build on the M53.

| Phase | Scope | Definition of done |
| --- | --- | --- |
| 0. Foundation | Expo project, TypeScript strict, ESLint, Prettier, Jest, CI, EAS config, theme tokens, Drizzle schema v1, migrations | CI green. Preview APK installs and opens on the M53. |
| 1. Profile and onboarding | FR-O1–O4, FR-P1–P5, FR-H1–H3, Edit profile | AC-P1 passes. Onboarding works in airplane mode. |
| 2. Weight | FR-W1–W9, carry-forward, journey, history, reminder | AC-W1–W5 pass. |
| 3. Workouts | FR-K1–K10, catalogue, exercise history | AC-K1, AC-K2 pass. |
| 4. Calendar | FR-C1–C6 | AC-C1, AC-C2 pass. |
| 5. Diet | FR-D1–D15, daily plan snapshot, meal items, suggestions | AC-D1–D6 pass. |
| 6. Recommendations and Home | FR-R1–R7, FR-HM1–HM2 | AC-R1, AC-R2 pass. |
| 7. Settings and data | FR-S1–S5, export, import, delete all | AC-S1, AC-S2 pass. |
| 8. Hardening | Accessibility, performance budgets, Maestro flows, device checklist | AC-A1, AC-A2, AC-PERF1, AC-N1 pass. |
| 9. Release | Production AAB, Play internal testing, then production | Release checklist in SETUP.md complete. |

---

## 17. Assumptions, decisions, and open items

### 17.1 Decisions made in this version

| # | Question from v1.0 | Decision |
| --- | --- | --- |
| D1 | Fixed or selectable deficit? | Default 500 kcal, user may choose 250, 500, or 750 (FR-D3). |
| D2 | Strict four-day cycle or rest day? | Cycle is Push → Pull → Legs → Cardio. Rest is a suggestion, not a cycle position (FR-R2, FR-R3). |
| D3 | Exercise library size? | Bundled catalogue of about 60 common exercises, plus free-text names (Section 18.3). |
| D4 | Should target change mid-day after a weight edit? | No. The daily plan is a snapshot. "Recalculate today" applies changes on demand (FR-D2). |

### 17.2 Assumptions

- One user, one device, no shared data.
- Adult user (age 13 or older). Formula is for adults; no paediatric guidance.
- The user enters weights at least occasionally. Carry-forward covers gaps.
- Calorie and macro targets are estimates. The app is not a medical device.

### 17.3 Open items for the owner (non-blocking)

- Preferred app name and package identifier (needed before first build).
- Whether to enable crash reporting (default: off).
- Whether a PIN or fingerprint app lock is wanted in v1.1.

---

## 18. Appendix

### 18.1 Device setup reference

See `docs/SETUP.md` for installing the app on the Galaxy M53 5G.

### 18.2 Bundled food suggestions (seed, illustrative values)

| Slot | Suggestion | kcal | Protein g | Carb g | Fat g |
| --- | --- | --- | --- | --- | --- |
| Snack | Greek yogurt cup | 150 | 15 | 12 | 3 |
| Snack | Protein shake | 200 | 25 | 10 | 3 |
| Dinner | Side salad | 60 | 2 | 8 | 2 |
| Breakfast | Greek yogurt bowl | 320 | 28 | 35 | 7 |
| Lunch | Tuna and rice wrap | 410 | 34 | 45 | 9 |
| Breakfast | Egg and avocado toast | 350 | 18 | 30 | 16 |
| Lunch | Chicken rice bowl | 620 | 45 | 70 | 14 |
| Dinner | Salmon, potatoes, broccoli | 640 | 40 | 55 | 25 |

These values are starting data for the owner to check against labels. The app shows them as estimates.

### 18.3 Exercise catalogue (starter set, by split)

- **Push:** Bench press, Incline dumbbell press, Overhead press, Triceps pushdown, Lateral raise.
- **Pull:** Lat pulldown, Seated cable row, Barbell row, Biceps curl, Face pull.
- **Legs:** Back squat, Romanian deadlift, Leg press, Leg curl, Calf raise.
- **Cardio:** Steady-state treadmill, Stationary bike, Rowing machine, Intervals.

The full catalogue (about 60 entries) is generated from this list with muscle group and equipment fields in `src/data/seed/catalogue.ts`.

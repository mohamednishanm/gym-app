# Gym & Diet Tracker — Handoff Package

A production-ready specification and design package for a personal Android app (React Native + Expo) that tracks body weight, gym workouts, attendance, workout recommendations, and daily calories and meals. It is built for one user on a Samsung Galaxy M53 5G.

## What is in this package

```
gym-app-handoff/
  README.md            This file: overview and contents
  CLAUDE.md            Instructions for Claude Code (read first by the agent)
  docs/
    FRD.md             Production Functional Requirements Document (source of truth)
    SETUP.md           How to give this package to Claude Code, and how to install the app on the M53
  design/
    README.md          Index of the approved screens and how they map to the FRD
    *.dc.html          Light screens (Home, Edit weight, Calendar, Workout, Diet, Add meal,
                       Profile, Edit profile, First login)
    Dark-*.dc.html     The same screens in the dark theme
    canvas.json        Layout index for the design canvas (positions and titles)
```

## Quick start

1. Read `docs/SETUP.md` Part 1 and push this folder to a private Git repository.
2. Open the repository in Claude Code and send the first prompt from SETUP.md.
3. Claude Code follows `CLAUDE.md`, builds the app phase by phase from `docs/FRD.md`, and stops after each phase so you can check it.
4. Install builds on the phone using SETUP.md Part 2.

## Key decisions (summary)

| Area | Decision | Where to read |
| --- | --- | --- |
| Platform | Android-first, React Native with Expo SDK 57 (New Architecture only) | FRD §2, §9 |
| Target device | Samsung Galaxy M53 5G (Dimensity 900, 6/8 GB RAM, 1080×2400, 120 Hz) | FRD §2.1 |
| Data | Local-first SQLite with Drizzle ORM. No backend, no accounts, no network in v1 | FRD §7, §10 |
| Calories | Mifflin–St Jeor, daily snapshot, deficit selectable 250/500/750 kcal | FRD §6, §5.4 |
| Weight | Stored in kg, shown in kg or lb, carry-forward for missed days | FRD §5.2, §6.3 |
| Edit gesture | Long press to edit, with a visible button and accessibility action | FRD §5.2, §5.5 |
| Workout cycle | Push → Pull → Legs → Cardio, advances only on logged workouts | FRD §5.7 |
| Release | EAS Build (APK for personal use, AAB for Google Play) | FRD §14, SETUP §2 |

## What changed from the earlier FRD (v1.0)

- Adds production architecture, stack, performance budgets, device profile, testing strategy, and release process.
- Adds profile editing, the edit-profile screen, the daily plan snapshot, the meal planner, and suggestions that fit each slot's budget.
- Resolves the open questions from v1.0 (see FRD §17.1).
- Replaces the sample numbers in the mockups with the calculation rules from FRD §6.

## Notes

- Design files are the exported source of the approved screens. The live design is also on Claude's canvas (the artifact link in the chat). If they differ, the live design is the newer one; tell Claude Code which to follow.
- Calorie and macro targets are estimates, not medical advice. This is stated in the app.

# Design Index

These files are the approved screens for the Gym & Diet Tracker. Each file is a self-contained HTML design page (`.dc.html`) that renders at 390 × 844 dp (the mockup size). The app targets a 412 × 915 dp screen (Samsung Galaxy M53 5G), so layouts must stretch, not scale.

Light files use plain names. Dark files start with `Dark-`. Both sets share the same layout and content.

## Screens

| File (light / dark) | Screen | Requirements |
| --- | --- | --- |
| `Onboarding.dc.html` / `Dark-Onboarding.dc.html` | First login, setup form (step 1 of 2) | FR-O1–O4 |
| `Home.dc.html` / `Dark-Home.dc.html` | Home: weight (long press to edit), calories, journey, recommended workout | FR-HM1, FR-W1, FR-R |
| `EditWeight.dc.html` / `Dark-EditWeight.dc.html` | Edit weight bottom sheet with number pad and kg/lb toggle | FR-W2, FR-W6 |
| `Calendar.dc.html` / `Dark-Calendar.dc.html` | Month calendar with gym-day dots, streak summary, day detail | FR-C1–C5 |
| `Workout.dc.html` / `Dark-Workout.dc.html` | Workout detail: exercises, sets, start and last weight in lb, add exercise | FR-K1–K5, FR-K9 |
| `Diet.dc.html` / `Dark-Diet.dc.html` | Diet: planned and eaten totals, macros, meal planner, suggestions | FR-D1–D14 |
| `AddMeal.dc.html` / `Dark-AddMeal.dc.html` | Add a meal sheet: meal type, name, calories, protein, time, quick picks | FR-D9, FR-D15 |
| `Profile.dc.html` / `Dark-Profile.dc.html` | Profile: details, Edit button, units, appearance, reminders | FR-P1, FR-P4, FR-P5, FR-W9 |
| `EditProfile.dc.html` / `Dark-EditProfile.dc.html` | Edit profile, pre-filled form with Cancel and Save | FR-P2, FR-P3 |

## Interaction notes

- **Long press to edit:** the weight on Home and each set in Workout detail open editors on a 400 ms long press. Each also has a visible Edit control and an accessibility action (FRD FR-W2, FR-K9).
- **Light and dark:** both themes follow the tokens in FRD §8.3. Dark is not an inverted light theme. It uses a charcoal background and one mint accent.
- **Navigation:** five tabs (Home, Workouts, Calendar, Diet, Profile). Onboarding, Edit profile and sheets appear over the tabs.

## Values in the mockups

The numbers in the mockups are illustrative sample data (for example 1,840 kcal and 2,250 kcal, 128 g protein). The app must calculate real values from the rules in FRD §6. The worked example in FRD §6.2 shows the correct calculation for the sample profile.

## Canvas index

`canvas.json` lists each screen's position on the design canvas. It is for reference only. Claude Code does not need to run the canvas.

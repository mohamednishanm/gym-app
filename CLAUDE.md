# CLAUDE.md — Gym & Diet Tracker

You are building a production Android app (React Native + Expo) for one user on a Samsung Galaxy M53 5G.
Read this file, then `docs/FRD.md` (the source of truth), then `design/README.md`, before writing code.

## Your working rules

1. **The FRD is the spec.** Implement requirement IDs exactly (FR-W4, FR-D2, …). If something is ambiguous, follow the decision table in FRD §17.1, then leave a `// FRD: <id>` comment. Do not invent features outside FRD §1.2 scope.
2. **Phases.** Work through FRD §16 in order. Finish a phase only when its acceptance criteria tests pass. Stop at the end of each phase and summarise what passed.
3. **Design is visual reference.** `design/*.dc.html` files are the approved screens (light files are the plain names, dark files start with `Dark-`). Match layout, hierarchy and spacing. Use the tokens in FRD §8.3, not the sample numbers in the mockups (mockup values are illustrative; calculate real values per FRD §6).
4. **Never weaken a test to make it pass.** If a test conflicts with the FRD, stop and report the conflict.

## Stack (do not substitute without an ADR)

- Expo SDK 57, React Native 0.86, New Architecture only, Hermes, TypeScript `strict`.
- Expo Router (file-based routes under `app/`).
- Local data: `expo-sqlite` + Drizzle ORM, migrations committed under `src/data/db/migrations/`.
- Zod for all validation (forms, import). Zustand only for transient UI state.
- `@shopify/flash-list` for long lists. `react-native-reanimated` and `react-native-gesture-handler` for gestures and motion.
- `expo-notifications` for local reminders. No network calls in v1.
- Jest + jest-expo + React Native Testing Library, fast-check for property tests, Maestro for end-to-end flows.
- ESLint (flat config, typescript-eslint) and Prettier.

Check versions with `npx expo install --check` after scaffolding. Use `npx expo install <pkg>` for Expo-compatible versions.

## Architecture rules

- Layers: UI → hooks → `src/domain` (pure TypeScript) → `src/data/repositories` → SQLite.
- `src/domain` imports nothing from React, React Native, Expo or Drizzle.
- Screens never import the database client. They use hooks that call repositories.
- Every multi-row write runs in one transaction.
- Feature folders under `src/features/<feature>`. Shared UI under `src/shared/ui`.

## Commands

```bash
npm run lint          # ESLint + Prettier check
npm run typecheck     # tsc --noEmit
npm test              # Jest unit + integration + property tests
npm run test:e2e      # Maestro flows (needs emulator or device)
npx expo start        # dev server
npx expo run:android  # dev build on connected device (USB debugging)
eas build -p android --profile preview      # installable APK
eas build -p android --profile production   # Play Store AAB
```

Run `lint`, `typecheck` and `test` before every commit. All three must pass.

## Code standards

- No `any` outside tests. No non-null assertions without a comment explaining why.
- Components are small and typed. Props are explicit interfaces.
- Strings live in one resource file (`src/shared/lib/strings.ts`). No hard-coded user-facing text in components.
- Dates: `YYYY-MM-DD` local strings in storage. Use `date-fns` and local-time helpers in `src/shared/lib/dates.ts`.
- Units: store kg and cm. Convert only in `src/domain/units.ts`.
- Long-press actions must also have a visible button and an accessibility action (FRD FR-W2, FR-K9).
- Every interactive element: minimum 48 × 48 dp, an accessibility label.
- Commits: Conventional Commits, with the requirement ID in the body when relevant.

## Device target

- Samsung Galaxy M53 5G: 1080 × 2400 px, 6.7", about 412 × 915 dp, 120 Hz, Android 12 (One UI 4.1) upgradable to 13.
- minSdkVersion 26. Portrait only. Respect safe-area insets (punch-hole camera).
- Do not add permissions beyond `POST_NOTIFICATIONS` (FRD §14.4). Set `allowBackup` to false (FRD NFR-3).

## Definition of done (every feature)

- Requirement IDs covered and cross-referenced in tests.
- Unit or integration tests for every acceptance criterion in FRD §12 that the feature owns.
- Works in airplane mode.
- Light and dark themes both checked against the design file.
- No lint, type or test failures.

## First task

Start Phase 0 (FRD §16): scaffold the project with `npx create-expo-app@latest` (template: blank TypeScript), add the stack above, configure Jest, ESLint, Prettier, the EAS `eas.json` profiles from FRD §14.1, the theme tokens, and the Drizzle schema for FRD §7.2 with migration 0001. Then stop and report.

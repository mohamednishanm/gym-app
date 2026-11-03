# Setup Guide — Gym & Diet Tracker

This guide covers two things:

1. How to give this handoff package to Claude Code so it builds the app.
2. How to install the finished app on your Samsung Galaxy M53 5G.

---

## Part 1 — Give the package to Claude Code

### Recommended method: private GitHub repository

This is the best way. Claude Code works inside a git repository, it can commit each phase, and you can review every change.

**Requirements on your computer**

| Tool | Check | Install |
| --- | --- | --- |
| Git | `git --version` | https://git-scm.com |
| Node.js (current LTS) | `node --version` | https://nodejs.org |
| Claude Code | `claude --version` | `npm install -g @anthropic-ai/claude-code` (then run `claude` once to sign in) |
| GitHub account | — | https://github.com |

**Steps**

1. Unzip `gym-app-handoff.zip` into an empty folder, for example `~/projects/gym-diet-tracker`.
2. Open a terminal in that folder and create the repository:

   ```bash
   cd ~/projects/gym-diet-tracker
   git init -b main
   git add .
   git commit -m "docs: add FRD, design and handoff package"
   ```

3. Create an empty **private** repository on GitHub named `gym-diet-tracker` (no README, no .gitignore). Then:

   ```bash
   git remote add origin https://github.com/<your-username>/gym-diet-tracker.git
   git push -u origin main
   ```

4. Start Claude Code in the same folder:

   ```bash
   claude
   ```

5. Send this first prompt:

   > Read CLAUDE.md, docs/FRD.md and design/README.md. Then complete Phase 0 from FRD section 16 and stop. Commit each step with a Conventional Commit message. Summarise what passed at the end.

6. Review the result, then continue phase by phase with prompts like:

   > Start Phase 1 from FRD section 16. Stop when AC-P1 passes and summarise.

### Alternative: local folder only

If you do not want GitHub, skip steps 3 and the `git remote` line. Run `git init` and `claude` in the folder. You still get commits locally.

### Alternative: attach the zip in a chat

You can attach `gym-app-handoff.zip` to a Claude chat and ask it to unpack the files. Claude Code itself works best from a folder or repository, so the GitHub method is still preferred for building.

### Keep the package in sync

- Edit `docs/FRD.md` first when requirements change, then tell Claude Code: "FRD section X changed; update the code and tests."
- Replace files in `design/` when the design changes, and commit.

---

## Part 2 — Install the app on the Samsung Galaxy M53 5G

There are three routes. Route A is for development. Route B is for personal use. Route C is for the Play Store.

### Phone preparation (all routes)

1. **Enable Developer options:** Settings → About phone → Software information → tap **Build number** seven times. Menu names can differ slightly by One UI version.
2. **Enable USB debugging:** Settings → Developer options → **USB debugging** (on).
3. Connect the phone to your computer with a USB cable (data cable, not charge-only). Accept the "Allow USB debugging?" prompt on the phone.
4. Check the connection:

   ```bash
   adb devices
   ```

   You should see your device listed as `device`. If you do not have `adb`, install **Android platform-tools** from https://developer.android.com/tools/releases/platform-tools.

### Route A — Development build (hot reload on your phone)

Use this while coding. It needs the Android SDK on your computer, which Android Studio installs.

1. Install **Android Studio** (https://developer.android.com/studio). In its SDK Manager, install the latest Android SDK Platform and Build-Tools.
2. In the project folder:

   ```bash
   npm install
   npx expo run:android
   ```

   The first build takes several minutes. After that, `npx expo start` reloads the app on the phone when you save a file.

### Route B — Installable APK (personal use, no computer needed afterwards)

Use this to install a release build on the phone. EAS Build compiles the app in the cloud, so you do not need Android Studio.

1. Create a free Expo account at https://expo.dev and install the EAS CLI:

   ```bash
   npm install -g eas-cli
   eas login
   ```

2. In the project folder, link the project once:

   ```bash
   eas build:configure
   ```

3. Build the preview APK (this uses the `preview` profile in `eas.json`):

   ```bash
   eas build -p android --profile preview
   ```

   When it finishes, EAS prints a download link and QR code.

4. Install on the M53:
   - Open the download link on the phone (or scan the QR code).
   - When Android asks, allow installs from that browser or Files app: **Settings → Apps → Special access → Install unknown apps** (menu names can vary by One UI version).
   - Tap the downloaded APK and choose **Install**.
   - Open **Gym & Diet**. Allow the notification permission when asked.

Repeat step 3 whenever you want a new version. Each build increments the version code automatically.

### Route C — Google Play (internal testing, then production)

Use this for a stable install that updates automatically.

1. Create a **Google Play Console** developer account at https://play.google.com/console (one-time registration fee applies).
2. Create the app in the console. Use the package name set in `app.config.ts` (it cannot be changed after the first upload).
3. Build the store bundle:

   ```bash
   eas build -p android --profile production
   ```

4. Upload with EAS Submit (needs a Google Play service-account key, which the EAS CLI walks you through):

   ```bash
   eas submit -p android --latest
   ```

5. In the console, create an **Internal testing** track, add your Google account as a tester, and open the opt-in link on the M53 to install from the Play Store.
6. When it works, promote the release to **Production**.

### Over-the-air updates for JavaScript-only fixes

After a store release, JavaScript-only fixes can ship without a new store build:

```bash
eas update --branch production --message "Fix: carry-forward on empty history"
```

Changes to native code, permissions, or the Expo SDK need a new build (Route B or C).

---

## Part 3 — First-run checklist on the phone

1. Launch the app. The First login screen appears.
2. Enter name, age, sex, height, current weight, goal weight, activity, and goal. Tap Continue.
3. On Home, long-press the weight to edit it (or use the Edit button).
4. Set the weigh-in reminder time in Settings.
5. Log one workout and confirm it appears on Calendar.
6. Turn on airplane mode and confirm everything still works (FRD AC-N1).
7. Use Settings → Export to save a backup JSON. Keep it somewhere safe.

---

## Part 4 — Troubleshooting

| Problem | Fix |
| --- | --- |
| `adb devices` shows nothing | Use a data cable. Re-enable USB debugging. Try another USB port. Accept the prompt on the phone. |
| "App not installed" on the M53 | Uninstall any older build with the same package name first. Check the APK finished downloading. |
| "Install blocked" | Allow installs from the browser or Files app (Route B step 4). |
| Build fails on EAS | Run `npx expo-doctor` and `npx expo install --check`, fix what it lists, and commit. |
| Notifications do not appear | Settings → Apps → Gym & Diet → Notifications → allow. Check Samsung battery settings do not restrict the app. |
| Text looks clipped | Check Settings → Display → Font size and screen zoom. The app must scale to 200% (FRD AC-A1); report it if not. |
| Claude Code asks for a permission repeatedly | Allow the command for the session, or add it to `.claude/settings.json` in the project. |

---

## Part 5 — What you need to provide

- Your app name and package identifier (for example `com.yourname.gymdiet`). Set these before the first build.
- An Expo account and, for Play Store, a Google Play Console account.
- Your weigh-in reminder time and any default preferences.

<h1 align="center">G-Labs Voiceover</h1>

<p align="center"><b>A desktop app that turns a script into a finished voice-over - CapCut voices and 322 Microsoft Edge voices in one place, exported as one merged MP3 with matching SRT subtitles.</b></p>

<p align="center">
  <b>English</b> ·
  <a href="README.vi.md">Tiếng Việt</a>
</p>

<p align="center">
  <a href="https://github.com/duckmartians/G-Labs-Voiceover/releases/latest"><img alt="Download for Windows" src="https://img.shields.io/badge/Download-Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white"></a>&nbsp;
  <a href="https://github.com/duckmartians/G-Labs-Voiceover/releases/latest"><img alt="Download for macOS (Apple Silicon)" src="https://img.shields.io/badge/Download-macOS%20Apple%20Silicon-000000?style=for-the-badge&logo=apple&logoColor=white"></a>
</p>

---

## Install

### Step 1 - Choose the right build

Download the latest version from **[Releases](https://github.com/duckmartians/G-Labs-Voiceover/releases/latest)** and pick the file for your machine:

| Your machine | Download file | Note |
|---|---|---|
| 🪟 **Windows (64-bit)** | [`GLabsVoiceover-<version>-setup.exe`](https://github.com/duckmartians/G-Labs-Voiceover/releases/latest) | Installer |
| 🍎 **Mac with Apple chip (M1/M2/M3/M4)** | [`GLabsVoiceover-<version>-arm64.dmg`](https://github.com/duckmartians/G-Labs-Voiceover/releases/latest) | Apple Silicon only |

> There is **no Intel Mac build** - the `arm64` file will not open on an Intel Mac.

### Step 2 - Install

<details open>
<summary><b>🪟 On Windows</b></summary>

1. Open the **`GLabsVoiceover-<version>-setup.exe`** file you downloaded.
2. If a **"Windows protected your PC"** box appears (SmartScreen): click **More info** → **Run anyway**. *(The app is not code-signed with a Microsoft certificate, so Windows warns about it - it is not a virus.)*
3. Follow the installer - you can choose the install folder. It installs for your user account and creates **Start Menu** and **Desktop** shortcuts.
4. Open **G-Labs Voiceover** from the Start Menu or the Desktop shortcut.

</details>

<details open>
<summary><b>🍎 On macOS</b></summary>

1. Open the **`.dmg`** you downloaded, then **drag the G-Labs Voiceover icon onto the Applications folder**.
2. In **Applications**, **right-click** (or Control-click) **G-Labs Voiceover** → **Open** → click **Open** again in the confirmation box. *(The app is not signed by Apple, so you must open it this way the **first time**; after that it opens like any other app.)*
3. If macOS says the app **"is damaged / can't be opened"** or there is no Open button, open **Terminal**, paste this and press Enter:
   ```bash
   xattr -dr com.apple.quarantine "/Applications/G-Labs Voiceover.app"
   ```
   Then open the app again.

</details>

### Step 3 - Sign in (access comes with your G-Labs plan)

**Voiceover is not sold on its own.** It unlocks for **your account with an active paid plan on any G-Labs tool (Lite or higher)**, or an active **G-Labs Voice Studio** (Voice add-on) subscription. The free Basic plan does not unlock it. See [plans & tools](https://duckmartians.info).

Click **Sign in with Google** and use the Google account linked to your G-Labs plan - the app opens your system browser to sign in. The license server confirms your plan each time the app opens and keeps re-checking it while the app runs; when the underlying plan expires, the app locks again. The voice-service configuration is delivered only to an entitled session, so **both engines (CapCut and Edge) need an eligible account**.

The app **updates itself** from GitHub Releases: on Windows it downloads the new installer and runs it; on macOS it downloads the new `.dmg` and opens it for you to drag into Applications.

---

## First run

1. **Open the app and sign in with Google** using your account on a paid plan.
2. **Open the Text to speech tab** and paste your script - or **Import file** to load a `.txt` or `.srt`.
3. **Choose how to split** (per sentence, per line, or smart-pack by character count), pick the **CapCut** or **Microsoft Edge** engine and a **voice**.
4. Press **Generate all**. Listen to each segment and regenerate any you don't like.
5. Adjust **speed** and the **pause** between segments if needed, then **Export audio** (MP3), **Export subtitles** (SRT) or **Export bundle** (ZIP with both).

---

## Features

![Main window](docs/screenshots/01-main.png)

- **Two engines, one workflow** - switch between **CapCut** and **Microsoft Edge** without changing anything else: same segment table, same settings, same exports. Edge has no quota, so it also serves as the fallback when CapCut throttles.
- **Sentence-aware splitting** - one segment per sentence, per line, or **smart-pack** to a character budget; every segment generates, replays, regenerates and downloads on its own.
- **Import .txt / .srt** - an SRT import keeps each cue's timing, and **"fit to subtitle timing"** speeds up any line longer than its cue (speed-up only, capped at 1.8×) so the merged audio follows the subtitle timeline.
- **Dialogue mode** - write `<Name> line` per line, assign a voice per character, and the whole conversation generates in one run; the SRT keeps the speaker names.
- **Speed & pause without regenerating** - the speed and silence-gap sliders re-merge locally: no new service calls, no quota spent.
- **Exports** - merged MP3, SRT, or both in a ZIP. File names carry an index, the first words and a timestamp, so a new export never overwrites an earlier one.
- **Voice preview** - every voice has a bundled sample to hear before generating; multilingual voices have one per language.
- **Proxy pool** - paste proxies in many formats; CapCut/Edge requests rotate through them, with a re-check that detects the proxy type and flags dead ones.
- **Webhook API** (opt-in) - a local HTTP server that scripts and AI agents can drive (see below).
- **Remembers your work** - script draft, segment table, chosen voice, speed, pause, favourites, UI zoom, language and the open tab survive a relaunch.
- **Whole-UI zoom** (70-140%) in the title bar, with real reflow.
- **ffmpeg bundled** - nothing else to install, no GPU needed.
- **11 interface languages** - Tiếng Việt, English, हिन्दी, Türkçe, Português, 简体中文, اردو (right-to-left), বাংলা, Русский, Español, ไทย.

### Which engine should I use?

| | CapCut | Microsoft Edge |
|---|---|---|
| Voices | 121 | **322** across 142 languages/locales |
| Quota | limited per session | none |
| Best for | the signature CapCut sound | volume work, rare languages |

Some CapCut voices are one **multilingual** model rather than a native speaker - they carry a 🌐 marker because they read with a slight accent outside their primary language. Native voices sound natural; the marker lets you choose knowingly.

---

## Pages

### 🔊 Text to speech

![Text to speech](docs/screenshots/01-main.png)

The main workspace. Paste or import a script, choose the split mode, engine and voice, then **Generate all**. Each row of the segment table can be played, regenerated and downloaded on its own. With an imported SRT, turn on **fit to subtitle timing** to keep every line on its original cue. The **Play all & export** panel plays the merged result and exports MP3, SRT or a ZIP bundle.

### 💬 Dialogue

![Dialogue](docs/screenshots/02-dialogue.png)

Write a conversation as `<Name> line`, one per line, and assign a voice to each character. The whole dialogue is generated in one run with a different voice per block, and the exported SRT keeps each speaker's name.

### 🎙 Voices

![Voices](docs/screenshots/03-voices.png)

The voice catalog - 322 Edge voices and 121 CapCut voices. Preview each voice from its bundled sample and mark favourites; multilingual CapCut voices carry the 🌐 marker.

### 🕘 History

![History](docs/screenshots/05-history.png)

Every generation is saved automatically for the current session, searchable by content, and reloads into the tab it came from.

### ⚙️ Settings - proxy pool

![Proxy pool](docs/screenshots/04-proxy.png)

Paste proxies in any format (`host:port:user:pass`, `user:pass@host:port`, `socks5://…`, IPv6) into a saved list; CapCut/Edge requests rotate through it round-robin. **Re-check** probes each proxy, auto-detects its type (HTTP / SOCKS4 / SOCKS5) and flags the dead ones so you can remove them in one click. Passwords are masked in the list.

### 🌐 Webhook API

![Webhook API](docs/screenshots/07-webhook.png)

Enable the **Webhook API** tab and Voiceover runs a small local HTTP server (default `127.0.0.1:8788`) that your own tools - or an AI agent - can call:

```bash
curl -X POST http://127.0.0.1:8788/api/tts \
  -H "X-API-Key: <key from the panel>" -H "Content-Type: application/json" \
  -d '{"provider":"edge","text":"Xin chào","voice":"vi-VN-HoaiMyNeural","srt":true}'
# → { "task_id": "…" }  → poll GET /api/status/{id}  → download from /api/files
```

Submit `text` or a `segments` array and get one merged `master.mp3` (plus per-segment files and an SRT on request). Auth is a per-install API key generated by the app; generation only runs while your account is entitled; the server binds to `127.0.0.1` unless you open it to your LAN. The complete reference - every endpoint, body field and response shape - is in **[WEBHOOK.md](WEBHOOK.md)**, written so an AI agent can integrate from it directly.

### 🌍 Right-to-left interface

![Urdu interface](docs/screenshots/06-rtl-urdu.png)

The interface switches between 11 languages; Urdu (اردو) uses a full right-to-left layout.

---

## Where your data lives

| What | macOS | Windows |
|---|---|---|
| Settings, script draft, proxy list, webhook key (`prefs.json`), sign-in session and device ID | `~/Library/Application Support/G-Labs Voiceover` | `%APPDATA%\G-Labs Voiceover` |
| Webhook output files | `$TMPDIR/voiceover-webhook-files` | `%TEMP%\voiceover-webhook-files` |

Your text is sent to the CapCut or Microsoft Edge voice service (through your proxies if configured) to be spoken. The G-Labs license server is used only to sign in and check access.

---

## Troubleshooting

**"Account not eligible" after signing in** - the account has no active paid G-Labs plan (Lite or higher) or Voice add-on. Buy or renew a plan, then press **Try again**.

**"Can't reach the server"** - the app could not contact the license server. Check your connection and press **Try again**.

**"Device limit reached for this account"** - the account has reached the number of devices the license server allows.

**CapCut stops generating or says the quota is used up** - CapCut limits requests per session. Switch the engine to **Microsoft Edge** (no quota), or add proxies in **Settings**.

**Windows blocks it with "Windows protected your PC"** - click **More info → Run anyway**. The app is not code-signed, so Windows warns about it; it is not a virus.

**macOS says the app is damaged / can't be opened** - it is not signed by Apple. Right-click → **Open** the first time, or run `xattr -dr com.apple.quarantine "/Applications/G-Labs Voiceover.app"`.

**An update didn't install** - download the latest version manually from [Releases](https://github.com/duckmartians/G-Labs-Voiceover/releases/latest).

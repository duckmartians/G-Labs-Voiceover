# G-Labs Voiceover — Webhook API Integration Guide

A guide for automating **G-Labs Voiceover** over its local HTTP API. Written to be
read by an AI agent or a developer: every endpoint, request body, response shape,
and the full async workflow are specified with copy-paste examples.

---

## 1. Overview & key concepts

G-Labs Voiceover turns text into natural voice-over using two engines:

- **`edge`** — Microsoft Edge read-aloud neural voices (~322 voices, many locales).
- **`capcut`** — CapCut TTS voices (~121 voices).

When you enable the **Webhook API** tab in the app, Voiceover starts a **separate
local HTTP server** (default `http://127.0.0.1:8788`) that you drive from your own
tools. Core ideas:

- **It's local.** By default it binds to `127.0.0.1`, reachable only from the same
  machine. The user may re-bind it to `0.0.0.0` / a LAN IP; then other machines on
  the network can call it too (the API key then travels the network in the clear).
- **It's authenticated.** Every endpoint except `GET /api/health` and
  `GET /api/files/{name}` requires an API key (see §2).
- **It's entitlement-gated.** Generation (`POST /api/tts`) only works while the
  signed-in account is entitled. Otherwise it returns `403`.
- **It's asynchronous.** You submit a job, get a `task_id` immediately (HTTP `202`),
  poll until it is `completed`, then download the result files.

```
POST /api/tts  ──▶  202 { task_id }
                     │
GET /api/status/{id} ──▶  running … running … completed { results:[…] }
                                                          │
GET /api/files/{name} ──▶  the audio / .srt bytes
```

---

## 2. Authentication

Auth applies to all endpoints **except** `GET /api/health` and
`GET /api/files/{name}`. Send the key as **either** header:

```
X-API-Key: <your-key>
```
```
Authorization: Bearer <your-key>
```

The key is shown (and can be rotated / copied) in the app's **Webhook API** tab.
Rotating it invalidates the old key immediately. A missing or wrong key → `401`.

---

## 3. Endpoints

| Method | Path | Auth | Purpose |
| --- | --- | :---: | --- |
| `GET`  | `/api/health` | ❌ | Server health + whether the account is entitled |
| `GET`  | `/api/voices` | ✅ | List every usable voice (CapCut + Edge) |
| `POST` | `/api/tts` | ✅ | Submit a text-to-speech job → returns `task_id` |
| `GET`  | `/api/status/{task_id}` | ✅ | Poll task status; carries results once done |
| `GET`  | `/api/result/{task_id}` | ✅ | Get the result (only once `completed`) |
| `GET`  | `/api/files/{name}` | ❌ | Download a generated audio / `.srt` file |
| `GET`  | `/api/tasks` | ✅ | List the 50 most recent tasks |

Base URL is `http://<host>:<port>` — default `http://127.0.0.1:8788`. The examples
below assume that default.

---

## 4. The async workflow (step by step)

### Step 1 — Health check (optional)

```bash
curl http://127.0.0.1:8788/api/health
```
```json
{ "ok": true, "app": "G-Labs Voiceover", "entitled": true }
```
If `entitled` is `false`, generation will be refused with `403`; ask the user to
sign in with an entitled account in the app.

### Step 2 — Discover voices

```bash
curl http://127.0.0.1:8788/api/voices -H "X-API-Key: <key>"
```
```json
{
  "count": 443,
  "voices": [
    { "id": "vi-VN-HoaiMyNeural", "name": "Hoai My", "lang": "vi-VN", "provider": "edge" },
    { "id": "BV562_streaming",    "name": "Mai",     "lang": "vi-VN", "provider": "capcut" }
  ]
}
```
Use a voice's **`id`** as the `voice` field when you submit. For CapCut you may also
pass the human display name (e.g. `"Mai"`) — the server resolves it. An unknown
voice makes the task fail with a clear error, so pick from this list.

### Step 3 — Submit a job

`POST /api/tts` with a JSON body (see §5). The response is **HTTP 202**:

```bash
curl -X POST http://127.0.0.1:8788/api/tts \
  -H "X-API-Key: <key>" -H "Content-Type: application/json" \
  -d '{"provider":"edge","text":"Xin chào từ Voiceover","voice":"vi-VN-HoaiMyNeural","srt":true}'
```
```json
{
  "task_id": "7ea17bc6a18f",
  "status": "pending",
  "message": "TTS task queued",
  "poll_url": "/api/status/7ea17bc6a18f"
}
```

### Step 4 — Poll status

```bash
curl http://127.0.0.1:8788/api/status/7ea17bc6a18f -H "X-API-Key: <key>"
```
`status` goes `pending` → `running` → `completed` (or `failed`). Poll every ~1–2 s.

Completed:
```json
{
  "task_id": "7ea17bc6a18f",
  "status": "completed",
  "error": null,
  "results": [
    { "name": "7ea17bc6a18f_master.mp3", "url": "http://127.0.0.1:8788/api/files/7ea17bc6a18f_master.mp3", "type": "audio/mpeg", "role": "master" }
  ]
}
```
Failed:
```json
{ "task_id": "7ea17bc6a18f", "status": "failed", "error": "Edge không tạo được audio nào. Lý do: …" }
```

### Step 5 — Download the files

Every result carries a ready-to-use `url`. `GET` it (no API key needed):

```bash
curl http://127.0.0.1:8788/api/files/7ea17bc6a18f_master.mp3 -o master.mp3
```

---

## 5. Request body — `POST /api/tts`

| Field | Type | Required | Default | Notes |
| --- | --- | :---: | --- | --- |
| `provider` | string | — | `"edge"` | `"edge"` or `"capcut"`. |
| `text` | string | one of | — | A single string. Use this **or** `segments`. |
| `segments` | array | one of | — | `[{ "text": "...", "voice": "..." }]` — one clip per item, merged in order. |
| `voice` | string | ✅* | — | Voice `id` from `GET /api/voices`. Required for `text`; per-item `voice` overrides it inside `segments`. |
| `lang` | string | — | auto | Locale override, e.g. `"vi-VN"`. |
| `speed` | number | — | `1.0` | Playback speed, `0.5`–`3.0` (pitch preserved). |
| `gapMs` | number | — | `0` | Silence between segments in ms, `0`–`5000`. |
| `srt` | boolean | — | `false` | Also produce a `.srt` subtitle file with timings. |
| `perSegment` | boolean | — | `false` | Also return each segment as its own `.mp3`. |

\* `voice` is required when using `text`. With `segments`, give each item a `voice`
(or a top-level `voice` as the fallback for all of them).

**Single line:**
```json
{ "provider": "edge", "text": "Xin chào", "voice": "vi-VN-HoaiMyNeural" }
```

**Multi-segment + subtitles + per-segment files:**
```json
{
  "provider": "edge",
  "segments": [
    { "text": "Đoạn một.", "voice": "vi-VN-HoaiMyNeural" },
    { "text": "Đoạn hai.", "voice": "vi-VN-NamMinhNeural" }
  ],
  "speed": 1.0,
  "gapMs": 300,
  "srt": true,
  "perSegment": true
}
```

---

## 6. Result files (`results[]`)

Each entry: `{ name, url, type, role, index? }`.

| `role` | Produced | Description |
| --- | --- | --- |
| `master` | always | The full clip: all segments merged, with `speed` + `gapMs` applied. |
| `segment` | when `perSegment: true` | One `.mp3` per segment; `index` is its 1-based position. |
| `srt` | when `srt: true` | A `.srt` subtitle file whose cues match the master's timeline. |

Files live in a temp directory and are served by `GET /api/files/{name}`. Download
what you need soon after completion; don't rely on them persisting across restarts.

---

## 7. Status & error reference

| HTTP | When |
| --- | --- |
| `202` | `POST /api/tts` accepted; body has `task_id`. |
| `200` | Any successful `GET`. |
| `400` | `POST /api/tts` with neither `text` nor a non-empty `segments`. |
| `401` | Missing / wrong API key on an authed endpoint. |
| `403` | Account not entitled — generation refused. |
| `404` | Unknown `task_id` or missing file. |

Task-level failures return HTTP `200` on `/api/status` with `status: "failed"` and a
human-readable `error` (e.g. an unknown voice, or the provider returning no audio).
Always check `status`, not just the HTTP code, before downloading.

---

## 8. Full worked example (bash)

```bash
BASE=http://127.0.0.1:8788
KEY=<your-key>

# 1) submit
TID=$(curl -s -X POST $BASE/api/tts \
  -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"provider":"edge","segments":[
        {"text":"Xin chào, đây là bản demo.","voice":"vi-VN-HoaiMyNeural"},
        {"text":"Đoạn thứ hai.","voice":"vi-VN-NamMinhNeural"}],
       "gapMs":300,"srt":true}' \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['task_id'])")

# 2) poll until done
while :; do
  S=$(curl -s $BASE/api/status/$TID -H "X-API-Key: $KEY")
  ST=$(echo "$S" | python3 -c "import sys,json;print(json.load(sys.stdin)['status'])")
  [ "$ST" = completed ] && break
  [ "$ST" = failed ] && { echo "$S"; exit 1; }
  sleep 2
done

# 3) download every result file
echo "$S" | python3 -c "import sys,json;[print(f['url']) for f in json.load(sys.stdin)['results']]" \
  | while read u; do curl -s -O "$u"; done
```

---

## 9. Notes for agents

- **Discover, don't guess.** Always resolve a real `voice` `id` from
  `GET /api/voices` (filter by `provider` and `lang`) before submitting.
- **One concept per call.** Submit is fire-and-forget; the audio is ready only after
  `status` is `completed`. Never treat the `202` body as the result.
- **Batch as segments.** To narrate many lines, send them as one `segments` array in
  a single `POST /api/tts` rather than many separate jobs — you get one merged
  `master.mp3` (and, with `srt`, aligned subtitles).
- **Handle 403.** It means the human needs to sign in with an entitled account in the
  app; there is no API-side workaround.
- **The server may be off.** If requests fail to connect, the user has not enabled
  the Webhook API tab, or bound it to a different host/port — ask them to check.

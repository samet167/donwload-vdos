# 🎬 VideoDown — Video Downloader Web App

Web application សម្រាប់ទាញយកវីដេអូពី YouTube, Facebook, TikTok, Vimeo, X/Twitter និង 1000+ sites។

| Layer    | Stack                                    | Hosting |
|----------|------------------------------------------|---------|
| Frontend | HTML5 · CSS3 · Vanilla JS                | Vercel  |
| Backend  | Python 3.12 · Flask · yt-dlp · FFmpeg    | Render  |
| Server   | Gunicorn + gevent                        | Render  |

---

## 🌐 Production URLs

| Service  | URL |
|----------|-----|
| Backend API | `https://video-downloader-backend-1-5oer.onrender.com` |
| Health Check | `https://video-downloader-backend-1-5oer.onrender.com/health` |
| Frontend | *(Your Vercel deployment URL)* |

### API Endpoints (Production)

```
GET  https://video-downloader-backend-1-5oer.onrender.com/health
POST https://video-downloader-backend-1-5oer.onrender.com/api/info
POST https://video-downloader-backend-1-5oer.onrender.com/api/download
GET  https://video-downloader-backend-1-5oer.onrender.com/api/progress/<task_id>
```

---

## 📁 Project Structure

```
video-downloader/
│
├── backend/
│   ├── app.py               # Flask app + REST API
│   ├── downloader.py        # yt-dlp engine + progress tracking
│   ├── gunicorn.conf.py     # Gunicorn production config
│   ├── requirements.txt     # Python dependencies
│   ├── render.yaml          # Render Blueprint deploy config
│   └── .env.example         # Environment variable template
│
├── frontend/
│   ├── index.html           # UI (responsive dark theme)
│   ├── css/style.css        # CSS design tokens + layout
│   ├── js/
│   │   ├── config.js        # Single source of truth — BACKEND_URL
│   │   ├── api.js           # API client (all fetch calls)
│   │   └── app.js           # UI controller + event handling
│   ├── script.js            # Standalone download script
│   ├── env-config.js        # Runtime API_BASE injection (Vercel build)
│   ├── vercel.json          # Vercel routing + security headers
│   └── .env.example         # Frontend env vars template
│
└── README.md
```

---

## ⚙️ Local Development

### Prerequisites

| Tool    | Install |
|---------|---------|
| Python 3.12+ | [python.org](https://python.org) |
| FFmpeg  | `brew install ffmpeg` (macOS) · `sudo apt install ffmpeg` (Ubuntu) |

### 1. Clone repository

```bash
git clone https://github.com/YOUR_USERNAME/video-downloader.git
cd video-downloader
```

### 2. Backend setup

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate        # macOS/Linux
# .venv\Scripts\activate         # Windows

pip install -r requirements.txt

cp .env.example .env
# Edit .env — set FRONTEND_URL=http://127.0.0.1:5500
```

### 3. Start backend

```bash
python3 app.py
# Server starts at http://127.0.0.1:5000
```

### 4. Start frontend (local dev)

For local development, edit `frontend/js/config.js`:
```js
BACKEND_URL: "http://127.0.0.1:5000",
```

Then open `frontend/index.html` in a browser — **or** use a local static server:

```bash
cd frontend
python3 -m http.server 8080
# Open http://localhost:8080
```

> **Note:** For production, `config.js` should point to the Render backend URL.

---

## 🚀 Deploy to Render (Backend)

### Step 1 — Push to GitHub

```bash
git add .
git commit -m "initial commit"
git push origin main
```

### Step 2 — Create Render Web Service

1. Go to [render.com](https://render.com) → **New** → **Blueprint**
2. Connect your GitHub repository
3. Render detects `render.yaml` and configures automatically
4. Click **Apply**

### Step 3 — Set Environment Variables in Render Dashboard

Go to **Environment** tab and add:

| Variable | Value | Note |
|----------|-------|------|
| `SECRET_KEY` | *(generate with `python -c "import secrets; print(secrets.token_hex(32))"`)* | **Required — sensitive** |
| `FRONTEND_URL` | `https://your-app.vercel.app` | Your Vercel domain for CORS |
| `FFMPEG_PATH` | `/usr/bin/ffmpeg` | Set by build command |
| `TMP_DIR` | `/tmp/videodl` | Temp download dir |
| `MAX_VIDEO_SECONDS` | `3600` | 1 hour max |
| `MAX_FILE_MB` | `500` | 500 MB max |
| `LOG_LEVEL` | `INFO` | Production log level |
| `PYTHON_VERSION` | `3.12.0` | Already in render.yaml |

### Step 4 — Verify Deployment

Health check URL:
```
https://video-downloader-backend-1-5oer.onrender.com/health
```

Expected response:
```json
{"status": "ok", "service": "video-downloader"}
```

> **Note:** Free tier sleeps after 15 min of inactivity. First request after sleep takes ~30s (cold start).

---

## 🌐 Deploy to Vercel (Frontend)

### Step 1 — Create Vercel Project

1. Go to [vercel.com](https://vercel.com) → **Add New Project**
2. Import your GitHub repository
3. Set **Root Directory** to `frontend`
4. Framework Preset: **Other**

### Step 2 — Set Environment Variables

In Vercel Dashboard → **Settings** → **Environment Variables**:

| Variable | Value |
|----------|-------|
| `API_BASE` | `https://video-downloader-backend-1-5oer.onrender.com/api` |

### Step 3 — Set Build Command

In Vercel Dashboard → **Settings** → **General** → **Build & Output Settings**:

| Setting | Value |
|---------|-------|
| **Build Command** | `echo "window.ENV={API_BASE:'$API_BASE'};" > env-config.js` |
| **Output Directory** | `.` (root of frontend/) |
| **Install Command** | *(leave empty)* |

This generates `env-config.js` at build time with the real backend URL.

### Step 4 — Deploy

```bash
# Push to main branch → Vercel deploys automatically
git push origin main
```

### Step 5 — Update Backend CORS

After getting your Vercel URL (e.g. `https://videodl.vercel.app`), go to Render Dashboard and set:

```
FRONTEND_URL = https://videodl.vercel.app
```

---

## 🔐 Security

| Feature | Implementation |
|---------|----------------|
| CORS | Only allows `FRONTEND_URL` env var domain |
| Rate Limiting | 30 req/min for `/api/info`, 10 req/min for `/api/download` |
| URL Validation | Server-side scheme + netloc check |
| Video Duration Limit | `MAX_VIDEO_SECONDS` (default 3600s) |
| File Size Limit | `MAX_FILE_MB` (default 500 MB) |
| Temp File Cleanup | Auto-delete after stream completes |
| Security Headers | Vercel adds X-Frame-Options, CSP, etc. via `vercel.json` |

---

## 📡 API Documentation

### `GET /health`

Server liveness probe. Used by Render health check.

**Response**
```json
{"status": "ok", "service": "video-downloader"}
```

---

### `POST /api/info`

Fetch video metadata without downloading.

**Request**
```json
{ "url": "https://www.youtube.com/watch?v=xxxxx" }
```

**Response 200**
```json
{
  "success": true,
  "title": "Video Title",
  "thumbnail": "https://i.ytimg.com/...",
  "duration": "4:05",
  "quality": "1080p",
  "formats": [
    {
      "format_id": "137",
      "resolution": "1920x1080",
      "ext": "mp4",
      "filesize": 52428800,
      "height": 1080
    }
  ]
}
```

**Error responses**

| Code | Meaning |
|------|---------|
| 400 | Missing or invalid URL |
| 422 | yt-dlp cannot process URL (private, geo-blocked, etc.) |
| 429 | Rate limit exceeded |
| 500 | Server error |

---

### `POST /api/download`

Start video download.

**Request**
```json
{
  "url": "https://www.youtube.com/watch?v=xxxxx",
  "quality": "720p"
}
```

**Response** — MP4 binary stream or SSE progress events depending on endpoint.

---

## 🔧 Environment Variables Reference

### Backend (Render)

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | *required* | Flask session signing key |
| `FRONTEND_URL` | — | CORS allowed origin (Vercel URL) |
| `FFMPEG_PATH` | `/usr/bin/ffmpeg` | FFmpeg binary path |
| `TMP_DIR` | `/tmp/videodl` | Temp download directory |
| `MAX_VIDEO_SECONDS` | `3600` | Max video duration (seconds) |
| `MAX_FILE_MB` | `500` | Max file size (MB) |
| `LOG_LEVEL` | `INFO` | Log level |
| `PORT` | `10000` | HTTP port (set by Render automatically) |
| `WEB_CONCURRENCY` | `1` | Number of Gunicorn workers |

### Frontend (Vercel)

| Variable | Description |
|----------|-------------|
| `API_BASE` | Backend API URL: `https://video-downloader-backend-1-5oer.onrender.com/api` |

### Frontend Configuration File (js/config.js)

```js
const CONFIG = {
  BACKEND_URL: "https://video-downloader-backend-1-5oer.onrender.com",
  REQUEST_TIMEOUT_MS: 30000,
  SSE_HEARTBEAT_MS: 500,
  SSE_CLOSE_DELAY_MS: 1500,
};
```

---

## ❗ Troubleshooting

### Backend

**`SSL: CERTIFICATE_VERIFY_FAILED`**
```bash
pip install --upgrade certifi
```

**`ffmpeg: command not found`**
```bash
# Local macOS
brew install ffmpeg

# Local Ubuntu
sudo apt install ffmpeg

# Render — handled by render.yaml buildCommand automatically
```

**`yt-dlp: ERROR: Sign in to confirm you're not a bot`**

Some videos require authentication. yt-dlp cannot bypass this server-side without cookies. This is a YouTube restriction, not a bug.

**Rate limit 429**

Wait 1 minute and try again. Limits: 30/min for info, 10/min for download.

**Render free tier — slow first request**

Free tier sleeps after 15 min. First request takes ~30s for cold start. Consider adding an uptime monitor (e.g. [UptimeRobot](https://uptimerobot.com)) to ping `/health` every 14 min.

### Frontend

**API Offline badge**
- Check Render Dashboard → your service is running
- Check `FRONTEND_URL` on Render includes your Vercel domain
- Check `API_BASE` in Vercel env vars is correct (no trailing slash issues)

**CORS errors in browser console**
- Ensure backend `FRONTEND_URL` env var matches your exact Vercel deployment URL
- Do not include trailing slash in the URL

**`env-config.js` not working on Vercel**
- Verify Build Command in Vercel: `echo "window.ENV={API_BASE:'$API_BASE'};" > env-config.js`
- Verify `API_BASE` env var is set in Vercel Dashboard

**Video downloads but no file appears (mobile)**
On iOS Safari, the browser opens the video in a new tab instead of downloading. Tap the share icon → Save to Files.

---

## 📝 License

MIT — Free to use and modify.

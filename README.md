<p align="center">
  <img src="assets/logo.svg" width="128" alt="Zhiyin Logo" />
</p>

<h1 align="center">Zhiyin Music 知音音乐</h1>

<p align="center">A Rust-based remote NAS music library + Web player system</p>

<p align="center">
  <a href="README_CN.md">中文文档</a>
</p>

[![Rust](https://img.shields.io/badge/rust-1.75+-orange.svg)](https://www.rust-lang.org/)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## 📖 Overview

Zhiyin Music is a music streaming service designed for NAS environments, offering:

- 🎼 **Large-scale Music Library**: Easily manage tens of thousands of tracks
- 🚀 **High Performance**: Rust + SQLite + async architecture, memory optimized to ~30-50MB
- 📦 **Ready to Use**: One-click Docker deployment
- 🔍 **Smart Recommendations**: Offline recommendation engine based on listening history
- 🎨 **Full Metadata Support**: Automatic parsing of cover art, artist, and album info
- 🌐 **RESTful API**: Front-end / back-end separation, easy integration
- 📡 **Subsonic API Compatible**: Works with Symfonium, DSub, Ultrasonic, and other popular clients
- 🔐 **User Authentication**: JWT + Basic Auth + bcrypt with role-based access control
- 🖥️ **Frontend Hosting**: Optional built-in Web frontend service with SPA support

## ✨ Features

### 1. Music Library Management
- ✅ Multiple music root directories
- ✅ Incremental scanning (based on mtime/size/hash)
- ✅ Real-time file watching (optional)
- ✅ Supported formats: MP3, FLAC, WAV, M4A, OGG, OPUS, APE

### 2. Metadata Parsing
- ✅ Audio metadata parsing with TagLib
- ✅ Automatic extraction of embedded cover art
- ✅ External cover file support (cover.jpg, folder.jpg)
- ✅ Cover deduplication via hash to save storage

### 3. Audio Playback
- ✅ HTTP Range request support (seamless seeking)
- ✅ **Multi-tier quality selection (128k/192k/320k/lossless)**
- ✅ **Smart transcode caching (based on play count)**
- ✅ **FFmpeg pipe streaming** (real-time transcode, play while encoding)
- ✅ Original format playback (no transcoding)
- ✅ Play history tracking

### 3.5 Subsonic API Compatibility
- ✅ **Subsonic API v1.16.1 compatibility layer** (mounted at `/rest/`, 40+ endpoints)
- ✅ Works with Ultrasonic, Symfonium, DSub, and other mainstream clients
- ✅ Multiple auth methods (plaintext password / hex-encoded / token auth)
- ✅ Dual XML/JSON response format (`f=json` / `f=xml`)
- ✅ **Implemented endpoints**:
  - 🔹 System: `ping`, `getLicense`, `getUser`
  - 🔹 Browsing: `getMusicFolders`, `getIndexes`, `getArtists`, `getArtist`, `getAlbum`, `getSong`, `getRandomSongs`, `getAlbumList2`
  - 🔹 Media: `stream`, `download`, `getCoverArt`
  - 🔹 Search: `search3`, `search2` (ID3 search)
  - 🔹 Annotation: `scrobble` (play history reporting)
  - 🔹 Library: `getScanStatus`, `startScan` (real-time progress)
- ⬜ **Stub endpoints** (routes registered, returning empty responses for client compatibility):
  - Playlists, favorites/ratings, bookmarks, lyrics, similar recommendations, radio/podcasts, etc.

### 4. User Authentication & Permissions
- ✅ User system (SQLite `users` table, bcrypt password hashing)
- ✅ JWT Token authentication (REST API)
- ✅ Basic Auth authentication (REST API)
- ✅ Role-based access control (admin / user)
- ✅ Initial admin creation (environment variable or `POST /api/setup`)
- ✅ User management API (create/update/delete/reset password)
- ✅ Subsonic API uses unified database user authentication

### 5. Web Frontend Hosting (Optional)
- ✅ Built-in HTTP static file server
- ✅ SPA route fallback (auto fallback to `index.html`)
- ✅ Mount via Docker volume or place frontend build output directly
- ✅ Swap in any frontend framework build output at any time

### 6. Personalized Recommendations
- ✅ Frequently played recommendations
- ✅ Recently listened recommendations
- ✅ Similarity-based recommendations (artist/album/directory)
- ✅ Async background computation, non-blocking API

### 7. Docker Deployment
- ✅ Multi-stage build, small image size
- ✅ Supports x86_64 and ARM64
- ✅ Health checks and auto-restart
- ✅ Data persistence

## 🏗️ Architecture

```
API Layer → Service Layer → Job System → Data Access
    ↓           ↓               ↓            ↓
  Axum     Business Logic   Background    SQLite
                              Jobs
```

## 🚀 Quick Start

### Option 1: Docker Compose (Recommended)

**Configure Settings**

Edit `docker-compose.yml` to set your music directory:
```yaml
volumes:
  - /path/to/your/music:/music:ro  # Change to your actual path
```

Create the configuration file:
```bash
cp config.toml.example.en config.toml
```

**Start the Service**
```bash
docker-compose up -d
```

**Create Admin User**

Option A: Via the Web frontend — the system will guide you through setup on first launch.

Option B: Via environment variables (set in `docker-compose.yml`):
```yaml
environment:
  - MUSIC_ADMIN_USER=admin
  - MUSIC_ADMIN_PASSWORD=your_secure_password
```

Option C: Via API:
```bash
curl -X POST http://localhost:8080/api/setup \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

**Trigger a Scan**

Use the Web frontend settings page to manually trigger or configure scans.

Or via API:

```bash
curl -X POST http://localhost:8080/api/scan \
  -H "Authorization: Bearer <your-jwt-token>"
```

**Access the API**

- API docs: http://localhost:8080/swagger-ui

## 📚 API Reference

### Core Endpoints

#### Public Endpoints (No Auth Required)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/status` | GET | Check system initialization status |
| `/api/setup` | POST | Create initial admin (only when system is uninitialized) |
| `/api/auth/login` | POST | Login to obtain JWT token |
| `/api/health` | GET | Health check |

#### Authenticated Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/songs` | GET | List songs (paginated) |
| `/api/songs/:id` | GET | Get song details |
| `/api/songs/batch` | POST | Batch query song details |
| `/api/albums` | GET | List albums |
| `/api/artists` | GET | List artists (with song count, album count, cover) |
| `/api/stream/{id}?quality=high` | GET | Stream audio (quality selection + Range) |
| `/api/covers/{id}` | GET | Get cover art |
| `/api/scan` | POST | Trigger scan job |
| `/api/recommend` | GET | Get recommended songs (with full song info) |
| `/api/history/recent` | GET | Get recently played songs |
| `/api/config` | GET | Get current config (with restart flags) |
| `/api/config` | PUT | Update config (auto-save and reload) |
| `/api/users` | GET | List users (admin) |
| `/api/users` | POST | Create user (admin) |
| `/api/users/me` | GET | Get current user info |
| `/api/users/:id` | PUT | Update user |
| `/api/users/me/password` | PUT | Change own password |
| `/api/users/:id/reset-password` | POST | Reset user password (admin) |
| `/api/users/:id` | DELETE | Delete user (admin) |

#### Subsonic Compatible Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/rest/ping` | GET | Heartbeat |
| `/rest/getArtists` | GET | Get artist index |
| `/rest/stream?id=1` | GET | Stream playback |
| `/rest/search3?query=xxx` | GET | Search |

---

### Songs

#### 1. List Songs (Paginated)

```bash
curl "http://localhost:8080/api/songs?limit=10&offset=0"
```

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "file_path": "/music/song.mp3",
      "title": "Song Title",
      "artist_id": 1,
      "album_id": 1,
      "cover_id": 1,
      "track_no": 1,
      "disc_no": 1,
      "year": 2024,
      "duration_secs": 245,
      "bitrate": 320000,
      "sample_rate": 44100,
      "codec": "mp3",
      "file_size": 10485760,
      "file_mtime": 1234567890,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 100,
  "limit": 10,
  "offset": 0
}
```

#### 2. Get Song Details

```bash
curl http://localhost:8080/api/songs/1
```

**Response:**
```json
{
  "id": 1,
  "file_path": "/music/song.mp3",
  "title": "Song Title",
  "artist_id": 1,
  "album_id": 1,
  "cover_id": 1,
  "track_no": 1,
  "disc_no": 1,
  "year": 2024,
  "duration_secs": 245,
  "bitrate": 320000,
  "sample_rate": 44100,
  "codec": "mp3",
  "file_size": 10485760,
  "file_mtime": 1234567890,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

**Error Response (404):**
```json
{
  "error": "not_found",
  "message": "Song ID 1 not found"
}
```

#### 3. Batch Query Songs

```bash
curl -X POST http://localhost:8080/api/songs/batch \
  -H "Content-Type: application/json" \
  -d '{"ids": [1, 2, 3, 4, 5]}'
```

**Response:**
```json
[
  {
    "id": 1,
    "title": "Song 1",
    "artist_id": 1,
    "album_id": 1,
    ...
  },
  {
    "id": 2,
    "title": "Song 2",
    "artist_id": 2,
    "album_id": 1,
    ...
  }
]
```

> Note: Non-existent song IDs are automatically filtered out. Results are returned in the order of input IDs.

---

### Artists

#### List Artists (with Statistics)

```bash
curl "http://localhost:8080/api/artists?limit=20&offset=0"
```

**Response:**
```json
{
  "items": [
    {
      "id": 1,
      "name": "Taylor Swift",
      "song_count": 245,
      "album_count": 12,
      "cover_id": 123
    },
    {
      "id": 2,
      "name": "Ed Sheeran",
      "song_count": 180,
      "album_count": 8,
      "cover_id": 456
    },
    {
      "id": 3,
      "name": "Local Artist",
      "song_count": 15,
      "album_count": 2,
      "cover_id": null
    }
  ],
  "total": 100,
  "limit": 20,
  "offset": 0,
  "page": 1,
  "total_pages": 5,
  "has_next": true,
  "has_prev": false
}
```

**Fields**:
- **song_count**: Total number of songs by this artist
- **album_count**: Total number of albums by this artist
- **cover_id**: Representative cover art, can be `null`

**Cover Selection Strategy** (by priority):
1. **Latest album cover** — prefer the most recent album cover (sorted by year descending)
2. **First album with cover** — if no year info, pick the first album that has a cover
3. **Song cover** — if no album covers exist, pick a cover from the artist's songs
4. **NULL** — if no covers at all, return `null`; frontend can show a placeholder

**Fetching cover art**:
```bash
# If cover_id is not null
curl http://localhost:8080/api/covers/123 > artist_cover.jpg
```

---

### Recommendations

#### Get Recommended Songs (with Full Song Info)

```bash
curl http://localhost:8080/api/recommend
```

**Response:**
```json
[
  {
    "score": 0.95,
    "id": 1,
    "file_path": "/music/song.mp3",
    "title": "Recommended Song",
    "artist_id": 1,
    "album_id": 1,
    "cover_id": 1,
    "duration_secs": 245,
    "bitrate": 320000,
    ...
  }
]
```

> Results are sorted by score (highest first) and include full song metadata, ready for player display.

---

### Play History

#### Get Recently Played Songs (with Full Song Info)

```bash
# Get last 20 (default)
curl http://localhost:8080/api/history/recent

# Get last 50
curl "http://localhost:8080/api/history/recent?limit=50"

# Get last 100 (maximum)
curl "http://localhost:8080/api/history/recent?limit=100"
```

**Query Parameters**:
- `limit` (optional): Number of songs to return, default 20, max 100

**Response:**
```json
[
  {
    "played_at": "2024-02-04T12:30:00Z",
    "id": 1,
    "file_path": "/music/song.mp3",
    "title": "Recently Played Song",
    "artist_id": 1,
    "album_id": 1,
    "cover_id": 1,
    "duration_secs": 245,
    ...
  }
]
```

**Behavior**:
- Returns recently played songs (deduplicated, showing only the last play time per song)
- Sorted by play time descending (most recent first)
- Includes full song info and `played_at` timestamp
- Supports custom result count (1-100)

---

### Configuration Management

#### 1. Get Current Configuration

```bash
curl http://localhost:8080/api/config
```

**Response:**
```json
{
  "scan": {
    "roots": ["/Users/user/Music"],
    "mode": "manual",
    "interval_hours": 24,
    "_meta": {
      "mode": {
        "description": "Scan mode: manual/scheduled/watch",
        "requires_restart": false,
        "default_value": "manual"
      }
    }
  },
  "recommend": {
    "job_interval_hours": 24,
    "play_threshold": 10,
    "max_results": 50
  },
  ...
}
```

> The `_meta` field for each config item indicates whether a service restart is required for the change to take effect.

#### 2. Update Configuration

```bash
curl -X PUT http://localhost:8080/api/config \
  -H "Content-Type: application/json" \
  -d '{
    "scan": {
      "mode": "scheduled",
      "interval_hours": 12
    },
    "recommend": {
      "max_results": 100
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Successfully updated 3 config items",
  "requires_restart": false,
  "updated_fields": [
    "scan.mode",
    "scan.interval_hours",
    "recommend.max_results"
  ]
}
```

**Behavior**:
- Supports partial updates (only update specified fields)
- Auto-saves to `config.toml`
- Auto-triggers config reload (Unix systems)
- Clearly indicates if restart is needed

**Updatable config items**:
- Scan: `scan.roots`, `scan.mode`, `scan.interval_hours`
- Recommendations: `recommend.job_interval_hours`, `recommend.play_threshold`, `recommend.max_results`
- Transcoding: `transcode.enabled`, `transcode.cache_strategy`, `transcode.cache_threshold`
- Maintenance: `maintenance.*` (all maintenance-related settings)

**Non-updatable config** (requires restart):
- `server.host`, `server.port`
- `database.path`, `database.pool_size`
- `transcode.cache_path`

> Full API docs available at Swagger UI: `http://localhost:8080/swagger-ui/`

---

### Authentication

#### 1. System Setup (Create First Admin)

```bash
# Check initialization status
curl http://localhost:8080/api/auth/status
```

```json
{
  "initialized": false,
  "message": "System not initialized. Please create an admin account."
}
```

```bash
# Create admin (first-time only; endpoint is disabled after initialization)
curl -X POST http://localhost:8080/api/setup \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

#### 2. Login

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 1,
    "username": "admin",
    "role": "admin",
    "display_name": null,
    "is_active": true
  }
}
```

#### 3. Accessing Protected Endpoints

```bash
# Bearer Token (recommended)
curl -H "Authorization: Bearer <token>" http://localhost:8080/api/songs

# Or Basic Auth
curl -u admin:your_secure_password http://localhost:8080/api/songs
```

---

### Audio Streaming

#### Stream Audio (Quality Selection + Pipe Streaming Transcode)

```bash
# Play original quality (default, supports Range requests)
curl http://localhost:8080/api/stream/1 > song.mp3

# Select high quality (320kbps)
curl "http://localhost:8080/api/stream/1?quality=high" > song_high.mp3

# Quality options: low(128k) / medium(192k) / high(320k) / lossless / original

# Range requests (for original or cached files)
curl -H "Range: bytes=0-1023" "http://localhost:8080/api/stream/1?quality=high"
```

**Streaming Transcode Details:**
- Original / cached files → `ServeFile` response (supports Range/ETag/resume)
- Real-time transcode without cache → FFmpeg pipe streaming (`Transfer-Encoding: chunked`, play while encoding)
- Cache strategy controlled by `transcode.cache_strategy`: `none` (all piped), `all` (cache everything), `smart` (cache after play count threshold)

### Subsonic API Endpoints

The project implements a Subsonic API v1.16.1 compatibility layer, mounted at `/rest/`. Connect directly using Ultrasonic, Symfonium, DSub, and other clients.

#### Enabling

In `config.toml`:

```toml
[subsonic]
enabled = true
```

#### Client Connection Settings

| Setting | Value |
|---------|-------|
| Server URL | `http://your-server:8080` |
| Username | A user created in the system (via `/api/setup` or environment variable) |
| Password | User password |
| Auth Mode | Plaintext password / Token auth both supported |

#### Supported Endpoints

| Category | Endpoint | Description |
|----------|----------|-------------|
| System | `/rest/ping` | Heartbeat |
| System | `/rest/getLicense` | License info |
| Browsing | `/rest/getMusicFolders` | Get music directories |
| Browsing | `/rest/getIndexes` | Artist index by letter |
| Browsing | `/rest/getArtists` | Get all artists (ID3) |
| Browsing | `/rest/getArtist?id=1` | Get artist and their albums |
| Browsing | `/rest/getAlbum?id=1` | Get album and its songs |
| Browsing | `/rest/getSong?id=1` | Get single song info |
| Browsing | `/rest/getRandomSongs?size=10` | Random songs |
| Media | `/rest/stream?id=1` | Stream playback (supports `maxBitRate`) |
| Media | `/rest/download?id=1` | Download original file |
| Media | `/rest/getCoverArt?id=1` | Get cover art |
| Search | `/rest/search3?query=keyword` | Search songs/albums/artists |
| Annotation | `/rest/scrobble?id=1` | Record play history |
| Library | `/rest/getScanStatus` | Get scan status |
| Library | `/rest/startScan` | Trigger scan |

All endpoints also support the `.view` suffix (e.g., `/rest/ping.view`).

#### Authentication Methods

Three standard Subsonic authentication methods are supported:

**1. Plaintext Password:**
```
/rest/ping?u=admin&p=your-password&v=1.16.1&c=myapp
```

**2. Hex-Encoded Password:**
```
/rest/ping?u=admin&p=enc:796f75722d70617373776f7264&v=1.16.1&c=myapp
```

**3. Token Authentication (Recommended):**
```
/rest/ping?u=admin&t=md5-token&s=random-salt&v=1.16.1&c=myapp
```

> Token auth works by having the client generate a token using `MD5(password + salt)`. The server verifies against the encrypted stored password. After the first plaintext login, the system automatically enables Token auth support for that user.

#### Response Format

Default is XML. Add `f=json` parameter to switch to JSON:
```bash
curl "http://localhost:8080/rest/ping?u=admin&p=your-password&v=1.16.1&c=curl&f=json"
```

---

## ⚙️ Configuration

### Configuration File (config.toml)

```toml
[server]
host = "0.0.0.0"
port = 8080

[scan]
roots = ["/music"]              # Music root directories
mode = "manual"                 # manual / scheduled / watch
interval_hours = 24             # Scheduled scan interval

[database]
path = "./data/db.sqlite"       # Database path

[covers]
cache_path = "./covers"         # Cover art cache directory
max_dimension = 1000            # Max dimension for original images (px)
quality = 85                    # Thumbnail encoding quality
cache_size_mb = 100             # Thumbnail in-memory cache (MB)

[recommend]
job_interval_hours = 6          # Recommendation job interval
play_threshold = 10             # Play count threshold for recommendations
max_results = 50                # Max recommendation results

[transcode]
enabled = true                  # Enable transcoding
cache_strategy = "smart"        # Cache strategy: none/all/smart
cache_threshold = 10            # Smart cache threshold (play count)
max_concurrent_transcodes = 2   # Max concurrent transcodes

[maintenance]
enabled = true                  # Enable automatic DB maintenance
interval_hours = 24             # Maintenance job interval (hours)
history_retention_days = 90     # Play history retention (days)
recommendation_retention_days = 30  # Recommendation cache retention (days)
enable_vacuum = true            # Enable defragmentation
vacuum_threshold = 30.0         # VACUUM trigger threshold (fragmentation %)

[subsonic]
enabled = true                  # Enable Subsonic API compatibility

[web]
enabled = true                  # Enable frontend static file hosting
path = "./web"                  # Frontend build output directory
```

> For the complete configuration reference with detailed comments, see [`config.toml.example.en`](config.toml.example.en).

### Key Configuration Items

The following settings are critical for proper operation and security. Review them carefully before your first deployment:

#### 1. Music Scan Root Directories `scan.roots`

```toml
[scan]
roots = ["/music"]
```

- Specify directories for the service to scan, supports **multiple directories**
- For Docker deployment, mount host music directories into the container and use the **container path** here
- For local development, use the actual absolute path on the host

```toml
# Multiple directories example
roots = ["/music/chinese", "/music/english", "/music/classical"]
```

#### 2. JWT Secret `security.jwt_secret`

```toml
[security]
jwt_secret = "your-random-hex-secret-here"
```

- Used to sign and verify Web frontend / REST API login tokens
- **Not configured** (default): A random key is generated on each restart; all users must re-login
- **Fixed value**: Tokens remain valid across restarts, transparent to users

Generate a recommended value:

```bash
openssl rand -hex 32
```

> During development, you can leave it empty (easier debugging). For production, you **must** set a fixed value. Security depends on length and secrecy, not frequent rotation.

#### 3. Subsonic Password Encryption Key `security.encryption_key`

```toml
[security]
encryption_key = "your-random-hex-key-here"
```

- Used to encrypt stored passwords needed for Subsonic Token authentication
- Default value is a public placeholder string — **security is equivalent to unencrypted**
- With a custom value, even if the database leaks, an attacker also needs this key to decrypt

Generate a recommended value:

```bash
openssl rand -hex 32
```

> **Warning**: Once set and user data exists, do not change this key arbitrarily. Changing it will make all encrypted Subsonic passwords unrecoverable; affected users will need to reset their passwords.

### Environment Variables (Docker Priority)

```bash
# Service config
MUSIC_SERVER_PORT=8080
MUSIC_SCAN_ROOTS=/music
MUSIC_DATABASE_PATH=/data/db.sqlite
MUSIC_COVERS_CACHE_PATH=/covers

# Initial admin (used on first launch)
MUSIC_ADMIN_USER=admin
MUSIC_ADMIN_PASSWORD=your_secure_password
```

## 🐛 Troubleshooting

### Problem: Scan job not responding

**Solution:**
1. Check music directory permissions
2. Check logs: `docker-compose logs -f`
3. Manually trigger scan: `POST /api/scan`

### Problem: Audio won't play

**Solution:**
1. Verify Range request support
2. Check file path is correct
3. Verify file format is supported

### Problem: Recommendations are empty

**Solution:**
1. Ensure enough play history exists (default requires 10 plays)
2. Check if the recommendation job is running: check logs
3. Manually trigger recommendation calculation

## 🤝 Contributing

Issues and Pull Requests are welcome!

1. Fork this project
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Tokio](https://tokio.rs/) — Async runtime
- [Axum](https://github.com/tokio-rs/axum) — Web framework
- [TagLib](https://taglib.org/) — Audio metadata library
- [SQLite](https://www.sqlite.org/) — Embedded database

## 💖 Sponsor & Support

Zhiyin is an open-source project, completely free to use. If you find it helpful, consider supporting development:

- ⭐ Star this project
- 🐛 Submit an Issue or PR
- ☕ Buy the author a coffee

<div align="center">
  <img src="https://raw.githubusercontent.com/qwex888/zhiyin-web/main/docs/donate/alipay.jpg" alt="Alipay" width="200" />
</div>

## 📧 Contact

- Project Home: https://github.com/qwex888/zhiyin
- Issue Tracker: https://github.com/qwex888/zhiyin/issues

---

**Made with ❤️ and Rust**

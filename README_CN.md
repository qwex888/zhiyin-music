<p align="center">
  <img src="assets/logo.svg" width="128" alt="Zhiyin Logo" />
</p>

<h1 align="center">Zhiyin Music 知音音乐</h1>

<p align="center">基于 Rust 的远程 NAS 音乐库 + Web 播放器系统</p>

<p align="center">
  <a href="README.md">English</a>
</p>

[![Rust](https://img.shields.io/badge/rust-1.75+-orange.svg)](https://www.rust-lang.org/)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## 📖 项目简介

Zhiyin Music 知音音乐 是一个专为 NAS 场景设计的音乐流媒体服务，旨在提供：

- 🎼 **大规模音乐库支持**：轻松管理数万首音乐
- 🚀 **高性能**：Rust + SQLite + 异步架构，内存优化至 ~30-50MB
- 📦 **开箱即用**：Docker 一键部署
- 🔍 **智能推荐**：基于播放历史的离线推荐系统
- 🎨 **完整元数据**：封面、艺术家、专辑信息自动解析
- 🌐 **RESTful API**：前后端分离，易于集成
- 📡 **Subsonic API 兼容**：支持音流、Symfonium、DSub 等主流客户端直连
- 🔐 **用户认证**：JWT + Basic Auth + bcrypt，角色权限控制
- 🖥️ **前端托管**：可选的内置 Web 前端服务，支持 SPA
- 🎯 **刮削（Scraping）**：从多个音乐平台自动抓取元数据与封面
- ☁️ **STRM 网盘直链**：支持 .strm 文件接入阿里云盘等网盘资源，服务端代理播放

## ✨ 核心特性

### 音乐库管理
- ✅ 支持多个音乐根目录
- ✅ 增量扫描（基于 mtime/size/hash）
- ✅ 实时文件监听（可选）
- ✅ 支持格式：MP3、FLAC、WAV、M4A、OGG、OPUS、APE
- ✅ 支持 .strm 网盘直链文件（阿里云盘、WebDAV 等）

### 元数据解析
- ✅ 使用 TagLib 解析音频元信息
- ✅ 自动提取内嵌封面
- ✅ 支持外部封面文件（cover.jpg、folder.jpg）
- ✅ 封面哈希去重，节省存储空间

### STRM 网盘音源
- ✅ 支持读取 `.strm` 文件中的远程 URL（阿里云盘、WebDAV、Alist 等）
- ✅ `source_type` 字段区分本地歌曲与网盘歌曲
- ✅ 服务端代理模式（proxy）：使用 curl 强制 IPv4 转发远程流，兼容国内 CDN
- ✅ 重定向模式（redirect）：307 跳转客户端直连，节省服务端带宽
- ✅ 从文件名和目录结构智能推断元数据（艺术家/专辑/曲号）
- ✅ 可选 HTTP HEAD 远程探测文件大小，呈现真实音频属性
- ✅ 与本地歌曲完全一致的功能体验（封面、歌词、播放历史、推荐）
- ✅ 自动刮削兼容：智能解析 "艺术家 - 歌名" 文件名格式

### 刮削（Scraping）
- ✅ 支持平台：网易云音乐、QQ 音乐、酷狗、酷我、咪咕
- ✅ 自动刮削模式（按评分阈值自动应用最优匹配）
- ✅ 人工审核模式（低置信度结果暂存，人工确认后再写入）
- ✅ 封面下载并嵌入音频文件（支持 MP3/FLAC/M4A/OGG 等）
- ✅ 鲁棒封面写入：按文件内容魔数检测格式（而非扩展名），正确处理 FLAC/ID3v2/MP3 误标文件
- ✅ 刮削后封面直接持久化到 cover-store，`cover_id` 实时更新 DB（文件嵌入失败不影响封面显示）
- ✅ 歌词抓取并写入嵌入标签 / 侧车 `.lrc` 文件
- ✅ STRM 歌曲刮削：不嵌入标签，仅保存外部封面和 `.lrc` 歌词文件
- ✅ 刮削日志与操作历史

### 音频播放
- ✅ HTTP Range 请求支持（无缝 seek）
- ✅ **多档音质选择（128k/192k/320k/无损）**
- ✅ **智能转码缓存（基于播放次数）**
- ✅ **FFmpeg 管道流式输出**（实时转码边转边播，无需等待完整文件）
- ✅ 原格式播放（无需转码）
- ✅ 播放历史记录

### Subsonic API 兼容
- ✅ **Subsonic API v1.16.1 兼容层**（挂载于 `/rest/`，40+ 端点）
- ✅ 支持音流、Ultrasonic、Symfonium、DSub 等主流客户端
- ✅ 多种认证方式（明文密码 / Hex 编码 / Token 认证）
- ✅ XML/JSON 双格式响应（`f=json` / `f=xml`）
- ✅ **已实现端点**：
  - 🔹 系统：`ping`、`getLicense`、`getUser`
  - 🔹 浏览：`getMusicFolders`、`getIndexes`、`getArtists`、`getArtist`、`getAlbum`、`getSong`、`getRandomSongs`、`getAlbumList2`
  - 🔹 媒体：`stream`、`download`、`getCoverArt`
  - 🔹 搜索：`search3`、`search2`（ID3 搜索）
  - 🔹 记录：`scrobble`（播放记录上报）
  - 🔹 扫描：`getScanStatus`、`startScan`（实时进度）
- ⬜ **Stub 端点**（已注册路由，返回空响应保证客户端不报错）：
  - 播放列表、收藏/评分、书签、歌词、相似推荐、电台/播客等

### 用户认证与权限
- ✅ 用户系统（SQLite `users` 表，bcrypt 密码存储）
- ✅ JWT Token 认证（REST API）
- ✅ Basic Auth 认证（REST API）
- ✅ 角色权限控制（admin / user）
- ✅ 初始管理员创建（环境变量 或 `POST /api/setup`）
- ✅ 用户管理 API（创建/修改/删除/重置密码）
- ✅ Subsonic API 统一使用数据库用户认证

### Web 前端托管（可选）
- ✅ 内置 HTTP 静态文件服务
- ✅ SPA 路由 fallback（自动回退到 `index.html`）
- ✅ 通过 Docker volume 挂载或直接放置前端构建产物
- ✅ 可随时替换为任意前端框架构建产物

### 个性化推荐
- ✅ 高频播放推荐
- ✅ 最近常听推荐
- ✅ 相似度推荐（艺术家/专辑/目录）
- ✅ 后台异步计算，不阻塞 API

### Docker 部署
- ✅ 多阶段构建，镜像体积小
- ✅ 支持 x86_64 和 ARM64
- ✅ 健康检查与自动重启
- ✅ 数据持久化

## 🏗️ 架构设计

```
API Layer → Service Layer → Job System → Data Access
    ↓           ↓               ↓            ↓
 Axum      业务逻辑        后台任务      SQLite
```

## 🚀 快速开始

### 方式一：Docker Compose（推荐）

**修改配置**

编辑 `docker-compose.yml`，设置你的音乐目录：
```yaml
volumes:
  - /path/to/your/music:/music:ro  # 修改为实际路径
```

创建配置文件
```bash
cp config.toml.example config.toml
```

**启动服务**
```bash
docker-compose up -d
```

**创建管理员**

方式一：通过前端页面，会在系统初始化时自动引导创建。

方式二：通过环境变量（在 `docker-compose.yml` 中设置）：
```yaml
environment:
  - MUSIC_ADMIN_USER=admin
  - MUSIC_ADMIN_PASSWORD=your_secure_password
```

方式三：通过 API 初始化：
```bash
curl -X POST http://localhost:8080/api/setup \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

**触发扫描**

前端设置页面，手动触发扫描和配置扫描

或

```bash
curl -X POST http://localhost:8080/api/scan \
  -H "Authorization: Bearer <your-jwt-token>"
```

8. **访问 API**

- API 文档地址：http://localhost:8080/swagger-ui

## 📚 API 文档

### 核心接口

#### 公开接口（无需认证）

| 接口 | 方法 | 描述 |
|------|-----|------|
| `/api/auth/status` | GET | 查询系统初始化状态 |
| `/api/setup` | POST | 首次创建管理员（仅限系统未初始化时） |
| `/api/auth/login` | POST | 登录获取 JWT Token |
| `/api/health` | GET | 健康检查 |

#### 需要认证的接口

| 接口 | 方法 | 描述 |
|------|-----|------|
| `/api/songs` | GET | 获取歌曲列表（分页） |
| `/api/songs/:id` | GET | 获取单个歌曲详情 |
| `/api/songs/batch` | POST | 批量查询歌曲详情 |
| `/api/albums` | GET | 获取专辑列表 |
| `/api/artists` | GET | 获取艺术家列表（含歌曲数、专辑数、封面） |
| `/api/stream/{id}?quality=high` | GET | 播放音频（支持音质选择+Range） |
| `/api/covers/{id}` | GET | 获取封面图片 |
| `/api/scan` | POST | 触发扫描任务 |
| `/api/recommend` | GET | 获取推荐歌曲（含完整歌曲信息） |
| `/api/history/recent` | GET | 获取最近播放的歌曲 |
| `/api/config` | GET | 获取当前配置（含重启标记） |
| `/api/config` | PUT | 更新配置（自动保存并重载） |
| `/api/users` | GET | 获取用户列表（管理员） |
| `/api/users` | POST | 创建用户（管理员） |
| `/api/users/me` | GET | 获取当前用户信息 |
| `/api/users/:id` | PUT | 更新用户信息 |
| `/api/users/me/password` | PUT | 修改自己的密码 |
| `/api/users/:id/reset-password` | POST | 重置用户密码（管理员） |
| `/api/users/:id` | DELETE | 删除用户（管理员） |

#### Subsonic 兼容接口

| 接口 | 方法 | 描述 |
|------|-----|------|
| `/rest/ping` | GET | 心跳检测 |
| `/rest/getArtists` | GET | 获取艺术家索引 |
| `/rest/stream?id=1` | GET | 流式播放 |
| `/rest/search3?query=xxx` | GET | 搜索 |

---

### 歌曲接口

#### 1. 获取歌曲列表（分页）

```bash
curl "http://localhost:8080/api/songs?limit=10&offset=0"
```

**响应：**
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

#### 2. 获取单个歌曲详情

```bash
curl http://localhost:8080/api/songs/1
```

**响应：**
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

**错误响应（404）：**
```json
{
  "error": "not_found",
  "message": "歌曲 ID 1 不存在"
}
```

#### 3. 批量查询歌曲详情

```bash
curl -X POST http://localhost:8080/api/songs/batch \
  -H "Content-Type: application/json" \
  -d '{"ids": [1, 2, 3, 4, 5]}'
```

**响应：**
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

> 注意：不存在的歌曲 ID 会被自动过滤，结果按输入 ID 顺序返回。

---

### 艺术家接口

#### 获取艺术家列表（含统计信息）

```bash
# 获取艺术家列表
curl "http://localhost:8080/api/artists?limit=20&offset=0"
```

**响应示例**:
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

**增强功能**:
- **歌曲数量** (`song_count`): 该艺术家的歌曲总数
- **专辑数量** (`album_count`): 该艺术家的专辑总数
- **封面 ID** (`cover_id`): 艺术家代表封面，可为 `null`

**封面获取策略**（优先级顺序）:
1. **最新专辑封面** - 优先选择该艺术家最新发行的专辑封面（按 year 降序）
2. **首张有封面专辑** - 如果没有 year 信息，则选择第一张有封面的专辑
3. **歌曲封面** - 如果专辑都没有封面，则从该艺术家的歌曲中选择一个封面
4. **NULL** - 如果都没有封面，返回 `null`，前端可显示默认占位图

**获取封面图片**:
```bash
# 如果 cover_id 不为 null
curl http://localhost:8080/api/covers/123 > artist_cover.jpg
```

---

### 推荐接口

#### 获取推荐歌曲（含完整歌曲信息）

```bash
curl http://localhost:8080/api/recommend
```

**响应：**
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

> 推荐结果按评分从高到低排序，包含完整的歌曲元数据，可直接用于播放器展示。

---

### 播放历史接口

#### 获取最近播放的歌曲（含完整歌曲信息）

```bash
# 获取最近 20 首（默认）
curl http://localhost:8080/api/history/recent

# 获取最近 50 首
curl "http://localhost:8080/api/history/recent?limit=50"

# 获取最近 100 首（最大值）
curl "http://localhost:8080/api/history/recent?limit=100"
```

**查询参数**:
- `limit` (可选): 返回的歌曲数量，默认 20，最大 100

**响应：**
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

**特性**:
- 返回最近播放的歌曲（去重，每首歌只返回最后一次播放时间）
- 按播放时间倒序排列（最新的在前）
- 包含完整歌曲信息和最后播放时间 `played_at`
- 支持自定义返回数量（1-100 首）

---

### 配置管理接口

#### 1. 获取当前配置

```bash
curl http://localhost:8080/api/config
```

**响应示例**:
```json
{
  "scan": {
    "roots": ["/Users/user/Music"],
    "mode": "manual",
    "interval_hours": 24,
    "_meta": {
      "mode": {
        "description": "扫描模式：manual/scheduled/watch",
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

> 每个配置项的 `_meta` 字段标注了该配置是否需要重启服务才能生效。

#### 2. 更新配置

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

**响应示例**:
```json
{
  "success": true,
  "message": "成功更新 3 项配置",
  "requires_restart": false,
  "updated_fields": [
    "scan.mode",
    "scan.interval_hours",
    "recommend.max_results"
  ]
}
```

**特性**:
- 支持部分更新（只更新指定的字段）
- 自动保存到 `config.toml` 文件
- 自动触发配置重载（Unix 系统）
- 明确标注是否需要重启

**可更新的配置项**:
- 扫描配置：`scan.roots`、`scan.mode`、`scan.interval_hours`
- 推荐配置：`recommend.job_interval_hours`、`recommend.play_threshold`、`recommend.max_results`
- 转码配置：`transcode.enabled`、`transcode.cache_strategy`、`transcode.cache_threshold`
- 维护配置：`maintenance.*`（所有维护相关配置）

**不可修改的配置**（需要重启）:
- `server.host`、`server.port`
- `database.path`、`database.pool_size`
- `transcode.cache_path`

> 完整 API 文档请访问 Swagger UI：`http://localhost:8080/swagger-ui/`

---

### 认证接口

#### 1. 系统初始化（创建首个管理员）

```bash
# 查看初始化状态
curl http://localhost:8080/api/auth/status
```

```json
{
  "initialized": false,
  "message": "系统尚未初始化，请创建管理员账户"
}
```

```bash
# 创建管理员（仅限首次，初始化后此接口自动关闭）
curl -X POST http://localhost:8080/api/setup \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

#### 2. 登录

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your_secure_password"}'
```

**响应：**
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

#### 3. 使用 Token 访问受保护接口

```bash
# Bearer Token（推荐）
curl -H "Authorization: Bearer <token>" http://localhost:8080/api/songs

# 或 Basic Auth
curl -u admin:your_secure_password http://localhost:8080/api/songs
```

---

### 音频流接口

#### 播放音频（支持音质选择 + 管道流式转码）

```bash
# 播放原始音质（默认，支持 Range 请求）
curl http://localhost:8080/api/stream/1 > song.mp3

# 选择高品质（320kbps）
curl "http://localhost:8080/api/stream/1?quality=high" > song_high.mp3

# 音质选项：low(128k) / medium(192k) / high(320k) / lossless / original

# 支持 Range 请求（原始文件或已缓存文件）
curl -H "Range: bytes=0-1023" "http://localhost:8080/api/stream/1?quality=high"
```

**流式转码说明：**
- 原始文件 / 已缓存文件 → `ServeFile` 响应（支持 Range/ETag/断点续传）
- 需实时转码且不缓存 → FFmpeg 管道流式输出（`Transfer-Encoding: chunked`，边转边播，无需等待完整文件）
- 缓存策略由 `transcode.cache_strategy` 控制：`none`（全走管道流）、`all`（全缓存）、`smart`（播放超过阈值后缓存）

### Subsonic API 兼容接口

项目实现了 Subsonic API v1.16.1 兼容层，挂载在 `/rest/` 路径下，可直接使用 Ultrasonic、Symfonium、DSub 等客户端连接。

#### 启用方式

在 `config.toml` 中启用：

```toml
[subsonic]
enabled = true
```

#### 客户端连接配置

| 配置项 | 值 |
|--------|-----|
| 服务器地址 | `http://your-server:8080` |
| 用户名 | 系统中创建的用户名（通过 `/api/setup` 或环境变量创建） |
| 密码 | 用户密码 |
| 认证模式 | 明文密码 / Token 认证均支持 |

#### 支持的端点

| 分类 | 端点 | 描述 |
|------|------|------|
| System | `/rest/ping` | 心跳检测 |
| System | `/rest/getLicense` | 许可证信息 |
| Browsing | `/rest/getMusicFolders` | 获取音乐目录 |
| Browsing | `/rest/getIndexes` | 按字母索引艺术家 |
| Browsing | `/rest/getArtists` | 获取所有艺术家（ID3） |
| Browsing | `/rest/getArtist?id=1` | 获取艺术家及其专辑 |
| Browsing | `/rest/getAlbum?id=1` | 获取专辑及其歌曲 |
| Browsing | `/rest/getSong?id=1` | 获取单首歌曲信息 |
| Browsing | `/rest/getRandomSongs?size=10` | 随机歌曲 |
| Media | `/rest/stream?id=1` | 流式播放（支持 `maxBitRate`） |
| Media | `/rest/download?id=1` | 下载原始文件 |
| Media | `/rest/getCoverArt?id=1` | 获取封面图 |
| Search | `/rest/search3?query=keyword` | 搜索歌曲/专辑/艺术家 |
| Annotation | `/rest/scrobble?id=1` | 记录播放历史 |
| Library | `/rest/getScanStatus` | 获取扫描状态 |
| Library | `/rest/startScan` | 触发扫描 |

所有端点同时支持 `.view` 后缀（如 `/rest/ping.view`）。

#### 认证方式

支持以下三种 Subsonic 标准认证方式：

**1. 明文密码认证：**
```
/rest/ping?u=admin&p=your-password&v=1.16.1&c=myapp
```

**2. Hex 编码密码认证：**
```
/rest/ping?u=admin&p=enc:796f75722d70617373776f7264&v=1.16.1&c=myapp
```

**3. Token 认证（推荐）：**
```
/rest/ping?u=admin&t=md5-token&s=random-salt&v=1.16.1&c=myapp
```

> Token 认证的工作原理：客户端使用 `MD5(密码 + salt)` 生成 token，服务端用加密存储的密码验证。首次使用明文方式登录后，系统会自动为该用户启用 Token 认证支持。

#### 响应格式

默认返回 XML，添加 `f=json` 参数切换为 JSON：
```bash
curl "http://localhost:8080/rest/ping?u=admin&p=your-password&v=1.16.1&c=curl&f=json"
```

---

## ⚙️ 配置说明

### 配置文件（config.toml）

```toml
[server]
host = "0.0.0.0"
port = 8080

[scan]
roots = ["/music"]              # 音乐根目录
mode = "manual"                 # manual / scheduled / watch
interval_hours = 24             # 定时扫描间隔
scan_strm = true                # 是否扫描 .strm 网盘直链文件
strm_mode = "proxy"             # proxy（服务端代理） / redirect（307 重定向）
strm_probe_remote = false       # 是否远程探测 strm 音频属性（文件大小等）

[database]
path = "./data/db.sqlite"       # 数据库路径

[covers]
cache_path = "./covers"         # 封面缓存目录
max_dimension = 1000            # 原图最大边长（像素）
quality = 85                    # 缩略图编码质量
cache_size_mb = 100             # 缩略图内存缓存大小（MB）

[recommend]
job_interval_hours = 6          # 推荐任务间隔
play_threshold = 10             # 触发推荐的播放次数
max_results = 50                # 推荐结果数量

[transcode]
enabled = true                  # 启用转码功能
cache_strategy = "smart"        # 缓存策略：none/all/smart
cache_threshold = 10            # 智能缓存阈值（播放次数）
max_concurrent_transcodes = 2   # 最大并发转码数

[maintenance]
enabled = true                  # 启用自动数据库维护
interval_hours = 24             # 维护任务执行间隔（小时）
history_retention_days = 90     # 播放历史保留天数
recommendation_retention_days = 30  # 推荐记录保留天数
enable_vacuum = true            # 启用碎片清理
vacuum_threshold = 30.0         # VACUUM 触发阈值（碎片率%）

[subsonic]
enabled = true                  # 启用 Subsonic API 兼容层

[web]
enabled = true                  # 启用前端静态文件托管
path = "./web"                  # 前端构建产物目录
```

### 重点配置项

以下配置项对服务的正常运行和安全性至关重要，请在首次部署前仔细确认：

#### 1. 音乐扫描根目录 `scan.roots`

```toml
[scan]
roots = ["/music"]
```

- 指定服务需要扫描的音乐文件目录，支持配置**多个目录**
- Docker 部署时，需将宿主机的音乐目录挂载到容器内，并在此处填写**容器内路径**
- 本地开发时，填写宿主机上的实际绝对路径

```toml
# 多目录示例
roots = ["/music/chinese", "/music/english", "/music/classical"]
```

#### 2. JWT 密钥 `security.jwt_secret`

```toml
[security]
jwt_secret = "your-random-hex-secret-here"
```

- 用于签发和验证 Web 前端 / REST API 的登录 Token
- **不配置**（默认）：每次服务重启后随机生成密钥，所有用户需重新登录
- **配置固定值**：服务重启后 Token 仍然有效，用户无感知

生成推荐值：

```bash
openssl rand -hex 32
```

> 开发阶段可以留空（方便调试），正式部署前**必须**配置固定值。密钥的安全性取决于其长度和保密性，而非是否频繁更换。

#### 3. Subsonic 密码加密密钥 `security.encryption_key`

```toml
[security]
encryption_key = "your-random-hex-key-here"
```

- 用于加密存储 Subsonic Token 认证所需的明文密码
- 默认值为公开的占位字符串，**安全性等同于未加密**
- 配置自定义值后，即使数据库泄露，攻击者也需要同时获取此密钥才能解密

生成推荐值：

```bash
openssl rand -hex 32
```

> **注意**：一旦确定此密钥并产生用户数据后，不可随意更改。更改后所有已加密的 Subsonic 密码将无法解密，受影响用户需要重新修改密码。

### 环境变量（Docker 优先）

```bash
# 服务配置
MUSIC_SERVER_PORT=8080
MUSIC_SCAN_ROOTS=/music
MUSIC_DATABASE_PATH=/data/db.sqlite
MUSIC_COVERS_CACHE_PATH=/covers

# 初始管理员（首次启动时使用）
MUSIC_ADMIN_USER=admin
MUSIC_ADMIN_PASSWORD=your_secure_password
```

## 🐛 故障排查

### 问题：扫描任务没有响应

**解决方案：**
1. 检查音乐目录权限
2. 查看日志：`docker-compose logs -f`
3. 手动触发扫描：`POST /api/scan`

### 问题：音频无法播放

**解决方案：**
1. 确认 Range 请求支持
2. 检查文件路径是否正确
3. 验证文件格式是否支持

### 问题：推荐结果为空

**解决方案：**
1. 确保有足够的播放历史（默认需要 10 次）
2. 检查推荐任务是否运行：查看日志
3. 手动触发推荐计算（需实现触发接口）

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m '添加某某功能'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

- [Tokio](https://tokio.rs/) - 异步运行时
- [Axum](https://github.com/tokio-rs/axum) - Web 框架
- [TagLib](https://taglib.org/) - 音频元数据库
- [SQLite](https://www.sqlite.org/) - 嵌入式数据库

## 💖 赞助与支持

Zhiyin Music 知音是一个开源项目，完全免费使用。如果它对你有帮助，可以通过以下方式支持开发：

- ⭐ 给项目一个 Star
- 🐛 提交 Issue 或 PR
- ☕ 请作者喝杯咖啡

<div align="center">
  <img src="https://raw.githubusercontent.com/qwex888/zhiyin-music-web/main/docs/donate/alipay.jpg" alt="支付宝" width="200" />
</div>

## 📧 联系方式

- 项目主页：https://github.com/qwex888/zhiyin-music
- 问题反馈：https://github.com/qwex888/zhiyin-music/issues

---

**Made with ❤️ and Rust**


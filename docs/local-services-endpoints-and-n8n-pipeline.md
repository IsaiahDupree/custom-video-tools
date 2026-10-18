# Local Services: Endpoints and n8n Chaining Guide

This document catalogs the local services provided by this repo, their endpoints, and how to chain them together in n8n to build an end‑to‑end pipeline.

- Enhanced Music & Video API (Docker service: `video-generator`)
- FlaskDrive (Docker service: `flaskdrive`)
- n8n (Docker service: `n8n`)

References to source code:
- Enhanced API: `music_system/enhanced_flask_api.py`
- FlaskDrive: `flaskdrive/app.py`

Notes on ports:
- Default (per `docker-compose.yml`):
  - n8n → http://localhost:5678
  - Enhanced API → http://localhost:5001
  - FlaskDrive → http://localhost:5000
- Some stacks may map to alternate host ports (e.g., 5679/5003/5005). If unsure, check Docker UI or run `docker compose ps`.
- Inside the n8n container, use service DNS names and container ports:
  - Enhanced API → http://video-generator:5001
  - FlaskDrive → http://flaskdrive:5000

---

## Enhanced Music & Video API (video-generator)
Base URL (host): `http://localhost:5001`
Base URL (from n8n container): `http://video-generator:5001`

### Health & System
- GET `/health`
  - Use case: Liveness check.
- GET `/system/status`
  - Use case: Full status including ffmpeg, music DB stats, library counts.

### Music Database
- POST `/music/preprocess`
  - Body: `{ "force_reanalyze": false }`
  - Use case: Scan and index local music for search/matching.
- GET `/music/database/stats`
  - Use case: Quick statistics of the indexed library.
- POST `/music/search`
  - Body (any subset):
    ```json
    {
      "mood": "energetic",
      "genre": "dubstep",
      "energy_min": 0.4,
      "energy_max": 0.9,
      "tempo_min": 90,
      "tempo_max": 160,
      "duration_min": 10,
      "duration_max": 180,
      "limit": 50
    }
    ```
  - Use case: Filtered music discovery for a video.

### Analysis & Matching
- POST `/analyze-music-match`
  - Body:
    ```json
    {
      "video_path": "C:/path/to/video.mp4",
      "transcript": "optional",
      "visual_analysis": "optional",
      "description": "optional"
    }
    ```
  - Use case: Pick a best-fit music track for the video.
- POST `/video/analyze-content`
  - Body: `{ "video_path": "C:/path/to/video.mp4" }`
  - Use case: Intelligent video analysis to produce `suggested_overlay`, transcript, and context.
- POST `/video/analyze-with-openai`
  - Body: `{ "video_path": "...", "openai_api_key": "optional-override" }`
  - Use case: OpenAI-driven overlay suggestion (explicit key override supported).

### Processing
- POST `/video/process`
  - Body (required: `video_path`, `music_path`):
    ```json
    {
      "video_path": "C:/path/to/video.mp4",
      "music_path": "C:/path/to/music.mp3",
      "text_lines": ["Line 1", "Line 2"],
      "music_volume_db": -25.0,
      "text_position": "bottom",
      "preserve_audio": true,
      "output_name": "processed_custom_name.mp4"
    }
    ```
  - Use case: Core processor to add background music and overlay text.
- POST `/video/process-enhanced`
  - Body (required: `video_path`; `music_path` optional or `"auto"`):
    ```json
    {
      "video_path": "C:/path/to/video.mp4",
      "music_path": "auto",
      "text_lines": ["Auto overlay or custom"],
      "music_volume_db": -25.0,
      "preserve_audio": true,
      "smart_positioning": true,
      "font_style": "auto|creative|bold|clean|modern",
      "output_name": "enhanced_processed.mp4"
    }
    ```
  - Use case: Smarter text placement, font selection, and mixing.
- POST `/video/analyze-and-process`
  - Body:
    ```json
    {
      "video_path": "C:/path/to/video.mp4",
      "transcript": "optional",
      "visual_analysis": "optional",
      "text_lines": ["Overlay text"],
      "music_volume_db": -25.0
    }
    ```
  - Use case: One-call pipeline that matches music then processes the video.
- POST `/video/batch-process`
  - Body:
    ```json
    {
      "video_directory": "C:/path/to/folder",
      "music_volume_db": -25.0,
      "include_text": true
    }
    ```
  - Use case: Batch a folder of videos end‑to‑end.

### Fonts
- GET `/fonts/list`
  - Use case: Enumerate custom/system fonts available to the processor.
- POST `/fonts/test`
  - Body:
    ```json
    {
      "video_path": "C:/path/to/video.mp4",
      "font_style": "auto",
      "test_text": ["Font Test", "Sample Text"]
    }
    ```
  - Use case: Quickly preview a font rendering on a sample video.

### Download
- GET `/download/file/<path:file_path>`
  - Use case: Download a processed output by absolute path returned from processing endpoints.

### Example curl snippets
```bash
# Health
curl http://localhost:5001/health

# Analyze content
curl -X POST http://localhost:5001/video/analyze-content \
  -H "Content-Type: application/json" \
  -d '{"video_path":"C:/videos/clip.mp4"}'

# Enhanced process (auto-pick music)
curl -X POST http://localhost:5001/video/process-enhanced \
  -H "Content-Type: application/json" \
  -d '{"video_path":"C:/videos/clip.mp4","music_path":"auto","text_lines":["Hello world"],"music_volume_db":-25}'

# Download (replace with actual output_path from response)
curl -L "http://localhost:5001/download/file/C:/Users/.../processed_videos/enhanced_processed_20250101_120000.mp4" -o out.mp4
```

---

## FlaskDrive (flaskdrive)
Base URL (host): `http://localhost:5000`
Base URL (from n8n container): `http://flaskdrive:5000`

- GET `/` → If not authenticated, redirects to `/authorize`. If authenticated, returns `{ status: "authenticated" }`.
- GET `/authorize` → Starts OAuth; if already authenticated returns a small JSON with `already_authorized`.
- GET `/oauth2callback` → OAuth callback.
- POST `/upload` (multipart with binary)
  - Form fields:
    - `file` (binary)
    - `folder_id` (optional Drive folder)
    - Optional metadata passthrough: `videoPath`, `videoName`, `detectedAt`, `fileSize`, `binaryId`
  - Returns: `{ id, download_link, view_link, name, size, success: true, ... }`
- GET `/health` → `{ status: "healthy", authenticated: <bool> }`
- GET `/status` → Credentials status summary.
- GET `/file-info/<file_id>` → Drive file metadata and links.
- GET `/search?q=<name>` → Search by name, returns files list.

### Example curl (upload)
```bash
# Upload a local file to Drive (host → container)
curl -X POST http://localhost:5000/upload \
  -F file=@"C:/path/to/out.mp4" \
  -F folder_id="YOUR_DRIVE_FOLDER_ID"
```

---

## n8n: Chaining the Processes
This section outlines a practical flow to analyze a video, process it with music and text, then upload the output to Google Drive.

All node URLs below assume you run inside the n8n container (use `video-generator` and `flaskdrive` DNS names). If you run the nodes from your host, switch to `http://localhost:<port>`.

### Single‑Video Pipeline (analyze → process → download → upload)
1. Set (input)
   - Fields: `video_path` (absolute path on the host that is also bind‑mounted into the containers if needed)
2. HTTP Request: Analyze Content
   - Method: POST
   - URL: `http://video-generator:5001/video/analyze-content`
   - JSON: `{ "video_path": "={{ $json.video_path }}" }`
3. Set: Build text_lines
   - Map `text_lines` to an array, e.g., `[ {{$json.suggested_overlay}} ]`
4. HTTP Request: Match Music (optional if using auto)
   - Method: POST
   - URL: `http://video-generator:5001/analyze-music-match`
   - JSON: `{ "video_path": "={{ $json.video_path }}" }`
5. HTTP Request: Process Enhanced
   - Method: POST
   - URL: `http://video-generator:5001/video/process-enhanced`
   - JSON example:
     ```json
     {
       "video_path": "={{ $json.video_path }}",
       "music_path": "={{ $json.music_recommendation?.file_path || 'auto' }}",
       "text_lines": "={{ $json.text_lines }}",
       "music_volume_db": -25,
       "preserve_audio": true,
       "smart_positioning": true
     }
     ```
6. HTTP Request: Download Output (binary)
   - Method: GET
   - URL: `http://video-generator:5001/download/file/={{ encodeURIComponent($json.output_path) }}`
   - Options: `Download: true`, Binary Property: `data`
7. HTTP Request: Upload to Google Drive (via FlaskDrive)
   - Method: POST
   - URL: `http://flaskdrive:5000/upload`
   - Options: `Send Binary Data: true`, Binary Property: `data`
   - Additional Form Fields (optional): `folder_id`
8. (Optional) HTTP Request: `/file-info/{{ $json.id }}` to confirm or gather metadata for downstream steps.

Tip: You can reuse the example workflow `workflows/google-drive-upload/google-drive-upload-fixed.json` for steps 6–7 (download → upload), wiring its input from the processing node response.

### Batch Pipeline (folder)
- Use HTTP Request → `POST http://video-generator:5001/video/batch-process` with `{ "video_directory": "..." }`.
- Loop through `summary.results` and for each successful item, `GET /download/file/...` → upload via FlaskDrive.

---

## Operational Tips
- Environment variables (recommended):
  - `OPENAI_API_KEY` for analysis endpoints
  - Ensure the host directories in `docker-compose.yml` (`shared/videos`, `soundcloud_music`, `cache`, etc.) exist and are accessible in the containers.
- When calling from n8n, prefer container DNS names to avoid host/port mapping issues.
- If an endpoint returns a file path, use `encodeURIComponent()` when putting it into a URL path.
- For large files, `FlaskDrive` uploads in 16MB chunks and returns both a `download_link` and `view_link`.

---

## Quick Verification Checklist
- `GET http://localhost:5001/health` → Enhanced API healthy
- `GET http://localhost:5000/health` → FlaskDrive healthy and/or authenticated
- `POST http://localhost:5001/video/analyze-content` with a valid `video_path` → returns `suggested_overlay`
- `POST http://localhost:5001/video/process-enhanced` → returns `output_path`
- `GET /download/file/<output_path>` → streams the processed file
- `POST http://localhost:5000/upload` (binary) → returns Drive `id`, `download_link`, `view_link`

If you want, I can also generate a ready‑to‑import n8n workflow JSON that wires these nodes together. Let me know and I’ll add it under `workflows/pipelines/`.

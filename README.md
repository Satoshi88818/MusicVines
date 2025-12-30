# MusicVines Pro V2.2

**Grok Imagine-powered short music video app**

MusicVines Pro is a Flask-based backend API that transforms user-provided (or Grok-generated) base videos into polished, vertical short music videos optimized for platforms like TikTok and Instagram Reels.

It adds professional enhancements such as:
- Parallax zoom effects
- Custom reactive waveform overlays
- Text title/caption overlays
- Proper audio normalization, looping/trimming, and fades
- Intelligent caption optimization and viral analysis powered by Grok 4

**Fully legal**: Only processes **user-provided audio** — no bundled music, ensuring zero copyright issues.

## Features

- **Video Enhancement Pipeline**:
  - Parallax zoom for dynamic feel
  - Real-time reactive audio waveform (matplotlib-based, with cyan fill)
  - Title and caption overlays with stroke for readability
  - Peak-based audio normalization (safe for silent clips)
  - Audio fading and perfect looping/trimming to video duration

- **Grok 4 Integration**:
  - Analyzes final video keyframes to suggest optimized captions, 8–12 hashtags, and 3 viral tips

- **Asynchronous Processing**:
  - Celery + Redis for background tasks with progress updates

- **Security & Rate Limiting**:
  - Per-user/IP rate limits
  - CSRF protection
  - CORS configuration
  - Secure file uploads

- **Future-Ready**:
  - Hooks for upcoming Grok text-to-video generation (currently falls back to user-uploaded base videos)

## How It Works

1. **Upload** (optional):
   - `/upload_audio` – Upload your music/track
   - `/upload_base` – Upload a base video (e.g., generated in Grok app/web and downloaded)

2. **Generate**:
   - POST to `/clip_grok` with:
     - `prompt` (used as title and for future video gen)
     - `caption`
     - `audio_filename` (optional)
     - `base_video_filename` (required until text-to-video API is public)

3. **Track Progress**:
   - Poll `/task_status/<task_id>`

4. **Result**:
   - Final 1080×1920 MP4 URL
   - Grok 4-optimized caption, hashtags, and viral analysis

## Setup & Running

```bash
# Clone and install dependencies
git clone <your-repo>
cd musicvines
pip install -r requirements.txt  # (create from imports if needed)

# Environment variables (.env)
XAI_API_KEY=your_xai_api_key
FLASK_SECRET_KEY=strong_secret
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0
REDIS_URL=redis://localhost:6379/0
ALLOWED_ORIGINS=*

# Create uploads folder
mkdir uploads

# Run Redis (docker or local)
docker run -p 6379:6379 redis

# Start Celery worker
celery -A app.celery worker --loglevel=info

# Run Flask app
python MusicVinesv2.2.txt  # or your script name
```

App runs on `http://localhost:5000`

## Current Limitations and Known Issues

As of **December 31, 2025**:

1. **No Public Text-to-Video API from xAI**  
   Grok Imagine supports text-to-video generation directly in the Grok app/web (including short clips with audio), but there is **no public API endpoint** for programmatic video generation.  
   → The `generate_grok_video()` function will always fail/fallback. Users must generate videos manually in the Grok interface, download them, and upload via `/upload_base`.

2. **Performance Bottlenecks**  
   - Reactive waveform uses matplotlib per frame → CPU-intensive and potential memory leaks on long/high-volume runs.  
   - Full audio loaded multiple times during waveform rendering.  
   - Keyframe extraction decodes the final video twice (once for rendering, once for analysis).

3. **Hard-Coded Assumptions**  
   - Fixed 1080×1920 resolution and 9:16 aspect ratio. Non-matching inputs may distort.  
   - Fallback audio from base video often silent or low-quality (Grok videos typically lack strong audio).

4. **Resource Management**  
   - No automatic cleanup of old uploaded/base/final files → disk can fill up over time.  
   - Potential minor memory leaks from repeated matplotlib figure creation.

5. **Scalability & Robustness**  
   - Relies on local Redis/Celery — needs proper queuing/supervision for production.  
   - Limited input validation (e.g., duration caps, mime checks).  
   - Waveform and enhancement pipeline can be slow (30–120s per clip depending on hardware).

6. **Dependency on External APIs**  
   - Requires valid `XAI_API_KEY` for Grok 4 analysis and image fallbacks. Rate limits/costs apply.

These are primarily due to current xAI API constraints and optimization opportunities. The core enhancement engine works excellently for polishing existing Grok-generated clips.

Contributions welcome to address these (e.g., pre-computed waveforms, auto-cleanup cron, resolution detection)! 🚀

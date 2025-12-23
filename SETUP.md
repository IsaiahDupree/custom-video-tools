# Setup Guide

This guide will help you set up the Custom Video Processing Tools suite.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux)
- **Docker Compose** (usually included with Docker Desktop)
- **Git** (for cloning the repository)
- **Python 3.8+** (for local development, optional)

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd custom-video-tools
```

### 2. Configure Environment Variables

Create a `.env` file in the root directory:

```bash
# .env
OPENAI_API_KEY=your-openai-api-key-here
FLASK_SECRET_KEY=your-secure-random-string-here
```

**Note**: Never commit the `.env` file to git. It's already in `.gitignore`.

### 3. Set Up Google Drive Authentication (for FlaskDrive)

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Drive API
4. Create OAuth 2.0 credentials:
   - Go to "Credentials" → "Create Credentials" → "OAuth client ID"
   - Select "Web application"
   - Add authorized redirect URI: `http://localhost:5000/oauth2callback`
   - Download the JSON file
5. Place the JSON file in `flaskdrive/` directory as `client_secret.json`

**Important**: The `client_secret.json` file is in `.gitignore` and should not be committed.

### 4. Prepare Shared Resources

Ensure the following directories exist and contain the necessary files:

- `shared/fonts/` - Font files for text overlays
- `shared/videos/` - Input videos (optional, can be empty)
- `soundcloud_music/` - Music tracks (optional, can be downloaded later)

### 5. Start Services

```bash
docker-compose up -d
```

This will start:
- FlaskDrive on port 5000
- Video Generator (Music System) on port 5001

### 6. Verify Installation

Check that services are running:

```bash
# Check Docker containers
docker-compose ps

# Test FlaskDrive
curl http://localhost:5000/health

# Test Video Generator
curl http://localhost:5001/system/status
```

### 7. Complete Google Drive Authentication

On first run, visit `http://localhost:5000` in your browser to complete the OAuth flow. This will create a `token.pickle` file in `flaskdrive/data/`.

## Component-Specific Setup

### Music System

The music system will automatically:
- Create cache directories
- Download SoundCloud tracks (if configured)
- Build music index

**Optional**: To download SoundCloud tracks:
1. Place an Excel file with SoundCloud URLs in `WordsOnText/SoundCloud to X Reposter.xlsx`
2. Run the download script (see music-system README)

### Words on Text

For GPU acceleration (optional):
1. Install NVIDIA Docker runtime
2. Ensure CUDA is available
3. The system will automatically detect and use GPU

## Development Setup

If you want to develop locally without Docker:

### 1. Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 2. Install Dependencies

```bash
# Install system dependencies
pip install -r requirements.txt

# Or install per component:
cd music-system && pip install -r requirements.txt
cd ../flaskdrive && pip install -r requirements.txt
cd ../video-generator && pip install -r requirements.txt
cd ../words-on-text && pip install -r requirements.txt
```

### 3. Install System Tools

- **FFmpeg**: Required for video processing
  - Windows: Download from [ffmpeg.org](https://ffmpeg.org/download.html)
  - Mac: `brew install ffmpeg`
  - Linux: `sudo apt-get install ffmpeg`

### 4. Run Services Locally

```bash
# Music System / Video Generator
cd music-system
python app.py

# FlaskDrive
cd flaskdrive
python app.py
```

## Troubleshooting

### Docker Issues

**Problem**: Services won't start
- **Solution**: Check Docker Desktop is running
- **Solution**: Check ports 5000 and 5001 are not in use

**Problem**: Permission denied errors
- **Solution**: On Linux, ensure Docker has proper permissions
- **Solution**: Check volume mount paths are correct

### Authentication Issues

**Problem**: Google Drive authentication fails
- **Solution**: Verify `client_secret.json` is in `flaskdrive/` directory
- **Solution**: Check redirect URI matches Google Cloud Console
- **Solution**: Delete `flaskdrive/data/token.pickle` and re-authenticate

### API Issues

**Problem**: 500 errors from APIs
- **Solution**: Check service logs: `docker-compose logs <service-name>`
- **Solution**: Verify environment variables are set
- **Solution**: Check file paths are accessible

### Path Issues

**Problem**: File not found errors
- **Solution**: Use Docker volume paths inside containers
- **Solution**: Check Windows path format (use forward slashes or double backslashes)
- **Solution**: Verify shared volumes are mounted correctly

## Next Steps

- Read the [README.md](README.md) for overview
- Check component-specific documentation:
  - [Music System](music-system/README.md)
  - [FlaskDrive](flaskdrive/README.md)
  - [Video Generator](video-generator/README.md)
  - [Words on Text](words-on-text/README.md)
- Review [Architecture](ARCHITECTURE.md) documentation
- See [Workflow Integration Guide](docs/video-workflow-replication-guide.md) for n8n setup

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review component-specific README files
3. Check Docker logs for detailed error messages
4. Open an issue on GitHub


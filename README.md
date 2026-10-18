# Custom Video Processing Tools

A comprehensive suite of custom video processing tools designed to work with n8n workflows. This repository contains modular services for music analysis, video processing, text overlay, and Google Drive integration.

## 🎯 Overview

This collection of tools provides:

- **Intelligent Music Selection**: AI-powered music matching based on video content analysis
- **Smart Video Processing**: Automated text overlay with intelligent positioning
- **Google Drive Integration**: Seamless file upload and management
- **Video Generation**: Combine audio and images into video files
- **Batch Processing**: Efficient handling of multiple videos

## 📦 Components

### 1. Music System (`music-system/`)
A comprehensive music analysis and video integration system that:
- Downloads and analyzes SoundCloud tracks
- Performs bulk music analysis
- Provides intelligent music recommendations based on video content
- Integrates with n8n workflows via REST API

**Key Features:**
- Modular architecture with separate analyzers
- Persistent caching for fast operations
- Trend integration for current music recommendations
- AI-powered music matching

[Read More →](music-system/README.md)

### 2. FlaskDrive (`flaskdrive/`)
A Flask microservice for uploading large video files to Google Drive:
- Handles chunked uploads for large files (>10GB)
- OAuth 2.0 authentication
- Returns direct download links
- Designed for n8n workflow integration

**Key Features:**
- Large file support (up to 10GB)
- Chunked uploads for reliability
- Direct download link generation
- Health check endpoints

[Read More →](flaskdrive/README.md)

### 3. Video Generator (`video-generator/`)
A Flask service that combines audio files with static images:
- Merges multiple audio tracks
- Creates video from image + audio
- Containerized for easy deployment
- Designed for podcast creator workflows

**Key Features:**
- Multiple audio file support
- Static image video generation
- Docker containerization
- Health monitoring

[Read More →](video-generator/README.md)

### 4. Words on Text (`words-on-text/`)
An intelligent text-on-video overlay tool:
- AI-powered text generation from transcripts
- Smart placement algorithm (avoids faces and UI elements)
- Batch processing capabilities
- GPU acceleration support

**Key Features:**
- Intelligent text generation (30% transcript + 30% vision + 40% prompt)
- Face detection for smart placement
- Multiple font presets
- Webhook integration
- GUI wrapper for non-technical users

[Read More →](words-on-text/README.md)

### 5. Shared Resources (`shared/`)
Shared assets used across all services:
- Font library for text overlays
- Video library for processing
- Music library for background tracks
- Utility scripts

[Read More →](shared/README.md)

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose
- Python 3.8+ (for local development)
- FFmpeg (for video processing)
- OpenAI API key (for AI features)

### Installation

1. **Clone this repository:**
   ```bash
   git clone <repository-url>
   cd custom-video-tools
   ```

2. **Set up environment variables:**
   ```bash
   export OPENAI_API_KEY="your-api-key-here"
   ```

3. **Start services with Docker Compose:**
   ```bash
   docker-compose up -d
   ```

### Service Endpoints

Once running, services are available at:

- **Music System API**: `http://localhost:5001`
- **FlaskDrive**: `http://localhost:5000`
- **Video Generator**: `http://localhost:5001` (same as music system)

## 📖 Documentation

- [Music System Documentation](music-system/README.md)
- [FlaskDrive Documentation](flaskdrive/README.md)
- [Video Generator Documentation](video-generator/README.md)
- [Words on Text Documentation](words-on-text/README.md)
- [Shared Resources](shared/README.md)
- [Workflow Integration Guide](docs/video-workflow-replication-guide.md)
- [Dockerized Endpoints Guide](docs/dockerized-endpoints-guide.md)
- [Large File Handling](docs/large-file-handling.md)

## 🔧 Configuration

### Docker Compose Setup

The services are designed to work together in a Docker Compose environment. Key configuration points:

- **Volume Mounts**: Shared directories for videos, fonts, and music
- **Network**: All services on the same Docker network for communication
- **Environment Variables**: API keys and configuration passed via environment

### n8n Integration

These tools integrate seamlessly with n8n workflows:

1. Use HTTP Request nodes to call service APIs
2. Services are accessible via Docker network names (e.g., `http://video-generator:5001`)
3. File paths use shared volume mounts
4. Binary data handling supported for video files

## 🏗️ Architecture

```
custom-video-tools/
├── music-system/          # Music analysis and matching
├── flaskdrive/            # Google Drive upload service
├── video-generator/       # Audio + image video creation
├── words-on-text/         # Smart text overlay tool
├── shared/                # Shared resources (fonts, videos, music)
├── scripts/               # Utility scripts
├── docs/                  # Documentation
├── cache/                 # Processing cache
├── output/                # Generated outputs
└── soundcloud_music/      # Downloaded music tracks
```

## 🔄 Workflow Integration

### Example n8n Workflow

1. **Video Input**: Receive video via webhook
2. **Music Analysis**: Call music-system API to find matching music
3. **Video Processing**: Use words-on-text for intelligent text overlay
4. **Upload**: Send processed video to Google Drive via FlaskDrive
5. **Notification**: Send completion notification

### API Endpoints

**Music System:**
- `POST /analyze-music-match` - Find music for video
- `GET /system/status` - System health check
- `POST /video/analyze-with-openai` - AI video analysis

**FlaskDrive:**
- `POST /upload` - Upload file to Google Drive
- `GET /health` - Health check
- `GET /file-info/<file_id>` - Get file metadata

**Video Generator:**
- `POST /generate-video` - Create video from audio + image
- `GET /health` - Health check

## 🧪 Testing

Each component includes test scripts:

```bash
# Test music system
cd music-system
python test_system.py

# Test FlaskDrive
cd flaskdrive
python -m pytest tests/

# Test video generator
cd video-generator
python test_api.py
```

## 📊 Performance

- **Music Analysis**: Cached results for fast repeated operations
- **Video Processing**: GPU acceleration available (CUDA)
- **File Uploads**: Chunked uploads for large files
- **Batch Processing**: Parallel processing where possible

## 🔒 Security

- OAuth 2.0 for Google Drive authentication
- Environment variables for sensitive keys
- Docker network isolation
- Input validation on all endpoints

## 🤝 Contributing

This is a custom toolset. For modifications:

1. Test changes in development environment
2. Update documentation
3. Verify Docker compatibility
4. Test n8n integration

## 📝 License

Custom software - See individual component READMEs for details.

## 🆘 Troubleshooting

### Common Issues

1. **Services not accessible from n8n:**
   - Ensure all services are on the same Docker network
   - Check service names match in docker-compose.yml
   - Verify ports are exposed correctly

2. **File path issues:**
   - Use Docker volume mount paths inside containers
   - Check Windows/Linux path format differences
   - Verify file permissions

3. **API errors:**
   - Check service logs: `docker-compose logs <service-name>`
   - Verify environment variables are set
   - Test endpoints directly with curl/Postman

### Getting Help

- Check component-specific README files
- Review workflow integration guides in `docs/`
- Check Docker logs for detailed error messages

## 🎯 Use Cases

- **Content Creation**: Automated video processing with music and text
- **Podcast Production**: Generate video from audio + cover art
- **Social Media**: Batch process videos with intelligent overlays
- **Video Libraries**: Organize and enhance video collections

---

**Built for n8n workflows | Modular | Production-Ready**


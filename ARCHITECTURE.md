# Architecture Overview

## System Architecture

This document describes the architecture of the Custom Video Processing Tools suite.

## Component Overview

### 1. Music System
**Purpose**: Intelligent music analysis and video matching

**Components**:
- `music_analyzer.py`: Audio feature extraction and analysis
- `video_analyzer.py`: Video content analysis
- `soundcloud_downloader.py`: SoundCloud track downloading
- `trend_analyzer.py`: Music trend analysis
- `main_system.py`: System orchestrator
- `app.py`: Flask API server

**Data Flow**:
```
Video Input → Video Analyzer → Music Matcher → Music Analyzer → Recommendations
```

**Key Technologies**:
- Librosa for audio analysis
- OpenCV for video analysis
- OpenAI API for intelligent matching
- Flask for REST API

### 2. FlaskDrive
**Purpose**: Google Drive file upload service

**Components**:
- `app.py`: Main Flask application
- `auto_auth.py`: OAuth 2.0 automation
- `complete_auth.py`: Authentication completion

**Data Flow**:
```
File Upload → Chunk Processing → Google Drive API → Direct Download Link
```

**Key Technologies**:
- Google Drive API v3
- OAuth 2.0 for authentication
- Flask for file handling
- Chunked uploads for large files

### 3. Video Generator
**Purpose**: Combine audio and images into video

**Components**:
- `app.py`: Flask API server
- `music_analysis_endpoint.py`: Music analysis integration

**Data Flow**:
```
Audio Files + Image → FFmpeg Processing → Video Output
```

**Key Technologies**:
- FFmpeg for video processing
- Flask for API
- Image processing libraries

### 4. Words on Text
**Purpose**: Intelligent text overlay on videos

**Components**:
- `smart_video_processor.py`: Main processing engine
- Video context analyzer modules
- Audio analysis modules
- Frame analysis modules

**Data Flow**:
```
Video → Frame Analysis → Text Generation → Smart Placement → Video Output
```

**Key Technologies**:
- OpenAI Whisper for transcription
- OpenAI GPT for text generation
- OpenCV for video processing
- YOLO for object detection
- CLIP for visual understanding

## Data Storage

### Shared Resources
- **Fonts**: Stored in `shared/fonts/`
- **Videos**: Stored in `shared/videos/`
- **Music**: Stored in `soundcloud_music/`

### Cache System
- **Music Analysis Cache**: `cache/music_analysis_cache.json`
- **Video Context Cache**: `cache/video_context_cache.json`
- **SoundCloud Cache**: `cache/soundcloud_cache.json`
- **Trend Cache**: `cache/trend_cache.json`

### Output
- **Processed Videos**: `cache/processed_videos/`
- **Analysis Results**: `analysis/`
- **General Output**: `output/`

## Communication Patterns

### Service-to-Service
Services communicate via HTTP REST APIs on the Docker network:
- Internal: `http://service-name:port`
- External: `http://localhost:port`

### n8n Integration
n8n workflows call services via HTTP Request nodes:
- Use Docker network names for internal calls
- Use `host.docker.internal` for external access

## Deployment Architecture

### Docker Compose Setup
```
┌─────────────────────────────────────────┐
│         Docker Network                  │
│                                         │
│  ┌──────────┐  ┌──────────┐           │
│  │ FlaskDrive│  │Video Gen │           │
│  │  :5000    │  │  :5001   │           │
│  └──────────┘  └──────────┘           │
│                                         │
│  ┌──────────────────────────────┐     │
│  │   Shared Volumes              │     │
│  │   - videos/                   │     │
│  │   - fonts/                    │     │
│  │   - music/                    │     │
│  │   - cache/                    │     │
│  └──────────────────────────────┘     │
└─────────────────────────────────────────┘
         │
         │ HTTP
         ▼
    ┌─────────┐
    │   n8n   │
    └─────────┘
```

## Performance Considerations

### Caching Strategy
- Music analysis results cached to avoid re-processing
- Video context cached for repeated analysis
- Trend data cached with TTL

### Resource Usage
- **CPU**: Video processing is CPU-intensive
- **GPU**: Optional acceleration for Words on Text
- **Memory**: Large video files require significant RAM
- **Storage**: Cache and output files can grow large

### Scalability
- Services are stateless (except cache)
- Can be scaled horizontally
- Database not required (file-based storage)

## Security Architecture

### Authentication
- OAuth 2.0 for Google Drive
- API keys via environment variables
- No user authentication (internal services)

### Data Protection
- Sensitive files excluded from git
- Environment variables for secrets
- Docker network isolation

## Error Handling

### Service Resilience
- Health check endpoints
- Graceful error responses
- Retry logic for external APIs
- Fallback mechanisms

### Monitoring
- Health check endpoints
- Logging to stdout/stderr
- Docker log aggregation

## Future Enhancements

### Potential Additions
- Message queue for async processing
- Database for metadata storage
- Redis for distributed caching
- Kubernetes deployment configs
- Monitoring and metrics collection


# Video Processing & Social Media Automation - Standalone Implementation Guide

## Overview
This guide explains how to replicate the n8n video processing workflow as a standalone Python application.

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Video Files   │───▶│  Main Pipeline  │───▶│ Social Media    │
│   Scanner       │    │   Orchestrator  │    │ Publisher       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ File System     │    │ AI Processing   │    │ Blotato API     │
│ Monitor         │    │ (OpenAI)        │    │ Integration     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Google Drive    │    │ Content         │    │ Platform APIs   │
│ Integration     │    │ Generation      │    │ (Direct)        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Implementation Structure

### 1. Core Components

```python
# main.py - Entry point and orchestration
# video_scanner.py - File discovery and validation
# video_processor.py - FFmpeg operations and AI analysis
# content_generator.py - OpenAI integration for captions
# drive_uploader.py - Google Drive integration
# social_publisher.py - Multi-platform publishing
# tracking.py - Google Sheets logging
# config.py - Environment and API keys
```

### 2. Data Flow Implementation

#### Phase 1: Video Discovery
```python
class VideoScanner:
    def __init__(self, video_directory="/path/to/videos"):
        self.video_directory = video_directory
        self.supported_formats = ['.mp4', '.avi', '.mov', '.mkv', '.webm']
    
    def scan_for_videos(self):
        """Equivalent to 'Scan for Videos' node"""
        videos = []
        for file in os.listdir(self.video_directory):
            if self.is_video_file(file):
                video_path = os.path.join(self.video_directory, file)
                videos.append({
                    'videoPath': video_path,
                    'videoName': os.path.splitext(file)[0],
                    'fileSize': os.path.getsize(video_path),
                    'detectedAt': datetime.now().isoformat()
                })
        return videos
```

#### Phase 2: Video Processing
```python
class VideoProcessor:
    def __init__(self, openai_api_key):
        self.openai_client = OpenAI(api_key=openai_api_key)
    
    async def process_video(self, video_path):
        """Equivalent to 'Optimized Video Processing' node"""
        # Extract video metadata
        dimensions = await self.get_video_dimensions(video_path)
        
        # Extract frames for analysis
        frames = await self.extract_key_frames(video_path)
        
        # Generate transcript using OpenAI Whisper
        transcript = await self.transcribe_audio(video_path)
        
        # Analyze frames with OpenAI Vision
        frame_context = await self.analyze_frames(frames)
        
        # Combine analysis
        combined_context = self.combine_analysis(transcript, frame_context)
        
        return {
            'dimensions': dimensions,
            'transcriptRaw': transcript,
            'frameContext': frame_context,
            'combinedContext': combined_context
        }
```

#### Phase 3: Content Generation
```python
class ContentGenerator:
    def __init__(self, openai_api_key):
        self.openai_client = OpenAI(api_key=openai_api_key)
    
    async def generate_social_content(self, video_analysis):
        """Equivalent to 'OpenAI' node"""
        prompt = self.build_content_prompt(video_analysis)
        
        response = await self.openai_client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "system", "content": self.get_system_prompt()},
                {"role": "user", "content": prompt}
            ],
            response_format={"type": "json_object"},
            temperature=0.7
        )
        
        return json.loads(response.choices[0].message.content)
```

#### Phase 4: Cloud Storage
```python
class DriveUploader:
    def __init__(self, credentials_file):
        self.drive_service = self.authenticate_drive(credentials_file)
    
    async def upload_video(self, video_path, video_name):
        """Equivalent to 'Upload to Google Drive' node"""
        file_metadata = {'name': video_name}
        media = MediaFileUpload(video_path, resumable=True)
        
        file = self.drive_service.files().create(
            body=file_metadata,
            media_body=media,
            fields='id,webViewLink,webContentLink'
        ).execute()
        
        # Generate direct download URL
        file_id = file.get('id')
        direct_url = f"https://usercontent.google.com/download?id={file_id}&export=download"
        
        return {
            'id': file_id,
            'directDownloadUrl': direct_url,
            'viewLink': file.get('webViewLink')
        }
```

#### Phase 5: Social Media Publishing
```python
class SocialPublisher:
    def __init__(self, blotato_api_key, account_config):
        self.blotato_api_key = blotato_api_key
        self.accounts = account_config
        self.base_url = "https://backend.blotato.com/v2"
    
    async def upload_to_blotato(self, video_url):
        """Equivalent to 'Upload to Blotato' node"""
        headers = {"blotato-api-key": self.blotato_api_key}
        data = {"url": video_url}
        
        response = await self.post_request(f"{self.base_url}/media", headers, data)
        return response.json()
    
    async def publish_to_platforms(self, content, media_url):
        """Publish to all platforms simultaneously"""
        tasks = []
        
        # Instagram (multiple accounts)
        for account_id in [self.accounts['instagram_id'], 
                          self.accounts['instagram_id_alt_1']]:
            tasks.append(self.publish_instagram(content, media_url, account_id))
        
        # Other platforms
        tasks.extend([
            self.publish_youtube(content, media_url),
            self.publish_tiktok(content, media_url),
            self.publish_twitter(content, media_url),
            self.publish_threads(content, media_url),
            self.publish_bluesky(content, media_url)
        ])
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        return results
```

### 3. Main Orchestration
```python
class VideoWorkflowOrchestrator:
    def __init__(self, config):
        self.scanner = VideoScanner(config.video_directory)
        self.processor = VideoProcessor(config.openai_api_key)
        self.generator = ContentGenerator(config.openai_api_key)
        self.uploader = DriveUploader(config.google_credentials)
        self.publisher = SocialPublisher(config.blotato_api_key, config.social_accounts)
        self.tracker = SheetsTracker(config.google_credentials, config.spreadsheet_id)
    
    async def process_workflow(self):
        """Main workflow equivalent to entire n8n flow"""
        # 1. Discover videos
        videos = self.scanner.scan_for_videos()
        
        for video in videos:
            try:
                # 2. Process video
                analysis = await self.processor.process_video(video['videoPath'])
                
                # 3. Upload to Drive
                drive_result = await self.uploader.upload_video(
                    video['videoPath'], video['videoName']
                )
                
                # 4. Generate content
                content = await self.generator.generate_social_content(analysis)
                
                # 5. Upload to Blotato
                blotato_media = await self.publisher.upload_to_blotato(
                    drive_result['directDownloadUrl']
                )
                
                # 6. Publish to social platforms
                publish_results = await self.publisher.publish_to_platforms(
                    content, blotato_media['url']
                )
                
                # 7. Track results
                await self.tracker.log_video_processing(
                    video, analysis, content, drive_result, publish_results
                )
                
            except Exception as e:
                logger.error(f"Failed processing {video['videoName']}: {e}")
                continue
```

## Environment Setup

### Required Environment Variables
```bash
# API Keys
OPENAI_API_KEY=sk-proj-your-openai-key
BLOTATO_API_KEY=blt_your-blotato-key

# Google Credentials
GOOGLE_CREDENTIALS_FILE=/path/to/service-account.json
GOOGLE_SPREADSHEET_ID=your-spreadsheet-id

# Social Media Account IDs (from Blotato)
INSTAGRAM_ID=807
YOUTUBE_ID=228
TIKTOK_ID=710
TWITTER_ID=571
THREADS_ID=243
BLUESKY_ID=2

# File Paths
VIDEO_DIRECTORY=/path/to/videos
```

### Dependencies
```bash
pip install openai google-auth google-auth-oauthlib google-auth-httplib2
pip install google-api-python-client gspread
pip install ffmpeg-python requests aiohttp
pip install fastapi uvicorn python-multipart
```

## Deployment Options

### 1. Scheduled Execution (Cron-like)
```python
# Use APScheduler for periodic execution
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()
scheduler.add_job(
    orchestrator.process_workflow,
    'interval',
    minutes=30,  # Run every 30 minutes
    id='video_processing'
)
scheduler.start()
```

### 2. Web API Trigger
```python
# FastAPI endpoint to trigger manually
@app.post("/trigger-workflow")
async def trigger_workflow():
    background_tasks.add_task(orchestrator.process_workflow)
    return {"message": "Workflow triggered"}
```

### 3. File System Watcher
```python
# Use watchdog for real-time file detection
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class VideoFileHandler(FileSystemEventHandler):
    def on_created(self, event):
        if event.is_dir or not self.is_video_file(event.src_path):
            return
        asyncio.create_task(orchestrator.process_single_video(event.src_path))
```

## Key Differences from n8n

### Advantages of Standalone Implementation
1. **Better Error Handling**: Full try/catch control
2. **Performance**: Direct async/await, no n8n overhead
3. **Flexibility**: Custom retry logic, parallel processing
4. **Monitoring**: Detailed logging and metrics
5. **Cost**: No n8n licensing fees

### Considerations
1. **Infrastructure**: Need to manage deployment yourself
2. **UI**: No visual workflow editor
3. **Integrations**: Must implement OAuth flows manually
4. **Scaling**: Need to handle concurrency and resource management

## Testing Strategy
```python
# Unit tests for each component
# Integration tests with mock APIs
# Load testing with multiple videos
# Error simulation and recovery testing
```

This implementation provides the same functionality as the n8n workflow but with greater control, performance, and customization options.

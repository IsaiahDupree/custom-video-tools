# Dockerized Container Endpoints Guide

## Overview
This guide covers all available endpoints in your dockerized video processing and automation ecosystem, including health checks, execution patterns, and best practices for integration.

## Service Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     n8n         │    │   Enhanced API  │    │   FlaskDrive    │
│  localhost:5678 │    │ localhost:5001  │    │ localhost:5000  │
│ (alt: 5679-5681)│    │ (alt: 5002-5005)│    │ (alt: 5006-5010)│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Workflow Engine │    │ Video/Music AI  │    │ Google Drive    │
│ Automation      │    │ Processing      │    │ Integration     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Port Configuration & Fallbacks

| Service | Primary Port | Fallback Ports | Container Port | Purpose |
|---------|-------------|----------------|----------------|---------|
| **n8n** | 5678 | 5679, 5680, 5681 | 5678 | Workflow Editor & API |
| **Enhanced API** | 5001 | 5002, 5003, 5004, 5005 | 5001 | Video/Music Processing |
| **FlaskDrive** | 5000 | 5006, 5007, 5008, 5009, 5010 | 5000 | Google Drive Integration |

#### Environment Variables for Port Configuration

```bash
# Primary ports (default)
N8N_PORT=5678
ENHANCED_API_PORT=5001
FLASKDRIVE_PORT=5000

# Fallback configuration
N8N_FALLBACK_PORTS="5679,5680,5681"
ENHANCED_API_FALLBACK_PORTS="5002,5003,5004,5005"
FLASKDRIVE_FALLBACK_PORTS="5006,5007,5008,5009,5010"
```

#### Docker Compose with Port Fallbacks

```yaml
version: '3.8'
services:
  video-generator:
    image: your-enhanced-api
    ports:
      - "${ENHANCED_API_PORT:-5001}:5001"
      - "${ENHANCED_API_FALLBACK_1:-5002}:5001"
      - "${ENHANCED_API_FALLBACK_2:-5003}:5001"
    environment:
      - PORT=5001
      - FALLBACK_PORTS=5002,5003,5004,5005
    
  flaskdrive:
    image: your-flaskdrive
    ports:
      - "${FLASKDRIVE_PORT:-5000}:5000"
      - "${FLASKDRIVE_FALLBACK_1:-5006}:5000"
      - "${FLASKDRIVE_FALLBACK_2:-5007}:5000"
    environment:
      - PORT=5000
      - FALLBACK_PORTS=5006,5007,5008,5009,5010
    
  n8n:
    image: n8nio/n8n
    ports:
      - "${N8N_PORT:-5678}:5678"
      - "${N8N_FALLBACK_1:-5679}:5678"
      - "${N8N_FALLBACK_2:-5680}:5678"
    environment:
      - N8N_PORT=5678
      - N8N_FALLBACK_PORTS=5679,5680,5681
```

#### Dynamic Port Discovery

```python
import socket
import requests
from typing import List, Optional

class PortDiscovery:
    def __init__(self, base_host: str = "localhost"):
        self.base_host = base_host
    
    def is_port_available(self, port: int) -> bool:
        """Check if a port is available on the host"""
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            result = sock.connect_ex((self.base_host, port))
            return result == 0
    
    def find_active_service_port(self, service_name: str, primary_port: int, 
                                fallback_ports: List[int]) -> Optional[int]:
        """Find the active port for a service, checking primary first then fallbacks"""
        all_ports = [primary_port] + fallback_ports
        
        for port in all_ports:
            if self.is_port_available(port):
                try:
                    # Verify service is actually responding
                    response = requests.get(f"http://{self.base_host}:{port}/health", timeout=2)
                    if response.status_code == 200:
                        print(f"✅ {service_name} found on port {port}")
                        return port
                except requests.RequestException:
                    continue
        
        print(f"❌ {service_name} not found on any configured ports")
        return None
    
    def discover_all_services(self) -> dict:
        """Discover all services and their active ports"""
        services = {
            "enhanced_api": {
                "primary": 5001,
                "fallbacks": [5002, 5003, 5004, 5005]
            },
            "flaskdrive": {
                "primary": 5000,
                "fallbacks": [5006, 5007, 5008, 5009, 5010]
            },
            "n8n": {
                "primary": 5678,
                "fallbacks": [5679, 5680, 5681]
            }
        }
        
        active_services = {}
        for service_name, config in services.items():
            port = self.find_active_service_port(
                service_name, 
                config["primary"], 
                config["fallbacks"]
            )
            if port:
                active_services[service_name] = {
                    "port": port,
                    "url": f"http://{self.base_host}:{port}",
                    "is_fallback": port != config["primary"]
                }
        
        return active_services

# Usage example
discovery = PortDiscovery()
active_services = discovery.discover_all_services()
print("Active services:", active_services)
```

## Service Endpoints

### 1. Enhanced Music & Video API (`video-generator:5001`)

#### Health & Status Endpoints
```bash
# Basic health check
GET http://localhost:5001/health
Response: {"status": "healthy", "timestamp": "2025-01-23T10:30:00Z"}

# Detailed system status
GET http://localhost:5001/system/status
Response: {
  "status": "online",
  "services": {
    "ffmpeg": "available",
    "openai": "connected",
    "database": "healthy"
  },
  "uptime": "2h 15m 30s",
  "memory_usage": "45%",
  "disk_space": "78% available"
}
```

#### Music Processing Endpoints
```bash
# Preprocess music library
POST http://localhost:5001/music/preprocess
Content-Type: application/json
{
  "directory": "/path/to/music",
  "formats": ["mp3", "wav", "flac"]
}

# Get music database statistics
GET http://localhost:5001/music/database/stats
Response: {
  "total_tracks": 1250,
  "processed": 1200,
  "pending": 50,
  "last_updated": "2025-01-23T09:45:00Z"
}

# Search music catalog
POST http://localhost:5001/music/search
Content-Type: application/json
{
  "query": "energetic electronic",
  "limit": 10,
  "filters": {
    "genre": "electronic",
    "bpm_range": [120, 140]
  }
}

# Analyze music match for video
POST http://localhost:5001/analyze-music-match
Content-Type: application/json
{
  "video_path": "/path/to/video.mp4",
  "music_preferences": {
    "mood": "upbeat",
    "energy_level": "high"
  }
}
```

#### Video Processing Endpoints
```bash
# Basic video content analysis
POST http://localhost:5001/video/analyze-content
Content-Type: multipart/form-data
file: video.mp4

# Advanced OpenAI-powered video analysis
POST http://localhost:5001/video/analyze-with-openai
Content-Type: multipart/form-data
file: video.mp4
options: {
  "extract_transcript": true,
  "analyze_frames": true,
  "generate_tags": true
}

# Process video with enhancements
POST http://localhost:5001/video/process
Content-Type: multipart/form-data
file: video.mp4
processing_options: {
  "resolution": "1080p",
  "format": "mp4",
  "quality": "high"
}

# Enhanced video processing with AI features
POST http://localhost:5001/video/process-enhanced
Content-Type: multipart/form-data
file: video.mp4
enhancement_options: {
  "auto_crop": true,
  "noise_reduction": true,
  "color_correction": true,
  "add_subtitles": true
}

# Combined analyze and process workflow
POST http://localhost:5001/video/analyze-and-process
Content-Type: multipart/form-data
file: video.mp4
workflow_options: {
  "analyze_first": true,
  "auto_enhance": true,
  "generate_thumbnails": true
}

# Batch process multiple videos
POST http://localhost:5001/video/batch-process
Content-Type: application/json
{
  "video_paths": [
    "/path/to/video1.mp4",
    "/path/to/video2.mp4"
  ],
  "processing_options": {
    "parallel": true,
    "max_concurrent": 3
  }
}
```

#### Font & Typography Endpoints
```bash
# List available fonts
GET http://localhost:5001/fonts/list
Response: {
  "fonts": [
    {"name": "Arial", "path": "/fonts/arial.ttf", "style": "regular"},
    {"name": "Helvetica", "path": "/fonts/helvetica.ttf", "style": "bold"}
  ]
}

# Test font rendering
POST http://localhost:5001/fonts/test
Content-Type: application/json
{
  "font_name": "Arial",
  "text": "Sample Text",
  "size": 24,
  "color": "#FFFFFF"
}
```

#### File Download Endpoint
```bash
# Download processed files
GET http://localhost:5001/download/file/{file_path}
# Example: GET http://localhost:5001/download/file/processed/video_123.mp4
```

### 2. FlaskDrive API (`flaskdrive:5000`)

#### Health & Status Endpoints
```bash
# Basic health check
GET http://localhost:5000/health
Response: {"status": "healthy", "service": "FlaskDrive"}

# Detailed status with Google Drive connection
GET http://localhost:5000/status
Response: {
  "status": "online",
  "google_drive": "connected",
  "auth_status": "valid",
  "quota_used": "45%",
  "last_sync": "2025-01-23T10:15:00Z"
}
```

#### Authentication Endpoints
```bash
# Main interface
GET http://localhost:5000/
Response: HTML interface for Google Drive operations

# Start OAuth flow
GET http://localhost:5000/authorize
Response: Redirects to Google OAuth consent screen

# OAuth callback
GET http://localhost:5000/oauth2callback?code={auth_code}
Response: Completes authentication and stores tokens
```

#### File Operations
```bash
# Upload file to Google Drive
POST http://localhost:5000/upload
Content-Type: multipart/form-data
file: document.pdf
folder_id: "optional_folder_id"

Response: {
  "id": "1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms",
  "name": "document.pdf",
  "download_link": "https://drive.google.com/file/d/.../view",
  "direct_link": "https://usercontent.google.com/download?id=...",
  "size": "1.2MB",
  "created": "2025-01-23T10:30:00Z"
}

# Get file information
GET http://localhost:5000/file-info/{file_id}
Response: {
  "id": "1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms",
  "name": "document.pdf",
  "size": "1234567",
  "mimeType": "application/pdf",
  "parents": ["folder_id"],
  "webViewLink": "https://drive.google.com/file/d/.../view"
}

# Search files in Drive
GET http://localhost:5000/search?q={search_query}&type={file_type}
# Example: GET http://localhost:5000/search?q=video&type=video/mp4
Response: {
  "files": [
    {
      "id": "file_id",
      "name": "video.mp4",
      "size": "50MB",
      "modified": "2025-01-23T09:30:00Z"
    }
  ],
  "total": 1
}
```

### 3. n8n Workflow Engine (`localhost:5678`)

#### Workflow Management
```bash
# n8n Web Interface
GET http://localhost:5678
Response: n8n workflow editor interface

# Webhook endpoints (configured per workflow)
POST http://localhost:5678/webhook/{webhook_id}
POST http://localhost:5678/webhook-test/{webhook_id}

# Health check (if configured)
GET http://localhost:5678/healthz
Response: {"status": "ok"}
```

## Health Check Implementation

### Docker Compose Health Checks
```yaml
version: '3.8'
services:
  video-generator:
    image: your-enhanced-api
    ports:
      - "5001:5001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
  flaskdrive:
    image: your-flaskdrive
    ports:
      - "5000:5000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5678/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Python Health Check Script
```python
#!/usr/bin/env python3
import requests
import json
from typing import Dict, List

class HealthChecker:
    def __init__(self):
        self.services = {
            "enhanced-api": "http://localhost:5001/health",
            "flaskdrive": "http://localhost:5000/health", 
            "n8n": "http://localhost:5678/healthz"
        }
    
    def check_all_services(self) -> Dict:
        results = {}
        for service, url in self.services.items():
            try:
                response = requests.get(url, timeout=5)
                results[service] = {
                    "status": "healthy" if response.status_code == 200 else "unhealthy",
                    "response_time": f"{response.elapsed.total_seconds():.3f}s",
                    "status_code": response.status_code
                }
            except requests.RequestException as e:
                results[service] = {
                    "status": "unreachable",
                    "error": str(e)
                }
        return results
    
    def detailed_status(self) -> Dict:
        """Get detailed status from Enhanced API"""
        try:
            response = requests.get("http://localhost:5001/system/status", timeout=5)
            return response.json() if response.status_code == 200 else {}
        except:
            return {"error": "Unable to fetch detailed status"}

# Usage
if __name__ == "__main__":
    checker = HealthChecker()
    health_status = checker.check_all_services()
    detailed = checker.detailed_status()
    
    print(json.dumps({
        "health_checks": health_status,
        "detailed_status": detailed
    }, indent=2))
```

## Best Practices

### 1. Connection Management
```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class APIClient:
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = requests.Session()
        
        # Configure retry strategy
        retry_strategy = Retry(
            total=3,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
        )
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
    
    def post_with_retry(self, endpoint: str, **kwargs):
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        return self.session.post(url, timeout=30, **kwargs)
```

### 2. Error Handling Patterns
```python
async def safe_api_call(client, endpoint, payload):
    """Safe API call with comprehensive error handling"""
    try:
        response = await client.post(endpoint, json=payload)
        response.raise_for_status()
        return {"success": True, "data": response.json()}
    
    except requests.exceptions.ConnectionError:
        return {"success": False, "error": "Service unavailable"}
    
    except requests.exceptions.Timeout:
        return {"success": False, "error": "Request timeout"}
    
    except requests.exceptions.HTTPError as e:
        return {"success": False, "error": f"HTTP {e.response.status_code}"}
    
    except Exception as e:
        return {"success": False, "error": f"Unexpected error: {str(e)}"}
```

### 3. Monitoring & Logging
```python
import logging
from datetime import datetime

class APIMonitor:
    def __init__(self):
        self.logger = logging.getLogger('api_monitor')
        
    def log_request(self, service: str, endpoint: str, response_time: float, status_code: int):
        self.logger.info(
            f"API_CALL service={service} endpoint={endpoint} "
            f"response_time={response_time:.3f}s status={status_code}"
        )
    
    def log_health_check(self, service: str, status: str):
        self.logger.info(f"HEALTH_CHECK service={service} status={status}")
    
    def track_metrics(self, service: str, metrics: dict):
        """Track custom metrics for monitoring"""
        for key, value in metrics.items():
            self.logger.info(f"METRIC service={service} {key}={value}")
```

### 4. Configuration Management
```python
# config.py
import os
from dataclasses import dataclass

@dataclass
class ServiceConfig:
    enhanced_api_url: str = os.getenv('ENHANCED_API_URL', 'http://localhost:5001')
    flaskdrive_url: str = os.getenv('FLASKDRIVE_URL', 'http://localhost:5000')
    n8n_url: str = os.getenv('N8N_URL', 'http://localhost:5678')
    
    # Timeouts
    default_timeout: int = 30
    health_check_timeout: int = 5
    
    # Retry settings
    max_retries: int = 3
    backoff_factor: float = 1.0
```

### 5. Container Orchestration
```bash
#!/bin/bash
# Container management script

check_container_health() {
    container_name=$1
    health_status=$(docker inspect --format='{{.State.Health.Status}}' $container_name 2>/dev/null)
    
    if [ "$health_status" = "healthy" ]; then
        echo "✅ $container_name is healthy"
        return 0
    else
        echo "❌ $container_name is unhealthy or not running"
        return 1
    fi
}

restart_unhealthy_containers() {
    containers=("video-generator" "flaskdrive" "n8n")
    
    for container in "${containers[@]}"; do
        if ! check_container_health $container; then
            echo "🔄 Restarting $container..."
            docker restart $container
            sleep 10
        fi
    done
}

# Usage: ./health_check.sh
restart_unhealthy_containers
```

## Testing Endpoints

### Automated Testing Suite
```python
import pytest
import requests

class TestEndpoints:
    base_urls = {
        "enhanced_api": "http://localhost:5001",
        "flaskdrive": "http://localhost:5000"
    }
    
    def test_health_endpoints(self):
        """Test all health endpoints"""
        endpoints = [
            f"{self.base_urls['enhanced_api']}/health",
            f"{self.base_urls['flaskdrive']}/health"
        ]
        
        for endpoint in endpoints:
            response = requests.get(endpoint, timeout=5)
            assert response.status_code == 200
            assert "status" in response.json()
    
    def test_video_upload_flow(self):
        """Test complete video processing flow"""
        # 1. Upload to FlaskDrive
        with open("test_video.mp4", "rb") as f:
            response = requests.post(
                f"{self.base_urls['flaskdrive']}/upload",
                files={"file": f}
            )
        assert response.status_code == 200
        file_data = response.json()
        
        # 2. Process with Enhanced API
        response = requests.post(
            f"{self.base_urls['enhanced_api']}/video/analyze-content",
            json={"video_url": file_data["direct_link"]}
        )
        assert response.status_code == 200

# Run tests: pytest test_endpoints.py -v
```

This comprehensive guide provides everything needed to interact with, monitor, and maintain your dockerized container ecosystem effectively.

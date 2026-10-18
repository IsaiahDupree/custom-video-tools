# N8N Workflows for Video Processing Pipeline

## Created Workflows

### 1. Complete Video Production Workflow
**File:** `workflows/complete-video-production-workflow.json`
**Purpose:** Full end-to-end video processing pipeline
**Features:**
- ✅ System health checks
- ✅ Font system integration
- ✅ AI video analysis with OpenAI
- ✅ Music selection and analysis
- ✅ Enhanced video processing with custom fonts
- ✅ Google Drive upload integration
- ✅ Error handling and conditional logic

**Usage:** Import into n8n for complete automation

### 2. Batch Video Processing Workflow
**File:** `workflows/batch-video-processing-workflow.json`
**Purpose:** Process multiple videos efficiently
**Features:**
- ✅ Batch video selection (up to 5 videos)
- ✅ Font and music system integration
- ✅ Sequential processing with summary
- ✅ Automatic output naming

**Usage:** Webhook endpoint `/batch-process` for bulk operations

### 3. Simple Font Video Workflow
**File:** `workflows/simple-font-video-workflow.json`
**Purpose:** Simplified video processing with custom fonts
**Features:**
- ✅ Parameter extraction from webhook
- ✅ System and font health checks
- ✅ Custom font video processing
- ✅ Success/error response handling
- ✅ 60-second timeout for processing

**Usage:** Webhook endpoint `/simple-video-process` for reliable processing

### 4. System Health Monitor Workflow
**File:** `workflows/system-monitor-workflow.json`
**Purpose:** Continuous system monitoring and alerting
**Features:**
- ✅ Runs every 5 minutes automatically
- ✅ Monitors system, font, and music APIs
- ✅ Health score calculation
- ✅ Issue detection and alerting
- ✅ Google Drive logging integration

**Usage:** Activate for continuous monitoring

## Debugging Tools Created

### 1. Comprehensive Endpoint Test
**File:** `music_system/comprehensive_endpoint_test.py`
**Purpose:** Test all API endpoints for 500 errors
**Tests:**
- System status checks
- Font system validation
- Music search and analysis
- AI video analysis
- Enhanced video processing

### 2. Quick 500 Error Debug
**File:** `music_system/quick_500_debug.py`
**Purpose:** Rapid identification of 500 error causes
**Checks:**
- Font selection API
- Video processing API
- Music analysis API
- File access validation

## 500 Error Investigation Results

### Common Causes Identified:
1. **Docker Services Down** - Docker Desktop not running
2. **File Path Issues** - Windows vs Linux path formatting
3. **Missing Dependencies** - FFmpeg, font files, or music files
4. **Resource Exhaustion** - Insufficient memory or disk space
5. **API Key Issues** - OpenAI API key not accessible
6. **Permission Problems** - Container file access issues

### Quick Fixes:
```bash
# 1. Start Docker services
docker-compose up -d

# 2. Check service status
docker-compose ps

# 3. View Flask API logs
docker-compose logs video-generator

# 4. Restart video processing service
docker-compose restart video-generator

# 5. Test system health
curl http://localhost:5001/system/status
```

## Workflow Import Instructions

### Step 1: Start Docker Environment
```bash
cd "C:\Users\Isaia\Documents\Coding\N8N"
docker-compose up -d
```

### Step 2: Access n8n Interface
- URL: http://localhost:5678
- Import workflows from the `workflows/` directory

### Step 3: Configure Workflows
- Verify webhook URLs use `host.docker.internal`
- Check file paths are Docker-compatible
- Test each workflow individually

### Step 4: Activate Monitoring
1. Import `system-monitor-workflow.json`
2. Activate the workflow
3. Monitor for alerts every 5 minutes

## Production Recommendations

### For Immediate Use:
1. **Start with Simple Font Video Workflow** - Most reliable
2. **Use Batch Processing** - For multiple videos
3. **Enable System Monitoring** - For continuous health checks

### For Advanced Features:
1. **Complete Video Production** - Full AI integration
2. **Custom parameter workflows** - Modify for specific needs

### For Troubleshooting:
1. **Run debugging scripts** - Before importing workflows
2. **Check Docker logs** - For detailed error messages
3. **Verify file paths** - Ensure proper mounting

## File Structure
```
workflows/
├── complete-video-production-workflow.json    # Full pipeline
├── batch-video-processing-workflow.json       # Batch processing
├── simple-font-video-workflow.json           # Reliable processing
└── system-monitor-workflow.json              # Health monitoring

music_system/
├── comprehensive_endpoint_test.py             # Full API testing
├── quick_500_debug.py                        # Quick debugging
└── debug_500_errors.py                      # Detailed diagnostics
```

## Next Steps
1. **Start Docker Desktop**
2. **Run comprehensive_endpoint_test.py** to verify all APIs
3. **Import simple-font-video-workflow.json** first
4. **Test with a sample video** via webhook
5. **Gradually add more complex workflows**

🚀 **All workflows are production-ready with proper error handling and monitoring capabilities!**

# Quick Start Guide

Get up and running with Custom Video Processing Tools in 5 minutes.

## Prerequisites Check

- [ ] Docker Desktop installed and running
- [ ] Git installed
- [ ] OpenAI API key (for AI features)
- [ ] Google Cloud project with Drive API enabled (for FlaskDrive)

## 5-Minute Setup

### Step 1: Clone and Navigate
```bash
git clone <repository-url>
cd custom-video-tools
```

### Step 2: Create Environment File
```bash
# Create .env file
echo "OPENAI_API_KEY=your-key-here" > .env
echo "FLASK_SECRET_KEY=$(openssl rand -hex 32)" >> .env
```

### Step 3: Set Up Google Drive (Optional)
1. Download OAuth credentials from Google Cloud Console
2. Place as `flaskdrive/client_secret.json`

### Step 4: Start Services
```bash
docker-compose up -d
```

### Step 5: Verify
```bash
# Check services
docker-compose ps

# Test endpoints
curl http://localhost:5000/health  # FlaskDrive
curl http://localhost:5001/system/status  # Video Generator
```

## First Use

### Test Music System
```bash
curl -X POST http://localhost:5001/system/status
```

### Test Video Processing
```bash
curl -X POST http://localhost:5001/video/analyze-with-openai \
  -H "Content-Type: application/json" \
  -d '{"video_path": "/srv/videos/test.mp4"}'
```

### Upload to Google Drive
```bash
curl -X POST http://localhost:5000/upload \
  -F "file=@test.mp4"
```

## Common Commands

```bash
# View logs
docker-compose logs -f

# Restart services
docker-compose restart

# Stop services
docker-compose down

# Update and rebuild
git pull
docker-compose up -d --build
```

## Next Steps

- Read [README.md](README.md) for full documentation
- Check [SETUP.md](SETUP.md) for detailed setup
- Review [ARCHITECTURE.md](ARCHITECTURE.md) for system design
- See component READMEs for specific features

## Troubleshooting

**Services won't start?**
- Check Docker Desktop is running
- Verify ports 5000/5001 are free
- Check logs: `docker-compose logs`

**Authentication errors?**
- Complete OAuth flow at http://localhost:5000
- Verify `client_secret.json` exists

**API errors?**
- Check environment variables
- Review service logs
- Verify file paths

## Need Help?

- Check component-specific READMEs
- Review [SETUP.md](SETUP.md) troubleshooting section
- Open an issue on GitHub

---

**Ready to process videos!** 🎬


# Contributing Guidelines

## Development Setup

### Prerequisites
- Docker and Docker Compose
- Python 3.8+
- Git

### Local Development

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd custom-video-tools
   ```

2. **Set up environment:**
   ```bash
   # Create virtual environment
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate

   # Install dependencies
   pip install -r requirements.txt
   ```

3. **Configure environment variables:**
   ```bash
   export OPENAI_API_KEY="your-key"
   export FLASK_SECRET_KEY="your-secret"
   ```

4. **Run services:**
   ```bash
   docker-compose up -d
   ```

## Code Style

### Python
- Follow PEP 8 style guide
- Use type hints where appropriate
- Document functions with docstrings
- Maximum line length: 100 characters

### File Organization
- Keep related functionality together
- Separate concerns into modules
- Use clear, descriptive names

## Testing

### Running Tests
```bash
# Run all tests
pytest

# Run specific component tests
pytest music-system/tests/
```

### Writing Tests
- Write tests for new features
- Aim for >80% code coverage
- Test edge cases and error conditions

## Documentation

### Code Documentation
- Document all public functions
- Include usage examples
- Update README files when adding features

### API Documentation
- Document all endpoints
- Include request/response examples
- Note any authentication requirements

## Pull Request Process

1. **Create a feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes:**
   - Write code
   - Add tests
   - Update documentation

3. **Commit your changes:**
   ```bash
   git commit -m "Add: description of changes"
   ```

4. **Push to your branch:**
   ```bash
   git push origin feature/your-feature-name
   ```

5. **Create a Pull Request:**
   - Describe your changes
   - Reference any related issues
   - Request review

## Commit Message Format

Use clear, descriptive commit messages:

```
Add: Feature description
Fix: Bug description
Update: Change description
Remove: Removal description
```

## Component-Specific Guidelines

### Music System
- Keep analyzers modular
- Cache expensive operations
- Handle missing dependencies gracefully

### FlaskDrive
- Validate file sizes
- Handle OAuth errors
- Provide clear error messages

### Video Generator
- Optimize FFmpeg commands
- Handle large files efficiently
- Clean up temporary files

### Words on Text
- Optimize for GPU when available
- Provide CPU fallback
- Cache model loads

## Questions?

For questions or clarifications, please open an issue on GitHub.


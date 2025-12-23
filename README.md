# 🔍 TruthScan AI - Deepfake Detection Platform

A comprehensive, production-ready deepfake detection system using advanced forensic analysis powered by AI. This project combines visual, audio, and audio-visual synchronization detection to identify manipulated media.

## Features

✅ **Multi-Modal Analysis**
- Visual deepfake detection (RetinaFace + XceptionNet)
- Audio authenticity detection (RawNet2)
- Audio-Visual synchronization analysis (SyncNet)
- Temporal analysis with timeline visualization

✅ **Modern Tech Stack**
- Backend: Flask + Celery + Redis + SQLAlchemy
- Frontend: React + Vite + TailwindCSS + Recharts
- Database: SQLite (dev) + PostgreSQL (production)
- Deployment: Docker + Docker Compose

✅ **Advanced Features**
- Real-time job status polling
- Interactive timeline with timeline-to-video sync
- Forensic-style reporting with LLaMA-2 integration
- Suspicious segment detection and highlighting
- Progress tracking and error handling

## Quick Start

### Prerequisites
- Python 3.11+
- Node.js 18+
- Docker & Docker Compose (for containerized deployment)
- Redis (or use Docker)
- PostgreSQL (optional, for production)

### Local Development

#### Backend Setup

```bash
# Clone and navigate
cd truthscan-ai/backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cp ../.env.example .env

# Initialize database
flask db upgrade  # or manually with Flask shell

# Run development server
flask run
# Server runs on http://localhost:5000
```

#### Celery Worker (separate terminal)

```bash
cd truthscan-ai/backend
celery -A celery_worker worker --loglevel=info
```

#### Frontend Setup

```bash
cd truthscan-ai/frontend

# Install dependencies
npm install

# Start dev server
npm run dev
# Frontend runs on http://localhost:5173
```

#### Redis (required for Celery)

```bash
# Using Docker
docker run -d -p 6379:6379 redis:7-alpine

# Or install locally and run
redis-server
```

### Verify Setup

Open http://localhost:5173 in your browser. You should see the TruthScan AI interface.

## Docker Deployment

### Build and Run

```bash
# Build images
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop services
docker-compose down
```

### Access Services

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:5000/api
- **Celery Flower** (monitoring): http://localhost:5555
- **Health Check**: http://localhost:5000/api/health

### Database Initialization

```bash
# For PostgreSQL (in Docker)
docker-compose exec postgres createdb -U truthscan truthscan_db

# Or connect to postgres service and run migrations
docker-compose exec backend flask db upgrade
```

## API Documentation

### Upload Media for Analysis

**POST** `/api/analyze`

Request (multipart/form-data):
```json
{
  "file": "<binary file data>",
  "media_type": "video|image|audio"
}
```

Response:
```json
{
  "job_id": "uuid-string",
  "status": "pending",
  "message": "Analysis job queued"
}
```

### Get Job Status

**GET** `/api/status/:job_id`

Response:
```json
{
  "job_id": "uuid-string",
  "status": "pending|processing|done|error",
  "progress": 45.5,
  "error_message": null,
  "created_at": "2025-01-01T12:00:00",
  "updated_at": "2025-01-01T12:05:00",
  "completed_at": null
}
```

### Get Analysis Results

**GET** `/api/results/:job_id`

Response:
```json
{
  "overall_score": 0.82,
  "verdict": "likely_real",
  "timeline": [
    {
      "start": 0,
      "end": 1,
      "visual": 0.12,
      "audio": 0.05,
      "sync": 0.08,
      "overall": 0.075
    },
    {
      "start": 1,
      "end": 2,
      "visual": 0.91,
      "audio": 0.23,
      "sync": 0.67,
      "overall": 0.60
    }
  ],
  "suspicious_segments": [
    {
      "start": 1.2,
      "end": 1.8,
      "suspiciousness": 0.75,
      "reason": "High face manipulation + lip-sync mismatch"
    }
  ],
  "report": "Forensic analysis report text...",
  "media_url": "/storage/uploads/{job_id}/filename"
}
```

## Project Structure

```
truthscan-ai/
├── backend/
│   ├── app.py                    # Flask application
│   ├── celery_worker.py          # Celery worker and pipeline
│   ├── requirements.txt          # Python dependencies
│   ├── Dockerfile               # Backend container
│   ├── config/
│   │   └── settings.py          # Configuration
│   ├── models/
│   │   └── __init__.py          # SQLAlchemy models
│   ├── routes/
│   │   ├── analyze.py           # Upload endpoint
│   │   ├── status.py            # Status endpoint
│   │   └── results.py           # Results endpoint
│   ├── preprocess/
│   │   ├── video_utils.py
│   │   ├── image_utils.py
│   │   └── audio_utils.py
│   ├── visual/
│   │   ├── face_detector.py     # RetinaFace mock
│   │   ├── image_detector.py    # XceptionNet mock
│   │   └── __init__.py
│   ├── audio/
│   │   ├── rawnet2_model.py     # RawNet2 mock
│   │   └── __init__.py
│   ├── avsync/
│   │   ├── syncnet_model.py     # SyncNet mock
│   │   └── __init__.py
│   ├── fusion/
│   │   ├── timeline_builder.py
│   │   ├── verdict_module.py
│   │   └── __init__.py
│   ├── llm/
│   │   ├── llama2_client.py
│   │   ├── report_service.py
│   │   └── __init__.py
│   ├── storage/
│   │   └── file_manager.py
│   └── utils/
│       └── __init__.py
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── index.html
│   ├── src/
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   ├── index.css
│   │   ├── components/
│   │   │   ├── UploadForm.jsx
│   │   │   ├── JobStatus.jsx
│   │   │   ├── ResultsTimeline.jsx
│   │   │   ├── MediaPlayer.jsx
│   │   │   └── ReportPanel.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   └── utils/
│   │       └── hooks.js
│   └── public/
├── docker-compose.yml
├── .env.example
└── README.md
```

## Processing Pipeline

The analysis follows this workflow:

```
Upload Media
    ↓
[M1] Job Creation
    ↓
[M2] File Storage
    ↓
[M3] Preprocessing (extract frames/audio/normalize)
    ↓
[M4] Visual Analysis (face detection + deepfake scoring)
    ↓
[M5] Audio Analysis (voice authenticity scoring)
    ↓
[M6] AV Sync Analysis (lip-sync detection)
    ↓
[M7] Fusion (build timeline + compute verdict)
    ↓
[M8] LLaMA Report Generation
    ↓
Store Results → Update Status → Complete
```

## Configuration

### Environment Variables

```bash
# Flask
DEBUG=False
SECRET_KEY=your-secret-key
SQLALCHEMY_DATABASE_URI=sqlite:///truthscan.db

# Celery & Redis
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# CORS
CORS_ORIGINS=http://localhost:5173,http://localhost:3000

# Logging
LOG_LEVEL=INFO

# Frontend
VITE_API_URL=http://localhost:5000/api
```

## Model Architecture

### Visual Analysis (Image/Video)
- **RetinaFace**: Face detection and localization
- **XceptionNet**: Image-level deepfake classification
- Output: Per-frame deepfake probability (0.0-1.0)

### Audio Analysis
- **RawNet2**: Speaker verification for voice authentication
- Output: Per-chunk voice authenticity (0.0-1.0)

### AV Synchronization
- **SyncNet**: Detects lip-sync misalignment
- Output: Sync distance (0.0=perfect, 1.0=bad)

### Fusion & Verdict
- Weighted combination: 50% visual + 30% audio + 20% sync
- Thresholds: >0.7=Fake, 0.4-0.7=Inconclusive, <0.4=Real

## Mock Implementation

This project uses **mock implementations** of ML models for demonstration. To integrate real models:

1. **RetinaFace**: Replace `visual/face_detector.py` with actual model weights
2. **XceptionNet**: Replace `visual/image_detector.py` with trained classifier
3. **RawNet2**: Replace `audio/rawnet2_model.py` with voice verification model
4. **SyncNet**: Replace `avsync/syncnet_model.py` with actual sync detector
5. **LLaMA-2**: Integrate with actual LLaMA-2 API in `llm/llama2_client.py`

## Performance & Optimization

### Recommendations

- **Video Processing**: Use GPU acceleration (CUDA/cuDNN) for faster inference
- **Batch Processing**: Process multiple frames simultaneously
- **Caching**: Redis caches job results for 24 hours
- **Concurrency**: Adjust Celery worker concurrency based on hardware

### Resource Requirements

- CPU: 4+ cores recommended
- RAM: 8GB minimum (16GB+ for video processing)
- GPU: NVIDIA GPU recommended for real models
- Storage: Depends on media size (500MB-1GB typical)

## Troubleshooting

### Port Already in Use
```bash
# Change port in vite.config.js (frontend)
# Change port in app.py (backend)
```

### Redis Connection Error
```bash
# Ensure Redis is running
redis-cli ping
# Should return: PONG
```

### Database Issues
```bash
# Reset database (removes all data)
rm truthscan.db  # or rm -rf postgresql_data/
# Restart services
docker-compose down && docker-compose up -d
```

### Out of Memory
```bash
# Reduce sample rate for video processing
# Edit config/settings.py
CHUNK_DURATION = 0.5  # Smaller chunks
```

## Testing

### Test Upload
```bash
curl -X POST http://localhost:5000/api/analyze \
  -F "media_type=video" \
  -F "file=@test_video.mp4"
```

### Check Job Status
```bash
curl http://localhost:5000/api/status/job-id-here
```

### Get Results
```bash
curl http://localhost:5000/api/results/job-id-here
```

## Security Considerations

⚠️ **Important for Production:**

1. Change `SECRET_KEY` in `.env`
2. Use HTTPS/TLS in production
3. Implement authentication for API endpoints
4. Rate limit uploads (500MB max)
5. Sanitize file names
6. Use environment variables for sensitive data
7. Enable CORS only for trusted domains
8. Implement file scanning (ClamAV)

## Contributing

Contributions are welcome! Areas for improvement:

- [ ] Implement real ML models
- [ ] Add WebRTC for streaming analysis
- [ ] Implement user accounts and history
- [ ] Add batch processing
- [ ] Improve UI/UX
- [ ] Add more metrics and visualizations
- [ ] Performance optimization

## License

MIT License - See LICENSE file for details

## Citation

If you use this project, please cite:

```bibtex
@software{truthscan2025,
  title={TruthScan AI: Deepfake Detection Platform},
  author={Your Name},
  year={2025},
  url={https://github.com/yourusername/truthscan-ai}
}
```

## References

- RetinaFace: [Deng et al., 2020](https://arxiv.org/abs/1905.00641)
- XceptionNet: [Chollet, 2016](https://arxiv.org/abs/1610.02357)
- RawNet2: [Zhai et al., 2021](https://arxiv.org/abs/2010.13581)
- SyncNet: [Chung et al., 2017](https://arxiv.org/abs/1705.02969)

## Support

For issues, questions, or feedback:
- Open an issue on GitHub
- Check existing documentation
- Review mock data format in API docs

---

**⚠️ Disclaimer**: This tool is for research and educational purposes. Deepfake detection is an ongoing challenge. Results should not be considered definitive proof. For critical applications, professional forensic analysis is recommended.

**Made with ❤️ for forensic multimedia analysis**
#   T r u t h S c a n  
 
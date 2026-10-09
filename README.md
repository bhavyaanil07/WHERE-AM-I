# WHERE AM I - Campus Navigation System

Indoor navigation system for Amrita Vishwa Vidyapeetham with voice and text input, AI-powered routing, and interactive response.

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    15.6" Touchscreen Kiosk                  │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  🎤 Voice    │  │  ⌨️  Text    │  │  🗣️  Output │     │
│  │  Microphone  │  │  Keyboard    │  │  Speaker    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│         │                 │                   ▲            │
└─────────────────────────────────────────────────────────────┘
           │                 │                   │
           ├─────────────────┴───────────────────┤
           │                                     │
           ▼                                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend API (FastAPI + Python)                │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ GPT-4 Agent: Query parsing & dialog management      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Routing Engine: Dijkstra algorithm for pathfinding  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Google Cloud: TTS, STT, map services               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ SQLite Database: Campus map, rooms, connectivity    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Key Features

### Dual Input Modes
- **🎤 Voice Input**: Speak destination in any of 5 Indian languages
- **⌨️ Text Input**: Type location name or room number

### Intelligent Processing
- **GPT-4 Agent**: Natural language understanding with conversation context
- **Dijkstra Routing**: Efficient path planning through multi-floor buildings
- **Error Handling**: Graceful fallbacks for API failures

### Interactive Output
- **🔊 Voice Response**: Conversational like Gemini, responds to queries
- **🗺️ Visual Map**: Real-time route display on 15.6" screen
- **📱 QR Code**: Mobile access to route details

### Supported Languages
- English (en-US)
- Hindi (hi-IN)
- Tamil (ta-IN)
- Malayalam (ml-IN)
- Telugu (te-IN)
- Kannada (kn-IN)

## Project Structure

```
where-am-i/
├── config.py                 # Configuration management
├── database.py              # SQLite operations
├── google_apis.py           # Google Cloud integration (TTS/STT)
├── gpt4_agent.py            # GPT-4 query parsing & response
├── main.py                  # FastAPI server
├── qrcode_generator.py      # QR code generation
├── routing_engine.py        # Dijkstra algorithm & routing
├── ui.py                    # 15.6" Touchscreen UI states
├── requirements.txt         # Python dependencies
├── SETUP_GUIDE.md          # Detailed setup instructions
└── README.md               # This file
```

## Core Modules

### 1. **config.py** - Configuration Manager
Handles all system configuration with environment variables and defaults.

```python
from config import get_config
config = get_config()
# Access: config.openai.api_key, config.google.project_id, etc.
```

### 2. **database.py** - Campus Data Management
SQLite-based storage for buildings, floors, rooms, and routing graph.

```python
from database import get_database
db = get_database()
locations = db.search_locations("library")
route = db.load_graph()
```

### 3. **routing_engine.py** - Path Planning
Dijkstra's algorithm with multi-floor support and turn-by-turn directions.

```python
from routing_engine import RoutingEngine
engine = RoutingEngine()
route = engine.find_route("lib_main", "cafeteria")
print(route.get_summary())
```

### 4. **gpt4_agent.py** - Intelligent Conversation
GPT-4 powered agent for query understanding and interactive dialogue.

```python
from gpt4_agent import parse_query, generate_response
result = parse_query("Where is the library?", user_id="user1")
response = generate_response(result.destination, route_summary)
```

### 5. **google_apis.py** - Cloud Integration
Google Cloud Text-to-Speech (WaveNet) and Speech-to-Text APIs.

```python
from google_apis import GoogleAPIManager
google = GoogleAPIManager(project_id)
audio_bytes = google.tts.synthesize_speech("Hello world", "en-US")
text = google.stt.transcribe_audio(audio_bytes, "en-US")
```

### 6. **main.py** - REST API Server
FastAPI endpoints for text queries, voice queries, and route management.

```bash
# Start server
python main.py

# Access at http://localhost:8000
# Docs at http://localhost:8000/docs
```

### 7. **qrcode_generator.py** - Mobile Sharing
Generate and manage QR codes for mobile route access.

```python
from qrcode_generator import qr_service
sharing_info = qr_service.create_shareable_route(
    route_id, source_id, destination_id
)
```

### 8. **ui.py** - Touchscreen Interface
State-based UI manager for 15.6" display with text, voice, and map rendering.

```python
from ui import TouchscreenUI, UIState
ui = TouchscreenUI()
ui.set_state(UIState.SHOWING_ROUTE)
screen = ui.render_current_screen()
```

## API Reference

### Text Query
```bash
POST /api/query/text
{
  "query": "Where is the main library?",
  "language": "en-US",
  "user_id": "student_001"
}
```

### Voice Query
```bash
POST /api/query/voice
Content-Type: multipart/form-data

audio_file: <binary audio data>
language: "auto"
user_id: "student_001"
```

### Search Locations
```bash
GET /api/search?query=library&limit=10
```

### Get Route
```bash
GET /api/route/{route_id}
```

## Installation & Deployment

### Quick Start (Development)

```bash
# 1. Clone/setup
git clone <repo>
cd where-am-i

# 2. Create environment
python3 -m venv venv
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure
cp .env.example .env
# Edit .env with your API keys

# 5. Initialize database
python database.py

# 6. Run server
python main.py
```

### Raspberry Pi Deployment

See **SETUP_GUIDE.md** for complete Raspberry Pi 5 setup with systemd service, audio configuration, and display setup.

Key points:
- Use Raspberry Pi OS 64-bit or Ubuntu Server 22.04
- 32GB MicroSD card minimum
- USB microphone and speaker
- 15.6" USB/HDMI display

## Performance & Optimization

### Response Times
- **Text Query**: ~2-3 seconds (GPT-4 parsing + routing)
- **Voice Query**: ~3-5 seconds (STT + parsing + routing)
- **Route Display**: <500ms (map rendering)
- **TTS Output**: Varies by text length

### Resource Usage
- **Memory**: ~200-300MB base + API buffers
- **Disk**: ~1GB for OS, cache, and database
- **Network**: 1-5MB per query (primarily API calls)

### Optimization Features
- Query result caching (in-database)
- Route pre-computation for common destinations
- Audio compression and streaming
- Map tile caching

## Troubleshooting

### Common Issues

**1. OpenAI API Key Error**
```
Error: OPENAI_API_KEY environment variable not set
```
Solution: Add API key to `.env` file

**2. Google Cloud Credentials Error**
```
Error: GOOGLE_APPLICATION_CREDENTIALS file not found
```
Solution: Download JSON credentials from GCP console

**3. Database Lock Error**
```
sqlite3.OperationalError: database is locked
```
Solution: Close other database connections, increase timeout in config

**4. Audio Issues on Raspberry Pi**
```bash
# Check audio devices
arecord -l
aplay -l

# Test audio
arecord -d 5 test.wav
aplay test.wav
```

## Testing

```bash
# Run API tests
pytest tests/ -v

# Test text query
curl -X POST http://localhost:8000/api/query/text \
  -H "Content-Type: application/json" \
  -d '{"query": "library", "user_id": "test"}'

# Check health
curl http://localhost:8000/health
```

## Cost Analysis

Monthly estimates for active kiosk:

| Service | Cost |
|---------|------|
| Google Cloud TTS | $5-10 |
| Google Cloud STT | $5-10 (optional) |
| OpenAI GPT-4 | $10-20 |
| **Total** | **$20-40** |

Cost optimization strategies:
- Cache frequent queries
- Pre-compute common routes
- Use lower-cost APIs for non-critical paths

## Timeline & Status

### Week 1-2: Core Backend ✅
- [x] FastAPI server setup
- [x] Database schema design
- [x] Routing engine (Dijkstra)
- [x] GPT-4 agent integration
- [x] Google Cloud APIs

### Week 2-3: Frontend & Testing 🔄
- [ ] Web/Native UI implementation
- [ ] Raspberry Pi deployment
- [ ] End-to-end testing
- [ ] Performance optimization
- [ ] Security hardening

### Week 3+: Production
- [ ] User testing & feedback
- [ ] Deployment to campus kiosk
- [ ] Monitoring & maintenance

## Team & Contact

- **Developer**: Bhavya Anil (B.Tech, Amrita)
- **Advisor**: PhD Faculty, Amrita Vishwa Vidyapeetham
- **Institution**: Amrita Vishwa Vidyapeetham

## License

Internal use only for Amrita Vishwa Vidyapeetham

## Acknowledgments

- OpenAI for GPT-4 API
- Google Cloud for TTS/STT services
- FastAPI for web framework
- NetworkX for routing algorithms

---

**Last Updated**: October 9, 2026  
**Deadline**: October 31, 2026  
**Status**: Core backend complete, ready for frontend integration

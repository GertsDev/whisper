# 🏗️ Technical Architecture & Implementation

## **🎯 System Overview**

Our voice intelligence platform consists of two integrated products powered by a unified backend:

1. **Local Meeting Intelligence** - Privacy-first Otter.ai alternative
2. **System-Wide Voice Input** - Context-aware voice control for macOS

Both leverage local Whisper models and adaptive LLM selection based on hardware capabilities.

---

## **🧠 Adaptive Model Architecture**

### **Smart Model Selection Based on Hardware**

```python
class AdaptiveModelManager:
    def __init__(self):
        self.hardware_tier = self.detect_hardware_tier()
        self.models = {
            'basic': 'microsoft/Phi-3.5-mini-instruct',      # 3.8B - M1 8GB+
            'performance': 'deepseek-ai/DeepSeek-R1-Distill-Llama-8B', # 8B - 16GB+
            'enterprise': 'Qwen/Qwen2.5-Coder-7B-Instruct'    # 7.6B - High performance
        }
    
    def detect_hardware_tier(self) -> str:
        """Detect optimal model tier based on available hardware"""
        memory_gb = self.get_available_memory()
        cpu_info = self.get_cpu_info()
        
        if memory_gb >= 24 and 'M2 Pro' in cpu_info or 'M3' in cpu_info:
            return 'enterprise'
        elif memory_gb >= 16:
            return 'performance'  
        else:
            return 'basic'
```

### **Hardware Compatibility Matrix**

| Hardware | RAM | Primary Model | Fallback | Performance |
|----------|-----|---------------|----------|-------------|
| **M1 MacBook (8GB)** | 8GB | Phi-3.5-Mini (3.8B) | - | ⭐⭐⭐⭐ Fast |
| **M1 MacBook (16GB)** | 16GB | DeepSeek-R1-Distill (8B) | Phi-3.5-Mini | ⭐⭐⭐⭐ Excellent |
| **M1 Pro/Max (16GB+)** | 16-64GB | DeepSeek-R1-Distill (8B) | Qwen2.5-Coder | ⭐⭐⭐⭐⭐ Optimal |
| **M2/M3 (16GB+)** | 16-128GB | Qwen2.5-Coder (7.6B) | DeepSeek-R1 | ⭐⭐⭐⭐⭐ Superior |

---

## **⚡ Real-Time Processing Pipeline**

### **Whisper Integration for Voice Input**

```python
class StreamingWhisperProcessor:
    def __init__(self):
        # Always use turbo for real-time performance
        self.whisper_model = whisper.load_model("turbo")
        self.audio_buffer = collections.deque(maxlen=5)  # 5-second rolling buffer
        self.vad = VoiceActivityDetector()
        
    async def process_voice_stream(self, audio_chunk: bytes) -> dict:
        """Process voice input with <2s latency on all supported hardware"""
        
        # Step 1: Voice Activity Detection (10ms)
        if not self.vad.contains_speech(audio_chunk):
            return None
            
        # Step 2: Buffer management (1ms)
        self.audio_buffer.append(audio_chunk)
        
        # Step 3: Process when sufficient context (1.5s)
        if len(self.audio_buffer) >= 3:
            combined_audio = b''.join(self.audio_buffer)
            
            # Step 4: Whisper processing (600-800ms with turbo)
            result = await self.transcribe_async(combined_audio)
            
            return {
                'text': result['text'],
                'confidence': result.get('confidence', 0.0),
                'language': result.get('language', 'en')
            }
```

### **Context-Aware Processing Engine**

```python
class ContextEngine:
    def __init__(self):
        self.contexts = {
            'terminal': TerminalContext(),
            'code_editor': CodeEditorContext(), 
            'browser': BrowserContext(),
            'meeting': MeetingContext(),
            'general': GeneralContext()
        }
        
    def detect_active_context(self) -> str:
        """Real-time application context detection"""
        active_app = self.get_active_application()
        window_title = self.get_window_title()
        
        # Smart context hierarchy
        if any(term in active_app for term in ['Terminal', 'iTerm', 'Warp']):
            return 'terminal'
        elif any(editor in active_app for editor in ['Code', 'Xcode', 'Vim', 'Cursor']):
            return 'code_editor'
        elif any(browser in active_app for browser in ['Safari', 'Chrome', 'Firefox']):
            if any(meeting in window_title for meeting in ['Zoom', 'Meet', 'Teams']):
                return 'meeting'
            return 'browser'
        else:
            return 'general'
            
    def process_with_context(self, text: str, context: str) -> str:
        """Apply context-specific processing"""
        return self.contexts[context].process(text)
```

---

## **🌐 System Architecture Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUDIO CAPTURE LAYER                             │
├─────────────────┬─────────────────┬─────────────────────────────────┤
│  Browser Ext.   │   macOS Core    │    Direct Microphone            │
│  (Web Audio)    │     Audio       │    (Web Audio API)              │
│  Chrome/Safari  │   (Blackhole)   │                                 │
└─────────────────┴─────────────────┴─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  VOICE PROCESSING ENGINE                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────────────┐ │
│  │   Whisper   │ │   Audio     │ │    Voice Activity Detection     │ │
│  │   Turbo     │ │  Buffering  │ │         (VAD)                   │ │
│  │  (Local)    │ │ Management  │ │                                 │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    CONTEXT INTELLIGENCE                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────────────┐ │
│  │ Application │ │  Command    │ │      Smart Processing           │ │
│  │  Context    │ │Interpretation│ │   (Terminal/Code/Browser)       │ │
│  │ Detection   │ │             │ │                                 │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  ADAPTIVE AI PROCESSING                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────────────┐ │
│  │ Phi-3.5-Mini│ │ DeepSeek-R1 │ │    Qwen2.5-Coder                │ │
│  │   (M1 8GB)  │ │ (16GB RAM)  │ │   (High Performance)            │ │
│  │   3.8B      │ │    8B       │ │        7.6B                     │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      OUTPUT LAYER                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────────────┐ │
│  │   macOS     │ │   Web UI    │ │     Meeting Summaries           │ │
│  │Accessibility│ │ Dashboard   │ │    & Action Items               │ │
│  │     API     │ │             │ │                                 │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## **💾 Data Storage Architecture**

### **Local-First Database Design**

```sql
-- Core tables optimized for local performance
CREATE TABLE meetings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    status meeting_status DEFAULT 'active',
    audio_file_path VARCHAR(500),
    model_used VARCHAR(100), -- Track which AI model was used
    hardware_tier VARCHAR(20), -- Track hardware context
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE transcripts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    meeting_id UUID REFERENCES meetings(id) ON DELETE CASCADE,
    speaker_id VARCHAR(50),
    content TEXT NOT NULL,
    start_time DECIMAL(10,3) NOT NULL,
    end_time DECIMAL(10,3) NOT NULL,
    confidence DECIMAL(4,3),
    context VARCHAR(50), -- terminal, code_editor, browser, meeting
    processed_by VARCHAR(100), -- Which model processed this
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes optimized for common queries
CREATE INDEX idx_transcripts_meeting_time ON transcripts(meeting_id, start_time);
CREATE INDEX idx_transcripts_context ON transcripts(context);
CREATE INDEX idx_meetings_status ON meetings(status);
```

### **File Storage Organization**

```
~/Library/Application Support/VoiceIntelligence/
├── models/           # Cached Whisper and LLM models
│   ├── whisper/      # Whisper models
│   └── llm/          # Local LLM models via Ollama
├── audio/            # Meeting recordings (encrypted)
│   ├── 2025-01/      # Organized by month
│   └── 2025-02/
├── transcripts/      # Exported transcripts
│   ├── pdf/
│   ├── docx/
│   └── txt/
├── database/         # SQLite database files
└── logs/            # Application logs
```

---

## **🔧 macOS System Integration**

### **Universal Text Injection**

```python
class MacOSTextInjection:
    def __init__(self):
        self.accessibility_api = AccessibilityController()
        self.pasteboard = NSPasteboard.generalPasteboard()
        
    def inject_text_universal(self, text: str, context: str):
        """Works in ANY macOS application"""
        
        # Strategy 1: Accessibility API (Primary)
        if self.try_accessibility_injection(text):
            return True
            
        # Strategy 2: Pasteboard + Cmd+V (Fallback)
        if self.try_pasteboard_injection(text):
            return True
            
        # Strategy 3: Keystroke simulation (Last resort)
        return self.try_keystroke_simulation(text)
        
    def try_accessibility_injection(self, text: str) -> bool:
        """Direct text injection via Accessibility API"""
        try:
            focused_element = self.accessibility_api.get_focused_element()
            if focused_element and focused_element.editable:
                focused_element.set_value(focused_element.value + text)
                return True
        except Exception:
            pass
        return False
```

### **Global Audio Capture**

```python
class UnifiedAudioCapture:
    def __init__(self):
        self.sources = {
            'microphone': MicrophoneCapture(),
            'system': SystemAudioCapture(),  # Via Blackhole
            'browser': BrowserExtensionCapture()
        }
        
    def start_capture(self, source_type: str = 'auto'):
        """Start audio capture from specified or detected source"""
        if source_type == 'auto':
            source_type = self.detect_optimal_source()
            
        return self.sources[source_type].start_streaming(
            sample_rate=16000,
            chunk_size=1024,
            callback=self.process_audio_chunk
        )
        
    def detect_optimal_source(self) -> str:
        """Automatically detect best audio source"""
        active_app = self.get_active_application()
        
        if 'zoom' in active_app.lower() or 'meet' in active_app.lower():
            return 'system'  # Capture system audio for meetings
        else:
            return 'microphone'  # Direct microphone for voice input
```

---

## **⚡ Performance Optimizations**

### **Model Loading and Caching**

```python
class OptimizedModelLoader:
    def __init__(self):
        self.model_cache = {}
        self.hardware_tier = self.detect_hardware()
        
    async def load_model_optimized(self, model_type: str):
        """Load models with hardware-specific optimizations"""
        
        if model_type in self.model_cache:
            return self.model_cache[model_type]
            
        # Hardware-specific loading strategies
        if self.hardware_tier == 'basic':  # M1 8GB
            model = self.load_quantized_model('phi3.5:3.8b-mini-instruct-q4_0')
        elif self.hardware_tier == 'performance':  # 16GB+
            model = self.load_model('deepseek-r1:8b')
        else:  # Enterprise
            model = self.load_model('qwen2.5-coder:7b')
            
        self.model_cache[model_type] = model
        return model
```

### **Memory Management**

```python
class MemoryManager:
    def __init__(self):
        self.max_memory_usage = self.calculate_safe_memory_limit()
        
    def calculate_safe_memory_limit(self) -> int:
        """Calculate safe memory usage based on system specs"""
        total_memory = psutil.virtual_memory().total
        
        # Leave enough memory for macOS and other apps
        if total_memory <= 8 * 1024**3:  # 8GB systems
            return int(total_memory * 0.4)  # Use max 40%
        elif total_memory <= 16 * 1024**3:  # 16GB systems
            return int(total_memory * 0.6)  # Use max 60%
        else:  # 32GB+ systems
            return int(total_memory * 0.7)  # Use max 70%
```

---

## **🌐 Web Dashboard Architecture**

### **Frontend Technology Stack**

```typescript
// React + TypeScript for type safety
interface VoiceIntelligenceState {
  currentMeeting?: Meeting;
  transcriptionActive: boolean;
  realtimeTranscript: TranscriptSegment[];
  meetings: Meeting[];
  activeContext: ApplicationContext;
  modelTier: 'basic' | 'performance' | 'enterprise';
}

// Socket.IO for real-time communication
const useRealtimeTranscription = () => {
  const [transcript, setTranscript] = useState<TranscriptSegment[]>([]);
  
  useEffect(() => {
    socket.on('transcript_update', (segment: TranscriptSegment) => {
      setTranscript(prev => [...prev, segment]);
    });
    
    return () => socket.off('transcript_update');
  }, []);
};
```

### **Backend API Design**

```python
from fastapi import FastAPI, WebSocket
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="Voice Intelligence Platform")

@app.websocket("/ws/transcription/{meeting_id}")
async def websocket_transcription(websocket: WebSocket, meeting_id: str):
    """Real-time transcription WebSocket endpoint"""
    await websocket.accept()
    
    try:
        while True:
            # Receive audio data
            audio_data = await websocket.receive_bytes()
            
            # Process with appropriate model based on hardware
            result = await transcription_service.process_chunk(
                audio_data, meeting_id
            )
            
            # Send back transcription result
            if result:
                await websocket.send_json(result)
                
    except WebSocketDisconnect:
        await cleanup_meeting_session(meeting_id)
```

---

## **🔒 Security & Privacy Architecture**

### **Local-First Security**

```python
class PrivacyGuard:
    def __init__(self):
        self.encryption_key = self.load_or_generate_key()
        self.cipher = Fernet(self.encryption_key)
        
    def encrypt_audio_file(self, file_path: str):
        """Encrypt all stored audio files"""
        with open(file_path, 'rb') as f:
            encrypted_data = self.cipher.encrypt(f.read())
            
        with open(file_path + '.enc', 'wb') as f:
            f.write(encrypted_data)
            
        os.remove(file_path)  # Remove unencrypted version
        
    def ensure_no_network_calls(self):
        """Verify no data leaves the machine"""
        # Block all external network requests in production
        blocked_domains = [
            'openai.com', 'anthropic.com', 'google.com',
            'microsoft.com', 'amazonaws.com'
        ]
        for domain in blocked_domains:
            self.block_domain(domain)
```

---

## **📊 Monitoring & Analytics**

### **Local Performance Metrics**

```python
class PerformanceMonitor:
    def __init__(self):
        self.metrics = {
            'transcription_latency': [],
            'model_inference_time': [],
            'memory_usage': [],
            'context_switch_time': [],
            'accuracy_estimates': []
        }
        
    async def track_operation(self, operation_type: str, func):
        """Track performance of critical operations"""
        start_time = time.time()
        result = await func()
        end_time = time.time()
        
        latency = end_time - start_time
        self.metrics[f'{operation_type}_latency'].append(latency)
        
        # Alert if performance degrades
        if latency > self.get_threshold(operation_type):
            await self.optimize_for_hardware()
            
        return result
```

---

**Last Updated**: 2025-08-07  
**Owner**: Technical Team  
**Next Review**: 2025-08-14
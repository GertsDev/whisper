# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Whisper is OpenAI's robust speech recognition model that performs multilingual speech recognition, speech translation, and language identification. It's a Transformer sequence-to-sequence model trained on diverse audio data.

## Development Commands

### Installation and Setup
```bash
# Install the package in development mode
pip install -e .

# Install with development dependencies  
pip install -e .[dev]

# Install external dependency (required for audio processing)
# macOS: brew install ffmpeg
# Ubuntu: sudo apt update && sudo apt install ffmpeg
```

### Testing
```bash
# Run all tests
pytest

# Run specific test files
pytest tests/test_transcribe.py
pytest tests/test_audio.py

# Run with CUDA tests (if available)
pytest -m requires_cuda
```

### Code Quality
```bash
# Format code with Black
black .

# Sort imports with isort  
isort .

# Lint with flake8
flake8
```

### Running the CLI
```bash
# Basic transcription
whisper audio.wav --model turbo

# Transcription with language specification
whisper audio.wav --language Japanese --model medium

# Translation to English
whisper audio.wav --language Japanese --task translate --model medium

# See all options
whisper --help
```

## Core Architecture

### Main Components

- **`whisper/__init__.py`**: Entry point with model loading (`load_model()`) and available models list
- **`whisper/model.py`**: Core Whisper transformer model implementation with `ModelDimensions` and `Whisper` classes
- **`whisper/audio.py`**: Audio processing utilities including mel spectrogram generation and audio loading via ffmpeg
- **`whisper/transcribe.py`**: High-level transcription interface and CLI implementation
- **`whisper/decoding.py`**: Low-level decoding logic with `DecodingOptions` and language detection
- **`whisper/tokenizer.py`**: Text tokenization using tiktoken, language definitions
- **`whisper/timing.py`**: Word-level timestamp alignment functionality

### Audio Processing Pipeline

1. **Load audio** (`load_audio()`) - Uses ffmpeg to convert to 16kHz mono
2. **Generate mel spectrogram** (`log_mel_spectrogram()`) - Creates 80-channel mel spectrogram  
3. **Pad/trim** (`pad_or_trim()`) - Ensures 30-second chunks (480,000 samples)
4. **Model inference** - Transformer processes 3000-frame mel inputs
5. **Decode tokens** - Convert model outputs to text with timestamps

### Model Variants

Available models with different size/speed tradeoffs:
- `tiny`, `tiny.en` (39M params) - Fastest, least accurate
- `base`, `base.en` (74M params) 
- `small`, `small.en` (244M params)
- `medium`, `medium.en` (769M params)
- `large` (1550M params) - Most accurate
- `turbo` (809M params) - Optimized version of large-v3, English-only tasks

Models are downloaded from Azure CDN and cached in `~/.cache/whisper/`.

### Key Constants

```python
# Audio processing (from audio.py)
SAMPLE_RATE = 16000  # Target sample rate
N_FFT = 400          # FFT window size  
HOP_LENGTH = 160     # Hop length for STFT
CHUNK_LENGTH = 30    # 30-second audio chunks
N_SAMPLES = 480000   # Samples per chunk
N_FRAMES = 3000      # Frames in mel spectrogram
```

## Testing Setup

The test suite uses pytest with the following configuration:
- Random seeds fixed for reproducibility (seed=42)
- `requires_cuda` marker for CUDA-specific tests  
- Test audio files in `tests/` directory (e.g., `jfk.flac`)

## Python API Usage Patterns

### Basic transcription:
```python
import whisper
model = whisper.load_model("turbo")  
result = model.transcribe("audio.mp3")
print(result["text"])
```

### Lower-level access:
```python  
# Language detection
_, probs = model.detect_language(mel)

# Direct decoding  
options = whisper.DecodingOptions()
result = whisper.decode(model, mel, options)
```

## Development Notes

- The codebase uses PyTorch with optional Triton acceleration on Linux x86_64
- CUDA detection is automatic via `torch.cuda.is_available()`
- Models use `weights_only=True` for secure loading (PyTorch >=1.13)
- Audio processing requires ffmpeg system dependency
- Word-level timestamps use cross-attention alignment heads
- The `turbo` model is optimized for transcription but not translation tasks

## 🎯 Strategic Context & Team Onboarding

**CRITICAL**: This is not just a Whisper transcription tool - we're building a **revolutionary voice intelligence platform** that will transform how developers interact with their computers.

### **Our Mission**
Build the world's first privacy-first, context-aware voice intelligence platform with two products:
1. **Local Meeting Intelligence** - Privacy-first Otter.ai alternative
2. **System-Wide Voice Input** - Context-aware voice control for macOS

### **Why This Matters**
- **$2B+ market opportunity** growing to $47B by 2030
- **Current solutions are failing**: macOS dictation degraded to 80-90% accuracy
- **Privacy crisis**: 42% of enterprises now concerned about cloud processing
- **No context-aware solutions exist** - we're creating a new category

### **Strategic Documentation (MUST READ)**

**Before contributing any code, ALL team members must read:**

📚 **Core Strategy Documents** (in `/strategy/` directory):
- [`01-product-vision.md`](./strategy/01-product-vision.md) - What we're building and why it's revolutionary
- [`02-market-opportunity.md`](./strategy/02-market-opportunity.md) - $2B+ market analysis and competitive landscape
- [`04-business-model.md`](./strategy/04-business-model.md) - AGPL v3 + Commercial dual licensing strategy
- [`06-development-roadmap.md`](./strategy/06-development-roadmap.md) - 6-day sprint development plan

### **Key Strategic Principles**
1. **Privacy First**: All processing happens locally, no cloud dependencies
2. **Context Intelligence**: System understands Terminal vs Code Editor vs Browser
3. **Developer Obsessed**: Built by developers, for developers  
4. **Real-time Performance**: Sub-2 second latency for all operations
5. **Open Source Growth**: Community-driven adoption, commercial sustainability

### **Technology Evolution**
We're evolving from basic Whisper transcription to:
- **Context-aware voice processing** with application intelligence
- **Real-time meeting transcription** with AI summaries
- **System-wide voice input** that works in every macOS application
- **Adaptive LLM integration** with hardware-optimized model selection:
  - **Phi-3.5-Mini (3.8B)**: M1 MacBook 8GB+ (fast, efficient)
  - **DeepSeek-R1-Distill (8B)**: 16GB+ RAM (best quality)
  - **Qwen2.5-Coder (7.6B)**: High-performance systems (enterprise)

### **Keeping Strategy Current**
- **Strategy docs must be updated** as we learn and pivot
- **Weekly roadmap reviews** to track progress against plan
- **Monthly strategy reviews** to validate market assumptions
- **All major technical decisions** should align with strategic vision

### **For New Team Members**
1. Read all strategy docs before touching code
2. Understand we're building TWO products on one platform
3. Every feature should support our "context-aware voice intelligence" vision
4. Ask questions about strategic alignment before implementing features
5. Contribute insights about user needs and market validation

**Remember**: We're not just improving Whisper - we're revolutionizing how humans interact with computers through voice.
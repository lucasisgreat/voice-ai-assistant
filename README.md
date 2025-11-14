# Local AI Voice Assistant

A privacy-focused conversational AI chatbot built to run locally without any cloud dependencies on consumer hardware.

## Overview

This project creates a natural-sounding voice assistant aimed at running efficiently on your own hardware. It combines a large language model for conversation with XTTS voice synthesis for realistic speech output featuring custom voice cloning from audio samples.

## Requirements:

### Hardware Requirements
- **Minimum:**
  - CPU: Modern multi-core processor
  - RAM: 8GB+
  - GPU: NVIDIA GPU with 6GB+ VRAM (optional but recommended)
  - Storage: 10GB+ free space for models

### Software Requirements
- **Operating System:** Linux (tested on Ubuntu/WSL2), macOS
- **Python:** 3.8 or higher
- **CUDA:** 11.8+ (if using GPU acceleration)
- **FFmpeg:** For audio processing

### Python Dependencies
```
llama-cpp-python
TTS (Coqui TTS)
numpy
soundfile
torch
```

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/lucasisgreat/voice-ai-assistant.git
cd voice-ai-assistant
```

### 2. Install System Dependencies

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ffmpeg portaudio19-dev python3-pyaudio
```

**macOS:**
```bash
brew install ffmpeg portaudio
```

### 3. Install Python Dependencies
```bash
pip install llama-cpp-python
pip install TTS
pip install numpy soundfile
pip install torch  # or torch with CUDA support
```

**For GPU support with llama-cpp-python:**
```bash
CMAKE_ARGS="-DLLAMA_CUBLAS=on" pip install llama-cpp-python --force-reinstall --no-cache-dir
```

### 4. Download Required Models

**LLM Model:**
Download a quantized GGUF model (recommended: OpenHermes 2.5 Mistral 7B Q4_K_M)
- From [TheBloke on HuggingFace](https://huggingface.co/TheBloke)
- Place in a `models/` directory

**XTTS Model:**
The XTTS model downloads automatically on first run.

### 5. Prepare Voice Samples (Optional)

For custom voice cloning:
1. Create a directory: `voice_samples/your_voice_name/`
2. Add audio files (.mp3, .wav, .flac, or .m4a) of the voice you want to clone
3. The script will automatically process them into usable segments

## Configuration

Edit the configuration section at the top of `llama_bot.py`:

```python
# Update these paths to match your setup
MODEL_PATH = "/path/to/your/model.gguf"
MODEL_DIR = "/path/to/models/directory"
VOICE_BASE_DIR = "/path/to/voice_samples"
```

### Optional Configuration
- `MIN_CHARS_TO_FLUSH`: Minimum characters before sending phrase to TTS (default: 35)
- `SPEECH_SPEED`: Voice playback speed multiplier (default: 1.15)
- `MAX_TOKENS`: Maximum response length (default: 140)
- `MAX_HISTORY_MESSAGES`: Chat history size (default: 20)

## Usage

### Basic Usage
```bash
python llama_bot.py
```

On first run:
1. Select a voice from the menu (or use default)
2. Wait for models to load (~30-60 seconds)
3. Start chatting!

### Commands
While chatting, you can use these commands:
- `/character <prompt>` - Change the AI's personality
- `/show_character` - View current personality
- `/models` - List available models
- `/voices` - List available voices
- `exit` or `quit` - End the conversation

### Advanced Usage

**Specify a custom model:**
```bash
python llama_bot.py /path/to/different-model.gguf
```

**Use specific voice:**
```bash
python llama_bot.py --refs /path/to/voice_samples/specific_voice
```

**Force specific GPUs:**
```bash
GPU_LLM_FORCE_INDEX=0 GPU_TTS_FORCE_INDEX=1 python llama_bot.py
```

## How It Works

The system uses a two-process architecture:

1. **Main Process** (GPU 0) - Runs the language model and manages conversation
2. **TTS Worker** (GPU 1) - Handles voice synthesis in a separate subprocess

**Phrase Streaming Pipeline:**
- As the LLM generates text word-by-word, the system buffers tokens
- When a complete sentence is detected (35+ characters ending with punctuation), it's immediately sent to the TTS worker
- Audio synthesis and playback happen in parallel with continued text generation
- This results in ~2-3 second response time from query to first spoken word

**Voice Cloning Process:**
1. Raw audio files are automatically converted to 24kHz mono
2. Files are split into 6-second segments with silence removed
3. Up to 17 best segments are selected as reference clips
4. XTTS learns voice characteristics from these clips
5. Generated speech matches the tone, pitch, and style of the reference voice

## Privacy & Security

- **No internet required** - Everything runs locally after initial model downloads
- **Chat history stored in RAM only** - Automatically wiped on exit
- **No logging to disk** - Conversation data never written to files
- **Temporary audio files** - Deleted immediately after playback

## What I Built & Learned

This project started during a time when I was helping care for my grandfather and needed something engaging to work on. I wanted to understand how AI language models and voice synthesis actually work under the hood, rather than just using cloud APIs.

**Technical Challenges Solved:**
- **Memory management**: Running both LLM and TTS on consumer hardware required careful GPU allocation. I implemented automatic GPU detection and splitting workload across two cards to prevent crashes.
- **Latency optimization**: Initial versions had 8-10 second delays. I built a phrase streaming system that starts speaking after the first complete sentence, dropping response time to 2-3 seconds.
- **Voice quality**: Learned how audio sampling rates, segment length, and reference clip selection impact synthesis quality. Automated the entire pipeline with FFmpeg.
- **Robust error handling**: Built fallback systems at every level - audio players, GPU selection, model loading - so the system degrades gracefully rather than crashing.

**Design Thinking:**
Through iteration and testing with friends and family, I learned that users prioritize reliability and speed over feature complexity. The phrase streaming feature came from observing that people disengage when there's too much silence.

## Troubleshooting

**"Model file not found" error:**
- Verify MODEL_PATH points to a valid .gguf file
- Use absolute paths, not relative paths

**No audio output:**
- Install paplay: `sudo apt install pulseaudio-utils`
- Or install aplay: `sudo apt install alsa-utils`
- Check audio device: `paplay --list-sinks`

**GPU not detected:**
- Install nvidia-smi: `sudo apt install nvidia-utils-XXX`
- Check CUDA installation: `nvcc --version`
- Verify drivers: `nvidia-smi`

**TTS worker timeout:**
- First run takes longer (model downloads)
- Reduce MAX_REF_CLIPS if limited VRAM
- Check GPU assignment isn't causing conflicts

## Future Improvements
- [ ] Fine-tuned models for specific use cases
- [ ] Voice activity detection for hands-free mode
- [ ] Multi-language support beyond English
- [ ] Model hot-swapping without restart

## License

This project is provided as-is for educational and personal use.

## Acknowledgments

- **llama.cpp** - Efficient LLM inference
- **Coqui TTS** - Open-source voice synthesis
- **TheBloke** - Quantized model distributions

# SubSync Algorithm & Architecture

This algorithm has been created using Claude Sonnet 4.5 model at 2025-12-05.

## Overview

SubSync is a CLI application that generates high-quality subtitles for YouTube videos, optimized for YouTube upload and compliant with Netflix subtitle standards.

---

## Core Algorithm Flow

```text
┌─────────────────────────────────────────────────────────────────┐
│                     USER INPUT                                  │
│  YouTube URL + Language (optional) + Format (optional)         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 1: VALIDATION & PREPARATION                  │
│  • Parse YouTube URL                                            │
│  • Validate video accessibility                                │
│  • Extract video metadata (duration, title, etc.)              │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 2: AUDIO EXTRACTION                          │
│  • Use yt-dlp to identify available audio streams              │
│  • Select best quality audio (prefer opus/aac, 128kbps+)      │
│  • Download audio stream to temporary location                 │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 3: SPEECH-TO-TEXT TRANSCRIPTION              │
│  • Load OpenAI Whisper model (medium/large recommended)        │
│  • Transcribe audio with word-level timestamps                │
│  • Output: Segments with text, start time, end time, words     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 4: NETFLIX STANDARDS APPLICATION             │
│  • Apply timing rules (min 5/6s, max 7s)                       │
│  • Format text (max 2 lines, ~42 chars/line)                   │
│  • Apply intelligent line breaking                             │
│  • Validate character encoding (UTF-8)                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 5: FORMAT CONVERSION & EXPORT                │
│  • Convert to target format (SRT, VTT, or TTML)                │
│  • Apply format-specific styling/positioning                   │
│  • Write to output file                                        │
│  • Cleanup temporary files                                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     OUTPUT                                      │
│  Ready-to-upload subtitle file (YouTube compatible)            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Third-Party Services & Tools

### 1. **yt-dlp** - Video/Audio Download

- **Purpose**: Extract audio stream from YouTube videos
- **Why**: Well-maintained, handles YouTube API changes, supports authentication
- **Integration**: Python library (`yt_dlp`)
- **Key Features**:
  - Automatic format selection
  - Audio-only extraction
  - Metadata retrieval
  - Cookie support for age-restricted content

### 2. **OpenAI Whisper** - Speech Recognition

- **Purpose**: Transcribe audio to text with timestamps
- **Why**:
  - State-of-the-art accuracy (peer-reviewed by OpenAI)
  - Supports 99+ languages
  - Open-source and free
  - Robust to accents, background noise
- **Integration**: Python library (`openai-whisper`)
- **Models Available**:

  | Model  | Size  | Speed    | Accuracy | Use Case                |
  |--------|-------|----------|----------|-------------------------|
  | tiny   | 39M   | ~10x     | Fair     | Quick testing           |
  | base   | 74M   | ~7x      | Good     | Fast processing         |
  | small  | 244M  | ~4x      | Good     | Balanced                |
  | medium | 769M  | ~2x      | Better   | **Recommended default** |
  | large  | 1550M | 1x       | Best     | Maximum quality         |
  | turbo  | 809M  | ~8x      | Better   | Fast with good quality  |

### 3. **pysrt / webvtt-py** - Subtitle Format Libraries

- **Purpose**: Parse, manipulate, and write subtitle files
- **Why**: Handle format complexities, timing calculations, validation
- **Integration**: Python libraries

### 4. **ffmpeg** - Audio Processing

- **Purpose**: Audio format conversion, quality normalization
- **Why**: Required by Whisper, handles various audio codecs
- **Integration**: System dependency (already in project)

---

## Netflix Subtitle Standards Implementation

### Timing Rules

```python
MIN_DURATION = 5/6  # seconds (0.833s)
MAX_DURATION = 7.0  # seconds

# Algorithm for timing adjustment:
for segment in transcript_segments:
    if segment.duration < MIN_DURATION:
        # Merge with next segment
        merge_segments(segment, next_segment)
    elif segment.duration > MAX_DURATION:
        # Split at natural break (punctuation, pause)
        split_at_natural_break(segment, MAX_DURATION)
```

### Text Formatting Rules

```python
MAX_LINES = 2
MAX_CHARS_PER_LINE = 42  # Approximate Netflix standard

# Line breaking priorities (in order):
BREAK_PRIORITIES = [
    "after punctuation (. ! ? , ; :)",
    "before conjunctions (and, but, or, so)",
    "before prepositions (in, on, at, to, from)",
    "at natural pauses (detected by Whisper)"
]

# Never break between:
FORBIDDEN_BREAKS = [
    "article + noun (the car, a house)",
    "adjective + noun (red car, big house)",
    "first name + last name",
    "verb + subject pronoun",
    "verb + auxiliary/reflexive/negation"
]
```

### Positioning Rules

```python
# Default positioning
position = "bottom"  # Center justified
alignment = "center"

# Override if:
if video_has_bottom_text:
    position = "top"  # Avoid overlap with on-screen text
```

### Character Validation

- Use UTF-8 encoding
- Validate against Netflix Glyph List
- Handle special characters:

  ```python
  CHAR_REPLACEMENTS = {
      '"': '"',  # Smart quotes to straight quotes
      '"': '"',
      ''': "'",
      ''': "'",
      '—': '-',  # Em dash to hyphen
      # ... more replacements
  }
  ```

---

## Output Format Specifications

### SubRip (.srt) - **RECOMMENDED**

```
1
00:00:01,000 --> 00:00:04,000
First subtitle line here
Second line if needed

2
00:00:04,500 --> 00:00:07,000
Next subtitle segment
```

**Pros**:

- Most widely supported
- Simple, human-readable
- YouTube's primary format
- Easy to manually edit

### WebVTT (.vtt)

```
WEBVTT

00:00:01.000 --> 00:00:04.000
First subtitle line here
Second line if needed

00:00:04.500 --> 00:00:07.000
Next subtitle segment
```

**Pros**:

- Web standard
- Supports styling metadata
- Better for advanced features

### TTML - Not recommended for MVP

- Too complex for basic use case
- Can be added as premium feature later

---

## Modular Architecture (Extensibility Design)

### Component Structure

```
subsync/
├── url_handler/
│   ├── validator.py          # URL validation
│   ├── youtube_extractor.py  # YouTube-specific logic
│   └── metadata.py            # Video metadata handling
│
├── audio/
│   ├── extractor.py           # Audio extraction from video
│   └── processor.py           # Audio preprocessing if needed
│
├── transcription/
│   ├── base.py                # Abstract transcriber interface
│   ├── whisper_transcriber.py # Whisper implementation
│   └── [future: google_transcriber.py, aws_transcriber.py]
│
├── subtitle_processor/
│   ├── timing.py              # Timing rules and adjustments
│   ├── text_formatter.py      # Line breaking, character limits
│   ├── standards/
│   │   ├── base.py            # Abstract standards interface
│   │   ├── netflix.py         # Netflix standards implementation
│   │   └── youtube.py         # YouTube-specific adjustments
│
├── translation/               # Future feature
│   ├── base.py                # Translation interface
│   └── [future: google_translate.py, deepl.py]
│
├── export/
│   ├── base.py                # Abstract exporter
│   ├── srt_exporter.py        # SRT format
│   ├── vtt_exporter.py        # WebVTT format
│   └── ttml_exporter.py       # TTML format (future)
│
└── cli.py                     # Command-line interface
```

### Design Patterns Used

1. **Strategy Pattern** - Transcription services

   ```python
   class Transcriber(ABC):
       @abstractmethod
       def transcribe(self, audio_path, language) -> RawTranscript:
           pass

   class WhisperTranscriber(Transcriber):
       def transcribe(self, audio_path, language) -> RawTranscript:
           # Whisper-specific implementation
           pass
   ```

2. **Template Method** - Subtitle processing pipeline

   ```python
   class SubtitleProcessor(ABC):
       def process(self, transcript):
           validated = self.validate(transcript)
           timed = self.apply_timing_rules(validated)
           formatted = self.format_text(timed)
           return formatted

       @abstractmethod
       def apply_timing_rules(self, transcript):
           pass
   ```

3. **Factory Pattern** - Format exporters

   ```python
   class ExporterFactory:
       @staticmethod
       def create(format: str) -> Exporter:
           if format == 'srt':
               return SRTExporter()
           elif format == 'vtt':
               return VTTExporter()
           # ...
   ```

---

## Data Structures

### VideoInfo

```python
@dataclass
class VideoInfo:
    url: str
    video_id: str
    title: str
    duration: float
    language: Optional[str]
    is_accessible: bool
    has_audio: bool
    metadata: Dict[str, Any]
```

### RawTranscript

```python
@dataclass
class RawTranscript:
    segments: List[TranscriptSegment]
    language: str
    language_probability: float

@dataclass
class TranscriptSegment:
    text: str
    start: float          # seconds
    end: float            # seconds
    words: List[Word]
    confidence: float

@dataclass
class Word:
    word: str
    start: float
    end: float
    probability: float
```

### ProcessedSubtitle

```python
@dataclass
class ProcessedSubtitle:
    index: int
    start_time: float
    end_time: float
    lines: List[str]      # Max 2 lines
    position: str         # 'bottom', 'top'
    alignment: str        # 'center', 'left', 'right'
```

---

## CLI Interface Design

### Basic Usage

```bash
subsync generate https://www.youtube.com/watch?v=VIDEO_ID
```

### Advanced Options

```bash
subsync generate <youtube-url> \
  --language en \              # Specify language (or auto-detect)
  --model medium \             # Whisper model: tiny|base|small|medium|large|turbo
  --format srt \               # Output format: srt|vtt|ttml
  --standards netflix \        # Apply Netflix timing/formatting rules
  --output my-video.srt \      # Output file path
  --verbose                    # Show detailed progress
```

### Future Translation Command

```bash
subsync translate subtitles.srt \
  --from en \
  --to es \
  --output subtitles_es.srt
```

### Interactive Mode

```bash
subsync generate

? Enter YouTube URL: https://www.youtube.com/watch?v=...
? Select language (or auto-detect): [auto]
? Select output format: [SRT]
? Apply Netflix standards?: [Yes]

⏳ Downloading audio...
⏳ Transcribing (this may take a few minutes)...
   Progress: [████████████████████] 100%
✓ Transcription complete!
⏳ Processing subtitles...
✓ Subtitles generated: output.srt

Summary:
- Duration: 10:45
- Segments: 156
- Language: English (detected)
- Compliance: Netflix standards applied
```

---

## Error Handling

### Critical Errors

| Error | Handling Strategy |
|-------|-------------------|
| Invalid YouTube URL | Validate format before processing |
| Video not accessible | Check accessibility, provide clear error |
| No audio stream | Detect audio streams early, fail with explanation |
| Transcription failure | Provide confidence scores, suggest retry |
| Language detection fail | Require user to specify language |

### Quality Warnings

- Background noise detected
- Multiple overlapping speakers
- Low audio quality
- Very fast speech (may affect accuracy)

---

## Performance Optimization

### Configuration (subsync.config.yaml)

```yaml
transcription:
  default_model: "medium"
  device: "auto"         # auto, cpu, cuda, mps
  language: "auto"

subtitle_standards:
  default: "netflix"
  netflix:
    min_duration: 0.833
    max_duration: 7.0
    max_lines: 2
    max_chars_per_line: 42

output:
  default_format: "srt"

download:
  audio_format: "best"
  audio_quality: "128k"
  temp_dir: "/tmp/subsync"
  cleanup: true

performance:
  use_gpu: true
  cache_models: true
```

### Optimization Strategies

1. **GPU Acceleration**
   - Auto-detect CUDA/Metal (Apple Silicon)
   - 10-20x faster transcription with GPU

2. **Model Caching**
   - Cache Whisper model files
   - Reuse loaded models across runs

3. **Chunked Processing**
   - Process long videos in chunks
   - Show progress for better UX

4. **Smart Model Selection**
   - Auto-select based on video length
   - Short (<5 min): medium or turbo
   - Long (>30 min): base or small for speed

---

## Future Extensions

### Translation Feature

```python
# Future iteration: Translation pipeline
def translate_subtitles(subtitle_file, target_language):
    """
    Translate existing subtitles to another language
    """
    subtitles = parse_subtitle_file(subtitle_file)

    # Use translation service
    translator = create_translator('google')  # or 'deepl', 'gpt'

    translated = []
    for subtitle in subtitles:
        # Maintain timing, translate text only
        translated_text = translator.translate(
            subtitle.text,
            target_language
        )
        # Ensure translation still meets formatting rules
        formatted = format_for_netflix_standards(translated_text)
        translated.append(
            ProcessedSubtitle(
                index=subtitle.index,
                start_time=subtitle.start_time,
                end_time=subtitle.end_time,
                lines=formatted,
                position=subtitle.position
            )
        )

    return export_subtitles(translated, format='srt')
```

### Additional Features

- **Speaker Diarization**: Identify different speakers
- **Multi-platform Support**: Vimeo, Dailymotion, etc.
- **Custom Vocabulary**: Improve accuracy for technical content
- **Batch Processing**: Process multiple videos
- **Subtitle Editing UI**: Web interface for corrections
- **Quality Scoring**: Automatic quality assessment

---

## Technology Stack Summary

| Component | Technology | Purpose |
|-----------|------------|---------|
| Language | Python 3.13+ | Modern Python features |
| Package Manager | uv | Fast dependency management |
| CLI Framework | Click or Typer | User-friendly CLI |
| Video Download | yt-dlp | YouTube audio extraction |
| Speech-to-Text | OpenAI Whisper | Transcription |
| Audio Processing | ffmpeg | Audio handling |
| Subtitle Formats | pysrt, webvtt-py | Format I/O |
| Configuration | YAML | Settings management |
| Testing | pytest | Unit/integration tests |
| Type Checking | mypy | Static type analysis |
| Logging | Python logging | Progress & debugging |

---

## Estimated Processing Times

| Video Length | Whisper Model | GPU | CPU (estimate) |
|--------------|---------------|-----|----------------|
| 5 minutes    | medium        | ~30s | ~3 minutes     |
| 10 minutes   | medium        | ~1m  | ~6 minutes     |
| 30 minutes   | medium        | ~3m  | ~18 minutes    |
| 1 hour       | medium        | ~6m  | ~36 minutes    |

*Note: Times vary based on hardware and audio complexity*

---

## References

- [Netflix Timed Text Style Guide](https://partnerhelp.netflixstudios.com/hc/en-us/articles/215758617)
- [YouTube Caption Support](https://support.google.com/youtube/answer/2734796)
- [YouTube Data API - Captions](https://developers.google.com/youtube/v3/docs/captions)
- [OpenAI Whisper Paper](https://arxiv.org/abs/2212.04356)
- [yt-dlp Documentation](https://github.com/yt-dlp/yt-dlp)
- [SubRip Format Specification](https://en.wikipedia.org/wiki/SubRip)
- [WebVTT Standard](https://www.w3.org/TR/webvtt1/)

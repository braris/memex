---
title: "KittenML/KittenTTS"
source: "https://github.com/KittenML/KittenTTS"
type: repos
ingested: 2026-07-05
tags: [github-repo, text-to-speech, tts, onnx, python, edge-ai]
summary: "KittenTTS is an open-source lightweight text-to-speech Python library built on ONNX. Its README describes CPU-friendly models ranging from 15M to 80M parameters, built-in voices, quick-start installation, API usage, system requirements, roadmap, and Apache-2.0 licensing."
---

# KittenML/KittenTTS

Source: [KittenML/KittenTTS](https://github.com/KittenML/KittenTTS)

## Repository Snapshot

- Owner/repo: `KittenML/KittenTTS`
- Description: State-of-the-art TTS model under 25MB
- Visibility: Public
- Primary language: Python
- License: Apache-2.0
- Default branch shown: `main`
- Repository scale at ingest: 39 commits, about 14.9k stars, about 850 forks
- Latest release shown: `0.8.1`, dated 2026-02-24

## README Summary

Kitten TTS is an open-source lightweight text-to-speech library built on ONNX. The README says it provides high-quality voice synthesis on CPU without requiring a GPU, with model variants ranging from 15M to 80M parameters and 25-80 MB on disk.

The project is marked as a developer preview, so APIs may change between releases.

## Features

- Ultra-lightweight model sizes, including an int8 model around 25 MB
- CPU-optimized ONNX inference
- Eight built-in voices: Bella, Jasper, Luna, Bruno, Rosie, Hugo, Kiki, and Leo
- Adjustable speech speed through a `speed` parameter
- Text preprocessing for numbers, currencies, units, and similar normalization
- 24 kHz audio output

## Available Models

| Model | Parameters | Size | Location |
|------|------------|------|----------|
| `kitten-tts-mini` | 80M | 80 MB | Hugging Face: `KittenML/kitten-tts-mini-0.8` |
| `kitten-tts-micro` | 40M | 41 MB | Hugging Face: `KittenML/kitten-tts-micro-0.8` |
| `kitten-tts-nano` | 15M | 56 MB | Hugging Face: `KittenML/kitten-tts-nano-0.8` |
| `kitten-tts-nano` int8 | 15M | 25 MB | Hugging Face: `KittenML/kitten-tts-nano-0.8-int8` |

The README notes that some users have reported issues with the int8 nano model and asks users to open an issue if they encounter problems.

## Installation And Basic Usage

The README quick start installs the release wheel directly from GitHub:

```bash
pip install https://github.com/KittenML/KittenTTS/releases/download/0.8.1/kittentts-0.8.1-py3-none-any.whl
```

Basic usage loads a Hugging Face model, generates audio, and writes the result to a WAV file:

```python
from kittentts import KittenTTS

model = KittenTTS("KittenML/kitten-tts-mini-0.8")
audio = model.generate("This high-quality TTS model runs without a GPU.", voice="Jasper")

import soundfile as sf
sf.write("output.wav", audio, 24000)
```

The README also documents speech-speed adjustment, direct file generation, and voice listing:

```python
audio = model.generate("Hello, world.", voice="Luna", speed=1.2)
model.generate_to_file("Hello, world.", "output.wav", voice="Bruno", speed=0.9)
print(model.available_voices)
```

## GPU Usage

The README says GPU usage requires installing `requirements_gpu.txt` and loading the model with a CUDA backend:

```bash
pip install -r requirements_gpu.txt
```

```python
m = KittenTTS("KittenML/kitten-tts-mini-0.8", backend="cuda")
```

## API Surface

The README lists these primary APIs:

- `KittenTTS(model_name, cache_dir=None)`: loads a model from Hugging Face Hub.
- `model.generate(text, voice, speed, clean_text)`: synthesizes speech from text and returns NumPy audio samples at 24 kHz.
- `model.generate_to_file(text, output_path, voice, speed, sample_rate, clean_text)`: synthesizes speech and writes directly to an audio file.
- `normalize_text(text, locale="en-US", return_spans=False)`: normalizes text for TTS without generating audio.
- `model.available_voices`: returns the voice list.

## Requirements

- Operating system: Linux, macOS, or Windows
- Python: 3.8 or later
- Hardware: CPU is sufficient; GPU is not required
- Disk: 25-80 MB depending on model variant

The README recommends using a virtual environment to avoid dependency conflicts.

## Roadmap

The README lists planned work:

- Optimized inference engine
- Mobile SDK
- Higher quality TTS models
- Multilingual TTS
- KittenASR

## Commercial And Community Support

The README says commercial support is available for product integrations, custom voices, and enterprise licensing. Community and support channels include Discord, the project website, GitHub Issues, a request form, and email.

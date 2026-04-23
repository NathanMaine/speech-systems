# speech-systems

A collection of speech-AI systems I build and run on hardware I own. Covers automatic speech recognition (ASR), text-to-speech synthesis (TTS), and the orchestration layer that turns either into a shipping product.

Each project below is its own repository with its own runbook. This page is the index.

---

## Why these three

Speech-AI has three distinct engineering disciplines that rarely share a codebase:

1. **ASR** turns audio into text. The failure modes are word error rate, diarization quality, streaming latency, domain mismatch.
2. **TTS** turns text into audio. The failure modes are prosody, speaker consistency, emotion control, pronunciation edge cases.
3. **Orchestration** takes one or both of the above and makes them usable inside a product. The failure modes are latency budgets, cost, concurrency, evaluation.

The sections below cover each discipline in two layers: a flagship project that goes deep, and a portfolio table showing the full set of projects I have built in that discipline.

---

## 1. ASR Transcript Project — ASR depth

**Repo:** [NathanMaine/asr-transcript-project](https://github.com/NathanMaine/asr-transcript-project)

Batch transcription of a 1,013-episode YouTube corpus using NVIDIA Parakeet CTC 1.1B on a DGX Spark, with speaker diarization via pyannote.audio. The interesting engineering is a custom CTC frame-level word alignment: word-level timestamps extracted directly from CTC outputs without a separate alignment model.

**Stack:** Parakeet CTC 1.1B, pyannote.audio 3.1.1, torchaudio, PyTorch nightly (with custom pyannote compatibility patches), NVIDIA DGX Spark GPU.

**What it demonstrates:**
- Production-grade batch ASR on real-world audio at scale
- Custom CTC alignment rather than wrapping a black-box aligner
- Speaker diarization integrated with transcription output
- GPU-accelerated inference on unified-memory architecture

---

## 2. HDP Forge — TTS depth

**Repo:** coming soon
**Website:** [memoriant.ai](https://www.memoriant.ai/memoriant_landing)

Multi-speaker podcast synthesis using OpenMOSS MOSS-TTS 8B. Custom continuation chaining preserves prosodic continuity across speaker turns by passing prior audio as a prefix for the next synthesis step. A 23-emotion-to-sampling-params mapping steers reference-tier selection and decoder temperature per line.

**Stack:** MOSS-TTS 8B, MOSS-VoiceGenerator, MOSS-SoundEffect, PyTorch + transformers (Flash Attention 2 where available), Llama 3.1 70B via vLLM for script enhancement, pydub for timeline mixing.

**What it demonstrates:**
- Pipeline-internals TTS work, not API orchestration
- Custom prosodic continuity across multi-speaker timelines
- Emotion control that targets a real parameter surface, not tags on a third-party API
- GPU phase management across LLM → TTS → mixer lifecycle

---

## 3. Project Aurora Echo — orchestration depth

**Current version:** [NathanMaine/Project-Aurora-Echo-v2.0](https://github.com/NathanMaine/Project-Aurora-Echo-v2.0) — NVIDIA-accelerated AI meeting copilot with real-time transcription, speaker diarization, and structured meeting summaries.

Real-time meeting copilot that captures audio, runs faster-whisper for streaming transcription, pyannote for diarization, and a multi-provider LLM for summarization. Runs on the same DGX Spark that powers the other two projects.

**Stack:** faster-whisper, pyannote.audio, multi-provider LLM routing, FastAPI, WebSocket streaming, NVIDIA GPU acceleration.

**What it demonstrates:**
- ASR + diarization + LLM chained inside a latency budget
- Production orchestration of open-source speech components
- Multi-provider failover at the LLM layer
- Iteration through multiple architectures to find the production shape

### Iteration history

This project reached its current form through six public iterations. Each repo is preserved so the architectural evolution is inspectable:

| # | Repo | Architecture focus |
|---|---|---|
| v1 | [realtime-ai-assistant](https://github.com/NathanMaine/realtime-ai-assistant) | First pass: xAI Grok API + live transcription |
| v2 | [realtime-ai-assistant002](https://github.com/NathanMaine/realtime-ai-assistant002) | Refined xAI Grok integration + MIT license |
| v3 | [realtime-ai-assistant003-fast-api](https://github.com/NathanMaine/realtime-ai-assistant003-fast-api) | **FastAPI + WebSocket** architecture for real-time action-item extraction |
| v4 | [realtime-ai-assistant004-stream-lit](https://github.com/NathanMaine/realtime-ai-assistant004-stream-lit) | Streamlit UI variant for rapid iteration on user interaction |
| PoC | [Project-Aurora-Echo](https://github.com/NathanMaine/Project-Aurora-Echo) | **Pivot to self-hosted**: faster-whisper + pyannote + multi-provider LLM, running on DGX Spark |
| v2.0 | [Project-Aurora-Echo-v2.0](https://github.com/NathanMaine/Project-Aurora-Echo-v2.0) | **Production**: NVIDIA-accelerated, structured meeting summaries, hardened for daily use |

The narrative arc: start with a hosted API (Grok), iterate on the serving architecture (FastAPI versus Streamlit), then pivot to self-hosted open-source ASR + diarization on owned hardware, then harden into the NVIDIA-accelerated production version.

---

## ASR portfolio breadth

I am the **integration author and architect** on the projects below, not the author of the underlying speech models. I did not train Parakeet, faster-whisper, pyannote, or any of the third-party LLMs these systems call. NVIDIA, Systran, and the pyannote research group own that work. What I built is the layer on top: the glue code, the architectural decisions, the novel components where the shipped product needed something the model vendors didn't provide, and the production hardening that turns a research checkpoint into a system that runs unattended on owned hardware.

Six ASR projects below, covering different architectural patterns. The flagship ASR section above describes one of them in narrative form; this table is the quick reference showing what I authored versus what I integrated.

All six projects below are my original creations. The code, the architecture, the integration patterns, and the novel components are mine. The table splits each project into **I wrote** (original code I authored end-to-end) and **I use** (third-party models, APIs, and libraries I integrate but did not create).

| Project | Public repo | I wrote (original, by me) | I use (third-party, not by me) |
|---|---|---|---|
| **asr-transcript-project** | [repo](https://github.com/NathanMaine/asr-transcript-project) | `transcribe.py` (700 lines, my code), the custom CTC frame-level word alignment algorithm, 9 pyannote compatibility patches for PyTorch nightly + numpy 2.x + PyTorch 2.6+, torchaudio soundfile fallback for aarch64, batch orchestration for the 1,013-file corpus, all CLI + config + logging | NVIDIA Parakeet CTC 1.1B (NVIDIA), pyannote.audio 3.1.1 (Bredin et al.), torchaudio (PyTorch team), ffmpeg |
| **video analyzer (v3/v4)** | local repo | The provider abstraction layer with runtime Gemini/Ollama/Parakeet switching, two-pass extraction-then-analysis architecture, segment-based video extraction with 30-second overlap, 4-strategy JSON parse with truncation retry, real-time SSE progress streaming, all of the backend + UI code | Gemini API (Google), Ollama runtime (Ollama team) + Qwen2.5-VL/LLaVA weights, NVIDIA Parakeet, Docling (IBM Research) |
| **google voice** | local repo | The Chrome extension with two-channel capture (tab audio + mic), FastAPI backend, WebSocket PCM streaming protocol, three-WAV-per-call save architecture (remote, local, mixed), React dashboard with search and playback, all client + server code | Chrome extension APIs, faster-whisper (Systran), Ollama + Llama 3.1 weights (Meta), SQLite |
| **Project-Aurora-Echo (PoC)** | [repo](https://github.com/NathanMaine/Project-Aurora-Echo) | The real-time orchestration layer, multi-provider LLM routing with failover, the entire application code and pipeline wiring | faster-whisper, pyannote.audio, multi-provider LLM APIs |
| **Project-Aurora-Echo-v2.0** | [repo](https://github.com/NathanMaine/Project-Aurora-Echo-v2.0) | The NVIDIA-accelerated production refinement, structured meeting summaries pipeline, the full app around the NVIDIA runtime | NVIDIA NIM, TensorRT-LLM |
| **realtime-ai-assistant (v1-v4)** | [v1](https://github.com/NathanMaine/realtime-ai-assistant) / [v2](https://github.com/NathanMaine/realtime-ai-assistant002) / [v3-fastapi](https://github.com/NathanMaine/realtime-ai-assistant003-fast-api) / [v4-streamlit](https://github.com/NathanMaine/realtime-ai-assistant004-stream-lit) | Four progressive architectural iterations, all authored by me: xAI Grok integration, FastAPI + WebSocket rewrite, Streamlit UI variant, DGX-hosted production pivot | xAI Grok API, FastAPI framework, Streamlit framework |

The novel engineering sits at the integration layer, which is the layer I authored end-to-end: the CTC alignment that skips the forced-aligner step, the provider abstraction with runtime switching, the two-channel capture that sidesteps the diarization problem, and the real-time orchestration with multi-provider failover. The underlying models (NVIDIA Parakeet, Systran faster-whisper, pyannote.audio, xAI Grok, Google Gemini, Meta Llama via Ollama) are industry-standard components authored by their respective vendors and research groups. I integrate them. The architecture around them, the patches that made them run on a Blackwell-class GPU with PyTorch nightly, and the production glue that keeps a 1,013-file corpus or a live meeting pipeline running end-to-end is my work.

---

## TTS portfolio breadth

Same authorship split as the ASR table above. I am the integration author and architect, not the author of the underlying TTS models. I did not train MOSS-TTS, the ElevenLabs voices, Qwen3-TTS, or OpenAI TTS. OpenMOSS, ElevenLabs, the Alibaba Qwen team, and OpenAI own those. What I built is the production pipeline on top: the 23-emotion to sampling-params mapping, the continuation chaining that preserves prosodic continuity across speaker turns, the multi-speaker timeline mixer, the HDP profile director, and the episode orchestrator that turns a script into a mixed multi-speaker audio file.

Four TTS projects below:

| Project | Public repo | I wrote (original, by me) | I use (third-party, not by me) |
|---|---|---|---|
| **hdp-forge** (flagship) | coming soon | Full production pipeline (~4,100 LOC, 199 tests): `emotion_mapping.py` (23-emotion → TTS sampling params), `rule_based.py` director, `episode.py` orchestrator, `mixer.py` + `timeline.py` for multi-speaker audio composition, `moss/single.py` continuation chaining that preserves prosodic continuity across speaker turns, ElevenLabs integration with emotion-tuned stability/similarity, HDP profile loader | MOSS-TTS 8B (OpenMOSS), ElevenLabs API, transformers, pydub, torchaudio |
| **text to speech** (podcast production) | local repo | `produce_episode` v1-v8 orchestrator (script → TTS → SFX layering → mixed audio), custom SFX script syntax with nested layering, `emotional_tts.py` parameter tuning, `clone_voice.py` workflow, `comic_generator` pipeline (panel generation from episode transcripts), `batch_convert.py`, HeyGen avatar video wrapper | ElevenLabs API (TTS + voice clone), OpenAI API (TTS + image), HeyGen (avatar video), Imagine.art (comic image gen) |
| **HDP Sports** | local repo | `hdp_dialogue_generator.py` (multi-speaker ElevenLabs Text-to-Dialogue wrapper), `emotion_enhancer.py` emotion tagging, `humanistic_enhancer.py` dialogue naturalization. A separate variant from hdp-forge that targets the ElevenLabs Text-to-Dialogue API instead of a local MOSS checkpoint | ElevenLabs Text-to-Dialogue API |
| **MOSS-TTS CLI toolkit** | local repo | Four CLI apps on top of the OpenMOSS checkpoints: `moss_tts_app.py`, `moss_ttsd_app.py`, `moss_voice_generator_app.py`, `moss_sound_effect_app.py`, plus a Gradio real-time demo. These are shell-friendly wrappers; the inference code underneath is upstream | OpenMOSS MOSS-TTS 8B and sound-effect models |

The novel TTS engineering lives inside hdp-forge: the 23-emotion to sampling-params mapping that steers reference-tier selection and decoder temperature per line, the continuation chaining that preserves prosodic continuity across multi-speaker turns by passing prior audio as a prefix for the next synthesis step, the HDP profile system that drives per-speaker direction, and the timeline mixer that composes multi-speaker episodes. MOSS-TTS 8B (OpenMOSS), ElevenLabs, OpenAI TTS, and HeyGen are third-party components I integrate. The pipeline, director, mixer, multi-speaker timeline composition, and SFX layering syntax are my work.

---

## Hardware this runs on

All three projects run on **NVIDIA DGX Spark GB10** (128GB unified memory) with an RTX 4090 available for burst work. Local-first, air-gappable, zero cloud dependency for the inference path.

Networking: 10G office backbone connecting DGX Spark, NAS model storage, and workstations.

---

## Why separate repositories

Each project is independently runnable. Bundling them into a monorepo would couple their release cadences and force anyone trying to use one to pull the dependencies of all three. The individual-repo pattern lets each have its own issue tracker, CI, versioning, and README without noise from the others.

This repository is the index and the narrative. The code lives in the three child repos linked above.

---

## License

MIT for all three projects and this index. See individual repositories for details.

---

## Author

Nathan Maine. Open-source contributor to NVIDIA's garak LLM vulnerability scanner and the TurboQuant llama.cpp fork. NVIDIA Inception member through Memoriant, Inc.

- GitHub: [@NathanMaine](https://github.com/NathanMaine)
- LinkedIn: [nathanmaine](https://www.linkedin.com/in/nathanmaine/)

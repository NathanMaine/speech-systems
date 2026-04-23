# speech-systems

A collection of speech-AI systems I build and run on hardware I own. Covers automatic speech recognition (ASR), text-to-speech synthesis (TTS), and the orchestration layer that turns either into a shipping product.

Each project below is its own repository with its own runbook. This page is the index.

---

## Why these three

Speech-AI has three distinct engineering disciplines that rarely share a codebase:

1. **ASR** turns audio into text. The failure modes are word error rate, diarization quality, streaming latency, domain mismatch.
2. **TTS** turns text into audio. The failure modes are prosody, speaker consistency, emotion control, pronunciation edge cases.
3. **Orchestration** takes one or both of the above and makes them usable inside a product. The failure modes are latency budgets, cost, concurrency, evaluation.

The three projects below cover one discipline each, in depth.

---

## 1. ASR Transcript Project — ASR depth

**Repo:** coming soon

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

Multi-speaker podcast synthesis using OpenMOSS MOSS-TTS 8B. Custom continuation chaining preserves prosodic continuity across speaker turns by passing prior audio as a prefix for the next synthesis step. A 23-emotion-to-sampling-params mapping steers reference-tier selection and decoder temperature per line.

**Stack:** MOSS-TTS 8B, MOSS-VoiceGenerator, MOSS-SoundEffect, PyTorch + transformers (Flash Attention 2 where available), Llama 3.1 70B via vLLM for script enhancement, pydub for timeline mixing.

**What it demonstrates:**
- Pipeline-internals TTS work, not API orchestration
- Custom prosodic continuity across multi-speaker timelines
- Emotion control that targets a real parameter surface, not tags on a third-party API
- GPU phase management across LLM → TTS → mixer lifecycle

---

## 3. Project Aurora Echo — orchestration depth

**Repo:** [NathanMaine/Project-Aurora-Echo](https://github.com/NathanMaine/Project-Aurora-Echo)

Real-time meeting copilot: captures audio, runs faster-whisper for streaming transcription, pyannote for diarization, and a multi-provider LLM for summarization. Runs on the same DGX Spark that powers the other two projects.

**Stack:** faster-whisper, pyannote.audio, multi-provider LLM routing, FastAPI, WebSocket streaming.

**What it demonstrates:**
- ASR + diarization + LLM chained inside a latency budget
- Production orchestration of open-source speech components
- Multi-provider failover at the LLM layer

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

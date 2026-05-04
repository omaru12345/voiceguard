# VoiceGuard — 12-Week Development Roadmap

This is the 3-month plan to take the project from technical proof-of-concept to acquisition-ready visibility.

## 1. Dataset selection and evaluation protocol
* **Training / evaluation data**:
  * **Baseline**: ASVspoof 2019 / 2021 (academic generic detection), FakeAVCeleb (more diverse fakes).
  * **In-house data**: deepfake audio we generate from clean sources (LibriSpeech, CC-BY) using state-of-the-art open TTS (VITS, Bark, XTTSv2) and commercial APIs (e.g. ElevenLabs — *strictly within ToS for research use*).
  * **Noise / degradation**: G.711 (μ-law / A-law) telephony codec, Opus compression, MUSAN background noise applied on-the-fly inside the data loader.
* **Evaluation protocol**:
  * Cross-dataset evaluation is mandatory (e.g. train on ASVspoof, evaluate on the in-house ElevenLabs set).
  * **KPI**: TPR at fixed FPR `0.1%` on out-of-domain data.

## 2. Phase-by-phase tasks

### Phase 1: PoC and data pipeline (Weeks 1–4)
* **Week 1: Data foundation**
  * Tasks: Download and preprocess ASVspoof, LibriSpeech. Implement the PyTorch data loader.
  * DoD: Train / val / test splits ready; `__getitem__` returns a 3-second random-cropped tensor.
* **Week 2: Augmentation and feature extraction**
  * Tasks: On-the-fly noise injection (MUSAN), codec degradation (`torchaudio.functional`). Mel-spec extractor.
  * DoD: Mel-spec tensors are correctly produced from degraded audio; no visual artifacts.
* **Week 3: Baseline model training**
  * Tasks: MobileNetV3-Small classifier. Training loop with BCE + Focal Loss in PyTorch Lightning.
  * DoD: A checkpoint reaches AUC > 0.90 on in-domain data.
* **Week 4: Wav2Vec2 feature integration**
  * Tasks: Plug in a lightweight SSL model (e.g. DistilHuBERT). Train the hybrid architecture.
  * DoD: Initial model with AUC > 0.94 on OOD and FPR ≤ 1%.

### Phase 2: Edge optimization and inference SDK (Weeks 5–8)
* **Week 5: ONNX export and quantization**
  * Tasks: Export from PyTorch to ONNX, apply INT8 dynamic quantization.
  * DoD: AUC drop ≤ `0.01` after quantization, model file size ≤ `15 MB`.
* **Week 6: Inference pipeline (C++ / Python)**
  * Tasks: Audio stream buffering. Sliding-window continuous inference.
  * DoD: A local WAV file replayed as a pseudo-stream produces correctly logged scores over time.
* **Week 7: WASM / browser integration**
  * Tasks: TypeScript inference wrapper using `onnxruntime-web`. Audio capture via Web Audio API.
  * DoD: Mic input in the browser produces real-time Real / Fake scores in the console with ≤ 100 ms latency.
* **Week 8: Mobile API prototypes (iOS / Android)**
  * Tasks: Swift (CoreML / ONNX) and Kotlin (NNAPI) bindings.
  * DoD: A test app on each device (e.g. iPhone 13, Pixel 6) runs inference cleanly and logs runtime numbers.

### Phase 3: Demo apps and launch (Weeks 9–12)
* **Week 9: Chrome Extension demo**
  * Tasks: Package as Manifest V3. Capture audio from active YouTube tabs.
  * DoD: YouTube audio is analyzed in real time and a score overlay renders without lag.
* **Week 10: Hugging Face Spaces / OSS release**
  * Tasks: Gradio-based browser demo. Author the model card.
  * DoD: HF Hub repo is public; users can upload audio and see results.
* **Week 11: Tech report (arXiv)**
  * Tasks: Write up evaluation methodology, architecture rationale, edge performance numbers.
  * DoD: Preprint submitted to arXiv with a URL.
* **Week 12: Marketing and launch**
  * Tasks: Hacker News (Show HN), tech-community posts on X, pitches to tech press.
  * DoD: Quantitative traction (e.g. GitHub stars 500+, extension installs 1,000+) and a working contact funnel that includes Google enterprise.

## 3. Risks, mitigations, and uncertainty
| Risk / Uncertainty | Severity | Mitigation |
| :--- | :--- | :--- |
| **Cannot keep up with the latest generators (e.g. Voicebox, ElevenLabs V2), accuracy degrades** | High | Lean harder on wav2vec "phoneme-transition unnaturalness" instead of mel-spec alone. Build a CI/CD pipeline that periodically pulls fresh outputs from the latest models into training. |
| **Significant accuracy loss from quantization (FPR rises)** | Med | Switch from dynamic to static quantization with real-world noisy calibration data. Or introduce QAT (Quantization-Aware Training) in Phase 2. |
| **ONNX Runtime Web (WASM) performance / memory leaks** | Med | Tune WASM thread count and SIMD options. If wav2vec is the bottleneck, fork a "nano" variant with fewer layers exclusively for the web. |
| **Chrome Web Store rejection (MV3 constraints)** | Low–Med | Make the rationale for `tabCapture` permission explicit. State emphatically — in policy and reviewer notes — that the extension performs purely local processing. |

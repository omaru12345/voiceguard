# voiceguard

A **tiny deepfake voice detection model + inference SDK** that runs in the browser and on mobile.
Analyzes audio samples (phone calls, web meetings, YouTube videos) locally and returns the probability that the voice was AI-generated.

---

## 🎯 Goal: Acquisition by Google

This project targets a **Jetpac-style acqui-hire by Google** (2–5 person team / technical core + UX demo).

**The acquisition target is the trained model + inference SDK (the technical core); the Chrome extension is a distribution channel for visibility and proof-of-concept.**

| Likely buyer | Integration scenario |
|---|---|
| **Chrome Safe Browsing** | Warn users about suspicious audio / video content (the natural extension of the Hume AI acquisition). |
| **Google Meet** | "This voice may be a deepfake" warnings during calls (a B2B security feature). |
| **YouTube** | Auto-label uploads that contain AI-generated audio. |
| **Pixel / Phone by Google** | Real-time scam protection that flags AI-cloned voices on incoming calls. |

### Technical moat we are paid for
1. **Model size < 10 MB** (runs on mobile and in the browser)
2. **Multilingual coverage** (detection beyond English / Japanese is rare)
3. **Generalization across generators** (ElevenLabs / OpenAI TTS / Suno / unseen models)
4. **Low false positive rate** (FPR < 1% is the bar to ship inside Chrome)

---

## Tech stack

| Layer | Tech |
|---|---|
| Training | PyTorch (features: mel-spectrogram + wav2vec-derived embeddings) |
| Data | ASVspoof / FakeAVCeleb / synthetic data we generate from open-source TTS |
| Inference | ONNX Runtime (Web / iOS / Android) + WASM SIMD |
| Demo extension | Chrome Manifest V3 (tab audio captured via WebAudio → inference) |
| Distribution | Model on Hugging Face Hub, inference SDK on npm |

---

## Repository layout

```
voiceguard/
├── model/                  # Training scripts and dataset definitions
├── inference/              # Inference SDK (TypeScript / Swift / Kotlin)
├── examples/
│   └── chrome-extension/   # In-browser detection demo
└── docs/                   # Evaluation reports / acquisition pitch material
```

---

## Three-phase plan

### Phase 1 (≤ 2 months): Model development
- [ ] Reach AUC > 0.95 on ASVspoof2021
- [ ] Export to ONNX, INT8-quantize to < 10 MB
- [ ] Publish scores on public benchmarks (FakeAVCeleb, etc.)

### Phase 2 (≤ 4 months): SDK + multilingual coverage
- [ ] Support Japanese TTS (VOICEVOX, Style-Bert-VITS2, etc.)
- [ ] Publish the `voiceguard-web` / `voiceguard-mobile` SDKs
- [ ] Publish the model on Hugging Face (download count signals technical credibility)

### Phase 3 (≤ 6 months): Visibility
- [ ] Publish the Chrome extension (real-time warning UI)
- [ ] Submit a tech report to arXiv
- [ ] Send demos to contacts at Google Trust & Safety / DeepMind

---

## Development commands

```bash
# Training
cd model && python train.py --config configs/baseline.yaml

# Inference SDK
cd inference && npm install && npm test

# Run the extension locally
cd examples/chrome-extension && npm run dev
```

(Implementations land when Phase 1 starts.)

# 02 — Scope

A small team can only win by being narrower than the competition. This file defines
the hard scope. Anything outside it must be rejected, even if technically interesting.

## DO focus on

- Detection accuracy at very low FPR (target: TPR > 85% at FPR = 0.1%)
- Generalization to **unseen** TTS / vocoder generators (cross-dataset evaluation)
- Edge inference: model ≤ 15 MB, latency ≤ 100 ms / 3-second chunk
- Multilingual coverage (Japanese is a real differentiator)
- A clean SDK that drops into a Chrome extension, a Swift app, or a Kotlin app

## DO NOT build

- **Video deepfake detection.** Audio is hard enough. Stay laser-focused on audio.
- **A cloud inference API.** Going server-side destroys the privacy and latency moat.
- **Voice generators / TTS.** That is the offense; we are the defense. Don't blur the role.
- **A consumer SaaS product.** This is a model + SDK to be embedded by Google, not a B2C app.
- **Manual data labeling pipelines from scratch.** Use existing public benchmarks
  (ASVspoof, FakeAVCeleb) and synthesize new fakes from open TTS models.

## Why this matters

Every "DO NOT" item above has been considered and rejected because it would either
(a) duplicate an existing competitor, (b) dilute the technical moat (privacy, edge, audio-only),
or (c) push the team toward operational work that a 1–5 person crew cannot sustain.

If you believe a "DO NOT" item should be reopened, escalate to the human owner —
do not start implementing it.

# voiceguard

ブラウザ / モバイルで動く **超軽量 Deepfake 音声検出モデル + 推論 SDK**。
音声サンプル（電話 / Web 会議 / YouTube 動画）をローカルで解析し、AI 生成音声である確率を返す。

---

## 🎯 Goal: Googleに買収されること

このプロジェクトは「Google による Jetpac 型 acqui-hire（2〜5人 / 技術コア + UX デモ）」をエグジット目標とする。

**買収対象は学習済みモデル + 推論 SDK（コア技術）であり、Chrome 拡張は世間への露出と PoC のための配布チャネル。**

| 想定買収先 | 統合シナリオ |
|---|---|
| **Chrome セーフブラウジング** | 不審な音声/動画コンテンツに対する警告（Google Hume AI 買収の延長線）|
| **Google Meet** | 通話中の "Deepfake かもしれません" 警告（B2B セキュリティ機能）|
| **YouTube** | アップロード時の "AI 生成音声含有" 自動ラベリング |
| **Pixel / Phone by Google** | 通話相手が AI ボイスかをリアルタイム判定する詐欺対策機能 |

### 評価される技術の堀（moat）
1. **モデルサイズ < 10MB**（モバイル / ブラウザで動く）
2. **多言語対応**（日本語・英語以外でも判定できることが希少）
3. **複数の生成器に対する汎化性**（ElevenLabs / OpenAI TTS / Suno / 未知のモデル）
4. **誤検知率の低さ**（FPR < 1% でなければ Chrome に統合できない）

---

## 技術スタック

| レイヤー | 技術 |
|---|---|
| 学習 | PyTorch（特徴量: メルスペクトログラム + wav2vec 派生埋め込み） |
| データ | ASVspoof / FakeAVCeleb / 自作合成データ（OSS TTS から生成） |
| 推論 | ONNX Runtime（Web / iOS / Android）+ WASM SIMD |
| デモ拡張 | Chrome Manifest V3（タブ音声を WebAudio で取得 → 推論） |
| 公開 | Hugging Face Hub にモデル公開、npm に推論 SDK 公開 |

---

## ディレクトリ構成

```
voiceguard/
├── model/                  # 学習スクリプト・データセット定義
├── inference/              # 推論 SDK（TypeScript / Swift / Kotlin）
├── examples/
│   └── chrome-extension/   # ブラウザ向け検出デモ
└── docs/                   # 評価レポート / 買収ピッチ
```

---

## 3 フェーズ計画

### Phase 1（〜2か月）: モデル開発
- [ ] ASVspoof2021 で AUC > 0.95 を達成
- [ ] モデルを ONNX 化、INT8 量子化で < 10MB
- [ ] 公開ベンチマーク（FakeAVCeleb 等）でスコア掲載

### Phase 2（〜4か月）: SDK 化 + 多言語化
- [ ] 日本語 TTS（VOICEVOX, Style-Bert-VITS2 等）への対応
- [ ] `voiceguard-web` / `voiceguard-mobile` SDK を公開
- [ ] Hugging Face にモデル公開（ダウンロード数で技術力を示す）

### Phase 3（〜6か月）: 露出
- [ ] Chrome 拡張公開（リアルタイム警告 UI）
- [ ] arXiv に技術レポート
- [ ] Google Trust & Safety / DeepMind 関係者にデモ送付

---

## 開発コマンド

```bash
# 学習
cd model && python train.py --config configs/baseline.yaml

# 推論 SDK
cd inference && npm install && npm test

# 拡張のローカル起動
cd examples/chrome-extension && npm run dev
```

（実装は Phase 1 着手時に追加）

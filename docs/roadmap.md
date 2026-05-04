# VoiceGuard: 12-Week Development Roadmap

本ロードマップは、3ヶ月（12週間）で技術実証から買収候補となる露出までを完了させるための実行計画である。

## 1. データセット選定基準と評価プロトコル
* **学習/評価データセット**:
  * **ベースライン**: ASVspoof 2019 / 2021 (学術ベースの汎用検出), FakeAVCeleb (より多様なフェイク)
  * **独自構築データ**: LibriSpeech (CC-BY) などのクリーンな音声を元に、最新のOSS TTS (VITS, Bark, XTTSv2) および商用API (ElevenLabs - *利用規約の許諾範囲内で研究用途*) を用いて生成した最新のディープフェイク音声。
  * **ノイズ・劣化**: G.711 (μ-law/A-law) 電話コーデック、Opus圧縮、背景雑音 (MUSAN) をデータローダー内で動的に付与 (On-the-fly augmentation)。
* **評価プロトコル**:
  * Cross-dataset評価を必須とする。（例: ASVspoofで学習したモデルを、独自構築のElevenLabsデータセットで評価）
  * **KPI**: 未知のドメイン（Out-of-domain）において、FPR(誤検知率) `0.1%` を固定した際の TPR(真陽性率) を計測。

## 2. フェーズ別タスク分解

### Phase 1: PoCとデータパイプライン構築 (Weeks 1-4)
* **Week 1: データ基盤の構築**
  * タスク: ASVspoof, LibriSpeechのダウンロード・前処理。データローダー（PyTorch）の実装。
  * DoD: 訓練/検証/テストのデータ分割が完了し、`__getitem__` で3秒のランダムクロップされた音声テンソルが出力されること。
* **Week 2: データ拡張と特徴量抽出パイプライン**
  * タスク: On-the-flyでのノイズ付加（MUSAN）、コーデック劣化（torchaudio.functional）実装。Mel-spec抽出器の実装。
  * DoD: 音響劣化を経た音声から正常にMel-specテンソルが生成され、視覚的にバグがないことの確認。
* **Week 3: ベースラインモデル学習**
  * タスク: MobileNetV3-Smallベースの分類器実装。BCE + Focal Lossを用いた学習ループ（PyTorch Lightning）の構築。
  * DoD: In-domainデータでAUC > 0.90 を達成するチェックポイントが保存されること。
* **Week 4: Wav2Vec2特徴量の統合**
  * タスク: DistilHuBERT等の軽量SSLモデルの組み込み、ハイブリッドアーキテクチャの学習。
  * DoD: OOD（未知データ）においてAUC > 0.94、FPR 1% 以下の初期モデルが完成すること。

### Phase 2: エッジ最適化と推論SDK開発 (Weeks 5-8)
* **Week 5: ONNX変換と量子化**
  * タスク: PyTorchモデルのONNXエクスポート、INT8 Dynamic Quantizationの適用。
  * DoD: 量子化前後でAUCの低下が `0.01` 以内であり、モデルファイルサイズが `15MB` 以下になること。
* **Week 6: 推論パイプライン（C++/Python）の構築**
  * タスク: 音声ストリームのバッファリング、スライディングウィンドウによる連続推論ロジックの実装。
  * DoD: ローカルのWAVファイルを擬似ストリームとして入力し、時間経過に伴う推論スコアのログが正しく出力されること。
* **Week 7: WASM / Browser統合**
  * タスク: `onnxruntime-web` を用いたTypeScript推論クラスの実装。Web Audio APIからのオーディオキャプチャ。
  * DoD: ブラウザ上でマイク入力を受け取り、100ms以下のレイテンシでリアルタイムにスコア（Real/Fake）をコンソール出力できること。
* **Week 8: モバイルAPIのプロトタイピング (iOS/Android)**
  * タスク: Swift (CoreML/ONNX) および Kotlin (NNAPI) のバインディング作成。
  * DoD: 各実機（例: iPhone 13, Pixel 6）のテストアプリで、推論処理がエラーなく実行され、実行時間ログが取得できること。

### Phase 3: デモアプリ開発と公開戦略実行 (Weeks 9-12)
* **Week 9: Chrome Extension デモの実装**
  * タスク: Manifest V3形式でのパッケージング、YouTube等のアクティブなタブからの音声キャプチャロジック実装。
  * DoD: YouTubeの音声をリアルタイム解析し、画面上に推論スコアのオーバーレイUIが遅延なく表示されること。
* **Week 10: Hugging Face Spaces / OSS 公開**
  * タスク: Gradioを用いたブラウザで試せるデモUIの構築。モデルリポジトリのREADME（Model Card）作成。
  * DoD: HF Hub上でリポジトリがPublicになり、誰でも音声ファイルをアップロードして判定結果を得られる状態になること。
* **Week 11: テクニカルペーパー (arXiv) 執筆**
  * タスク: 評価指標、アーキテクチャの妥当性、エッジ環境でのパフォーマンステスト結果をまとめた論文の執筆。
  * DoD: arXivへプレプリントがSubmitされ、URLが発行されること。
* **Week 12: マーケティング・ローンチ活動**
  * タスク: Hacker Newsへの投稿 (Show HN)、X (Twitter) での技術コミュニティへの発信、テック系メディアへのプレスリリース（Pitch）。
  * DoD: 定量的なトラフィック（例: Github Star 500+, 拡張機能インストール 1000+）を獲得し、Google関係者を含むエンタープライズからのコンタクト導線（専用フォーム）が機能していること。

## 3. 想定リスクと回避策 / 技術的不確実性
| 想定リスク / 不確実性 | 影響度 | 回避策 (Mitigation Strategy) |
| :--- | :--- | :--- |
| **最新の生成器（例: Voicebox, ElevenLabs V2）に追従できず精度が低下する** | 高 | 音響特徴量（Mel-spec）だけでなく、Wav2vecによる「音素遷移の不自然さ」の重みを高める。定期的に最新モデルの生成音声を学習データに組み込むCI/CDパイプラインを構築する。 |
| **量子化による著しい精度劣化（FPRの増大）** | 中 | Dynamic Quantizationで劣化が激しい場合は、実環境のノイズデータを用いたStatic Quantization（キャリブレーション付き）へ移行する。または量子化対応学習 (QAT: Quantization-Aware Training) をPhase 2で導入する。 |
| **ONNX Runtime Web (WASM) でのパフォーマンス不足・メモリリーク** | 中 | WASMスレッド数やSIMDオプションのチューニングを行う。重いWav2vec層がボトルネックになる場合、Wav2vecブロックをさらに軽量化（層の削減）したナノモデルをWeb専用にフォークする。 |
| **ブラウザ拡張機能の審査リジェクト (Manifest V3の制約)** | 低〜中 | タブの音声キャプチャ権限 (`tabCapture`) の要求理由を明確にし、外部サーバーへのデータ送信を行わない「完全ローカル処理」であることをプライバシーポリシーと審査用ノートで強く主張する。 |

# VoiceGuard: Acquisition Thesis (Google Acqui-hire Target)

## 1. なぜGoogleが買うのか（Why Google?）
Googleは「AIの民主化」を進める一方で、「AIによる悪用（詐欺、フェイクニュース、スパム）」の矢面に立たされている。
サーバーサイドに音声を送信して解析するアプローチは、通話内容や会議内容の盗聴と同義であり、GDPRやプライバシー保護の観点で実現不可能。
**「ローカル（エッジ）で完結し、ユーザーのプライバシーを保護しながら超低遅延でAI生成音声をブロックする技術」**は、Googleの既存プラットフォームの安全性を担保するためのミッシングピースである。

## 2. ターゲットとなる具体的チームと統合シナリオ
VoiceGuardのコア技術は、以下のチームの直近のOKR（Objective and Key Results）に直接貢献する。

* **Android / Pixel チーム (Call Screen / Spam Filtering)**
  * *課題*: 巧妙化するVishing（音声詐欺）、特に身内の声をクローンした詐欺電話への対応。
  * *統合シナリオ*: PixelのCall Screen（電話スクリーニング機能）にVoiceGuardのINT8モデルをオンデバイス統合。通話開始の最初の3秒でディープフェイク判定を行い、ユーザーに警告UIを表示。
* **Chrome チーム (Safe Browsing)**
  * *課題*: YouTubeやWeb上の悪意あるディープフェイク動画/音声のブラウザベースでのフィルタリング。
  * *統合シナリオ*: Chrome本体またはGoogle標準の拡張機能として、メディアストリーム（WebRTC / HTML5 Audio）をローカルでリアルタイム監視。
* **Google Meet チーム (Trust & Safety / Anti-abuse)**
  * *課題*: 会議におけるなりすまし（CEO詐欺など）の防止。
  * *統合シナリオ*: Meetクライアント内でマイク入力/受信音声を監視し、合成音声の疑いがある参加者にフラグを立てる。
* **(参考文脈) Hume AI買収の延長線**: Googleは感情認識・音声解析の強化に動いており、音声基盤技術を持つ小規模かつエッジ特化のエリートチームを「acqui-hire」する素地は十分にある。

## 3. 競合環境と差別化要素 (Moat)
| 競合企業 | アプローチ | 弱点・課題 | VoiceGuardの優位性 (Moat) |
| :--- | :--- | :--- | :--- |
| **Reality Defender** | API / サーバーサイドの大規模モデル | レイテンシが大きく、プライバシー制約上コンシューマーの通話に適用不可。 | 完全なエッジ処理。外部通信ゼロでセキュア。 |
| **Pindrop** | コールセンター向け、クローズドAPI | B2Bエンタープライズ特化。スマートフォンOSやブラウザレベルへの統合が困難。 | 超軽量・クロスプラットフォームSDK (モバイル/WASM)。 |
| **Hiya** | 電話番号やネットワークパターンの分析 | 未知の番号や正規回線を乗っ取ったVishing、Web上の音声には無力。 | 音声の「音響的・生成痕跡」そのものを直接解析。 |

**【VoiceGuardの技術の堀 (Moat)】**
精度の高い重いモデルを作ることは容易だが、「汎化性能（未知のTTSモデルへの対応）」と「エッジ向け極限圧縮（<15MB, <100ms）」を両立させた「推論SDK」の存在自体が最大の堀となる。具体的には、**FPR 0.1%の制約下での精度維持ノウハウ**と、**実世界の電話網（コーデック劣化・パケットロス）に耐えるデータ拡張パイプライン**である。

## 4. 買収を引き寄せる露出戦略 (Go-to-Market for Acqui-hire)
Googleのエンジニアリングディレクターやリサーチャーの目に留まるよう、学術的・OSS的な「Proof of Competence（技術力の証明）」を戦略的に展開する。

1. **Hugging Face公開とデモ展開**
   * `voiceguard-nano-v1` モデルをHF Hubで公開。Gradio / Spaces で実際に手元の音声をアップロードして試せるデモを提供。
   * **KPI**: HF Trending 1位獲得、月間10,000ダウンロード。
2. **Hacker News (Y Combinator) でのローンチ**
   * 「Show HN: We built a 12MB local Deepfake audio detector that runs in your browser」としてChrome Manifest V3デモを公開。
   * Webブラウザ上でリアルタイムにYouTube動画の音声を判定する拡張機能の実用性をアピールし、HNのトップページ獲得を狙う。
3. **arXivへのテクニカルレポート投稿**
   * タイトル案: *"Sub-15MB Deepfake Audio Detection for Edge Devices: Achieving 98% AUC with 0.1% FPR"*
   * 手法そのものの新規性より「実運用に耐えうるエンジニアリングアプローチと厳しい評価指標（FPR=0.1%）での達成」を強調。Googleリサーチャーが引用・ベンチマークとして使いやすい形にする。

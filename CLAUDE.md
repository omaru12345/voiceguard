# CLAUDE.md — voiceguard

## プロジェクト概要

ブラウザ・モバイルで動く Deepfake 音声検出モデル + 推論 SDK。
**コア技術 = 学習済みモデル + 軽量推論ランタイム**。Chrome 拡張は PoC。

ゴール: Google による acqui-hire（Trust & Safety / Chrome / Pixel いずれか）。詳細は README 参照。

## 設計原則

- **モデルは小さく、汎化重視**: 1 つのデータセットだけで過学習しない
- **誤検知（FPR）を絶対的に下げる**: 真の音声を fake と誤判定する事故は致命傷
- **完全オフライン**: 推論は端末内で完結、ネットワーク送信は禁止
- **再現可能**: 学習スクリプトと評価スクリプトは Docker / uv で完全再現できる状態を維持

## ディレクトリ構成

`model/` で学習・評価、`inference/` が SDK 本体、`examples/chrome-extension/` がデモ。
詳細は README 参照。

## ブランチ命名

```
feature-{name}-{description}
bug-{name}-{description}
refactoring-{name}-{description}
```

メインブランチは `main`、PR は `main` に向ける。

## コミット前チェック

- 学習コード変更時: `pytest` + 小規模データセットで AUC が劣化していないこと
- SDK 変更時: `npm test`
- モデル更新時: `docs/eval-report.md` を更新

## やらないこと

- 動画の Deepfake 検出（音声に集中。スコープ拡大は買収後）
- 自前のクラウド推論 API 提供（オフライン性こそが価値）
- ジェネレーター（音声合成）の実装

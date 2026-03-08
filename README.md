# voxi

リアルタイム音声文字起こし Chrome 拡張機能。マイク入力とタブ音声（YouTube・会議など）の両方を、インターネット接続なしでオフライン文字起こしします。

## 特長

- **完全オフライン** — 音声データは外部に送信されません。Whisper モデルをブラウザ内で推論します
- **2ソース同時キャプチャ** — マイク（自分）とアクティブタブ音声（タブ）を同時に文字起こし
- **話者区別表示** — 「自分:」と「タブ:」で発話者を区別してリアルタイム表示
- **エクスポート** — クリップボードコピーまたは `.txt` ファイルでダウンロード
- **タイムスタンプ対応** — タイムスタンプ付きの出力にも対応

## 概要

| 項目 | 内容 |
|------|------|
| 対象ブラウザ | Chrome (Manifest V3) |
| 推論方式 | オフライン（Transformers.js） |
| モデル | `onnx-community/whisper-base`（日本語最適化） |
| 音声ソース | マイク + アクティブタブ音声 |
| 表示 | 拡張ポップアップ内 |

## インストール

### Chrome ウェブストア

（準備中）

### ソースからビルド

```bash
git clone https://github.com/lazyarea/voxi.git
cd voxi
npm install
npm run build
```

Chrome で `dist/` フォルダを「パッケージ化されていない拡張機能」として読み込んでください。

> **初回起動時**、Whisper モデル（約 145MB）をダウンロードします。進捗はポップアップ内に表示されます。

## 使い方

1. Chrome ツールバーの voxi アイコンをクリックしてポップアップを開く
2. **「開始」** ボタンをクリック（マイクの使用許可を求められたら許可する）
3. 話すか、音声が流れるタブをアクティブにする
4. 文字起こし結果がリアルタイムで表示される
5. **「コピー」** または **「ダウンロード」** でエクスポート

## アーキテクチャ

```
[マイク]  getUserMedia()      ─┐
                               ├─→ [Offscreen Document]
[タブ音声] tabCapture.capture() ─┘    AudioWorklet (16kHz リサンプリング)
                                      チャンク分割（5秒 + VAD）
                                           ↓
                                    [Whisper Worker]
                                    Transformers.js
                                    onnx-community/whisper-base
                                           ↓
                                  { source: 'mic'|'tab', text }
                                           ↓
                                    [Popup UI]
                                    自分: ○○○○○
                                    タブ: ○○○○○
```

## 技術スタック

| ライブラリ | 用途 |
|-----------|------|
| [Transformers.js v3](https://huggingface.co/docs/transformers.js) | Whisper ブラウザ内推論 |
| [Chrome Extension APIs](https://developer.chrome.com/docs/extensions/) | `tabCapture`, `offscreen` |
| Web Audio API | `AudioContext`, `AudioWorklet` |
| Web Workers | Whisper 推論をメインスレッドから分離 |
| [@ricky0123/vad-web](https://github.com/ricky0123/vad) | 音声区間検出（Silero VAD） |
| Tailwind CSS | UI スタイリング |
| Vite | バンドラー |
| Vitest | テストフレームワーク |

## 開発

```bash
npm install
npm run dev    # Vite dev build（watch モード）
npm run build  # dist/ に本番ビルド
npm run test   # テスト実行
```

## プラン

| プラン | 機能 | 価格 |
|--------|------|------|
| **Free Plan** | マイク音声の文字起こし（インストールから15日間） | 無料 |
| **Pro** | マイク＋タブ音声（YouTube・会議など）の文字起こし | $10/月 |

## プライバシー

voxi はすべての音声処理をブラウザ内でローカルに実行します。音声データ・文字起こし結果を外部サーバーに送信することは一切ありません。

## ライセンス

MIT

## 参考

- [whisper-web](https://github.com/xenova/whisper-web) — Transformers.js によるブラウザ内 Whisper のリファレンス実装
- [Transformers.js ドキュメント](https://huggingface.co/docs/transformers.js)
- [Chrome tabCapture API](https://developer.chrome.com/docs/extensions/reference/api/tabCapture)
- [Chrome Offscreen API](https://developer.chrome.com/docs/extensions/reference/api/offscreen)

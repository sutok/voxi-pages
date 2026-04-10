# voxi

リアルタイム音声文字起こし Chrome 拡張機能。  
Free プランは Chrome の Web Speech API、Pro プランは Cloudflare Workers AI（Whisper）を利用して文字起こしします。

## 特長

- **2ソース対応（Pro）** — マイク（自分）とアクティブタブ音声（タブ）を同時に文字起こし
- **話者区別表示** — 「自分:」と「タブ:」で発話者を区別してリアルタイム表示
- **エクスポート** — クリップボードコピーまたは `.txt` ファイルでダウンロード
- **プラン別処理** — Free は Web Speech API、Pro は Whisper（Cloudflare Workers AI）
- **Chrome 拡張（MV3）** — `tabCapture` / `offscreen` を活用した構成

## 概要

| 項目 | 内容 |
|------|------|
| 対象ブラウザ | Chrome (Manifest V3) |
| Free プラン推論方式 | Chrome 内蔵音声認識（Web Speech API） |
| Pro プラン推論方式 | Cloudflare Workers AI `@cf/openai/whisper` |
| 音声ソース | Free: マイク / Pro: マイク + アクティブタブ音声 |
| 表示 | 拡張ポップアップ内 |

## 使い方

1. Chrome ツールバーの voxi アイコンをクリックしてポップアップを開く
2. 「開始」ボタンをクリック（マイク使用許可を求められたら許可）
3. 話すか、音声が流れるタブをアクティブにする
4. 文字起こし結果がリアルタイムで表示される
5. 「コピー」またはダウンロードでエクスポート

## プラン

| プラン | 文字起こし方式 | 対応ソース | 価格 |
|--------|----------------|------------|------|
| **Free** | Chrome 内蔵音声認識（Web Speech API） | マイク | 無料 |
| **Pro** | Cloudflare Workers AI（Whisper） | マイク + タブ音声 | $10/月 |

### Pro プランについて

Pro プランでは、拡張機能の Offscreen Document で音声を処理し、音声区間を分割して Cloudflare Workers AI（Whisper）に送信して文字起こしします。  
マイク音声とタブ音声を同時に扱い、結果を「自分 / タブ」で分けて表示します。

- 決済は Stripe を利用
- サブスクリプション管理は Firebase / Firestore と連携
- 解約は Stripe Customer Portal から実施可能

## プライバシー

プライバシーとデータ取扱いの詳細は `docs/privacy.md` をご確認ください。  
本サービスはプランにより音声処理方式が異なります（Free: Web Speech API / Pro: Cloudflare Workers AI）。

## お問い合わせ

サービス内容および個人情報の取扱いに関するお問い合わせは、公開している連絡窓口をご利用ください。

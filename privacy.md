# プライバシーポリシー

**最終更新日**: 2026年4月10日

---

## 0. 重要なお知らせ（2026年4月 改定）

本ポリシーは Voxi v1.1 以降の仕様変更に合わせて全面改定されました。主な変更点は以下のとおりです。

- **Pro プランの推論基盤を変更**: Pro プランの音声文字起こしは、端末内のオフライン推論から **Cloudflare Workers AI（`@cf/openai/whisper`）によるクラウド推論** に変更されました。Pro プラン利用時は、文字起こし対象の音声チャンクが当社バックエンド経由で Cloudflare に送信されます。
- **認証・課金基盤の追加**: Google アカウントでのサインイン（Firebase Authentication）と Stripe によるサブスクリプション課金を追加しました。
- **Free プランの推論方式**: Free プランは従来どおり外部送信なしで動作しますが、内部実装が Transformers.js から**ブラウザ内蔵の Web Speech API** に変更されました。

Free プランのみを利用する場合、音声データが端末外に出ることはありません。Pro プランを利用する場合のみ、第 4 章・第 5 章に記載するクラウド送信が発生します。

---

## 1. Voxi について

Voxi はリアルタイム音声文字起こしを提供する Chrome 拡張機能（Manifest V3）です。マイク入力とブラウザタブの音声（YouTube・オンライン会議・動画講義など）をテキスト化します。

| 項目 | 内容 |
|------|------|
| 提供者 | k4zuhisa |
| 配布チャネル | [Chrome ウェブストア](https://chromewebstore.google.com/detail/voxi/eabfnnmlbcokddidhefenmdohjlfifcm) |
| 対応ブラウザ | Google Chrome (Manifest V3) |
| 対応言語 | 日本語 |

---

## 2. 提供するプラン

| プラン | 推論方式 | 対応音声 | 料金 |
|--------|---------|---------|------|
| **Free** | ブラウザ内蔵 Web Speech API（外部送信なし） | マイクのみ | 無料 |
| **Pro** | Cloudflare Workers AI `@cf/openai/whisper`（クラウド送信） | マイク ＋ タブ音声 | 月額 10 USD（Stripe サブスクリプション） |

Pro プランのお支払いは Stripe Checkout を通じた月額サブスクリプションです。解約は拡張機能内の「サブスクリプション解除」ボタンから Stripe Customer Portal 経由でいつでも可能です。解約後も現在の請求期間終了までは Pro 機能を利用できます。

---

## 3. 収集・保存する情報

### 3.1 端末内に保存するデータ（`chrome.storage.local`）

| データ | 用途 | 外部送信 |
|--------|------|---------|
| 言語設定 | 次回起動時に文字起こし言語を復元 | しない |
| プランステータス（`isPro` など） | Free / Pro の UI 表示切替 | しない |
| 直近の録音状態 | 不意の Service Worker 停止からの復帰用 | しない |

端末内データは拡張機能のアンインストール時にすべて削除されます。

### 3.2 当社バックエンドに保存するデータ（Firebase Firestore）

Pro プラン利用者およびサインイン済みユーザーについて、以下を Firestore の `users/{uid}` ドキュメントに保存します。

| フィールド | 内容 |
|-----------|------|
| `uid` | Firebase Authentication が発行する一意識別子 |
| `email` | Google アカウントのメールアドレス |
| `displayName`, `photoURL` | Google アカウントの表示名・アバター |
| `createdAt`, `lastLoginAt` | 初回作成・最終ログイン時刻 |
| `subscription.status` | `free` / `active` / `past_due` / `canceled` |
| `subscription.plan` | `free` / `pro` |
| `subscription.stripeCustomerId` | Stripe 顧客 ID（`cus_xxx`） |
| `subscription.stripeSubscriptionId` | Stripe サブスクリプション ID（`sub_xxx`） |
| `subscription.currentPeriodStart`, `currentPeriodEnd` | 現在の請求期間 |
| `subscription.cancelAtPeriodEnd` | 期間終了解約予約フラグ |
| `usage.totalMinutes`, `monthlyMinutes`, `lastResetAt` | 文字起こし利用時間の集計 |

これらは認証・課金・プラン判定の目的にのみ使用し、第三者に提供しません。

---

## 4. マイクへのアクセス

Voxi はユーザーが拡張機能のポップアップで「開始」ボタンを押した場合にのみ、ブラウザの `getUserMedia()` API を通じてマイクへのアクセスを要求します。

### Free プラン

- マイク音声は Content Script に注入した **ブラウザ内蔵の Web Speech API**（`webkitSpeechRecognition`）に渡されます
- 文字起こしはブラウザが OS またはブラウザベンダーの仕組みを用いて行います
- Voxi は音声データを受け取らず、外部サーバーにも送信しません
- **注**: Web Speech API の内部動作（ブラウザが音声をどのように処理するか）は Google Chrome の実装に依存します。詳細は [Google Chrome プライバシーホワイトペーパー](https://www.google.com/chrome/privacy/whitepaper.html) をご確認ください。

### Pro プラン

- マイク音声は MV3 Offscreen Document 内で Web Audio API により 16 kHz にリサンプリングされ、VAD（音声区間検出）で発話区間のみ抽出されます
- 発話チャンク（約 15 秒単位の WAV バイナリ）は第 6 章のクラウド経路で Cloudflare Workers AI へ送信されます

---

## 5. タブ音声へのアクセス（Pro プランのみ）

Pro プランでは、アクティブタブで再生中の音声（YouTube・オンライン会議・動画講義など）を Chrome の `tabCapture` API で取得し、マイクと同じ経路で文字起こしします。

- タブ音声のキャプチャは、ユーザーがポップアップで「開始」ボタンを押したアクティブタブに対してのみ行われます
- 取得した音声はブラウザ外再生を行わず、`AudioContext.createMediaStreamDestination()` を用いてタブ自身の音声再生は維持されます
- タブ音声もマイクと同様に 16 kHz WAV チャンクに変換され、第 6 章の経路で Cloudflare Workers AI に送信されます

---

## 6. Pro プランにおける音声データの送信経路

Pro プラン利用時、発話チャンクは以下の経路で処理されます。

```
[マイク / タブ音声]
      ↓
[拡張の Offscreen Document]
   - Web Audio API で 16 kHz にリサンプリング
   - VAD で無音をスキップ（無音チャンクは送信しません）
   - WAV バイナリにエンコード
      ↓
POST https://<voxi-worker>.workers.dev/transcribe
   Authorization: Bearer <Firebase ID Token>
      ↓
[当社 Cloudflare Worker]
   - Firebase ID Token を検証
   - サブスクリプション状態を確認
   - env.AI.run('@cf/openai/whisper', { audio, task: 'transcribe' }) を呼び出し
      ↓
[Cloudflare Workers AI]
   - @cf/openai/whisper モデルによる推論
      ↓
{ source: 'mic' | 'tab', text } を拡張機能に返却
      ↓
ポップアップ画面に表示
```

### 送信される情報

- **音声データ**: 16 kHz モノラル WAV、約 15 秒チャンク単位。無音区間は送信されません
- **Firebase ID Token**: サーバー側で uid を検証するために送信されます
- **メタデータ**: `source`（`mic` / `tab` の区別）、リクエスト識別用の相関 ID

### 送信されない情報

- 閲覧中タブの URL・タブタイトル・Cookie・ページ内容
- Google アカウントの認証情報そのもの（ID Token のみ送信）
- クレジットカード番号やその他の決済情報（これらは Stripe が直接取扱います）

### 保存・保持について

- 当社 Cloudflare Worker は、文字起こし結果を返却した後に音声データを保持しません（リクエストスコープで破棄）
- Cloudflare Workers AI 上での音声データの取扱いについては [Cloudflare のプライバシーポリシー](https://www.cloudflare.com/privacypolicy/) および [Workers AI のデータ取扱い](https://developers.cloudflare.com/workers-ai/) をご確認ください
- 運用監視のためのログには、uid / 相関 ID / 推論所要時間 / 音声長のみを記録し、音声本体やテキスト内容は記録しません

---

## 7. 認証（Firebase Authentication）

Pro プランの購入・利用には Google アカウントでのサインインが必要です。

- 認証には Google Identity Platform および Firebase Authentication を使用します
- 当社は Google アカウントのパスワードを取得・保管しません
- 取得する情報は uid・メールアドレス・表示名・アバター URL のみです
- サインアウト後、ローカルの認証状態は破棄されます

---

## 8. 決済（Stripe）

Pro プランの決済には Stripe を利用します。

- クレジットカード情報は Stripe が直接取得・保管し、Voxi および当社サーバーは**カード番号・セキュリティコード等を一切取得・保存しません**
- 当社が保持するのは Stripe が発行する顧客 ID（`cus_xxx`）とサブスクリプション ID（`sub_xxx`）、請求期間情報、プランステータスのみです
- サブスクリプション状態の同期は Stripe Webhook（`checkout.session.completed` / `customer.subscription.created` / `customer.subscription.updated` / `customer.subscription.deleted`）経由で行います
- 決済情報の詳しい取扱いは [Stripe プライバシーポリシー](https://stripe.com/privacy) をご確認ください

---

## 9. 使用する Chrome 権限

| 権限 | 使用目的 |
|------|---------|
| `tabCapture` | Pro プランでアクティブタブの音声を取得するため |
| `offscreen` | MV3 制約下で AudioContext を動作させるため |
| `storage` | プラン設定・言語設定の保存（ローカル） |
| `scripting` / `activeTab` | Free プランで Web Speech API 用 Content Script を注入するため |

マイクへのアクセスは `getUserMedia()` 呼び出し時にブラウザがランタイムで要求します。

---

## 10. 第三者への情報提供

当社は次の目的以外で第三者に情報を販売・提供することはありません。

| 提供先 | 提供する情報 | 目的 |
|--------|-------------|------|
| Google（Firebase Authentication） | メール・表示名・uid | サインイン認証 |
| Google（Cloud Firestore） | ユーザードキュメント（第 3.2 節参照） | サブスクリプション状態の保存 |
| Cloudflare（Workers / Workers AI） | 音声チャンク・Firebase ID Token | Pro プランの Whisper 推論 |
| Stripe | メール・顧客 ID | 決済処理・サブスクリプション管理 |

広告トラッキング・アナリティクスに第三者サービスを使用することはありません。

---

## 11. データの保持と削除

- **Free プラン**: ローカルストレージのみ。アンインストールですべて削除されます。
- **Pro プラン**:
  - Firestore 上のユーザードキュメントはアカウント削除申請まで保持されます
  - Cloudflare Worker 上の音声データはリクエスト処理後に破棄されます
  - Stripe 上の決済履歴は、Stripe の規約および各国の会計法令に従って保持されます

### アカウント削除のご依頼

Firebase Firestore 上のデータ削除をご希望の場合は、第 13 章のお問い合わせ先までご連絡ください。確認のうえ、当該 uid のドキュメントを削除します。Stripe 上の決済履歴は法令上の保持期間の間、削除できない場合があります。

---

## 12. ポリシーの変更

本ポリシーを変更した場合は、このページの「最終更新日」を更新します。重要な変更がある場合は拡張機能内または Chrome ウェブストアの説明文でもお知らせします。

---

## 13. お問い合わせ

プライバシーに関するご質問・アカウント削除のご依頼は、以下からお送りください。

- Chrome ウェブストア デベロッパー連絡先: [Voxi ストアページ](https://chromewebstore.google.com/detail/voxi/eabfnnmlbcokddidhefenmdohjlfifcm) 内の「サポート」タブ
- GitHub Issue: （voxi-pages リポジトリの Issue）

---

&copy; 2026 Voxi

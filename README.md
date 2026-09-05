## Go / Google Cloud / 生成 AI

**公開 Go ライブラリ 19 本**と、それらを統合して **Cloud Run + Cloud Tasks** 上で動く
生成系アプリケーション群を個人で作っています。

作っているものより、**境界の引き方**を残すことに時間を使っています。どのライブラリが何を
引き受けないか、なぜその層に置いたか、どの案を採らなかったか——1 つのリポジトリの中だけを
見ていては書けないことは、すべて [**public-docs**](https://github.com/shouni/public-docs) にまとめてあります。

* 🗺️ [**ライブラリリファレンス**](https://github.com/shouni/public-docs/blob/main/docs/libraries.md) — 19 本の境界の地図（担当範囲・隣との線引き・層の位置）
* 🚀 [**アプリケーションリファレンス**](https://github.com/shouni/public-docs/blob/main/docs/applications.md) — 各アプリがどこまで自分で持ち、どこから委譲しているか
* 📐 [**規約**](https://github.com/shouni/public-docs#documents) — URL 命名・ワーカー・README の、リポジトリ横断の決まり
* 📝 [**Zenn @snknsk**](https://zenn.dev/snknsk) — 設計判断の記録

はじめて読むなら [**個人開発エコシステムの全体像**](https://zenn.dev/snknsk/articles/shouni-go-ecosystem-overview) から。

---

### 🏗️ コア基盤ライブラリ

AI の話を一切含まない層。生成系以外のアプリケーションでもそのまま使えます。

| | 担当範囲 |
|---|---|
| [**netarmor**](https://github.com/shouni/netarmor) | 外部通信の安全性だけ。AI もクラウドも知らない。`require` は空 |
| [**go-utils**](https://github.com/shouni/go-utils) | `jobid` / `jst` / `slogctx` / `strlist`。外部依存ゼロ・I/O に触れない・2 つ以上から使われている、が収録条件 |
| [**go-serve-kit**](https://github.com/shouni/go-serve-kit) | 応答を返す側の定型（防御的ヘッダー・表現の出し分け・役割の宣言）。サーバーは持たない |
| [**gcp-kit**](https://github.com/shouni/gcp-kit) | Cloud Run の Web と Cloud Tasks のワーカー。`worker.Lifecycle` がジョブの一生の順序を持つ |
| [**go-http-kit**](https://github.com/shouni/go-http-kit) | SSRF 対策と指数バックオフを組み込んだ `net/http` 互換クライアント |
| [**go-remote-io**](https://github.com/shouni/go-remote-io) | GCS / S3 / ローカルを `Open(ctx, path)` 一本に。成果物の置き場 |
| [**go-web-reader**](https://github.com/shouni/go-web-reader) | `https://` / `gs://` / `s3://` を同じ入口で読む。素材の取得 |
| [**audio**](https://github.com/shouni/audio) | 音声バイナリと日本語の読みだけ |
| [**go-job-kit**](https://github.com/shouni/go-job-kit) | 投入・記録・一覧。成果物のドメインには踏み込まない |
| [**go-notify**](https://github.com/shouni/go-notify) | 実行結果を人へ届ける部分だけ。CLI を持たない |

### 🤖 AI 抽象化ライブラリ

`genai` SDK を公開 API に出さない層。

| | 担当範囲 |
|---|---|
| [**go-prompt-kit**](https://github.com/shouni/go-prompt-kit) | プロンプト管理と、レスポンスの Markdown / JSON → HTML 化 |
| [**go-gemini-client**](https://github.com/shouni/go-gemini-client) | Gemini API と Vertex AI のデュアルバックエンド |
| [**genai-kit**](https://github.com/shouni/genai-kit) | Vertex AI 専用。参照画像は `gs://` をモデル側に解決させる |
| [**gemini-image-kit**](https://github.com/shouni/gemini-image-kit) | 参照画像の取得・再圧縮・キャッシュまで要る画像生成 |
| [**go-character-kit**](https://github.com/shouni/go-character-kit) | キャラクター定義の読み込みと検証だけ |
| [**go-voicevox**](https://github.com/shouni/go-voicevox) | `[]ScriptLine` → 結合済み WAV。保存先は呼び出し側が決める |

### 🎨 生成オーケストレーター

工程の組み立てを持つ層。保存先も通知先も呼び出し側の実装。

| | 担当範囲 |
|---|---|
| [**go-comic-kit**](https://github.com/shouni/go-comic-kit) | キャラクターの一貫性を保ったまま、漫画を工程単位で組み立てる |
| [**go-veo-orchestrator**](https://github.com/shouni/go-veo-orchestrator) | 楽曲構成書から動画カット列を構造化し、Veo へ渡す |
| [**go-review-kit**](https://github.com/shouni/go-review-kit) | Git 差分 → AI レビュー → 公開 → 通知を 1 本のパイプラインに |

---

### 🚀 アプリケーション

Cloud Run / Cloud Tasks 上で動く成果物。認証・セッション・CSRF・OIDC はアプリ側に実装を持たず、
すべて `gcp-kit/auth` に寄せています。

| | 何をするか | 設計の要点 |
|---|---|---|
| [**ap-mv**](https://github.com/shouni/ap-mv) | 楽曲構成書から Veo でミュージックビデオを生成 | **カット 1 本ずつ生成し、ワーカーが自分で投げ直す**。Cloud Tasks の時間上限を越え、途中から再開できる |
| [**ap-story**](https://github.com/shouni/ap-story) | 原稿から章立て・ネーム・パネル・ページを生成 | **台本ゲート**。コマ数が分かる前に画像生成を始めない |
| [**ap-voice**](https://github.com/shouni/ap-voice) | 記事や文書を話者指定のナレーション音声へ | 台本と音声を別工程に。**生成をやり直さず合成だけかけ直せる** |
| [**adk-review**](https://github.com/shouni/adk-review) | Git 差分を AI エージェントにレビューさせる | 読み取り専用ツール 3 種で**差分の外を自分で調べる**。打ち切りは時間ではなく回数 |
| [**ap-music-poc**](https://github.com/shouni/ap-music-poc) | Lyria による音楽生成（PoC） | 非同期化の骨格がそのまま読めることを優先した構成 |

### 🔌 MCP サーバー

| | |
|---|---|
| **ap-mcp**（非公開） | 5 サービス・72 ツールを 1 エンドポイントへ集約するゲートウェイ。説明文は Go のリテラルではなく `{service}/{tool}.md`、annotations は推測ではなく宣言、**confirm ゲートはプレビューそのもの**（設計は [applications.md](https://github.com/shouni/public-docs/blob/main/docs/applications.md)） |
| [**ap-mcp-slack**](https://github.com/shouni/ap-mcp-slack) | サーバーを建てずに stdio で動く Slack の MCP。認証情報に応じてツールの登録自体が変わる |

### 🐍 Python のツール

Cloud Run に乗せず、手元で単体で動かすもの。映像と音の解析は既存の資産が Python 側にあります。

| | |
|---|---|
| [**lyric-video-maker**](https://github.com/shouni/lyric-video-maker) | MP3 とキーフレーム ZIP から**カラオケ字幕付き MP4** を生成。歌詞がプレーンテキストでもタイミングを付けられる（[記事](https://zenn.dev/snknsk/articles/lyric-video-maker)） |
| [**ap-audio-probe**](https://github.com/shouni/ap-audio-probe) | 生成した楽曲が**レシピの指示どおりに鳴っているか**を測る検証ツール。Demucs で伴奏を除いてから測り、閾値は曲ごとの相対で決める。曲の良し悪しは対象外 |

---

### 📝 設計判断の記録

| 記事 | 扱っていること |
|---|---|
| [個人開発エコシステムの全体像](https://zenn.dev/snknsk/articles/shouni-go-ecosystem-overview) | 層の関係と、ライブラリを分けた基準 |
| [エージェントは「差分の外」を読む](https://zenn.dev/snknsk/articles/adk-go-agent-review-practice) | ADK for Go でレビューをエージェントループに作り直した実践 |
| [MCP サーバーを API ラッパーで終わらせない](https://zenn.dev/snknsk/articles/mcp-server-is-not-api-wrapper) | confirm ゲートと、プレビュー専用ツールを作らない理由 |
| [コマ数が分かる前に、画像生成を始めない](https://zenn.dev/snknsk/articles/79fa0d1bbb57e1) | 費用のために工程を二段に割る判断 |
| [Cloud Run で 1 つのイメージを web/worker に分ける](https://zenn.dev/snknsk/articles/cloudrun-web-worker-split) | 役割で依存グラフと権限を切り替える |
| [モデル名を変えるたびに、アプリをフルビルドしていた](https://zenn.dev/snknsk/articles/cloudrun-config-terraform-import) | Cloud Run の設定を Terraform へ移す |
| [NetArmor「凡事徹底」の 4 つの柱](https://zenn.dev/snknsk/articles/netarmor-four-pillars) | SSRF 対策を最下層に置いた理由 |
| [AI の誤読を許さない](https://zenn.dev/snknsk/articles/go-kagome-prompt-defense) | プロンプトを安全な「読み形式」へトランスパイルする |

全 17 本は [zenn.dev/snknsk](https://zenn.dev/snknsk) にあります。

---

<sub>境界の地図と規約は [public-docs](https://github.com/shouni/public-docs)、使い方は各リポジトリの README、API は [pkg.go.dev](https://pkg.go.dev/search?q=github.com%2Fshouni)、内部の設計判断は各リポジトリの CLAUDE.md が持ちます。</sub>

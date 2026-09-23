以下に、Laya README の日本語訳を示します。

---

# Laya: あなたのAIコマンドセンター

![Demo](./laya.gif)

**プロフェッショナルなオーケストレーションのためのリズム。**

Laya は、オープンソースでローカルファーストな AI 通知コマンドセンターです。Slack、Gmail、GitHub、Jira、Notion、Outlook、Calendar の通知を統合し、Ollama や LM Studio を介したローカル LLM、またはご自身の API キーで Claude や GPT などのクラウドモデルを利用できます。プロフェッショナルなツールからのイベントを受信し、LLM 駆動のエージェントによって自律的な調査とアクションのステージングを行い、承認待ちの **Action Card（アクションカード）** として提示します。つまり、通知を開く前に、答えがすでに用意されている状態になります。

**対応バックエンド:**

- Ollama および LM Studio（ローカル LLM）
- Claude モデル（Anthropic）
- GPT モデル（OpenAI）
- Gemini モデル（Google）
- Llama モデルおよび LiteLLM 経由の OpenAI 互換エンドポイント

**統合対象:**

- Gmail
- Slack
- GitHub
- Bitbucket
- Jira
- Linear
- Notion
- Outlook（メールおよびカレンダー）
- Google Calendar

## 仕組み

```
あなたのツール（Jira、Slack、Gmail、Bitbucket、Calendar）
         |
         v
      n8n（ローカル Node.js） -- イベントを正規化
         |
         v
   Laya Engine（Python） -- 分類、調査、ステージング
         |
         v
    Laya UI（Tauri + Svelte） -- 承認または却下できる Action Cards
         |
         v
      n8n -- 承認されたアクションを実行（PR の作成、返信の送信など）
```

## 主な機能

- **マルチペルソナブレイン:** イベントを専門の AI ペルソナ（エンジニア、コミュニケーション、オペレーション、営業、人事、財務）にルーティングし、ドメイン固有のツールとプロンプトを使用。すべての通知を AI が優先順位付けします。
- **カードワークスペース:** 複雑なタスク（バグ修正、コードレビュー）のためのエージェントワークフロー。複数の承認ステップを通じて、コーディングエージェント（Claude Code、Gemini CLI、Codex、Pi CLI）と協働できるインタラクティブなワークスペースです。
- **カードリサーチ:** 任意のカードに対してオンデマンドの深い調査セッションを開始。コーディングエージェントがウェブ検索、セマンティックコンテキスト、サンドボックス化されたファイルアクセスを使って調査します。
- **エージェント推論バックエンド:** 分類・統合パイプラインを API キーではなく、インストール済みの CLI エージェント自身のクォータで実行。任意のステージで `agent/<id>/<model>` 形式のモデルを選択できます（Claude Code、Codex、Gemini、Pi）。Claude Code はネイティブに JSON スキーマを強制し、他のエージェントはベストエフォートのスキーマ＋リトライを使用します。
- **スペース:** イベントソースをグループ化し、スペースごとにモデルや API キー設定を行える、ユーザー定義のコンテキストです。
- **コンテキスト関連付け:** セマンティック類似性と LLM による確認を使用して、関連するカードをプラットフォーム間で自動的にリンク。修正内容から学習し、時間とともにグループ化の精度を向上させます。
- **クロスプラットフォームメモリ:** エンティティ解決により、Jira の「BUG-1234」を Bitbucket の「PR-891」、Slack の「支払いバグ」に紐付けます。
- **デイリーブリーフィング:** 一晩の活動、保留中のカード、今日のカレンダーをコンテキスト付きでまとめた朝のサマリーです。
- **アナリティクスダッシュボード:** 処理されたイベント数、節約時間、LLM コスト（機能およびパイプラインステップ別）、スループットの推移、承認率を追跡します。
- **予算追跡:** 機能別（Pulse、Omni、Chat、Coherence）の LLM コストを監視し、月次上限を設定。上限に達すると自動的に一時停止します。エージェント推論バックエンド使用時は、別のウィンドウベースの使用予算により、エージェントのローリングクォータが上限に近づくと取り込みを一時停止し、ウィンドウがリセットされると自動で再開します。
- **チャットサイドバー:** イベント、プロジェクト、コンテキストについて Laya に質問したり、会話からフィルタ・分類・処理ルールを直接作成・編集・削除したりできます。
- **ハイブリッド検索:** チャットおよび Coherence の取得では、ローカルベクトル検索（ChromaDB）と SQLite FTS5（`cards_fts`/`events_fts`）上の語彙的 BM25 ランキングを Reciprocal Rank Fusion で統合し、セマンティックおよび完全一致キーワードの両方がヒットします。
- **Coherence:** クロスプラットフォームのエンティティ検索により、ハイブリッドローカル検索を使用して人物、チケット、PR をすべてのプラットフォーム間で追跡し、AI 生成のナラティブを提供します。
- **Egress（送信）:** メール、Slack メッセージ、PR コメントなどの送信アクションを、送信前プレビュー付きで Laya から直接実行できます。
- **Omni:** 「今、自分はどこにいるのか？」に答えるローリングクロスプラットフォームサマリー。4 つの時間層（Attention、Recent、Period、Milestone）と段階的な AI 圧縮を備えています。
- **ブックマーク:** 重要なカードをピン留めし、日付やステータスに関係なくすばやくアクセスできます。
- **分類学習:** 優先度・ペルソナの修正からルールを抽出し、自動的に分類精度を向上させます。
- **コンテキスト学習:** リンク・アンリンクの修正から自然言語の **コンテキストルール** を抽出し、コンテキスト関連付けの精度を自動的に向上させます。学習済みルールと手動ルールは設定画面で閲覧・編集可能で、大規模なルールセットは自動的に LLM による統合が行われます。
- **処理ルール:** 受信カードに対して動作する、オプションで AI 評価可能な自動ルール（タグ付け、ルーティング、エージェント実行、egress 送信）。すべての発火は、結果（成功・エラー・スキップ）と理由を含む検索可能な **発火ログ** に記録されます。
- **監査とエクスポート:** デッドイベント、取り込みエラー、ルールでフィルタされたイベントを一箇所で確認。失敗の再試行やクリア、フィルタされたイベントや監査ログを任意の期間で JSON としてエクスポートできます。
- **オンプレミスリポジトリ:** セルフホスト型 Bitbucket Server / Data Center および GitHub Enterprise は、リポジトリごとの `host` フィールドでサポートされます（空またはクラウドドメインの場合はクラウドとして扱われます）。
- **デッドイベントリカバリ:** 失敗したイベントはエラーコンテキスト付きで追跡され、監査ログから手動で再試行できます。
- **プライバシー対応:** 3 段階のデータ分類と、クラウド／ローカル処理の選択肢があります。

## 技術スタック

| レイヤー | 技術 |
|---|---|
| デスクトップシェル | Tauri v2（Rust） |
| フロントエンド | Svelte 5（runes）+ Skeleton UI + Tailwind CSS v4 |
| バックエンド | Python 3.10+ / FastAPI / asyncio |
| LLM インターフェース | LiteLLM（Anthropic、OpenAI、Google、Ollama 対応） |
| 統合ゲートウェイ | n8n（ローカル Node.js、ポート 45678） |
| 構造化ストレージ | SQLite（aiosqlite 経由で非同期、WAL モード） |
| ベクトルストレージ | ChromaDB（埋め込み PersistentClient） |
| 埋め込み | ONNX（ChromaDB 組み込み）または sentence-transformers（オプション） |
| コーディングエージェント | Claude Code / Gemini CLI / OpenAI Codex CLI / Pi CLI（ワークスペースエージェントおよび推論バックエンドとして使用可能） |

## プロジェクト構成

```
laya/
├── engine/                  # Python FastAPI バックエンド
│   ├── laya/
│   │   ├── main.py          # エントリーポイント（uvicorn サーバー :8420）
│   │   ├── config.py        # 設定、パス、エージェント検出
│   │   ├── api/             # REST + WebSocket エンドポイント（27 個のルーター）
│   │   ├── db/              # SQLite（+ FTS5）+ ChromaDB + 70 のマイグレーション
│   │   ├── pipeline/        # イベント処理（ingest → route → stage → emit → trace → learn → context_learn → omni）
│   │   ├── llm/             # LiteLLM クライアント、エージェント推論バックエンド、プロンプト、ツール
│   │   ├── agents/          # コーディングエージェントアダプター（Claude、Gemini、Codex、Pi）
│   │   ├── workers/         # マルチペルソナ LLM ワーカー（engineer、comms、ops、sales、hr、finance）
│   │   ├── egress/          # 送信アクション実行（9 プラットフォーム）
│   │   ├── integrations/    # n8n ブートストラップおよびクライアント
│   │   └── security/        # OS キーチェーン統合
│   ├── requirements.txt     # コア Python 依存関係
│   └── requirements-ml.txt  # オプション: torch + sentence-transformers
│
├── ui/                      # SvelteKit + Tauri デスクトップアプリ
│   ├── src/                 # Svelte 5 フロントエンド（runes 構文）
│   │   ├── routes/          # ページ（feed、coherence、dashboard、settings、workspace、omni）
│   │   ├── lib/             # コンポーネント、API クライアント、ストア
│   │   ├── app.css          # Tailwind v4 + テーマシステム
│   │   └── app.html
│   ├── src-tauri/           # Rust/Tauri シェル
│   │   ├── src/
│   │   │   ├── lib.rs       # Tauri セットアップ、コマンド、ヘルスポーリング、トレイ
│   │   │   ├── sidecar.rs   # Python venv ライフサイクルおよびエンジン起動
│   │   │   └── n8n.rs       # n8n プロセス管理
│   │   ├── tauri.conf.json  # Tauri 設定（リソース、アイコン、ウィンドウ）
│   │   └── resources/       # バンドルされたエンジンソース（本番ビルド）
│   ├── package.json
│   └── svelte.config.js     # 静的アダプター（SPA モード）
│
├── n8n/
│   └── workflows/           # 統合ワークフロー（JSON、約 21 ファイル: プラットフォームごとの取り込み + 実行）
│
├── scripts/
│   ├── setup-dev.sh         # 初回開発環境セットアップ
│   ├── dev.sh               # エンジン + Tauri 開発サーバーの起動
│   ├── build.sh             # 本番ビルド
│   └── update_icons.sh      # アイコン生成
│
├── landing/                 # ランディングページ
└── docs/                    # アーキテクチャおよび設計ドキュメント
```

## インストール

Laya を最速で試すには、プリビルドリリースを使用してください。ツールチェーンは不要です。

1. [**Releases ページ**](https://github.com/aayushch/laya/releases) を開き、ご利用のプラットフォーム用のインストーラーをダウンロードします。

   | プラットフォーム | ダウンロード |
   |----------|----------|
   | macOS | `.dmg`（ユニバーサル: Apple Silicon + Intel） |
   | Windows | `.msi` または `.exe` |
   | Linux | `.deb` または `.AppImage` |

2. インストールして起動します。リリースビルドの *実行* には、Python や Node、Rust をインストールする必要はありません。初回起動時に、互換性のある Python（3.10+）および Node.js（20+）がマシン上に既に存在するか確認し、見つかった場合はそれらを使用します。見つからない場合は、バンドルされたランタイムを自動的にプロビジョニングします。いずれにせよ、ローカルの n8n インスタンスは `~/.laya/` 以下にセットアップされます。
3. API キー（Anthropic、OpenAI、Google など）を追加するか、ローカルの Ollama / LM Studio エンドポイントを指定し、設定画面からツールを接続します。

> **macOS:** リリースビルドは署名済みなので、通常通りダブルクリックで起動できます。

ソースからビルドしたり、エンジインをハックしたり、貢献したい場合は、以下の **Development** を参照してください。

## 開発

### 前提条件

3 つのランタイムがインストールされている必要があります。それぞれの入手方法は以下の通りです。

#### Python 3.10+

- **macOS:** `brew install python@3.12`（または [python.org](https://www.python.org/downloads/) からダウンロード）
- **Ubuntu/Debian:** `sudo apt install python3 python3-venv python3-pip`
- **Windows:** [python.org](https://www.python.org/downloads/) からダウンロード（インストール時に「Add to PATH」をチェック）

確認: `python3 --version`

#### Node.js 20+

- **すべてのプラットフォーム:** [nodejs.org](https://nodejs.org/) からダウンロード（LTS 推奨）、または [nvm](https://github.com/nvm-sh/nvm) / [fnm](https://github.com/Schniz/fnm) などのバージョンマネージャーを使用
- **macOS:** `brew install node`
- **Ubuntu/Debian:** `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt install -y nodejs`

確認: `node --version && npm --version`

#### Rust ツールチェーン

[rustup](https://rustup.rs/) 経由でインストール:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

確認: `cargo --version`

#### プラットフォーム固有の依存関係

**macOS:**

```bash
xcode-select --install
```

**Linux（Ubuntu/Debian）:**

Tauri v2 には、GTK、WebKit、アプリインジケーター対応のシステムライブラリが必要です:

```bash
sudo apt install -y libwebkit2gtk-4.1-dev libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev patchelf
```

### トラブルシューティング

<details>
<summary><strong>Linux: Tailwind CSS クラスが欠落している、またはスタイルが更新されない</strong></summary>

Linux のデフォルトの inotify ファイルウォッチャー上限（65,536）は、このプロジェクトにとって低すぎる可能性があります。Vite はソースファイルを監視する必要があり、Rust の `target/` ディレクトリが大部分のクォータを消費するため、Tailwind CSS がユーティリティクラスの生成に暗黙的に失敗することがあります。上限を増やしてください:

```bash
# 即時（再起動でリセット）
echo 524288 | sudo tee /proc/sys/fs/inotify/max_user_watches

# 永続化
echo 'fs.inotify.max_user_watches=524288' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

</details>

### セットアップ

```bash
scripts/setup-dev.sh
```

このスクリプトは以下を行います:

1. `python3`、`node`、`npm`、`cargo` が利用可能か確認
2. `engine/.venv/` に Python 仮想環境を作成し、`engine/requirements.txt` から依存関係をインストール
3. UI の npm パッケージをインストール（`ui/node_modules/`）
4. n8n をローカルの npm パッケージとして `~/.laya/n8n_module/` にインストール
5. `~/.laya/data/` および `~/.laya/logs/` にデータディレクトリを作成

### ローカルでの実行

```bash
scripts/dev.sh
```

これにより、2 つのプロセスが起動します:

1. **Python エンジン** -- `python -m laya.main`（ホットリロード付き）http://127.0.0.1:8420
2. **Tauri 開発サーバー** -- `npx @tauri-apps/cli dev`（Vite を http://localhost:5173 で起動し、Tauri ウィンドウを開きます）

n8n は Tauri アプリによって自動的に管理されます。起動時に開始（ポート 45678）、終了時に停止します。

> **注:** エンジンが「Address already in use」で失敗する場合、古いエンジンプロセスがポート 8420 を占有している可能性があります。エンジンは起動時に自動的にそれを終了させようとします。

### 設定

初回起動時に、エンジンは `~/.laya/` に設定ファイルを作成します:

| ファイル | 目的 |
|------|------|
| `settings.json` | モデル、エージェントパス、プライバシー設定、パイプラインパラメータ |
| `team.json` | チームメンバーのコンテキスト |
| `rules.json` | イベントフィルタリングルール |
| `repos.json` | Git リポジトリのパスとメタデータ |

API キー（Anthropic、OpenAI、Google など）は OS のキーチェーンに安全に保存され、設定 UI から構成できます。

エンジンのログレベルはデフォルトで `INFO` です。**Settings → Data → Engine Log Level**（`DEBUG` / `INFO` / `WARNING` / `ERROR`）から詳細度を変更できます。これは `settings.json` の `logging.level` キーにマッピングされ、再起動不要で即座に適用されます。`WARNING` に設定すると、警告とエラーのみが記録され、ログを小さく保てます。単一の実行で上書きするには `LAYA_LOG_LEVEL` 環境変数を使用します。これは設定より優先され、uvicorn のリクエストログレベルも設定します。

### カスタムプロンプト

Laya の AI パイプラインは、すべてのステージ（ルーティング、ステージング、要約、チャットなど）でシステムプロンプトを使用します。すべてのプロンプトには適切なデフォルトが用意されていますが、`~/.laya/prompts/` にファイルを配置することで、いずれも上書きできます:

```bash
mkdir -p ~/.laya/prompts

# ルータープロンプトを上書き（イベント分類を制御）
vim ~/.laya/prompts/router.md

# ワーカーペルソナを上書き
vim ~/.laya/prompts/engineer.md

# 再起動せずにリロード
curl -X POST http://127.0.0.1:8420/prompts/reload
```

利用可能なプロンプトファイル: `router.md`、`stager.md`、`omni.md`、`group_summary_initial.md`、`group_summary_rolling.md`、`briefing.md`、`summarizer.md`、`summarizer_status_change.md`、`engineer.md`、`comms.md`、`sales.md`、`hr.md`、`ops.md`、`finance.md`、`chat.md`、`chat_title.md`、`chat_polish.md`、`learner.md`、`context_learner.md`、`trace_narrative.md`、`trace_summary.md`、`trace_filter.md`

カスタムプロンプトは、そのステージの組み込みデフォルトを完全に置き換えます。ファイルが削除されると、自動的にハードコードされたデフォルトが使用されます。エンジンはこのディレクトリ内のファイルを作成または変更することはありません。現在どのプロンプトが上書きされているかは `GET /prompts` で確認できます。

### データストレージ

| ストア | 場所 | 目的 |
|-------|----------|------|
| SQLite | `~/.laya/data/laya.db` | イベント、カード、ワークスペース、スペース、トレース、egress、チャット |
| ChromaDB | `~/.laya/data/chroma/` | セマンティック検索用のベクトル埋め込み |
| n8n | `~/.laya/n8n/` | ワークフローデータ、認証情報（暗号化） |
| ログ | `~/.laya/logs/` | `engine.log` -- ローテーションするエンジンログ（10 MB × 5 ファイル）、詳細度は上記のログレベルで設定。`engine-stdout.log` および `n8n.log` も同様にローテーション（10 MB × 3 ファイル）。 |

## 配布用ビルド

Laya は Python エンジンソースを Tauri アプリにバンドルします。初回起動時に、アプリは `~/.laya/venv/` に Python 仮想環境を作成し、依存関係を自動的にインストールします。エンドユーザーのマシンには、アプリが管理するもの以外の Python インストールは不要です。

### ビルドコマンド

```bash
scripts/build.sh
```

これは以下の 2 つを行います:

1. **エンジンソースのバンドル** -- `engine/laya/`、`requirements.txt`、`requirements-ml.txt`、`n8n/workflows/` を `ui/src-tauri/resources/engine/` にコピー
2. **Tauri アプリのビルド** -- Rust シェルをコンパイルし、SvelteKit フロントエンドをバンドルし、すべてをプラットフォームネイティブなインストーラーにパッケージング

### ビルドオプション

```bash
scripts/build.sh                                   # 現在のプラットフォーム向けにビルド
scripts/build.sh --target x86_64-apple-darwin      # Intel Mac 向けにクロスコンパイル
scripts/build.sh --universal                       # ユニバーサルバイナリ（arm64 + x86_64）
scripts/build.sh --sign "Developer ID App: ..."    # macOS コード署名
scripts/build.sh --skip-engine                     # エンジンバンドルをスキップ（前回のものを再利用）
```

### ビルド出力

| プラットフォーム | 形式 | パス |
|----------|--------|------|
| macOS | `.app` | `ui/src-tauri/target/release/bundle/macos/Laya.app` |
| macOS | `.dmg` | `ui/src-tauri/target/release/bundle/dmg/Laya_0.1.0_<arch>.dmg` |
| Windows | `.msi` | `ui/src-tauri/target/release/bundle/msi/` |
| Windows | `.exe` | `ui/src-tauri/target/release/bundle/nsis/` |
| Linux | `.deb` | `ui/src-tauri/target/release/bundle/deb/` |
| Linux | AppImage | `ui/src-tauri/target/release/bundle/appimage/` |

> **注:** macOS ビルドはデフォルトで未署名です。未署名のアプリは Gatekeeper をトリガーするため、ユーザーは右クリック → 開くで回避する必要があります。Apple Developer ID を `--sign` に渡すと、署名済みビルドを生成できます。

## ドキュメント

アーキテクチャおよび設計ドキュメントは [`docs/`](./docs/) にあります:

- [**System Architecture**](./docs/architecture.md) -- コンポーネント図とサービス説明
- [**Event Schema**](./docs/event-schema.md) -- Laya Event スキーマ仕様
- [**API Contracts**](./docs/api-contracts.md) -- REST、WebSocket、およびサービス間 API 定義
- [**Database Schema**](./docs/database-schema.md) -- SQLite テーブル（FTS5 含む）、ChromaDB コレクション、およびマイグレーション
- [**Project Structure**](./docs/project-structure.md) -- リポジトリ構成と設定ファイルスキーマ
- [**Tuning Parameters**](./docs/tuning-parameters.md) -- 上書き可能なパイプライン、取得、エージェント予算設定
- [**n8n Data Persistence**](./docs/n8n-data-persistence.md) -- n8n ワークフローデータと認証情報の保存方法
- [**Decision Log**](./docs/decision-log.md) -- 根拠を含むアーキテクチャ上の決定

より深い設計ドキュメント（egress、AI 処理ルール、OAuth アプリ配布、パイプライフライフサイクル）は [`engine/docs/`](./engine/docs/) にあります。

# ローカルの実装に関して。低スペ
**結論から言うと、Radeon 6800 XT で Laya は動作する可能性が高いです。** ただし、いくつか注意点があります。

---

### Laya のモデルサイズ

Laya は約 **4.2億パラメータ**（ModernBERT-large backbone + decision head）のモデルです。<source-chip title="Hugging Face - Laya" url="https://huggingface.co/convaiinnovations/laya" />

入力は1問あたり **512トークン** で、GPU上での1問の推論時間は約 **38ms**、10問のバッチ処理でも **156ms** 程度です。非常に軽量なモデルです。

---

### Radeon 6800 XT の状況

**VRAM（16GB）に関しては全く問題ありません。** 4.2億パラメータのモデルなら、FP16でも数百MB〜1GB程度のVRAMしか使いません。

問題になるのは **ROCm（AMDのGPUコンピューティングプラットフォーム）の互換性** です。

| 項目 | 状況 |
|------|------|
| 公式ROCmサポート | 6800 XT（gfx1030）は**公式サポート対象外** |
| 実際の動作 | コミュニティでは**動作報告が多数あり** |
| 必要な対応 | `HSA_OVERRIDE_GFX_VERSION=10.3.0` などの環境変数設定が必要な場合あり |

<source-chip title="ROCm GitHub Issue" url="https://github.com/RadeonOpenCompute/ROCm/issues/1786" /><source-chip title="CodeGenes - 6800XT PyTorch" url="https://www.codegenes.net/blog/6800xt-pytorch/" />

---

### セットアップの流れ（Linux推奨）

1. **ROCm をインストール**（Linux上）
2. **PyTorch（ROCm版）をインストール**
3. **`pip install laya`**
4. 必要に応じて環境変数を設定して実行

```bash
# 環境変数の例（必要な場合）
export HSA_OVERRIDE_GFX_VERSION=10.3.0

python -c "
import laya
agent = laya.load('convaiinnovations/laya')
# ... 推論処理
"
```

---

### まとめ

- **VRAM的には余裕**: 16GBで4.2億パラメータは全く問題なし
- **推論速度も速い**: 38ms/問程度で動く見込み
- **注意点**: 6800 XTはROCm公式サポート外なので、環境変数の設定やLinux環境が必要になる場合があります
- **WindowsよりLinux推奨**: ROCmの consumer GPU サポートは Linux が基本です

# 似ているアーキテクチャ
https://github.com/hiroki-abe-58/sokudan
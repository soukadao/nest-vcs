# Nest

> **試験的なリポジトリです。** 仕様・API は予告なく変更される可能性があります。

Git/GitHub を置き換える次世代バージョン管理システム。CRDT ベースのリアルタイム協調編集により、複数エージェント・分離環境での同時開発を実現します。

## 特徴

- **CRDT ベース** — ファイル・Issue・Review・ドキュメント全てが CRDT で管理され、自動的に収束
- **ブランチ** — Git のブランチと同様の操作感。内部では複数ワーカーが同一ブランチ上で並行動作し、マルチエージェント同時作業を実現
- **統合プロジェクト管理** — Issue、Review（PR相当）、ドキュメントがリポジトリに内蔵。GitHub 不要
- **リアルタイム同期** — WebSocket による操作ストリーミング
- **マルチエージェント対応** — 分離コンテナ上の複数 AI エージェントが同一ブランチを同時編集可能

## アーキテクチャ

```
nest-crdt        CRDT エンジン (HLC, Text/RGA, LWW, Map, Set, Sequence)
  ↓
nest-protocol    通信プロトコル・API 型定義
  ↓
nest-core        コアライブラリ (Repository, Entity, Snapshot, Storage)
  ↓
nest-cli         CLI クライアント (nest コマンド)
nest-server      Relay サーバー (REST API + WebSocket)
nest-ui          Web UI (GitHub 的な画面)
```

### CRDT エンジン (nest-crdt)

| 型 | 用途 |
|----|------|
| HLC (Hybrid Logical Clock) | 分散因果順序の追跡 |
| TextCRDT (RGA) | 文字レベルの協調テキスト編集 |
| LWW-Register | 最終書き込み勝ちのスカラー値 |
| MapCRDT | ファイルツリー等のキーバリュー |
| SetCRDT (OR-Set) | ラベル・アサイニー等の集合 |
| SequenceCRDT | コメント等の追記専用リスト |

## コマンド一覧

### リポジトリ

| コマンド | 説明 |
|---------|------|
| `nest init [path] --user <name>` | リポジトリを初期化（`.nest/` を作成） |
| `nest status` | 現在のブランチとユーザーを表示 |
| `nest log` | Snapshot 履歴を表示 |
| `nest diff [from] [to]` | Snapshot 間の差分を表示 *(未実装)* |
| `nest record` | ファイル変更を検出し CRDT 操作として記録 |

### ブランチ

| コマンド | 説明 |
|---------|------|
| `nest branch list` | ブランチ一覧（現在のブランチに `*` 表示） |
| `nest branch create <name>` | ブランチを作成 |
| `nest branch switch <name>` | ブランチを切り替え |

### Snapshot

| コマンド | 説明 |
|---------|------|
| `nest snapshot create -m "<msg>"` | Snapshot を作成（Git の commit に相当） |
| `nest snapshot list` | Snapshot 一覧 |

### Issue

| コマンド | 説明 |
|---------|------|
| `nest issue create "<title>" [-b "<body>"]` | Issue を作成 |
| `nest issue list` | Issue 一覧 |
| `nest issue show <id>` | Issue 詳細（ラベル・アサイニー・コメント含む） |
| `nest issue close <id>` | Issue をクローズ |
| `nest issue comment <id> "<body>"` | Issue にコメント |

### Review

| コマンド | 説明 |
|---------|------|
| `nest review create "<title>" --source <branch> --target <branch>` | Review を作成 *(スタブ)* |
| `nest review list` | Review 一覧 *(スタブ)* |

### Document

| コマンド | 説明 |
|---------|------|
| `nest doc create "<title>" [-b "<body>"]` | ドキュメントを作成（`-b` 省略時は `$EDITOR` 起動） |
| `nest doc list` | ドキュメント一覧 |
| `nest doc show <id>` | ドキュメントを表示 |
| `nest doc edit <id>` | `$EDITOR` でドキュメントを編集 |

### リモート / 同期

| コマンド | 説明 |
|---------|------|
| `nest remote add <name> <url>` | リモートを追加 |
| `nest remote list` | リモート一覧 |
| `nest remote remove <name>` | リモートを削除 |
| `nest sync [remote]` | リモートと同期 *(未実装)* |

## サーバー

```bash
nest-server    # REST API + WebSocket (デフォルト: 0.0.0.0:3141)
nest-ui        # Web UI プロキシ  (デフォルト: 0.0.0.0:3142)
```

### API エンドポイント

| メソッド | パス | 説明 |
|---------|------|------|
| GET/POST | `/api/repos` | リポジトリ一覧 / 作成 |
| GET | `/api/repos/{repo}` | リポジトリ詳細 |
| GET | `/api/repos/{repo}/files` | ファイルツリー |
| GET | `/api/repos/{repo}/files/{*path}` | ファイル内容 |
| GET/POST | `/api/repos/{repo}/issues` | Issue 一覧 / 作成 |
| GET | `/api/repos/{repo}/issues/{id}` | Issue 詳細 |
| GET | `/api/repos/{repo}/reviews` | Review 一覧 |
| GET | `/api/repos/{repo}/streams` | Stream 一覧（内部） |
| GET | `/api/repos/{repo}/groups` | ブランチ一覧 |
| GET | `/api/repos/{repo}/snapshots` | Snapshot 一覧 |
| WS | `/api/repos/{repo}/sync` | WebSocket リアルタイム同期 |

## ビルド

```bash
cargo build                  # 全 crate をビルド
cargo test                   # テスト実行 (15 tests)
cargo run --bin nest         # CLI を実行
cargo run --bin nest-server  # サーバーを起動
cargo run --bin nest-ui      # Web UI を起動
```

## ドキュメント

| ファイル | 内容 |
|---------|------|
| [DESIGN.md](DESIGN.md) | 設計思想・コンポーネントアーキテクチャ・データフロー |
| [SPEC.md](SPEC.md) | 技術仕様・CRDT 定義・API 詳細・実装フェーズ |
| [KNOWLEDGE.md](KNOWLEDGE.md) | 技術的知見・改善記録 |

---

## ユースケース

### 1. 個人開発 — Git と同じ感覚で使う

Git の `init → add → commit` に対応する Nest の基本フロー。

```bash
# リポジトリを作成
mkdir my-project && cd my-project
nest init --user "alice"

# コードを書く
echo 'fn main() { println!("Hello"); }' > main.rs

# 変更を記録（Git の add + diff 検出を自動で行う）
nest record
#=> Recorded 35 operations from 1 file(s).

# 区切りの良いところで Snapshot を作成（Git の commit に相当）
nest snapshot create -m "Hello World を実装"

# 履歴を確認
nest log
#=> snapshot snap-0000019c...
#=> Author: alice
#=>     Hello World を実装

# さらにコードを編集して繰り返す
echo 'fn main() { println!("Hello, Nest!"); }' > main.rs
nest record
nest snapshot create -m "メッセージを変更"
```

### 2. 機能ブランチ — ブランチで分岐・統合

Git と同じ感覚でブランチを作成・切り替えする。

```bash
# main 上で作業中
nest status
#=> On branch: main

# 新機能用のブランチを作成
nest branch create feature-login

# 切り替え
nest branch switch feature-login

# 機能を実装
echo 'fn login() { /* ... */ }' > auth.rs
nest record
nest snapshot create -m "ログイン機能を実装"

# Issue を作成して作業を紐付け
nest issue create "ログイン機能" -b "OAuth 対応のログインを追加する"

# 完了したら Review を作成（Pull Request に相当）
nest review create "ログイン機能の追加" --source feature-login --target main

# main に戻る
nest branch switch main
```

### 3. チーム開発 — サーバー経由でリアルタイム同期

中央サーバーを立てて、チームメンバー間で変更をリアルタイムに同期する。

```bash
# ---- サーバー側 ----
cargo run --bin nest-server
#=> Nest server listening on 0.0.0.0:3141

# Web UI も起動（ブラウザで Issue や Review を管理）
cargo run --bin nest-ui
#=> Nest UI listening on http://0.0.0.0:3142

# ---- 開発者 Alice ----
nest init --user "alice"
nest remote add origin http://server:3141

# コードを編集・記録
nest record
nest sync origin   # サーバーに変更を送信

# ---- 開発者 Bob ----
nest init --user "bob"
nest remote add origin http://server:3141
nest sync origin   # サーバーから変更を取得

# Bob がコードを編集・記録
nest record
nest sync origin   # 送信。Alice の変更と CRDT で自動マージ
```

### 4. マルチ AI エージェント — 分離コンテナでの並行開発

3台の AI エージェントが別々のコンテナから同じコードベースを同時に編集する。
各エージェントは内部的に独立した操作ログを持ち、CRDT により衝突なく統合される。

```bash
# ---- 共通: サーバーを起動 ----
cargo run --bin nest-server

# ---- コンテナ A: フロントエンド担当 ----
nest init --user "agent-frontend"
nest remote add origin http://server:3141
nest sync origin

# React コンポーネントを生成
# ... エージェントがファイルを書き換え ...
nest record    # 変更を CRDT 操作に変換
nest sync origin   # サーバーに送信

# ---- コンテナ B: バックエンド担当 ----
nest init --user "agent-backend"
nest remote add origin http://server:3141
nest sync origin

# API エンドポイントを実装
# ... エージェントがファイルを書き換え ...
nest record
nest sync origin

# ---- コンテナ C: テスト担当 ----
nest init --user "agent-test"
nest remote add origin http://server:3141
nest sync origin

# テストコードを生成
# ... エージェントがファイルを書き換え ...
nest record
nest sync origin

# ---- 結果 ----
# 3つのエージェントの変更は全て同じブランチ (main) に属し、
# CRDT により自動的にマージされる。
# 同じファイルの同じ行を編集しても、文字レベルで衝突解決される。
# Git のようなコンフリクト解消作業は不要。
```

### 5. Issue 駆動開発 — Issue → 実装 → Review → マージ

GitHub のワークフローを Nest 内で完結させる。外部サービス不要。

```bash
# Issue を起票
nest issue create "検索機能を追加" -b "全文検索を実装する。対象はドキュメントとコード。"

# 作業用ブランチを作成して実装開始
nest branch create feature-search
nest branch switch feature-search

# 実装
echo 'pub fn search(query: &str) -> Vec<Result> { todo!() }' > search.rs
nest record
nest snapshot create -m "検索機能の骨組みを実装"

# 進捗を Issue にコメント
nest issue comment issue-1 "基本構造を実装しました。次はインデックス作成を行います。"

# 実装完了後、Review を作成（Pull Request に相当）
nest review create "検索機能の追加" --source feature-search --target main

# Issue をクローズ
nest issue close issue-1
```

### 6. ドキュメント管理 — コードと設計書を一元管理

設計書や仕様書をリポジトリ内に CRDT ドキュメントとして保存する。
Markdown ファイルとは異なり、複数人が同時編集してもコンフリクトしない。

```bash
# 設計ドキュメントを作成（-b 省略時は $EDITOR が起動）
nest doc create "API 設計書" -b "## エンドポイント一覧\n..."
nest doc create "アーキテクチャ概要"

# ドキュメント一覧
nest doc list
#=> [doc-1] API 設計書 (by alice)
#=> [doc-2] アーキテクチャ概要 (by alice)

# ドキュメントを表示
nest doc show doc-1

# $EDITOR でドキュメントを編集
nest doc edit doc-1

# Web UI (http://localhost:3142) からもドキュメントの閲覧・編集が可能
```

---

## Git との概念対応表

| Git | Nest | 違い |
|-----|------|------|
| `git init` | `nest init` | `.nest/` ディレクトリを作成 |
| `git add` + `git commit` | `nest record` + `nest snapshot create` | `record` がファイル差分を自動検出し CRDT 操作に変換 |
| `git branch` / `git checkout` | `nest branch create` / `nest branch switch` | 同一ブランチ上で複数人が同時作業可能 |
| commit | Snapshot | ブランチ上の操作ログの特定時点を記録 |
| `git merge` | 自動マージ (CRDT) | テキストは文字レベルで自動収束。コンフリクトマーカーなし |
| `git push` / `git pull` | `nest sync` | 双方向同期。CRDT 操作の交換 |
| GitHub Issues | `nest issue` | リポジトリに内蔵。外部サービス不要 |
| GitHub Pull Request | `nest review` | リポジトリに内蔵。ブランチ間のレビュー |
| GitHub Wiki | `nest doc` | リポジトリに内蔵。CRDT による同時編集対応 |

## 実装状況

| 機能 | 状態 |
|------|------|
| CRDT エンジン (HLC, Text, LWW, Map, Set, Sequence) | ✅ 完了 (13 tests) |
| ストレージ (Blob, State, Stream) | ✅ 完了 (2 tests) |
| リポジトリ操作 (init, status, record) | ✅ 完了 |
| ブランチ管理 (create, list, switch) | ✅ 完了 |
| Snapshot (create, list, log) | ✅ 完了 |
| Issue (create, list, show, close, comment) | ✅ 完了 |
| Document (create, list, show, edit) | ✅ 完了 |
| Remote (add, list, remove) | ✅ 完了 |
| REST API サーバー | ✅ 完了 |
| WebSocket ハンドラ | ✅ 完了 |
| Diff (snapshot 間差分) | 🚧 未実装 |
| Review (create, list, approve, merge) | 🚧 スタブ |
| Sync (WebSocket 同期クライアント) | 🚧 未実装 |
| Web UI | 🚧 スケルトン |

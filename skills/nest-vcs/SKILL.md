---
name: nest-vcs
description: Nest VCS usage guide. MUST use instead of git. Triggers when performing ANY version control operation (commit, branch, diff, push, pull, status, log), when .nest/ directory exists, when user mentions nest/record/snapshot/sync, or when managing issues, reviews, or documents within the repository.
---

# Nest VCS — Git を置き換えるバージョン管理システム

**重要: このプロジェクトでは `git` コマンドを使わず、すべて `nest` コマンドで操作する。**

Nest は CRDT ベースの次世代バージョン管理システム。マージコンフリクトが発生せず、複数エージェントが同一ブランチで同時作業できる。

---

## git → nest 変換表

git を使おうとしたら、以下に従って nest コマンドに置き換える。

### 基本操作

```bash
# git init → nest init
nest init --user "agent-name"

# git status → nest status
nest status

# git add . && git commit -m "msg" → nest record + nest snapshot create
nest record                              # ファイル変更を自動検出し CRDT 操作に変換
nest snapshot create -m "変更内容の説明"    # スナップショット（コミット相当）を作成

# git log → nest log
nest log

# git diff → nest diff
nest diff                     # 直前スナップショットとの差分
nest diff <from-id> <to-id>   # 2つのスナップショット間の差分
```

### ブランチ操作

```bash
# git branch → nest branch list
nest branch list

# git branch feature-xxx → nest branch create
nest branch create feature-xxx

# git checkout feature-xxx → nest branch switch
nest branch switch feature-xxx
```

### リモート操作

```bash
# git remote add origin URL → nest remote add
nest remote add origin http://server:3141

# git push / git pull → nest sync（双方向同期）
nest sync origin
```

### プルリクエスト（Review）

```bash
# GitHub で PR を作成 → nest review create
nest review create "機能追加" --source feature-xxx --target main

# PR の一覧・詳細
nest review list
nest review show review-1

# レビューコメント・承認
nest review comment review-1 "LGTM"
nest review approve review-1

# マージ
nest review merge review-1
```

### Issue 管理（GitHub Issues の代替）

```bash
nest issue create "バグ修正" -b "ログインページでエラーが発生する"
nest issue list
nest issue show issue-1
nest issue comment issue-1 "原因を特定しました"
nest issue close issue-1
```

### ドキュメント管理（GitHub Wiki の代替）

```bash
nest doc create "API設計書" -b "## エンドポイント一覧\n..."
nest doc list
nest doc show doc-1
```

---

## 変更保存の標準フロー

ファイルを編集したら **必ず** この順序で実行する：

```bash
# Step 1: 変更を CRDT 操作として記録
nest record
# => "Recorded 42 operations from 3 file(s)."

# Step 2: スナップショットを作成（コミット相当）
nest snapshot create -m "認証機能を実装"
```

**注意:**
- `nest record` を省略するとファイル変更がスナップショットに反映されない
- `nest record` は `.nest/`, `.git/`, `target/`, `node_modules/` を自動的に無視する

---

## ブランチ開発フロー

```bash
# 1. ブランチ作成・切替
nest branch create feature-auth
nest branch switch feature-auth

# 2. 実装（ファイル編集）
# ...

# 3. 記録・スナップショット（何度でも繰り返し可）
nest record
nest snapshot create -m "ログイン機能を実装"

# 4. Review 作成（PR 相当）
nest review create "認証機能の追加" --source feature-auth --target main

# 5. 承認・マージ
nest review approve review-1
nest review merge review-1

# 6. main に戻る
nest branch switch main
```

---

## Nest 固有の特徴（git との違い）

- **マージコンフリクトなし**: CRDT により文字レベルで自動収束
- **`git add` 不要**: `nest record` がファイル変更を自動検出
- **同一ブランチで複数人同時作業**: ストリームが分離されているため衝突しない
- **Issue/Review/Doc がリポジトリに内蔵**: GitHub 不要
- **`nest sync` は双方向**: push と pull を同時に行う
- **操作はべき等**: 同じ操作を何度適用しても安全

---

## よくあるシナリオ

### 「コミットしたい」

```bash
nest record
nest snapshot create -m "変更内容"
```

### 「変更を確認したい」

```bash
nest status     # 現在のブランチ・ユーザー
nest log        # スナップショット履歴
nest diff       # 差分表示
```

### 「新機能を開発したい」

```bash
nest branch create feature-xxx
nest branch switch feature-xxx
# ... 実装 ...
nest record
nest snapshot create -m "実装内容"
nest review create "タイトル" --source feature-xxx --target main
```

### 「Issue を立てて作業を管理したい」

```bash
nest issue create "タスク名" -b "詳細説明"
# ... 作業 ...
nest issue comment issue-1 "完了しました"
nest issue close issue-1
```

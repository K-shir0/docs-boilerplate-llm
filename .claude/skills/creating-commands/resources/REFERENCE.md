# コマンド作成詳細リファレンス

コマンド作成における詳細な実装パターン、高度なテクニック、完全なリファレンスガイドです。

## 目次

1. [Frontmatter 完全リファレンス](#frontmatter-完全リファレンス)
2. [allowed-tools 完全ガイド](#allowed-tools-完全ガイド)
3. [高度なテクニック](#高度なテクニック)
4. [ネームスペースの活用](#ネームスペースの活用)
5. [SlashCommand Tool との連携](#slashcommand-tool-との連携)
6. [パフォーマンス最適化](#パフォーマンス最適化)

---

## Frontmatter 完全リファレンス

### 必須フィールド

#### description

```yaml
description: コマンドの簡潔な説明
```

**要件:**
* 1-2行の簡潔な説明
* コマンドの目的を明確に
* SlashCommand tool による認識に使用

**例:**

```yaml
# 良い例
description: Conventional Commits準拠でコミットを作成（日本語説明）
description: GitHub PR を作成し、レビュアーをアサイン
description: ユニットテストを生成し、カバレッジを計測

# 悪い例
description: コミット  # 不明確
description: This command creates a commit  # 日本語推奨
description: コミットを作成します。PRも作成できます。レビューもします。  # 長すぎ、複数タスク
```

### 推奨フィールド

#### allowed-tools

```yaml
allowed-tools: Bash(git status:*), Read, Write
```

**形式:**
* `ToolName`: ツール全体を許可
* `ToolName(pattern:*)`: 特定パターンのみ許可
* カンマ区切りで複数指定

**利用可能なツール:**

| ツール | 説明 | 例 |
|--------|------|-----|
| `Read` | ファイル読み取り | `Read` |
| `Write` | ファイル書き込み | `Write` |
| `Edit` | ファイル編集 | `Edit` |
| `Bash` | シェルコマンド実行 | `Bash(git status:*)` |
| `Grep` | テキスト検索 | `Grep` |
| `Glob` | ファイルパターンマッチ | `Glob` |
| `Task` | エージェント起動 | `Task` |
| `Skill` | スキル実行 | `Skill` |
| `SlashCommand` | 他コマンド実行 | `SlashCommand` |
| `WebFetch` | Web ページ取得 | `WebFetch` |
| `WebSearch` | Web 検索 | `WebSearch` |

**パターン例:**

```yaml
# Git コマンド全般
allowed-tools: Bash(git *)

# 特定の Git コマンドのみ
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)

# npm コマンド
allowed-tools: Bash(npm install:*), Bash(npm test:*), Bash(npm run:*)

# 読み取り専用
allowed-tools: Read, Grep, Glob

# ファイル操作全般
allowed-tools: Read, Write, Edit

# 全ツール許可（非推奨）
# allowed-tools を指定しない場合、全ツールが許可されます
```

#### model

```yaml
model: claude-sonnet-4-5-20250929
```

**利用可能なモデル:**

| モデル | 特徴 | 使用例 |
|--------|------|--------|
| `claude-sonnet-4-5-20250929` | バランス型、推奨 | 一般的なタスク |
| `claude-opus-*` | 高性能、複雑なタスク | コードレビュー、設計 |
| `claude-haiku-*` | 高速、シンプルなタスク | フォーマット、簡単な変換 |

**選択基準:**

* **Sonnet（推奨）**: ほとんどのユースケース
* **Opus**: 複雑な推論、高度なコードレビュー、設計判断
* **Haiku**: テンプレート展開、シンプルな変換、フォーマット

### オプションフィールド

#### disable-model-invocation

```yaml
disable-model-invocation: true
```

**用途:**
* LLM を呼び出さずにプロンプトを展開
* シンプルなテンプレート置換
* 高速実行が必要な場合

**例:**

```markdown
---
description: Issue テンプレートを展開
disable-model-invocation: true
---

# Issue: $ARGUMENTS

## Description
[問題の詳細を記述]

## Steps to Reproduce
1.
2.
3.

## Expected Behavior
[期待される動作]

## Actual Behavior
[実際の動作]

## Environment
- OS:
- Version:
```

呼び出し: `/issue-template バグ報告`

結果: テンプレートが展開されるが、LLM は呼び出されない

---

## allowed-tools 完全ガイド

### パターンマッチング

`allowed-tools` はワイルドカードとパターンマッチングをサポートします。

#### ワイルドカード構文

```yaml
# * は任意の文字列にマッチ
Bash(git *)          # git で始まる全コマンド
Bash(*test*)         # test を含む全コマンド
Bash(npm run:*)      # npm run で始まる全コマンド
```

#### 具体例

**Git 操作:**

```yaml
# 読み取り専用 Git コマンド
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*)

# コミット操作を含む
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)

# ブランチ操作を含む
allowed-tools: Bash(git branch:*), Bash(git checkout:*), Bash(git merge:*)

# 全 Git コマンド（注意して使用）
allowed-tools: Bash(git *)
```

**Node.js / npm:**

```yaml
# npm install と test のみ
allowed-tools: Bash(npm install:*), Bash(npm test:*)

# npm run コマンド
allowed-tools: Bash(npm run:*)

# package.json の読み取りを含む
allowed-tools: Bash(npm *), Read
```

**Python:**

```yaml
# pytest 実行
allowed-tools: Bash(pytest:*), Bash(python -m pytest:*)

# pip install
allowed-tools: Bash(pip install:*)

# Python スクリプト実行
allowed-tools: Bash(python:*), Read
```

**Docker:**

```yaml
# Docker コンテナ操作
allowed-tools: Bash(docker ps:*), Bash(docker logs:*)

# Docker Compose
allowed-tools: Bash(docker-compose up:*), Bash(docker-compose down:*)
```

### ツール組み合わせパターン

#### パターン1: 読み取り専用

```yaml
allowed-tools: Read, Grep, Glob, Bash(git status:*), Bash(git diff:*)
```

用途: レビュー、調査、分析

#### パターン2: ファイル操作

```yaml
allowed-tools: Read, Write, Edit
```

用途: ドキュメント生成、コード生成

#### パターン3: Git ワークフロー

```yaml
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Read
```

用途: コミット、プッシュ

#### パターン4: テスト実行

```yaml
allowed-tools: Bash(npm test:*), Bash(pytest:*), Read, Write
```

用途: テスト実行とレポート生成

#### パターン5: 複合ワークフロー

```yaml
allowed-tools: Bash(git *), Bash(npm *), Read, Write, Edit, Task
```

用途: 複雑なワークフロー（ただしエージェントを検討）

---

## 高度なテクニック

### テクニック1: 条件付き実行

プロンプト内で条件分岐を記述:

```markdown
---
description: 環境に応じたデプロイ
allowed-tools: Bash(git *), Bash(npm *), Read
---

# デプロイコマンド

## 環境

環境: $ARGUMENTS

## 実行手順

環境が "production" の場合:
1. git pull origin main
2. npm ci
3. npm run build
4. npm run deploy:prod

環境が "staging" の場合:
1. git pull origin develop
2. npm install
3. npm run build:staging
4. npm run deploy:staging

それ以外の場合:
エラーを表示: "Invalid environment: $ARGUMENTS"
```

### テクニック2: 複数ファイルの処理

```markdown
---
description: 複数ファイルにヘッダーを追加
allowed-tools: Read, Edit
---

# ヘッダー追加コマンド

## 対象ファイル

$ARGUMENTS で指定されたファイル全てに以下のヘッダーを追加:

\`\`\`
/**
 * Copyright (c) 2025 Your Company
 * Licensed under MIT
 */
\`\`\`

## 処理

各ファイルに対して:
1. ファイルを読み取り
2. 既存のヘッダーがあるかチェック
3. なければヘッダーを先頭に追加
4. ファイルを保存
```

### テクニック3: エラーハンドリング

```markdown
---
description: 安全なコミット作成
allowed-tools: Bash(git *)
---

# 安全なコミット

## 事前チェック

以下を確認:
1. !`git status` - 変更があるか
2. !`git diff --check` - ホワイトスペースエラーがないか
3. !`git log -1` - 最新コミットを確認

## エラーケース

変更がない場合:
→ "No changes to commit" と表示し、終了

ホワイトスペースエラーがある場合:
→ エラー箇所を表示し、修正を促す

## 実行

チェックが全て通過した場合のみコミットを作成。
```

### テクニック4: ドライラン

```markdown
---
description: ドライランサポート付きコマンド
allowed-tools: Bash(git *), Read
---

# デプロイコマンド（ドライラン対応）

## 引数

$ARGUMENTS

引数に "--dry-run" が含まれる場合:
→ 実行内容を表示するが、実際には実行しない

それ以外:
→ 実際に実行

## 実行内容

1. ブランチを確認: !`git branch --show-current`
2. 未コミットの変更を確認: !`git status --short`
3. デプロイスクリプトを実行

ドライランの場合:
→ 上記の情報を表示し、「これらの操作を実行します」と表示

通常実行の場合:
→ 実際にデプロイを実行
```

### テクニック5: スキルとの深い連携

```markdown
---
description: スキルを活用したコード生成
allowed-tools: Read, Write, Skill
---

# コード生成コマンド

## スキルの参照

以下のスキルに従ってコードを生成:
* `coding-standards` - コーディング規約
* `naming-conventions` - 命名規則
* `testing-guidelines` - テスト記述ルール

## 生成対象

$ARGUMENTS で指定されたコンポーネント

## 手順

1. Skill tool で各スキルを読み込む
2. スキルのガイドラインに従ってコードを生成
3. テストコードも併せて生成
4. ドキュメントコメントを追加
```

---

## ネームスペースの活用

### ネームスペースの構造

```
.claude/commands/
├── git/
│   ├── commit.md       # /git:commit
│   ├── pr.md           # /git:pr
│   ├── sync.md         # /git:sync
│   └── review.md       # /git:review
├── test/
│   ├── unit.md         # /test:unit
│   ├── integration.md  # /test:integration
│   └── e2e.md          # /test:e2e
├── docs/
│   ├── api.md          # /docs:api
│   ├── readme.md       # /docs:readme
│   └── changelog.md    # /docs:changelog
└── deploy/
    ├── staging.md      # /deploy:staging
    └── production.md   # /deploy:production
```

### ネームスペースのベストプラクティス

#### 1. 論理的なグループ化

関連するコマンドを同じネームスペースに:

```
git/        → Git 関連操作
test/       → テスト関連
docs/       → ドキュメント関連
deploy/     → デプロイ関連
code/       → コード生成関連
```

#### 2. 共通設定の活用

ネームスペース内で共通のパターンを使用:

**git/ 以下の全コマンド:**
```yaml
allowed-tools: Bash(git *), Read
```

**test/ 以下の全コマンド:**
```yaml
allowed-tools: Bash(npm test:*), Bash(pytest:*), Read, Write
```

#### 3. 階層的な命名

```
project/
├── init.md           # /project:init
├── setup/
│   ├── frontend.md   # /project:setup/frontend (非推奨)
│   └── backend.md    # /project:setup/backend (非推奨)
```

注: 現在、ネストは1階層のみサポート。深いネストは避けてください。

---

## SlashCommand Tool との連携

### SlashCommand Tool とは

Claude が会話中にプログラマティックにコマンドを実行できるツールです。

### 使用条件

* コマンドに `description` フィールドが必須
* カスタムコマンドのみサポート（ビルトインコマンドは不可）
* グローバル設定で無効化可能

### 自動実行の設計

`description` を工夫して、適切なタイミングで自動実行されるようにします。

**例1: コミット作成**

```yaml
description: Git コミットを作成（Conventional Commits準拠）
```

ユーザーが「コミットを作成して」と言った時に自動実行される可能性があります。

**例2: PR 作成**

```yaml
description: GitHub Pull Request を作成
```

ユーザーが「PR を作成して」と言った時に自動実行される可能性があります。

### 連鎖実行

コマンド内から他のコマンドを呼び出す:

```markdown
---
description: コミットして PR を作成
allowed-tools: SlashCommand, Bash(git *)
---

# コミット + PR コマンド

## 手順

1. SlashCommand tool で /commit を実行
2. コミットが成功したら
3. SlashCommand tool で /pr を実行

## 引数

コミットメッセージ: $ARGUMENTS の前半
PR タイトル: $ARGUMENTS の後半（またはコミットメッセージを使用）
```

---

## パフォーマンス最適化

### 最適化1: モデル選択

タスクに応じて適切なモデルを選択:

```yaml
# シンプルなフォーマット → Haiku
model: claude-haiku-*

# 一般的なタスク → Sonnet
model: claude-sonnet-4-5-20250929

# 複雑な推論 → Opus
model: claude-opus-*
```

### 最適化2: disable-model-invocation

LLM 不要の場合は無効化:

```yaml
disable-model-invocation: true
```

**使用例:**
* テンプレート展開
* 単純な文字列置換
* 定型フォーマットの生成

### 最適化3: Bash コマンドの最適化

```markdown
# 悪い例: 大量の出力
!`git log --all`

# 良い例: 必要な情報のみ
!`git log -10 --oneline`

# 悪い例: 遅いコマンド
!`find . -name "*.js"`

# 良い例: 高速な代替
!`fd -e js`  # fd がインストールされている場合
```

### 最適化4: プロンプトの簡潔化

```markdown
# 悪い例: 冗長
このコマンドは、プロジェクトの全ての JavaScript ファイルに対して、
ESLint を実行し、エラーがあれば修正を試み、修正できないものは
レポートを生成し、最終的に結果をユーザーに報告します。

# 良い例: 簡潔
全 JS ファイルに ESLint を実行し、自動修正とレポート生成。
```

---

## まとめ

コマンド作成における重要ポイント:

1. **Frontmatter を適切に設定**
   * `description` は必須
   * `allowed-tools` でセキュリティ確保
   * 適切な `model` 選択

2. **明確なプロンプト構造**
   * セクション分割
   * 具体的な手順
   * エラーケースの考慮

3. **動的コンテンツの活用**
   * `$ARGUMENTS` で柔軟性
   * `!`bash`` で最新情報
   * スキルで一貫性

4. **ネームスペースで整理**
   * 論理的なグループ化
   * 発見しやすさの向上

5. **パフォーマンス最適化**
   * 適切なモデル選択
   * 簡潔なプロンプト
   * 効率的な Bash コマンド

---

バージョン: 1.0.0
最終更新: 2025-01-15

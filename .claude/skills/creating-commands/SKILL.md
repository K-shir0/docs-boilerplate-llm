---
name: creating-commands
description: Claude Code custom slash commands creation guide with structure, best practices, and examples
version: 1.0.0
category: development
keywords: [commands, slash-commands, CLI, automation, workflow]
---

# カスタムコマンド作成ガイド

Claude Code のカスタムスラッシュコマンド（slash commands）を作成するための包括的なガイドです。

## 重要な原則

**"コマンドは実行、スキルは知識"** - コマンドは特定のタスクを即座に実行するためのツールです。知識やガイドラインの提供にはスキルを使用してください。

**"シンプルに保つ"** - 1つのコマンドは1つの明確なタスクに集中します。複雑なワークフローにはエージェントを検討してください。

## 目次

1. [コマンドとは](#1-コマンドとは)
2. [コマンド vs スキル vs エージェント](#2-コマンド-vs-スキル-vs-エージェント)
3. [基本構造](#3-基本構造)
4. [Frontmatter 設定](#4-frontmatter-設定)
5. [プロンプトの記述](#5-プロンプトの記述)
6. [動的コンテンツ](#6-動的コンテンツ)
7. [実例解説](#7-実例解説)
8. [よくあるユースケース](#8-よくあるユースケース)
9. [ベストプラクティス](#9-ベストプラクティス)
10. [トラブルシューティング](#10-トラブルシューティング)
11. [参考資料](#11-参考資料)

---

## 1. コマンドとは

### 1.1 定義

カスタムコマンドは、Claude Code で使用できるスラッシュコマンド（`/command-name`）です。特定のタスクを素早く実行するための CLI スタイルのインターフェースを提供します。

### 1.2 コマンドの種類

* **プロジェクトコマンド**: `.claude/commands/` に配置
  * リポジトリで管理
  * チーム全体で共有
  * バージョン管理対象

* **パーソナルコマンド**: `~/.claude/commands/` に配置
  * 全プロジェクトで利用可能
  * ユーザー固有のカスタマイズ
  * 個人の設定として管理

### 1.3 コマンドを作成すべき場合

以下の場合にコマンド作成を検討してください:

* 頻繁に実行する特定のタスクがある
* 一連の手順を自動化したい
* プロジェクト固有のワークフローがある
* チーム全体で同じ操作を標準化したい
* 既存ツールの簡易ラッパーが必要

---

## 2. コマンド vs スキル vs エージェント

### 2.1 比較表

| 特徴 | コマンド | スキル | エージェント |
|------|----------|--------|------------|
| **目的** | タスクの即時実行 | 知識・ガイドラインの提供 | 複雑な多段階タスクの自動実行 |
| **配置** | `.claude/commands/*.md` | `.claude/skills/*/SKILL.md` | `.claude/agents/*.md` |
| **呼び出し** | `/command-name` | 自動参照または Skill tool | Task tool |
| **実行形態** | CLI スタイル | オンデマンド参照 | 自律的実行 |
| **引数** | サポート (`$ARGUMENTS`) | 非サポート | サポート（プロンプト経由） |
| **構造** | 単一ファイル + frontmatter | ディレクトリ + SKILL.md + resources/ | 単一ファイル + frontmatter |
| **例** | `/commit`, `/pr`, `/review` | `markdown-skills`, `creating-skills` | `code-quality-enhancer` |

### 2.2 使い分けの判断基準

**コマンドを使用:**
* Git 操作（コミット、PR作成）
* コード生成
* テスト実行
* ドキュメント生成
* 単純なワークフローの自動化

**スキルを使用:**
* コーディング規約
* スタイルガイド
* ベストプラクティス集
* リファレンス情報
* 設計パターン

**エージェントを使用:**
* コード品質改善
* 包括的なテスト作成
* 複数ファイルのリファクタリング
* 複雑な調査タスク
* 多段階の意思決定を伴う作業

---

## 3. 基本構造

### 3.1 ファイル配置

```
.claude/commands/
├── commit.md              # シンプルなコマンド
├── pr.md                  # シンプルなコマンド
└── git/                   # ネームスペース
    ├── review.md          # /git:review として呼び出し
    └── sync.md            # /git:sync として呼び出し
```

### 3.2 ファイル形式

コマンドは Markdown ファイルで、以下の構造を持ちます:

```markdown
---
description: コマンドの簡潔な説明
allowed-tools: Bash(git status:*), Read, Write
model: claude-sonnet-4-5-20250929
---

# コマンドタイトル

[自然言語でのプロンプト内容]

$ARGUMENTS を使用して引数を埋め込み。
!`bash コマンド` を使用してコマンド出力を埋め込み。

## セクション例
- 手順1
- 手順2
- 手順3
```

### 3.3 ネームスペース

ディレクトリを使用してコマンドを整理:

```
.claude/commands/
└── git/
    ├── commit.md      # /git:commit
    ├── pr.md          # /git:pr
    └── sync.md        # /git:sync
```

呼び出し: `/namespace:command-name`

---

## 4. Frontmatter 設定

### 4.1 必須フィールド

#### description

```yaml
description: コマンドの簡潔な説明（1-2行）
```

用途:
* SlashCommand tool がコマンドを認識するために使用
* コマンド一覧での表示
* 自動実行の判断基準

例:
```yaml
description: Conventional Commits準拠でコミットを作成（日本語説明）
```

### 4.2 推奨フィールド

#### allowed-tools

```yaml
allowed-tools: Bash(git status:*), Bash(git diff:*), Read, Write
```

Claude が使用できるツールを制限します。セキュリティと予測可能性の向上に重要。

形式:
* `ToolName`: ツール全体を許可
* `ToolName(pattern:*)`: 特定パターンのみ許可
* カンマ区切りで複数指定

例:
```yaml
# Git コマンドのみ許可
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)

# 読み取り専用
allowed-tools: Read, Grep, Glob

# ファイル操作
allowed-tools: Read, Write, Edit
```

#### model

```yaml
model: claude-sonnet-4-5-20250929
```

使用する Claude モデルを指定:
* `claude-sonnet-4-5-20250929`: バランス型（推奨）
* `claude-opus-*`: 複雑なタスク向け
* `claude-haiku-*`: シンプルなタスク向け

### 4.3 オプションフィールド

#### disable-model-invocation

```yaml
disable-model-invocation: true
```

`true` の場合、LLM を呼び出さずにプロンプトを単純に展開します。単純なテンプレート展開に有用。

---

## 5. プロンプトの記述

### 5.1 基本原則

1. **自然言語で記述**: Markdown と自然言語で指示を書く
2. **明確な構造**: セクション（`##`）で論理的に整理
3. **具体的な手順**: 実行すべきステップを明示
4. **例を含める**: 良い例と悪い例を提示

### 5.2 プロンプトの構成要素

```markdown
# コマンドタイトル

## 目的
[このコマンドの目的を1-2文で説明]

## 現在の状態
[動的に取得した情報を表示]
!`git status`

## ルール
[従うべき規則をリスト]

* ルール1
* ルール2
* ルール3

## 手順
[実行すべきステップ]

1. ステップ1
2. ステップ2
3. ステップ3

## 例
[良い例と悪い例]

良い例:
\`\`\`
例の内容
\`\`\`

悪い例:
\`\`\`
避けるべき内容
\`\`\`
```

### 5.3 スキルとの連携

既存スキルを参照して一貫性を保つ:

```markdown
## commit-guidelines スキルの規則

このプロジェクトでは commit-guidelines スキルに従います。
詳細は `.claude/skills/commit-guidelines/SKILL.md` を参照。
```

---

## 6. 動的コンテンツ

### 6.1 引数の使用

`$ARGUMENTS` プレースホルダーでコマンドライン引数を挿入:

```markdown
## タスク

以下の機能について作業してください: $ARGUMENTS
```

呼び出し:
```
/task-command ユーザー認証機能
```

展開結果:
```
以下の機能について作業してください: ユーザー認証機能
```

### 6.2 Bash コマンドの実行

`!`bash command`` で実行結果を埋め込み:

```markdown
## 現在の Git 状態

**Status:**
!`git status`

**Diff:**
!`git diff`

**最近のコミット:**
!`git log -5 --oneline`
```

実行時に各コマンドが実行され、出力がプロンプトに埋め込まれます。

### 6.3 ファイル参照

`@filename` でファイル内容を参照:

```markdown
## 設計書のレビュー

以下の設計書をレビューしてください:
@docs/design.md
```

---

## 7. 実例解説

### 7.1 プロジェクト内の `/commit` コマンド

このプロジェクトの `.claude/commands/commit.md` は優れた実例です。

**構造:**

```markdown
---
description: Conventional Commits準拠でコミットを作成（日本語説明）
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git log:*)
model: claude-sonnet-4-5-20250929
---

# コミット作成タスク

commit-guidelines スキルに従って、Conventional Commits形式（日本語説明）でコミットを作成してください。

## 現在の状態

**Git Status:**
!`git status`

**変更内容（未ステージング）:**
!`git diff`

**ステージング済み:**
!`git diff --cached`

**最近のコミット履歴（参考用）:**
!`git log -5 --oneline`

## commit-guidelines スキルの規則

[スキルの内容を参照]

## タスク手順

1. 変更内容の分析
2. 適切な type の選択
3. コミットメッセージの作成
4. ファイルのステージング
5. コミットの実行

## 良い例

\`\`\`
feat(skills): コミットガイドラインスキルを追加
\`\`\`

## 悪い例（避けること）

\`\`\`
add: 追加  # 何を追加したのか不明
\`\`\`
```

**優れた点:**

1. **動的情報の活用**: `!`git commands`` で現在の状態を取得
2. **スキルとの連携**: `commit-guidelines` スキルを参照して一貫性を保持
3. **明確な手順**: ステップバイステップの指示
4. **具体例**: 良い例と悪い例を提示
5. **ツール制限**: Git コマンドのみに `allowed-tools` を制限

### 7.2 シンプルなコマンド例

```markdown
---
description: README.md を更新
allowed-tools: Read, Write, Edit
---

# README 更新コマンド

README.md を更新してください。

## 更新内容

以下の内容を反映:
$ARGUMENTS

## 既存内容

@README.md

## ルール

* Markdown 形式を維持
* 既存のセクション構造を尊重
* 簡潔で明確な記述
```

---

## 8. よくあるユースケース

### 8.1 Git 操作

**コミット作成:**
```markdown
---
description: Git コミットを作成
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)
---
```

**PR 作成:**
```markdown
---
description: GitHub PR を作成
allowed-tools: Bash(git *), Bash(gh pr create:*)
---
```

### 8.2 コード生成

**テスト生成:**
```markdown
---
description: ユニットテストを生成
allowed-tools: Read, Write
---

# テスト生成

以下のファイルのユニットテストを生成:
$ARGUMENTS
```

**API エンドポイント生成:**
```markdown
---
description: REST API エンドポイントを生成
allowed-tools: Read, Write, Edit
---
```

### 8.3 ドキュメント作成

**設計書生成:**
```markdown
---
description: 設計書を生成
allowed-tools: Read, Write
---

# 設計書生成

docs/design.md.sample をベースに設計書を作成。
```

**Changelog 更新:**
```markdown
---
description: CHANGELOG.md を更新
allowed-tools: Bash(git log:*), Read, Write, Edit
---
```

### 8.4 レビュー

**コードレビュー:**
```markdown
---
description: コード変更をレビュー
allowed-tools: Bash(git diff:*), Read
---

# コードレビュー

## 変更内容
!`git diff`

## レビュー観点
* バグの可能性
* パフォーマンス
* セキュリティ
* 可読性
* テストカバレッジ
```

---

## 9. ベストプラクティス

### 9.1 Frontmatter 設定

* **必ず `description` を含める**: SlashCommand tool による自動実行に必須
* **`allowed-tools` で制限**: セキュリティと予測可能性の向上
* **適切な `model` 選択**: タスクの複雑さに応じて選択

### 9.2 プロンプト記述

* **明確な構造**: `##` セクションで論理的に整理
* **具体例を含める**: 良い例と悪い例を提示
* **スキルを参照**: 既存スキルを活用して一貫性を保持
* **動的情報を活用**: `!`bash`` で最新情報を取得

### 9.3 コマンド設計

* **1コマンド1タスク**: 単一責任の原則
* **引数で柔軟性**: `$ARGUMENTS` で再利用性を高める
* **エラーケースを考慮**: 失敗時の対処を明示
* **ドライラン可能**: 実行前に確認できる仕組み

### 9.4 チーム開発

* **命名規則の統一**: kebab-case を使用
* **ネームスペースで整理**: 関連コマンドをグループ化
* **ドキュメント化**: README でコマンド一覧を管理
* **バージョン管理**: `.claude/commands/` をリポジトリに含める

---

## 10. トラブルシューティング

### 10.1 コマンドが認識されない

**症状:** `/command-name` が機能しない

**確認項目:**
* ファイルが `.claude/commands/` に配置されているか
* ファイル名が `command-name.md` か（ハイフン使用）
* `description` フィールドが存在するか
* Frontmatter の YAML が正しいか

### 10.2 $ARGUMENTS が展開されない

**症状:** `$ARGUMENTS` がそのまま表示される

**解決策:**
* コマンド呼び出し時に引数を指定: `/command arg1 arg2`
* プロンプト内で `$ARGUMENTS` を使用（`${ARGUMENTS}` ではない）

### 10.3 Bash コマンドが実行されない

**症状:** `!`command`` が展開されない

**確認項目:**
* バッククォート（`` ` ``）を正しく使用しているか
* `allowed-tools` に Bash が含まれているか
* コマンドのパーミッションが正しいか

詳細なトラブルシューティングは [resources/TROUBLESHOOTING.md](./resources/TROUBLESHOOTING.md) を参照してください。

---

## 11. 参考資料

### このスキルのリソース

* [resources/REFERENCE.md](./resources/REFERENCE.md) - 詳細リファレンス
* [resources/EXAMPLES.md](./resources/EXAMPLES.md) - 実例集
* [resources/TROUBLESHOOTING.md](./resources/TROUBLESHOOTING.md) - トラブルシューティング詳細

### 関連スキル

* `creating-skills` - スキル作成のベストプラクティス
* `commit-guidelines` - コミットメッセージ規則
* `markdown-skills` - Markdown 記述ガイドライン

### プロジェクト内リソース

* `.claude/commands/commit.md` - 実際のコマンド例
* `.claude/skills/` - 参照可能なスキル集
* `CLAUDE.md` - プロジェクトガイドライン

### 外部リソース

* [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
* [Custom Commands Guide](https://docs.claude.com/en/docs/claude-code/custom-commands)

---

## 使用方法

### このスキルを呼び出す

```
/skill creating-commands
```

### 新しいコマンドを作成する

このガイドを参照しながら、以下のように依頼してください:

```
[タスク名]のコマンドを作成してください。
creating-commands スキルの手順に従って作成してください。
```

Claude Code が自動的に以下を実行します:
1. コマンドファイルの作成
2. Frontmatter の設定
3. プロンプトの記述
4. 動作確認

---

バージョン: 1.0.0
最終更新: 2025-01-15
メンテナー: K-shir0

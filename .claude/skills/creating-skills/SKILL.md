---
name: creating-skills
description: Guide for creating and editing custom Claude Code skills with best practices and step-by-step instructions
version: 2.1.0
category: development
---

# スキル作成ガイド

Claude Codeのカスタムスキルを作成するための包括的なガイドです。このガイドでは、実際に作成した `markdown-style-rules` スキルを実例として、具体的な手順とベストプラクティスを説明します。

## 重要な原則

**"評価をコードより先に作成"** - スキルの実装前に、成功基準を明確に定義してください。詳細は [resources/TESTING.md](resources/TESTING.md) を参照。

**"コンテキストウィンドウは公共財"** - スキルは会話履歴とトークンを共有します。簡潔さとトークン効率を最優先してください。

## 目次

1. [スキルとは](#1-スキルとは)
2. [スキルの構造](#2-スキルの構造)
3. [作成手順](#3-作成手順)
4. [既存スキル編集ワークフロー](#4-既存スキル編集ワークフロー)
5. [ベストプラクティス](#5-ベストプラクティス)
6. [実例: markdown-style-rules](#6-実例-markdown-style-rules)
7. [トラブルシューティング](#7-トラブルシューティング)
8. [参考資料](#8-参考資料)

---

## 1. スキルとは

### 1.1 スキルの定義

スキルは、特定のドメインやタスクに関する知識・手順・ガイドラインをまとめた再利用可能なドキュメントです。Claude Codeが必要に応じて参照し、一貫性のある作業を実行するために使用します。

### 1.2 スキル vs エージェント vs コマンド

| 種類 | 目的 | 実行形態 | 例 |
|------|------|----------|-----|
| スキル | 知識・ガイドラインの参照 | オンデマンド読み込み | `markdown-style-rules`, `creating-skills` |
| エージェント | 複雑な多段階タスクの自動実行 | 自律的実行 | `code-quality-enhancer`, `unit-test-writer` |
| コマンド | 特定の単一タスクの実行 | CLIスタイル実行 | `/pr`, `/commit-and-pr` |

### 1.3 スキルを作成・編集すべき場合

以下の場合にスキル作成を検討してください：

- 繰り返し参照する知識やガイドラインがある
- プロジェクト固有のルールや規約を文書化したい
- ベストプラクティスを体系的にまとめたい
- 一貫性のある作業を保証したい
- 新しいチームメンバーへのオンボーディング資料として使いたい

以下の場合には既存スキルの編集を検討してください：

- スキルの内容が古くなっている、または不正確になっている
- 新しいベストプラクティスやルールを追加する必要がある
- スキルの構造や可読性を改善したい
- トークン効率を向上させるために内容を最適化したい
- ユーザーフィードバックに基づいて改善が必要

---

## 2. スキルの構造

### 2.1 基本構造

```
.claude/skills/
└── <skill-name>/
    ├── SKILL.md          # メインファイル（400-500行推奨）
    └── resources/        # オプション: 詳細情報
        ├── REFERENCE.md
        └── TESTING.md
```

重要: 参照の深さは1段階のみ。SKILL.md → resources/*.md は可能ですが、resources/*.md → さらに別ファイルは避けてください。

### 2.2 ファイル命名規則

- スキル名: kebab-case（小文字 + ハイフン）
- メインファイル: 必ず `SKILL.md`（大文字）
- リソースファイル: `resources/` ディレクトリ内に配置

### 2.3 Frontmatter（メタデータ）

SKILL.mdの先頭に以下のYAML frontmatterを記述：

```yaml
---
name: skill-name          # 必須: kebab-case、64文字以内
description: Brief description in English   # 必須: 英語で記述、200文字以内
version: 1.0.0            # 推奨: セマンティックバージョニング
category: documentation   # 推奨: カテゴリ
---
```

**重要**: `description` フィールドは必ず英語で記述してください。これはClaude Codeのスキル一覧やヘルプシステムで表示されるため、国際的な互換性を保つ必要があります。

完全なfrontmatterリファレンスは [resources/REFERENCE.md](resources/REFERENCE.md) を参照。

### 2.4 Progressive Disclosure

**500行制限**: SKILL.mdは500行以内に収めてください。詳細情報は `resources/` ディレクトリに分割します。

実装パターン:
- パターン1: 単一ドメインスキル（300-500行、resources/ 不要）
- パターン2: 複数ドメインスキル（400-500行 + resources/）
- パターン3: 参照型スキル（200-300行 + 外部リンク集）

詳細は [resources/REFERENCE.md](resources/REFERENCE.md) を参照。

---

## 3. 作成手順

### ステップ1: 評価基準を定義

実装前に成功基準を明確化：

1. スキルの目的を1文で記述
2. 成功シナリオを3つ以上列挙
3. 各シナリオの期待される出力を定義

詳細な評価戦略は [resources/TESTING.md](resources/TESTING.md) を参照。

### ステップ2: ディレクトリとファイルを作成

```bash
# ディレクトリ作成
mkdir -p .claude/skills/<skill-name>

# メインファイル作成
touch .claude/skills/<skill-name>/SKILL.md

# 必要に応じてリソースディレクトリ作成
mkdir -p .claude/skills/<skill-name>/resources
```

### ステップ3: Frontmatterを作成

```yaml
---
name: <skill-name>
description: <Brief description in English, max 200 characters>
version: 1.0.0
category: <カテゴリ名>
---
```

**注意**: `description` は必ず英語で記述してください。

### ステップ4: 本文を構造化

推奨構造:

```markdown
# スキルタイトル

[簡潔な概要説明]

## 目次
[詳細な目次]

## セクション1: 基本概念
[コア情報のみ]

## セクション2: 使用方法
[実用的な手順]

## セクション3: 実例
[Before/After比較]

## 参考資料
- [resources/REFERENCE.md] - 詳細リファレンス
- [resources/TESTING.md] - テスト戦略
```

### ステップ5: 動作確認

```bash
# ファイルサイズ確認（500行以内が目標）
wc -l .claude/skills/<skill-name>/SKILL.md

# スキル呼び出しテスト
# Claude Codeで実行: /skill <skill-name>
```

### ステップ6: 複数モデルでテスト

Haiku、Sonnet、Opusの3モデルで動作確認。詳細は [resources/TESTING.md](resources/TESTING.md) を参照。

### ステップ7: 反復改善

2つのClaudeインスタンス（実装者 vs 使用者）でフィードバックループを実施。最低2回以上のイテレーションを推奨。

---

## 4. 既存スキル編集ワークフロー

既存のスキルを編集する際は、以下の手順に従ってください。このワークフローは `markdown-skills` のパターンに準拠しています。

### ステップ1: 現状の把握

編集前にスキル全体を理解する：

1. **スキル全体を読む**
   - スキルの目的と対象範囲を確認
   - 既存のセクション構造を把握
   - 記述スタイルとトーンを理解

2. **修正箇所を特定**
   - 古くなった情報の確認
   - 不正確な記述の特定
   - 改善が必要な箇所のリストアップ

3. **影響範囲の確認**
   - 関連するresourcesファイルの有無
   - 他のスキルとの参照関係
   - CLAUDE.mdでの自動活用設定

### ステップ2: 既存の記述規則に準拠

一貫性を保つための確認事項：

1. **Frontmatterの更新**
   - version番号をセマンティックバージョニングで更新
     - パッチ（0.0.x）: 誤字修正、軽微な変更
     - マイナー（0.x.0）: 新機能追加、セクション追加
     - メジャー（x.0.0）: 構造の大幅変更、破壊的変更
   - descriptionが英語で記述されているか確認（日本語の場合は英語に変更）
   - 最終更新日を更新

2. **既存のセクション構造を維持**
   - 見出しレベルの一貫性（H1-H3）
   - セクション番号の整合性
   - 目次との対応確認

3. **トーンとスタイルの統一**
   - 既存の記述スタイルに合わせる
   - 用語の統一（既存用語を優先）
   - 箇条書きスタイルの統一（アスタリスク使用）

### ステップ3: 内容の整理と最適化

トークン効率を考慮した編集：

1. **500行制限の遵守**
   - 現在の行数を確認（`wc -l SKILL.md`）
   - 超過している場合はresources/に分割検討
   - 冗長な説明の削除

2. **Progressive Disclosureの活用**
   - コア情報はSKILL.mdに保持
   - 詳細情報はresources/に移動
   - 適切な参照リンクを追加

3. **実例の見直し**
   - Before/After比較の有効性確認
   - 最小限の実例に絞る
   - 古い実例の更新または削除

### ステップ4: 変更の検証

編集後の品質確認：

1. **構造の妥当性確認**
   - 目次とセクション番号の一致
   - リンクの有効性確認
   - 参照の深さ確認（1段階のみ）

2. **複数モデルでテスト**
   - Haiku: 基本的な動作確認
   - Sonnet: 標準的なユースケース
   - Opus: 複雑なシナリオ

3. **トークン効率の検証**
   - ファイルサイズの確認
   - 不要な繰り返しの削除
   - 簡潔な表現への置き換え

### ステップ5: ドキュメント更新

関連ドキュメントの同期：

1. **CLAUDE.mdの確認**
   - 自動活用ポリシーの更新が必要か確認
   - スキルの説明が最新か確認

2. **resources/の更新**
   - 関連するリソースファイルを同期
   - 参照リンクの有効性確認

3. **変更履歴の記録**
   - コミットメッセージで変更内容を明記
   - 重要な変更はREADMEやドキュメントに記載

---

## 5. ベストプラクティス

### 5.1 トークン効率化

**原則**: "コンテキストウィンドウは公共財"

実装:
- SKILL.mdは500行以内
- 詳細情報はresources/に分割
- 冗長な説明を避ける
- 実例は必要最小限に

### 5.2 明確な構造

- H1: スキルタイトル（1つのみ）
- H2: 主要セクション
- H3: サブセクション
- H4以降: 詳細項目

### 5.3 実例重視

含めるべき実例:
- Before/After比較
- 正しい例 vs 間違い例
- ユースケース別サンプル

### 5.4 避けるべきアンチパターン

1. 時間依存情報（"2025年12月まで有効"など）
2. 矛盾した用語（"メインセクション" vs "プライマリセクション"）
3. Windowsスタイルパス（`C:\Users\...`）
4. 多数の選択肢提示（推奨デフォルト + 2-3の選択肢に絞る）
5. ネストされた参照構造（SKILL.md → ref1.md → ref2.md）

詳細は [resources/REFERENCE.md](resources/REFERENCE.md) を参照。

### 5.5 自由度の適切な設定

Claudeに適切な自由度を与える:

低自由度（厳格なルール）:
```markdown
フォーマット規則:
- 見出しはH1から始める
- コードブロックに必ず言語指定
```

中自由度（推奨事項）:
```markdown
推奨:
- 見出しはH1から始めることを推奨
- コードブロックに言語指定があると望ましい
```

高自由度（ガイダンスのみ）:
```markdown
考慮事項:
- 見出し階層の一貫性
- コードの可読性
```

### 5.6 メタデータの適切な設定

チェック項目:
- `name`: 64文字以内、kebab-case
- `description`: 200文字以内、**必ず英語で記述**、明確
- `version`: セマンティックバージョニング（x.y.z）

**descriptionの記述ルール**:
- 必ず英語で記述する（国際的互換性のため）
- 簡潔で明確な説明（200文字以内）
- スキルの目的と主な機能を含める
- 自動活用条件を含めると効果的

良い例:
```yaml
description: Guide for creating and editing custom Claude Code skills with best practices and step-by-step instructions
```

避けるべき例:
```yaml
description: スキル作成のためのガイド  # ❌ 日本語
description: A guide  # ❌ 不明確
```

---

## 6. 実例: markdown-style-rules

### 6.1 作成プロセス

```bash
# ステップ1: Plan modeで調査
Task tool で以下を調査:
- Kubernetesスタイルガイド
- Claudeスキルベストプラクティス

# ステップ2: ディレクトリ作成
mkdir -p .claude/skills/markdown-style-rules

# ステップ3: ファイル作成
Write tool で SKILL.md を作成（約756行）

# ステップ4: 反復改善
- 絵文字削除（ユーザーフィードバック）
- 太字削除（シンプル化）
- Kubernetes表記の汎用化
```

### 6.2 最終成果物

```
ファイル: .claude/skills/markdown-style-rules/SKILL.md
サイズ: 18KB（約756行）
構成:
- 詳細な目次（13セクション）
- 4部構成（基本ルール、プロジェクト固有、チェックリスト、実例）
- 豊富なBefore/After比較
- 自動チェックリスト機能

呼び出し方:
/skill markdown-style-rules
```

注: 756行は500行推奨を超えていますが、包括的なスタイルガイドとして許容範囲内です。

---

## 7. トラブルシューティング

### 7.1 スキルが認識されない

原因:
- ディレクトリ構造が間違っている
- ファイル名が `SKILL.md` でない（小文字やスペルミス）
- frontmatterが正しくない

解決方法:
```bash
# 構造を確認
ls -la .claude/skills/<skill-name>/

# 正しい構造:
# .claude/skills/<skill-name>/SKILL.md

# frontmatterを確認
head -n 10 .claude/skills/<skill-name>/SKILL.md
```

### 7.2 内容が長すぎる

対策:
1. Progressive Disclosureを使用してresources/に分割
2. 冗長な説明を削除
3. 実例を最小限に絞る

### 7.3 frontmatterのYAMLエラー

```yaml
# 正しい
---
name: skill-name
description: Simple description
version: 1.0.0
---

# 間違い（特殊文字のエスケープ漏れ）
---
name: skill-name
description: Description with: colon
---

# 正しい（クォートで囲む）
---
name: skill-name
description: "Description with: colon"
---
```

詳細なトラブルシューティングは [resources/REFERENCE.md](resources/REFERENCE.md) を参照。

---

## 8. 参考資料

### 公式ドキュメント

- [Claude Code Skills Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
- [How to Create Custom Skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)

### スタイルガイド

- [Kubernetes Style Guide](https://kubernetes.io/docs/contribute/style/style-guide/)
- [CommonMark Specification](https://commonmark.org/)

### このスキルのリソース

- [resources/TESTING.md](resources/TESTING.md) - 評価とテスト戦略
- [resources/REFERENCE.md](resources/REFERENCE.md) - 詳細リファレンスとアンチパターン

### プロジェクト内リソース

- `.claude/agents/` - エージェントの実例
- `.claude/commands/` - コマンドの実例
- `CLAUDE.md` - プロジェクトガイドライン

---

## 使用方法

### このスキルを呼び出す

```
/skill creating-skills
```

### 新しいスキルを作成する

このガイドを参照しながら、以下のように依頼してください：

```
[トピック]についてのスキルを作成してください。
creating-skillsスキルの手順に従って作成してください。
```

Claude Codeが自動的に以下を実行します：
1. 評価基準定義（"評価をコードより先に"）
2. ディレクトリ作成
3. SKILL.md作成（500行以内）
4. 必要に応じてresources/作成
5. 複数モデルでテスト
6. 反復改善

### 既存のスキルを編集する

既存のスキルを改善する際は、以下のように依頼してください：

```
.claude/skills/[skill-name]/SKILL.md を編集してください。
[具体的な変更内容や改善点]
creating-skillsスキルの編集ワークフローに従ってください。
```

Claude Codeが自動的に以下を実行します：
1. 現状の把握（スキル全体を読む）
2. 既存の記述規則に準拠した編集
3. 内容の整理と最適化（トークン効率化）
4. 変更の検証（構造、リンク、トークン効率）
5. 関連ドキュメント更新の確認

**自動活用**: このスキルは `.claude/skills/` 内の `SKILL.md` ファイルを編集する際に自動的に活用されます。

---

バージョン: 2.1.0
最終更新: 2025-11-15
メンテナー: K-shir0

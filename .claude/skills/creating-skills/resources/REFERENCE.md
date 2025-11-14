# スキル作成詳細リファレンス

スキル作成における詳細な実装パターン、アンチパターン、トラブルシューティングを網羅したリファレンスガイドです。

## 目次

1. [詳細な実装パターン](#詳細な実装パターン)
2. [アンチパターン詳細](#アンチパターン詳細)
3. [Progressive Disclosureの詳細実装](#progressive-disclosureの詳細実装)
4. [Frontmatter完全リファレンス](#frontmatter完全リファレンス)
5. [トラブルシューティング詳細](#トラブルシューティング詳細)

---

## 詳細な実装パターン

### パターン1: 単一ドメインスキル

単一の専門領域に特化したスキル。

構造:
```
SKILL.md (300-500行)
- 概要
- ルール集
- 実例
- チェックリスト
```

適用例:
- markdown-style-rules（Markdownスタイル専門）
- code-review-checklist（コードレビュー専門）
- naming-conventions（命名規則専門）

メリット:
- 焦点が明確
- メンテナンスが容易
- トークン消費が少ない

デメリット:
- 複数ドメインにまたがる場合は複数スキル必要

### パターン2: 複数ドメインスキル

複数の関連領域をカバーするスキル。

構造:
```
SKILL.md (400-500行)
- 概要
- ドメイン1の要点
- ドメイン2の要点
- ドメイン3の要点
- 各ドメインの詳細は resources/ に分離

resources/
├── domain1-details.md
├── domain2-details.md
└── domain3-details.md
```

適用例:
- creating-skills（スキル作成の全領域）
- fullstack-guidelines（フロント+バック+DB）
- ci-cd-practices（ビルド+テスト+デプロイ）

メリット:
- 関連情報が1箇所にまとまる
- 横断的な理解が促進される

デメリット:
- SKILL.mdが肥大化しやすい
- Progressive Disclosureが必須

### パターン3: 参照型スキル

主に外部リソースへの参照を提供するスキル。

構造:
```
SKILL.md (200-300行)
- 概要
- クイックリファレンス
- 外部リソースへのリンク集
- 最小限の実例
```

適用例:
- api-documentation-index（API仕様へのリンク集）
- tools-reference（ツール使用法のリンク集）
- learning-resources（学習リソースのキュレーション）

メリット:
- 非常に軽量
- メンテナンスが最小限

デメリット:
- 外部リソースが変更されると無効になる可能性

---

## アンチパターン詳細

### 1. 時間依存情報の問題

#### 問題の本質

スキルに時間依存の情報を含めると、時間経過で陳腐化します。

悪い例:
```markdown
## 最新機能（2025年11月時点）

新しいオプション `--experimental` が追加されました。
このガイドは2025年12月まで有効です。
来年1月のリリースで変更予定です。
```

なぜ悪いか:
- 2026年には情報が古くなる
- メンテナンスが必要
- ユーザーが混乱する

良い例:
```markdown
## 実験的機能

オプション `--experimental` が利用可能です（バージョン1.5.0以降）。

最新情報:
- 公式ドキュメント: [リンク]
- リリースノート: [リンク]
```

改善ポイント:
- バージョン番号で管理
- 外部リンクで最新情報を参照
- 時間表現を避ける

#### 検出方法

```bash
# 時間依存表現を検索
grep -E "202[0-9]年|[0-9]+月|来年|今年|最新" SKILL.md

# 見つかった場合は修正を検討
```

### 2. 矛盾した用語の検出と修正

#### 問題の本質

同じ概念を異なる用語で表現すると、理解が困難になります。

悪い例:
```markdown
## メインセクション

プライマリセクションには...
トップセクションは...
第一セクションでは...
```

3つの異なる用語が同じ意味で使用されている。

良い例:
```markdown
## メインセクション

メインセクションには...
各メインセクションは...
メインセクションでは...
```

統一された用語を使用。

#### 検出方法

1. 用語リストを作成:
```markdown
| 概念 | 使用する用語 | 避ける用語 |
|------|------------|-----------|
| 主要セクション | メインセクション | プライマリ、トップ、第一 |
| 補助セクション | サブセクション | 副、セカンダリ |
| コード例 | 例、サンプル | デモ、実例 |
```

2. スクリプトで検証:
```bash
# 矛盾する用語をチェック
grep -E "プライマリ|トップ|第一" SKILL.md
# 見つかったら "メインセクション" に統一
```

### 3. パス表記の統一

#### Windowsスタイルパスの問題

バックスラッシュ `\` はエスケープ文字として解釈される可能性があります。

悪い例:
```
C:\Users\user\project\file.md
src\components\Button.tsx
```

良い例:
```
/Users/user/project/file.md (Unix系)
C:/Users/user/project/file.md (Windows互換)
src/components/Button.tsx
```

#### 自動修正

```bash
# バックスラッシュをスラッシュに変換
sed -i 's/\\/\//g' SKILL.md

# 確認
grep '\\' SKILL.md
# 何も出力されなければOK
```

### 4. 選択肢提示のベストプラクティス

#### 問題: 多数の選択肢

多すぎる選択肢はClaude（とユーザー）の判断を困難にします。

悪い例:
```markdown
## セクション構造の選択

以下から選択してください:
1. オプションA: H1 → H2 → H3 → H4 → H5 → H6
2. オプションB: H1 → H2 → H3 → H4 → H5
3. オプションC: H1 → H2 → H3 → H4
4. オプションD: H1 → H2 → H3
5. オプションE: H1 → H2
6. オプションF: カスタム構造
```

6つの選択肢は多すぎる。

良い例:
```markdown
## 推奨セクション構造

基本構造（推奨）:
- H1: ドキュメントタイトル（1つのみ）
- H2: メインセクション
- H3: サブセクション
- H4以降: 必要に応じて使用（推奨しない）

特殊ケース:
- 非常に複雑な文書: H4まで許容
- シンプルな文書: H2-H3のみ
```

改善ポイント:
- デフォルト（推奨）を明示
- 選択肢を2-3個に制限
- 理由を説明

### 5. ネストされた参照構造の回避

#### 問題の本質

Claudeはネストされた参照を完全に読み込まない可能性があります。

悪い例（2段階ネスト）:
```
SKILL.md:
"詳細は resources/guide.md を参照"

resources/guide.md:
"さらに詳しくは resources/details/spec.md を参照"

resources/details/spec.md:
[実際の詳細情報]
```

Claudeが spec.md を読み込まない可能性がある。

良い例（1段階のみ）:
```
SKILL.md:
"基本ガイドは resources/guide.md を参照"
"詳細仕様は resources/spec.md を参照"

resources/guide.md:
[基本情報、外部参照なし]

resources/spec.md:
[詳細情報、外部参照なし]
```

#### 実装チェック

```bash
# resources/ 内のファイルが他のファイルを参照していないか確認
grep -r "resources/" resources/

# 出力があればネスト構造の可能性
# 手動で確認して修正
```

---

## Progressive Disclosureの詳細実装

### 基本原則

1. 重要情報はSKILL.mdに含める
2. 詳細情報は resources/ に分離
3. 参照は1段階のみ
4. 参照先を明示的に記載

### 実装パターンの詳細

#### パターン1: 高レベルガイド + 参照ファイル

使用場面: 基本ルールは簡単だが、例外や詳細が多い場合

SKILL.md:
```markdown
## 命名規則

基本ルール:
1. 変数名: camelCase
2. 定数名: UPPER_SNAKE_CASE
3. クラス名: PascalCase

例外や言語固有の詳細は `resources/REFERENCE.md` を参照してください。
```

resources/REFERENCE.md:
```markdown
## 命名規則の詳細

### TypeScript固有のルール
### Python固有のルール
### 例外ケース
### 歴史的背景
```

メリット:
- SKILL.mdが簡潔
- 必要に応じて詳細を参照
- トークン効率が良い

#### パターン2: ドメイン別構成

使用場面: 複数の独立したドメインをカバーする場合

SKILL.md:
```markdown
## フロントエンド規約

クイックリファレンス:
- コンポーネント: 関数コンポーネント + アロー関数
- スタイリング: Tailwind CSS
- 状態管理: useState（ローカル）、Jotai（グローバル）

詳細: `resources/frontend-guidelines.md`

## バックエンド規約

クイックリファレンス:
- API: tRPC
- 認証: Auth0
- データベース: Prisma + PostgreSQL

詳細: `resources/backend-guidelines.md`
```

resources/:
```
frontend-guidelines.md
backend-guidelines.md
database-guidelines.md
```

メリット:
- ドメインごとに整理
- 必要な部分のみ参照
- メンテナンスが容易

#### パターン3: 条件付き詳細情報

使用場面: 基本は簡単だが、特定の条件下で詳細が必要な場合

SKILL.md:
```markdown
## ファイル構造

基本構造:
\`\`\`
src/
├── components/
├── pages/
└── utils/
\`\`\`

大規模プロジェクトの場合: `resources/large-project-structure.md` を参照
マイクロサービスの場合: `resources/microservices-structure.md` を参照
```

メリット:
- 一般的なケースは簡潔
- 特殊ケースは分離
- 柔軟性が高い

### 参照の書き方

#### 明示的な参照

良い例:
```markdown
詳細は `resources/REFERENCE.md` を参照してください。
```

悪い例:
```markdown
詳細はREFERENCEを参照。
```

理由: Claudeがファイルパスを正確に認識できる。

#### セクション指定付き参照（オプション）

```markdown
コードレビューのベストプラクティスは `resources/REFERENCE.md` の「レビュー手順」セクションを参照してください。
```

Claudeがセクションを直接参照しやすくなる。

---

## Frontmatter完全リファレンス

### 必須項目

#### name

```yaml
name: skill-name
```

要件:
- kebab-case（小文字 + ハイフン）
- 64文字以内
- 一意である必要がある

例:
```yaml
# 良い例
name: markdown-style-rules
name: creating-skills
name: code-review-checklist

# 悪い例
name: Markdown_Style_Rules  # スネークケース
name: markdown style rules   # スペース
name: very-long-and-descriptive-skill-name-that-exceeds-sixty-four-characters  # 長すぎ
```

#### description

```yaml
description: Concise description (max 200 chars)
```

要件:
- 200文字以内
- スキルの目的を明確に
- 検索に適したキーワードを含める

例:
```yaml
# 良い例
description: Guide for creating custom Claude Code skills with best practices and step-by-step instructions

description: Kubernetes & Markdown style guide with comprehensive formatting rules

# 悪い例
description: This is a skill  # 短すぎ、不明確
description: A comprehensive and detailed guide covering all aspects of creating, maintaining, and optimizing custom Claude Code skills including best practices, common pitfalls, troubleshooting strategies, and much more...  # 長すぎ
```

### 推奨項目

#### version

```yaml
version: 1.0.0
```

セマンティックバージョニング:
- MAJOR.MINOR.PATCH
- MAJOR: 互換性のない変更
- MINOR: 後方互換性のある機能追加
- PATCH: 後方互換性のあるバグ修正

例:
```yaml
version: 1.0.0  # 初版
version: 1.1.0  # 機能追加
version: 1.1.1  # バグ修正
version: 2.0.0  # 大きな変更
```

#### category

```yaml
category: documentation
```

推奨カテゴリ:
- documentation: ドキュメント関連
- development: 開発プロセス
- testing: テスト関連
- deployment: デプロイ関連
- security: セキュリティ関連

カスタムカテゴリも可能。

### オプション項目

#### keywords

```yaml
keywords: [markdown, style, guide, formatting]
```

用途:
- スキル検索の改善
- 関連スキルの発見

推奨:
- 3-7個のキーワード
- 具体的で検索しやすい単語

#### dependencies

```yaml
dependencies: python>=3.8, nodejs>=22.14.0
```

用途:
- 必要なソフトウェアバージョンの明示
- 環境要件の文書化

形式:
- カンマ区切り
- バージョン指定は `>=`, `==`, `<` など

#### author

```yaml
author: K-shir0
```

用途:
- メンテナー情報
- 問い合わせ先

### 完全な例

```yaml
---
# 必須
name: advanced-typescript-patterns
description: Comprehensive TypeScript advanced patterns and best practices for large-scale applications

# 推奨
version: 1.2.3
category: development

# オプション
keywords: [typescript, patterns, advanced, architecture]
dependencies: typescript>=5.0.0, nodejs>=22.14.0
author: Platform Team
---
```

---

## トラブルシューティング詳細

### スキルが認識されない

#### 原因1: ディレクトリ構造の誤り

正しい構造:
```
.claude/skills/<skill-name>/SKILL.md
```

よくある間違い:
```
.claude/skills/<skill-name>.md          # ディレクトリではなくファイル
.claude/skills/<skill-name>/skill.md    # 小文字
.claude/skills/<skill-name>/README.md   # 名前が違う
```

確認方法:
```bash
ls -la .claude/skills/*/
```

#### 原因2: Frontmatterの構文エラー

よくあるエラー:
```yaml
# 間違い: インデントが不正
---
  name: skill-name
description: text
---

# 間違い: クォートが不足
---
name: skill-name
description: Text with: colon
---

# 正しい
---
name: skill-name
description: "Text with: colon"
---
```

確認方法:
```bash
head -n 10 .claude/skills/<skill-name>/SKILL.md
```

YAMLパーサーでチェック（オプション）:
```bash
python3 -c "import yaml; yaml.safe_load(open('.claude/skills/<skill-name>/SKILL.md').read().split('---')[1])"
```

#### 原因3: ファイルパーミッション

まれにパーミッション問題で読み込めない場合:

```bash
# 確認
ls -l .claude/skills/<skill-name>/SKILL.md

# 修正（必要に応じて）
chmod 644 .claude/skills/<skill-name>/SKILL.md
```

### パフォーマンス問題

#### 症状: スキル読み込みが遅い

原因:
- SKILL.mdが大きすぎる（500行以上）
- 画像や大きなファイルを含んでいる

対策:
1. ファイルサイズを確認:
```bash
wc -l .claude/skills/<skill-name>/SKILL.md
ls -lh .claude/skills/<skill-name>/SKILL.md
```

2. 500行以上なら分割:
```bash
# 前半を残す
head -n 500 SKILL.md > SKILL_temp.md

# 後半を resources/REFERENCE.md に
tail -n +501 SKILL.md > resources/REFERENCE.md

# SKILL.md を更新（参照を追加）
mv SKILL_temp.md SKILL.md
echo "\n詳細は \`resources/REFERENCE.md\` を参照してください。" >> SKILL.md
```

#### 症状: トークン消費が多い

原因:
- 冗長な説明
- 重複した情報
- 不要な例

対策:
1. 重複チェック:
```bash
# 同じ単語が連続していないか
grep -E '(\b\w+\b)\s+\1' SKILL.md
```

2. 冗長な表現を簡潔に:
```markdown
# Before
このセクションでは、スキルを作成する際に知っておくべき重要な情報について詳しく説明します。

# After
このセクションではスキル作成の重要な情報を説明します。
```

### リンク切れ

#### 内部リンク（アンカー）の問題

よくある間違い:
```markdown
# 間違い
## 目次
[セクション1](#section-1)

## 1. セクション1
[内容]

# 正しい
## 目次
[セクション1](#1-セクション1)

## 1. セクション1
[内容]
```

確認方法:
```bash
# アンカーを抽出
grep -o '#[a-z0-9-]*' SKILL.md

# 見出しを抽出して比較
grep '^##' SKILL.md
```

#### 外部リンクの検証

定期的に外部リンクを確認:

```bash
# URLを抽出
grep -oE 'https?://[^\s\)]+' SKILL.md

# 各URLをチェック（手動またはスクリプト）
```

自動チェックスクリプト（オプション）:
```bash
#!/bin/bash
for url in $(grep -oE 'https?://[^\s\)]+' SKILL.md); do
    if curl -s --head "$url" | grep "200 OK" > /dev/null; then
        echo "✓ $url"
    else
        echo "✗ $url"
    fi
done
```

---

## まとめ

スキル作成における重要ポイント:

1. Progressive Disclosureで500行以内に収める
2. 参照は1段階のみ
3. アンチパターンを避ける
4. Frontmatterを正確に設定
5. 定期的にメンテナンス

このリファレンスに従うことで、高品質で保守しやすいスキルを作成できます。

---

バージョン: 1.0.0
最終更新: 2025-11-14

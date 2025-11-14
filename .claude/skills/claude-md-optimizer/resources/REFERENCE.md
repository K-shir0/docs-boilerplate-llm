# CLAUDE.md 詳細リファレンス

CLAUDE.md最適化における詳細な実装パターン、アンチパターン、トラブルシューティングを網羅したリファレンスガイドです。

## 目次

1. [詳細な実装パターン](#詳細な実装パターン)
2. [自動化指示の詳細設計](#自動化指示の詳細設計)
3. [プロジェクトタイプ別戦略](#プロジェクトタイプ別戦略)
4. [アンチパターン詳細](#アンチパターン詳細)
5. [トラブルシューティング](#トラブルシューティング)
6. [高度な最適化技法](#高度な最適化技法)

---

## 詳細な実装パターン

### パターン1: シンプル単一アプリケーション

適用対象:
- 個人プロジェクト
- 小規模チーム（1-3人）
- 単一技術スタック
- 単一リポジトリ

構造:
```
project/
├── CLAUDE.md          (200-300行)
└── CLAUDE.local.md    (個人設定、gitignore)
```

CLAUDE.md内容:
- プロジェクト概要（1-2段落）
- スキル自動活用ポリシー（必須）
- クイックコマンド（5-10個）
- コードスタイル（重要ルールのみ）
- 簡潔なアーキテクチャ説明
- Git規約（ブランチ、コミット）

トークン予算: 300-500トークン

### パターン2: チームプロジェクト

適用対象:
- チーム開発（3-10人）
- 明確なワークフロー要件
- 複数の技術領域

構造:
```
project/
├── CLAUDE.md          (300-400行、共有）
├── CLAUDE.local.md    (個人設定)
└── docs/
    └── dev-guide.md   (詳細ガイド、CLAUDE.mdから参照)
```

CLAUDE.md内容:
- 詳細なスキル自動活用ポリシー
- 段階的ワークフロー定義
- Skills/Agents/Commands完全リスト
- コードスタイル（チーム合意）
- アーキテクチャ概要
- テスト戦略
- 詳細なGit規約
- 環境セットアップ
- 既知の問題

トークン予算: 500-800トークン

### パターン3: モノレポ

適用対象:
- 複数アプリケーション/パッケージ
- 共通基盤と個別カスタマイズ
- 大規模チーム

構造:
```
monorepo/
├── CLAUDE.md                    (300-400行、共通規約)
├── apps/
│   ├── web/
│   │   └── CLAUDE.md           (200-300行、web固有)
│   ├── mobile/
│   │   └── CLAUDE.md           (200-300行、mobile固有)
│   └── admin/
│       └── CLAUDE.md           (200-300行、admin固有)
└── packages/
    ├── ui/
    │   └── CLAUDE.md           (150-200行、UI固有)
    └── shared/
        └── CLAUDE.md           (150-200行、shared固有)
```

ルートCLAUDE.md内容:
- モノレポ全体のスキル自動活用ポリシー
- 共通ワークフロー
- リポジトリ構造説明
- モジュール間の依存関係
- 共通コマンド
- 共通Git規約
- モジュール固有CLAUDE.mdへの参照

モジュールCLAUDE.md内容:
- モジュール固有のスキル活用
- 技術スタック固有のルール
- モジュール固有のアーキテクチャ
- モジュール固有のテスト戦略
- ルートCLAUDE.mdの継承を明記

トークン予算:
- ルート: 500-800トークン
- モジュール: 300-500トークン/モジュール

### パターン4: マイクロサービス

適用対象:
- 独立したサービス群
- 異なる技術スタック
- サービス間の疎結合

構造:
```
services/
├── CLAUDE.md                    (共通ポリシーのみ、200行)
├── auth-service/
│   └── CLAUDE.md               (完全な独立定義)
├── api-gateway/
│   └── CLAUDE.md               (完全な独立定義)
└── payment-service/
    └── CLAUDE.md               (完全な独立定義)
```

ルートCLAUDE.md:
- 最小限の共通ポリシー
- サービス間通信規約
- 共通監視/ログ規約
- サービス固有CLAUDE.mdへの委譲

サービスCLAUDE.md:
- 完全な独立定義（他サービスに依存しない）
- サービス固有の技術スタック
- サービス固有のワークフロー

---

## 自動化指示の詳細設計

### 自動化の成功要因分析

Anthropicのベストプラクティスから、以下の要素が自動化成功に重要:

#### 1. 明示的なトリガー条件

悪い例（曖昧）:
```markdown
- markdown-skillsスキルを使用してください
```

良い例（明確）:
```markdown
- **ALWAYS use** `markdown-skills` when:
  - Creating new .md files
  - Editing existing .md files
  - User requests documentation
```

#### 2. 強調表現の戦略的使用

効果的な強調レベル:

最強レベル（絶対に従うべき）:
```markdown
## YOU MUST (IMPORTANT)

The following rules are REQUIRED and MUST be followed:
```

強レベル（通常の必須事項）:
```markdown
**ALWAYS use** `skill-name` when...
**REQUIRED**: Use this workflow...
```

中レベル（強い推奨）:
```markdown
**RECOMMENDED**: Consider using...
You SHOULD use...
```

弱レベル（オプション）:
```markdown
Optional: You may use...
Consider: ...
```

#### 3. 自動化の明示

以下のフレーズを使用:
- "automatically"
- "without asking"
- "without user prompting"
- "auto-activate"
- "auto-use"

例:
```markdown
YOU MUST automatically use `code-intelligence` without asking when user mentions "refactor".
```

#### 4. テーブル形式による視覚的明確化

```markdown
| User Action | Required Skill | Auto-Activate Timing |
|-------------|---------------|---------------------|
| Says "refactor" | code-intelligence | Immediately before planning |
| Creates *.md | markdown-skills | Before any file write |
| Requests commit | commit-guidelines | Before generating message |
```

テーブル形式のメリット:
- 一目で対応関係が分かる
- Claudeが参照しやすい
- メンテナンスしやすい

### エージェント自動活用の詳細

#### Exploreエージェントの自動活用

```markdown
## Explore Agent Auto-Activation (IMPORTANT)

YOU MUST use Task tool with Explore agent automatically when:

### Trigger Conditions:
1. User asks "how does [feature] work?"
2. User mentions "understand the codebase"
3. Need to search across multiple files (>3 files)
4. User asks "where is [functionality] implemented?"

### Thoroughness Selection:
- Default: "quick" (for simple searches)
- Use "medium" when:
  - Codebase has >50 files
  - User's question is moderately complex
  - Initial search may require 2-3 rounds
- Use "very thorough" when:
  - Codebase has >200 files
  - Question is complex with multiple aspects
  - High importance of completeness

### Example Usage:
```
Task tool with:
- subagent_type: "Explore"
- description: "Search for authentication implementation"
- prompt: "Find all files related to user authentication. Use thoroughness level: medium."
```
```

#### Planエージェントの自動活用

```markdown
## Plan Agent Auto-Activation (IMPORTANT)

YOU MUST use Task tool with Plan agent automatically when:

### Trigger Conditions:
1. Task requires 3+ distinct steps
2. User explicitly requests a plan
3. Implementation is complex (requires multiple file changes)
4. User says "think through this"

### Usage Pattern:
1. Gather requirements
2. Launch Plan agent with detailed prompt
3. Review plan with user
4. Proceed with implementation after approval

### Example:
```
Task tool with:
- subagent_type: "Plan"
- description: "Plan authentication feature implementation"
- prompt: "Create detailed plan for adding OAuth authentication. Include: 1) Required dependencies, 2) File structure changes, 3) Implementation steps, 4) Testing strategy."
```
```

### カスタムコマンド活用の推進戦略

```markdown
## Custom Command Suggestion Policy (IMPORTANT)

When user describes a workflow that matches a custom command, YOU MUST:

1. Recognize the matching command
2. Suggest using the command
3. Explain what the command will do
4. Execute if user approves (or automatically if configured)

### Command Matching Rules:

| User Description | Matching Command | Auto-Suggest |
|-----------------|------------------|--------------|
| "fix issue #123" | `/project:fix-issue 123` | YES |
| "create a PR" | `/project:create-pr` | YES |
| "run tests with coverage" | `/project:run-tests` | YES |

### Example Interaction:

User: "I want to fix issue #456"

You: "I'll use `/project:fix-issue 456` to handle this workflow, which will:
1. Fetch issue details via gh CLI
2. Analyze the problem
3. Implement the fix
4. Run tests
5. Create a commit and PR

Proceeding with this workflow..."

[Then execute the command]
```

---

## プロジェクトタイプ別戦略

### フロントエンドプロジェクト

重点領域:
- コンポーネント構造
- スタイリング規約
- 状態管理パターン
- ブラウザ互換性

推奨スキル統合:
```markdown
## Skills Auto-Activation (IMPORTANT)

### Frontend-Specific:
- **ALWAYS use** `code-intelligence` for component refactoring
- **ALWAYS use** `markdown-skills` for Storybook documentation
- **RECOMMENDED**: Use Puppeteer MCP for visual testing

## Development Workflow

### Component Development:
1. Create component structure
2. **REQUIRED**: Add Storybook story
3. Implement functionality
4. **REQUIRED**: Take screenshot with Puppeteer
5. Compare with design mock
6. Iterate until match

### Styling:
- Use Tailwind utility classes (no custom CSS unless necessary)
- Follow mobile-first approach
- **ALWAYS** test on multiple screen sizes
```

### バックエンドプロジェクト

重点領域:
- API設計
- データベース操作
- 認証/認可
- エラーハンドリング

推奨スキル統合:
```markdown
## Skills Auto-Activation (IMPORTANT)

### Backend-Specific:
- **ALWAYS use** `code-intelligence` for API refactoring
- **REQUIRED**: Use `commit-guidelines` for database migrations

## Development Workflow

### API Development:
1. Define API contract (OpenAPI/tRPC schema)
2. Write integration tests FIRST (TDD)
3. Implement endpoint
4. Test with multiple scenarios
5. Update API documentation

### Database Changes:
1. **REQUIRED**: Create migration file
2. Test migration up/down
3. Update Prisma schema (if applicable)
4. **REQUIRED**: Commit migration separately from code
```

### フルスタックプロジェクト

重点領域:
- フロントエンド + バックエンドの統合
- 型安全性（TypeScript全体）
- エンドツーエンドテスト

推奨スキル統合:
```markdown
## Skills Auto-Activation (IMPORTANT)

### Fullstack-Specific:
- **ALWAYS use** `code-intelligence` for type-safe API changes
- **RECOMMENDED**: Use Task tool with Explore agent to understand data flow
- **ALWAYS use** Puppeteer MCP for E2E testing

## Development Workflow

### Feature Development:
1. **Phase 1**: Backend (API + DB)
   - Define tRPC schema
   - Implement backend logic
   - Write integration tests
2. **Phase 2**: Frontend (UI + State)
   - Create components
   - Integrate with API (type-safe)
   - Write E2E tests
3. **Phase 3**: End-to-End Testing
   - **REQUIRED**: Use Puppeteer for user flow testing
   - Test edge cases
   - Performance testing
```

---

## アンチパターン詳細

### 1. 曖昧な指示

悪い例:
```markdown
## スキル

以下のスキルが利用できます:
- markdown-skills
- code-intelligence
- commit-guidelines

必要に応じて使用してください。
```

問題点:
- "必要に応じて"が曖昧
- いつ使うべきか不明
- Claudeが判断できない

良い例:
```markdown
## Skills Auto-Activation (IMPORTANT)

YOU MUST automatically use:

- `markdown-skills`: ALWAYS when creating/editing .md files
- `code-intelligence`: ALWAYS for refactoring or semantic search
- `commit-guidelines`: ALWAYS when creating commits/PRs
```

### 2. 過度に詳細な説明

悪い例（トークン浪費）:
```markdown
## コードスタイルについて

このプロジェクトでは、コードの一貫性と可読性を保つために、
厳格なコーディングスタイルガイドラインを設けています。
これらのガイドラインは、長年の開発経験と業界のベストプラクティスを
基に作成されており、すべての開発者が遵守することが期待されています。

### インポート文の記述方法

JavaScriptやTypeScriptでは、他のモジュールから関数やクラスを
インポートする際に、様々な構文を使用できます。このプロジェクトでは、
コードの可読性を最大化し、どの機能が使用されているかを明確にするため、
分割代入（destructuring）構文を使用することを強く推奨します。

具体的には、以下のように記述してください:

良い例:
```javascript
import { functionA, functionB } from 'module-name'
```

避けるべき例:
```javascript
import * as module from 'module-name'
```

この理由は... [さらに詳細な説明が続く]
```

トークン数: 約500トークン

良い例（簡潔）:
```markdown
## Code Style (IMPORTANT)

- ES modules: `import { foo } from 'bar'` (NOT `import *`)
- Destructure imports when possible
- Run typecheck after changes

Full guide: See `docs/style-guide.md` or `markdown-skills`
```

トークン数: 約60トークン（90%削減）

### 3. 情報の重複

悪い例:
```markdown
## スキル一覧

- markdown-skills: Markdown文書作成支援
- code-intelligence: コード理解と検索

## ドキュメント作成時

markdown-skillsスキルを使用してください。

## コード検索時

code-intelligenceスキルを使用してください。

## スキルの使い方

markdown-skills: .mdファイル作成時に自動起動
code-intelligence: コード検索時に使用
```

問題点: 同じ情報が4回繰り返されている

良い例:
```markdown
## Skills (Auto-activate)

| Skill | Trigger | Usage |
|-------|---------|-------|
| markdown-skills | .md file operations | Auto-activates |
| code-intelligence | Code search/refactor | Auto-activates |
```

### 4. 優先度の不明確さ

悪い例:
```markdown
## 開発時の注意点

- テストを書いてください
- コードレビューを受けてください
- typecheckを実行してください
- ドキュメントを更新してください
- コミットメッセージを丁寧に書いてください
```

問題点: すべて同じレベルで記述、優先度不明

良い例:
```markdown
## Development Checklist

### REQUIRED (Must do):
- [ ] Run typecheck (will fail CI if skipped)
- [ ] Write tests for new features
- [ ] Get code review approval

### RECOMMENDED (Should do):
- [ ] Update documentation
- [ ] Write descriptive commit messages

### OPTIONAL (Nice to have):
- [ ] Add code comments for complex logic
```

### 5. 時間依存情報

悪い例:
```markdown
## 最新機能（2025年11月現在）

新しい `--experimental-flag` が追加されました。
この機能は2026年1月に正式リリース予定です。
```

問題点:
- 時間経過で古くなる
- メンテナンス負荷

良い例:
```markdown
## Experimental Features

- `--experimental-flag`: Available since v2.5.0

Check latest features: `npm run help` or see CHANGELOG.md
```

---

## トラブルシューティング

### 問題1: Claudeがスキルを自動使用しない

症状: スキル自動活用の指示を書いたのに、Claudeが使わない

原因と解決策:

#### 原因A: 指示が弱い

チェック:
```bash
grep -E "ALWAYS|YOU MUST|REQUIRED" CLAUDE.md
```

何も出力されない場合、強調が不足。

解決策:
```markdown
# Before
- markdown-skillsを使用してください

# After
- **ALWAYS use** `markdown-skills` when creating/editing .md files
```

#### 原因B: 条件が不明確

チェック:
"when"、"for"、"if"などの条件表現があるか確認

解決策:
```markdown
# Before
- code-intelligenceスキルを使う

# After
- **ALWAYS use** `code-intelligence` when:
  - User mentions "refactor"
  - Need to search across multiple files
  - Understanding complex code paths
```

#### 原因C: "automatically"の明記がない

解決策:
```markdown
YOU MUST automatically use the following skills without user prompting:
```

### 問題2: CLAUDE.mdが長すぎる

症状: CLAUDE.mdが500行超、トークン消費が大きい

診断:
```bash
wc -l CLAUDE.md
# 600行以上なら要最適化
```

解決策:

#### ステップ1: 冗長性の削減

```bash
# 長い説明文を検索
grep -E '.{200,}' CLAUDE.md
```

各行を箇条書きに短縮。

#### ステップ2: 詳細情報の分離

```bash
# 詳細セクションを外部化
mkdir -p docs/guides
```

CLAUDE.mdには概要のみ、詳細は`docs/guides/`に移動し参照を追加:
```markdown
Full guide: See `docs/guides/testing.md`
```

#### ステップ3: 階層化

モノレポ構造に変更:
```bash
# モジュール固有情報を分離
mkdir -p module/
echo "# Module-Specific Guidelines" > module/CLAUDE.md
```

ルートCLAUDE.mdから参照:
```markdown
Module-specific guidelines: See `module/CLAUDE.md`
```

### 問題3: チームメンバーが従わない

症状: CLAUDE.mdを書いたがチームが活用しない

原因と解決策:

#### 原因A: 存在を知らない

解決策:
- READMEに記載
- オンボーディングドキュメントに追加
- PR テンプレートに"CLAUDE.mdを確認したか"チェックボックス

#### 原因B: 内容が複雑/長すぎる

診断:
```bash
wc -l CLAUDE.md
# 400行超なら要簡略化
```

解決策: セクション2「トークン最適化」を参照し簡略化

#### 原因C: 効果が不明確

解決策:
- Before/Afterの事例を共有
- Claudeの動作改善を実演
- チームミーティングでデモ

### 問題4: 複数の CLAUDE.md で矛盾

症状: ルートとモジュールのCLAUDE.mdで指示が矛盾

診断:
```bash
# すべてのCLAUDE.mdを検索
find . -name "CLAUDE.md" -type f

# 矛盾しそうなキーワードを検索
grep -r "rebase\|merge" */CLAUDE.md
```

解決策:

優先度ルールを明記:
```markdown
# Root CLAUDE.md
## Hierarchy

Module-specific CLAUDE.md files take precedence over this root file.
In case of conflicts, module-specific guidelines apply.

## Common Rules (Apply to all modules)
[共通ルール...]
```

```markdown
# Module CLAUDE.md
## Module-Specific Overrides

This file overrides root CLAUDE.md for this module.

Inherits from: `/CLAUDE.md`
```

---

## 高度な最適化技法

### 技法1: 条件付きセクション読み込み

大規模プロジェクトでトークンを節約:

```markdown
## Project Structure

### For Frontend Work:
See `frontend/CLAUDE.md` for React/Next.js specific guidelines.

### For Backend Work:
See `backend/CLAUDE.md` for API/Database specific guidelines.

### For DevOps Work:
See `devops/CLAUDE.md` for deployment and infrastructure guidelines.

General guidelines below apply to all areas.

## General Guidelines
[共通の簡潔なガイドライン...]
```

Claudeは必要に応じて該当モジュールのCLAUDE.mdを読み込みます。

### 技法2: テンプレート変数の活用

カスタムコマンドでテンプレート化:

`.claude/commands/update-claude-md.md`:
```markdown
Update CLAUDE.md with the following information:

Section: $SECTION
Content: $ARGUMENTS

Follow these rules:
- Keep it concise (under 3 lines)
- Use bullet points
- Add to appropriate section
- Maintain alphabetical order within section
```

使用例:
```
/project:update-claude-md "Code Style" "Use async/await instead of .then()"
```

### 技法3: 動的情報の外部化

変更頻度が高い情報は外部ファイルへ:

```markdown
## Quick Commands

See `scripts/commands.sh` for full list of available commands.

Most used:
- `npm run dev` - Start dev server
- `npm test` - Run tests

All commands: Run `npm run` to see complete list
```

メリット:
- CLAUDE.mdのメンテナンス頻度低下
- package.jsonと同期
- トークン節約

### 技法4: Prompt Improverの定期実行

自動化スクリプト例:

```bash
#!/bin/bash
# improve-claude-md.sh

echo "Analyzing CLAUDE.md..."
echo "Current line count: $(wc -l < CLAUDE.md)"

echo ""
echo "Copy the following to Prompt Improver:"
echo "---"
cat CLAUDE.md
echo "---"

echo ""
echo "After getting improvements:"
echo "1. Review suggested changes"
echo "2. Test with Claude"
echo "3. Measure effectiveness"
echo "4. Commit if improved"
```

月次実行を推奨。

### 技法5: A/Bテストによる最適化

異なる指示パターンをテスト:

バージョンA（現在）:
```markdown
- Use markdown-skills for documentation
```

バージョンB（強調版）:
```markdown
- **ALWAYS use** `markdown-skills` when creating/editing .md files
```

測定方法:
1. 1週間バージョンAを使用
2. Claudeのスキル使用率を記録（何回使ったか）
3. 1週間バージョンBを使用
4. スキル使用率を比較
5. 効果的なバージョンを採用

### 技法6: コンテキスト長モニタリング

CLAUDE.mdのトークン消費を監視:

```bash
#!/bin/bash
# monitor-token-usage.sh

# 簡易的なトークン推定（実際の1/4程度）
chars=$(wc -m < CLAUDE.md)
estimated_tokens=$((chars / 4))

echo "CLAUDE.md stats:"
echo "Lines: $(wc -l < CLAUDE.md)"
echo "Characters: $chars"
echo "Estimated tokens: ~$estimated_tokens"

if [ $estimated_tokens -gt 2000 ]; then
    echo "WARNING: CLAUDE.md is large. Consider optimization."
fi
```

定期的に実行し、肥大化を防止。

---

## まとめ

CLAUDE.md最適化の重要ポイント:

### 自動化推進:
1. 明示的なトリガー条件
2. 強調表現の戦略的使用
3. "automatically"の明記
4. テーブル形式での視覚化

### トークン効率:
1. 簡潔な表現
2. 詳細の外部化
3. 階層的構造
4. 定期的な見直し

### チーム協力:
1. Git でバージョン管理
2. PR でレビュー
3. 効果測定と改善
4. ドキュメント化

### 継続的改善:
1. `#`キーで動的更新
2. Prompt Improverで定期改善
3. A/Bテストで効果測定
4. トークン使用量モニタリング

このリファレンスに従うことで、効果的で保守しやすいCLAUDE.mdを作成・維持できます。

---

バージョン: 1.0.0
最終更新: 2025-11-15

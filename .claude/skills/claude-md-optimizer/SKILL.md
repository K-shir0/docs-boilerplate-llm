---
name: claude-md-optimizer
description: Comprehensive guide for creating, editing, and optimizing CLAUDE.md files with automatic skills/agents/commands activation strategies. Supports Japanese projects.
version: 1.1.0
category: development
keywords: [claude-md, optimization, skills-integration, workflow, automation, japanese]
---

# CLAUDE.md Optimizer

Claude Codeの動作を最適化するCLAUDE.mdファイルの作成・編集・最適化を支援するスキルです。特に、**Claudeが自動的にskills/agents/commandsを使用するように促す指示設計**に重点を置いています。

## 重要な原則

**"CLAUDE.mdはプロンプトである"** - CLAUDE.mdはClaudeへの永続的な指示です。プロンプトエンジニアリングの技法を適用し、反復改善してください。

**"簡潔性とトークン効率"** - 最も頻繁に必要な情報のみを含め、詳細は階層的に構造化してください。

**"自動化を促進"** - Claudeが明示的な指示なしでskills/agents/commandsを使用するように設計してください。

**"日本語プロジェクトでは日本語で記述"** - 日本語プロジェクトのCLAUDE.mdは日本語で作成してください。セクション見出し、指示文、説明はすべて日本語にします。ただし以下は英語のまま保持:
- スキル名（例: `markdown-skills`、`code-intelligence`）をバッククォートで囲む
- コマンド名（例: `/commit`）
- コードブロック内のコマンド例
- 技術用語は必要に応じて英語を併記（例: スキル（Skills）、エージェント（Agents））

## 目次

1. [CLAUDE.mdとは](#1-claudemdとは)
2. [自動スキル活用戦略](#2-自動スキル活用戦略)
3. [ファイル配置戦略](#3-ファイル配置戦略)
4. [推奨構造](#4-推奨構造)
5. [トークン最適化](#5-トークン最適化)
6. [実装パターン](#6-実装パターン)
7. [ベストプラクティス](#7-ベストプラクティス)
8. [実例](#8-実例)

---

## 1. CLAUDE.mdとは

### 1.1 基本概念

CLAUDE.mdは、Claudeが会話開始時に自動的に読み込む特別なファイルです。

目的:
- プロジェクト固有の知識と規約の文書化
- よく使うコマンドとツールの記録
- ワークフローと開発プロセスの標準化
- Skills/Agents/Commandsの活用促進
- Claudeの動作のカスタマイズ

### 1.2 なぜ最適化が重要か

最適化されたCLAUDE.mdは:
- Claudeの指示遵守率を向上させる
- トークン消費を削減する
- 開発効率を大幅に改善する
- チーム全体の一貫性を保証する
- 自動化ツール（skills/agents/commands）の活用を促進する

### 1.3 作成すべきタイミング

以下の場合にCLAUDE.md作成を検討:
- 新規プロジェクト開始時
- プロジェクト固有のルールが複数ある
- チームメンバーに共有したい規約がある
- Claudeに繰り返し同じ指示をしている
- カスタムskills/commands/MCPツールを導入した

---

## 2. 自動スキル活用戦略

### 2.1 自動化の核心原則

Claudeが**ユーザーの明示的な指示なし**でskills/agents/commandsを使用するようにするには、CLAUDE.md内で明確な自動化ルールを定義します。

重要な要素:
1. **明示的なトリガー条件**: いつスキルを使うべきか
2. **強調表現**: IMPORTANT, YOU MUST, REQUIRED, ALWAYS
3. **自動化の明記**: "without asking", "automatically"
4. **優先度の設定**: 必須 vs 推奨 vs オプション

### 2.2 自動化指示パターン

#### パターンA: タスクベースの自動トリガー

```markdown
## Skills Usage Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

### Documentation Tasks
- **ALWAYS use** `markdown-skills` when creating or editing .md files
- **ALWAYS use** `creating-skills` when creating new skills in .claude/skills/

### Code Tasks
- **ALWAYS use** `code-intelligence` for:
  - Any refactoring request
  - Understanding large codebases (>100 files)
  - Semantic code search needs

### Git Operations
- **ALWAYS use** `commit-guidelines` when:
  - User requests a commit
  - Creating pull requests
  - Reviewing commit messages
```

このパターンは:
- タスクの種類ごとにスキルを紐付け
- "ALWAYS use"で自動化を強調
- 具体的な条件を列挙

#### パターンB: ワークフローベースの自動化

```markdown
## Development Workflow (YOU MUST FOLLOW)

Every development task follows this workflow:

### Phase 1: Exploration (REQUIRED)
1. **REQUIRED**: Use `code-intelligence` skill to understand context
2. Use Task tool with Explore agent for complex codebases
3. Read relevant files mentioned by the user

### Phase 2: Planning
1. Create a detailed plan
2. **REQUIRED**: Use appropriate domain-specific skills:
   - Documentation tasks → `markdown-skills`
   - Skill creation → `creating-skills`
3. Get user approval before proceeding

### Phase 3: Implementation
1. Follow code style guidelines (see below)
2. **RECOMMENDED**: Use Test-Driven Development
3. Auto-activate domain-specific skills as needed

### Phase 4: Commit & PR (REQUIRED)
1. **REQUIRED**: Use `commit-guidelines` skill
2. Generate conventional commits format
3. Include co-author attribution
```

このパターンは:
- 開発フローの各段階でスキル使用を指示
- REQUIRED/RECOMMENDEDで優先度を明示
- 段階的なチェックポイントを設定

#### パターンC: 条件付き自動実行テーブル

```markdown
## Automatic Skill Activation Rules

The following skills MUST be used automatically when conditions are met:

| Condition | Required Skill | When to Activate |
|-----------|---------------|------------------|
| User mentions "refactor" | code-intelligence | Immediately before planning |
| Creating/editing *.md | markdown-skills | Before any file write |
| User requests commit | commit-guidelines | Before generating commit message |
| Creating new skill | creating-skills | At planning phase |
| Codebase exploration needed | code-intelligence + Explore agent | During research phase |
| PR creation requested | commit-guidelines | Before generating PR description |
```

このパターンは:
- 視覚的に条件とスキルを対応付け
- タイミングを明確化
- 参照しやすいテーブル形式

### 2.3 エージェント自動活用の促進

```markdown
## Agent Usage Policy (IMPORTANT)

YOU MUST use Task tool with appropriate agents without asking:

### Explore Agent
**Auto-activate when:**
- User asks "how does X work?"
- Need to understand file structure
- Searching for patterns across multiple files

**Thoroughness levels:**
- "quick": Simple file searches (use by default)
- "medium": Moderate exploration (2-3 search rounds)
- "very thorough": Comprehensive analysis (complex codebases)

### Plan Agent
**Auto-activate when:**
- Task requires multiple steps (3+)
- User requests a plan
- Complex implementation needed

**Usage:**
Always use in plan mode for multi-step tasks before implementation.
```

### 2.4 カスタムコマンド活用の促進

```markdown
## Custom Commands (Use When Appropriate)

Available slash commands for common workflows:

- `/project:fix-issue <number>` - Auto-fix GitHub issue with full workflow
- `/project:create-pr` - Create PR with proper formatting and checks
- `/project:run-tests` - Execute test suite with coverage report

**YOU SHOULD suggest these commands** when users describe matching workflows.

Example:
- User: "I want to fix issue #123"
- You: "I'll use `/project:fix-issue 123` to handle this workflow."
```

### 2.5 MCP統合の促進

```markdown
## MCP Tools (Auto-use When Available)

The following MCP servers are configured:

### Puppeteer (UI Testing)
- **Auto-use** for screenshot requests
- **Auto-use** when user provides visual mock
- Workflow: implement → screenshot → compare → iterate

### GitHub MCP
- **Prefer `gh` CLI** but fallback to GitHub MCP
- Use for: PR comments, issue management, check status
```

---

## 3. ファイル配置戦略

### 3.1 配置場所の選択

CLAUDE.mdは複数の場所に配置可能で、自動的に階層化されます:

#### ① プロジェクトルート（最も一般的）

```
/path/to/project/CLAUDE.md
```

用途:
- プロジェクト全体に適用される規約
- チーム全体で共有する情報
- Gitにコミットして共有

推奨内容:
- プロジェクト概要
- 共通コマンド
- コードスタイル
- Git規約
- Skills/Agents/Commands参照

#### ② ホームディレクトリ（個人設定）

```
~/.claude/CLAUDE.md
```

用途:
- 全プロジェクト共通の個人設定
- 個人的なワークフロー
- プライベートな情報

推奨内容:
- 個人的なエイリアス
- 好みのワークフロー
- プライベートなMCPツール設定

#### ③ 親ディレクトリ（モノレポ）

```
/monorepo/CLAUDE.md          (ルート: 共通規約)
/monorepo/frontend/CLAUDE.md (フロントエンド固有)
/monorepo/backend/CLAUDE.md  (バックエンド固有)
```

動作:
- `/monorepo/frontend/`で実行時、両方のCLAUDE.mdを読み込み
- 階層的に適用（子が親を補完/上書き）

推奨構造:
- ルート: 全体的なアーキテクチャと共通ルール
- サブディレクトリ: 各モジュール固有の詳細

#### ④ ローカル設定ファイル

```
/path/to/project/CLAUDE.local.md
```

用途:
- Gitにコミットしない個人設定
- 実験的な指示
- ローカル環境固有の情報

推奨:
- `.gitignore`に追加

### 3.2 Progressive Disclosure戦略

階層的にコンテキストを提供する戦略:

```
~/.claude/CLAUDE.md           (最も一般的)
  ↓ 継承
/project/CLAUDE.md            (プロジェクト固有)
  ↓ 継承
/project/module/CLAUDE.md     (モジュール固有)
  ↓ オンデマンド読み込み
/project/module/sub/CLAUDE.md (詳細情報)
```

原則:
- 頻繁に必要な情報を上位に配置
- 詳細情報を下位に配置
- トークン消費を最小化

---

## 4. 推奨構造

### 4.1 基本テンプレート

```markdown
# Project: [プロジェクト名]

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for .md files
2. **Refactoring**: Always use `code-intelligence` for code changes
3. **Commits**: Always use `commit-guidelines` for git operations
4. **Skill Creation**: Always use `creating-skills` when making skills

## Development Workflow (YOU MUST FOLLOW)

[ワークフロー定義...]

## Quick Commands

- `npm run build` - Build the project
- `npm test` - Run tests
- `/project:fix-issue <num>` - Auto-fix GitHub issue

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: Documentation tasks
- `code-intelligence`: Refactoring and search
- `creating-skills`: Skill creation
- `commit-guidelines`: Git operations

### Custom Commands
- `/project:fix-issue`: GitHub issue workflow
- `/project:create-pr`: PR creation workflow

### Agents
- Explore: Use for codebase understanding (thoroughness: "medium")
- Plan: Use for multi-step tasks

### MCP Servers
- Puppeteer: UI testing and screenshots
- GitHub: PR/Issue management

## Code Style (IMPORTANT)

[具体的なスタイルルール...]

## Architecture

[主要ファイルとパターン...]

## Testing

- TDD preferred: Write tests first
- Run tests before committing
- Use `npm test` for unit tests

## Git Workflow

- Branch: `feature/` or `fix/` prefix
- Commits: Conventional Commits (via commit-guidelines skill)
- IMPORTANT: Rebase, not merge

## Environment Setup

[環境要件...]

## Known Issues

[既知の問題と回避策...]
```

### 4.2 セクション詳細

#### Skills Auto-Activation Policy

目的: Claudeに自動的にスキルを使用させる

必須要素:
- "IMPORTANT"または"YOU MUST"で強調
- "automatically"、"without user prompting"を明記
- 具体的な条件を列挙
- ALWAYSなどの絶対表現

#### Development Workflow

目的: 一貫した開発プロセスの実行

必須要素:
- 段階的なステップ（Phase 1, 2, 3...）
- 各段階でのスキル/エージェント使用指示
- REQUIRED/RECOMMENDEDで優先度明示

#### Available Tools

目的: 利用可能なツールの発見可能性向上

推奨構成:
- Skills: 自動活用可能なスキルのリスト
- Custom Commands: スラッシュコマンドの使用例
- Agents: エージェントと推奨設定
- MCP Servers: 統合済みツール

---

## 5. トークン最適化

### 5.1 最適化原則

CLAUDE.mdはプロンプトの一部として全会話に含まれます。トークン効率が重要です。

原則:
1. **簡潔に**: 冗長な説明を避ける
2. **参照で**: 詳細は外部ファイルやスキルへの参照
3. **頻度で**: 頻繁に必要な情報のみ含める
4. **階層化**: Progressive Disclosureを活用

### 5.2 Before/After例

#### Before（冗長）:

```markdown
## コードスタイル

このプロジェクトでは、コードの一貫性と可読性を保つために、
以下のコーディングスタイルガイドラインに従う必要があります。
これらのガイドラインは、チーム全体で合意されたものであり、
すべての開発者が遵守することが期待されています。

### インポート文について

インポート文を記述する際は、可能な限り分割代入（destructuring）
の構文を使用してください。これにより、コードの可読性が向上し、
どの関数やクラスが使用されているかが明確になります。

例えば:
- 良い例: import { foo, bar } from 'module'
- 悪い例: import * as module from 'module'

### モジュールシステムについて

このプロジェクトでは、ES modules（import/export）構文を使用します。
CommonJS（require/module.exports）は使用しないでください。
理由は、ES modulesがモダンなJavaScriptの標準であり、
Tree shakingなどの最適化が可能だからです。
```

トークン数: 約400トークン

#### After（最適化済み）:

```markdown
## Code Style (IMPORTANT)

- Use ES modules (import/export), NOT CommonJS (require)
- Destructure imports: `import { foo } from 'bar'`
- Run typecheck after changes
- Prefer single test execution for performance

Full style guide: See `markdown-skills` for documentation standards
```

トークン数: 約80トークン（80%削減）

改善ポイント:
- 箇条書きで簡潔に
- 理由の説明を削除（必要なら外部参照）
- 実用的な情報のみ
- スキル参照で詳細を補完

### 5.3 重複排除

悪い例（重複あり）:

```markdown
## スキル

利用可能なスキル:
- markdown-skills: Markdown文書の作成と編集をサポート
- creating-skills: カスタムスキルの作成をガイド

## ドキュメント作成

Markdown文書を作成する際は、markdown-skillsスキルを使用してください。
このスキルは、一貫したフォーマットと構造化された情報整理を提供します。

## スキル作成

新しいスキルを作成する際は、creating-skillsスキルを参照してください。
このスキルには、スキル作成の手順とベストプラクティスが含まれています。
```

良い例（重複なし）:

```markdown
## Available Skills (Auto-activate)

- `markdown-skills`: Documentation tasks (auto-activates for .md files)
- `creating-skills`: Skill creation (auto-activates when creating skills)

Refer to skill documentation for detailed usage.
```

### 5.4 `#`キーでの動的更新

CLAUDE.mdは静的ファイルではなく、使いながら改善します。

使用方法:
1. Claude Code使用中に`#`キーを押す
2. 追加したい指示を入力
3. Claude が自動的に適切なCLAUDE.mdに追記

例:
```
User: # コミット時は必ずtypecheckを実行
→ CLAUDE.mdに自動追加:
  "- Run typecheck before every commit"
```

推奨ワークフロー:
- 繰り返し同じ指示をしていることに気づいたら`#`で追加
- チームで有用な指示はGitにコミット
- 個人的な指示はCLAUDE.local.mdへ

### 5.5 Prompt Improverの活用

定期的にCLAUDE.mdをPrompt Improverツールで改善:

手順:
1. CLAUDE.mdの内容をコピー
2. Anthropic Prompt Improverに入力
3. 提案された改善を確認
4. 効果的な変更を適用

改善される点:
- 指示の明確性
- 強調の効果的な使用
- 構造の改善
- 冗長性の削減

---

## 6. 実装パターン

### 6.1 パターン1: シンプルプロジェクト

適用対象: 小規模プロジェクト、個人開発

特徴:
- 単一のCLAUDE.md
- 200-300行
- 基本的なスキル参照

構造:
```markdown
# Project: [Name]

## Skills (Auto-activate)
[2-3個のスキル]

## Quick Commands
[5-10個のコマンド]

## Code Style
[重要ルールのみ]

## Git Workflow
[ブランチとコミット規約]
```

### 6.2 パターン2: チームプロジェクト

適用対象: 中規模プロジェクト、複数開発者

特徴:
- ルートCLAUDE.md + CLAUDE.local.md
- 300-400行
- 詳細なワークフロー定義

構造:
```markdown
# Project: [Name]

## Skills Auto-Activation Policy (IMPORTANT)
[詳細な自動化ルール]

## Development Workflow (YOU MUST FOLLOW)
[段階的なワークフロー]

## Available Tools
[Skills/Commands/Agents/MCP]

## Code Style (IMPORTANT)
[チーム合意のルール]

## Architecture
[主要なファイルと責務]

## Testing
[テスト戦略]

## Git Workflow
[詳細なGit規約]
```

### 6.3 パターン3: モノレポ

適用対象: 大規模プロジェクト、複数モジュール

特徴:
- 階層的なCLAUDE.md配置
- ルート: 共通規約（300-400行）
- モジュール: 固有ルール（200-300行）

構造:

`/monorepo/CLAUDE.md`:
```markdown
# Monorepo: [Name]

## Skills Auto-Activation (IMPORTANT)
[全モジュール共通]

## Repository Structure
[モノレポのレイアウト]

## Common Workflows
[共通ワークフロー]

## Module-Specific Guidelines
See module-specific CLAUDE.md files:
- `frontend/CLAUDE.md` - Frontend guidelines
- `backend/CLAUDE.md` - Backend guidelines
- `shared/CLAUDE.md` - Shared library guidelines
```

`/monorepo/frontend/CLAUDE.md`:
```markdown
# Frontend Module

## Additional Skills
- Use React/Next.js specific patterns

## Architecture
- Component structure
- State management (Jotai)

## Testing
- React Testing Library
- Storybook

Inherits: /monorepo/CLAUDE.md
```

---

## 7. ベストプラクティス

### 7.1 スキル自動活用の推進

Do:
- ✅ "ALWAYS use"、"YOU MUST"などの強い表現を使用
- ✅ 条件を明確に定義（"when", "for"）
- ✅ "automatically"、"without asking"を明記
- ✅ タスクとスキルを明示的に対応付け

Don't:
- ❌ "可能であれば"、"推奨します"など曖昧な表現
- ❌ 条件なしの一般的な記述
- ❌ スキルのリストのみで使用タイミングなし

### 7.2 ワークフロー定義

Do:
- ✅ 段階的なステップ（Phase 1, 2, 3...）
- ✅ 各段階での必須アクション（REQUIRED）
- ✅ スキル/エージェント使用を組み込む
- ✅ チェックポイントを設定

Don't:
- ❌ 一般的すぎる説明
- ❌ ステップの優先度が不明確
- ❌ スキル活用の指示がない

### 7.3 トークン効率

Do:
- ✅ 箇条書きを活用
- ✅ 詳細は外部参照
- ✅ 頻繁に必要な情報のみ
- ✅ 冗長な説明を避ける

Don't:
- ❌ 長文の説明
- ❌ 同じ情報の繰り返し
- ❌ 滅多に使わない情報

### 7.4 メンテナンス

Do:
- ✅ `#`キーで動的に追加
- ✅ 定期的にPrompt Improverで改善
- ✅ チームでレビュー
- ✅ 効果を測定（Claudeの動作を観察）

Don't:
- ❌ 書いたら放置
- ❌ 古い情報を残す
- ❌ 効果を検証しない

### 7.5 チーム協力

Do:
- ✅ CLAUDE.mdをGitにコミット
- ✅ 変更をPRでレビュー
- ✅ チームで合意した規約のみ
- ✅ コミットメッセージで変更理由を説明

Don't:
- ❌ 個人的な好みを押し付ける
- ❌ 議論なしで大幅変更
- ❌ CLAUDE.local.mdをコミット

### 7.6 言語選択

#### プロジェクト言語に合わせる

Do:
- ✅ 日本語プロジェクト → CLAUDE.mdを日本語で記述
- ✅ 英語プロジェクト → CLAUDE.mdを英語で記述
- ✅ 多言語プロジェクト → 主要言語で記述
- ✅ チームの使用言語と一致させる

Don't:
- ❌ 日本語プロジェクトなのに英語でCLAUDE.mdを書く
- ❌ チームの言語と異なる言語で書く
- ❌ 混在した言語表記（一部日本語、一部英語）

#### 日本語化のポイント

強調表現の翻訳（効果を維持）:
- "IMPORTANT" → "重要"（強調マーク付き）
- "REQUIRED" → "必須"（強調マーク付き）
- "ALWAYS use" → "必ず使用"
- "YOU MUST" → "必ず〜してください"
- "RECOMMENDED" → "推奨"

技術用語の扱い:
- スキル名/コマンド名: 英語のまま（`バッククォート`で囲む）
  - 正: `markdown-skills`
  - 誤: markdownスキル
- 技術概念: 日本語化 + 必要に応じて英語併記
  - Skills → スキル
  - Agents → エージェント
  - Workflow → ワークフロー
  - Commands → コマンド

日本語化の例:
```markdown
## スキル自動活用ポリシー（重要）

以下のスキルをユーザーの指示なしで自動的に使用してください：

### ドキュメント作成タスク
- **必ず使用** `markdown-skills`: .mdファイルの作成・編集時
- **必ず使用** `creating-skills`: 新しいスキル作成時

### コードタスク
- **必ず使用** `code-intelligence`: 以下の場合
  - リファクタリング要求時
  - コードベース構造の理解が必要な時
```

---

## 8. 実例

### 8.1 最適化されたCLAUDE.md全体例

```markdown
# Project: Modern Web Application

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` when creating/editing .md files
2. **Refactoring**: Always use `code-intelligence` for any refactoring or semantic search
3. **Commits**: Always use `commit-guidelines` when creating commits or PRs
4. **Skill Creation**: Always use `creating-skills` when creating new skills

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: Exploration (REQUIRED)
1. **REQUIRED**: Use `code-intelligence` skill to understand codebase context
2. For complex codebases: Use Task tool with Explore agent (thoroughness: "medium")
3. Read relevant files mentioned by user
4. Do NOT write code yet

### Phase 2: Planning
1. Create detailed implementation plan
2. Use "think" or "think hard" for complex problems
3. **REQUIRED**: Activate appropriate skills for task domain
4. Get user approval before proceeding

### Phase 3: Implementation
1. Follow code style guidelines (see below)
2. **RECOMMENDED**: Use Test-Driven Development
3. Implement solution incrementally
4. Run tests after each change

### Phase 4: Commit & PR (REQUIRED)
1. **REQUIRED**: Use `commit-guidelines` skill
2. Generate conventional commits format
3. Run final checks: typecheck, lint, tests
4. Include co-author attribution

## Quick Commands

- `npm run dev` - Start development server (localhost:3000)
- `npm run build` - Build for production
- `npm run typecheck` - Run TypeScript type checker
- `npm test` - Run test suite
- `/project:fix-issue <num>` - Auto-fix GitHub issue with full workflow
- `/project:create-pr` - Create PR with proper formatting

## Available Tools

### Skills (Auto-activate when possible)
- `markdown-skills`: Documentation tasks (auto-activates for .md files)
- `code-intelligence`: Refactoring, semantic search (auto-activates for code tasks)
- `creating-skills`: Skill creation (auto-activates when creating skills)
- `commit-guidelines`: Git operations (auto-activates for commits/PRs)

### Custom Commands
- `/project:fix-issue`: Complete GitHub issue fixing workflow
- `/project:create-pr`: PR creation with checks and formatting
- `/project:run-tests`: Test execution with coverage report

### Agents (Use via Task tool)
- **Explore**: Codebase understanding (use "medium" thoroughness by default)
- **Plan**: Multi-step task planning (use for complex implementations)

### MCP Servers
- **Puppeteer**: UI testing and screenshots (auto-use for visual tasks)
- **GitHub**: PR/Issue management (prefer `gh` CLI when available)

## Code Style (IMPORTANT)

- Use ES modules (import/export), NOT CommonJS (require)
- Destructure imports: `import { foo } from 'bar'`
- React: Function components + arrow functions only
- Styling: Tailwind CSS utility classes
- State: useState (local), Jotai (global)
- API: tRPC for type-safe API calls
- IMPORTANT: Run typecheck after code changes

Full documentation standards: See `markdown-skills`

## Architecture

Key files and their responsibilities:

- `src/app/`: Next.js 13+ App Router pages
- `src/components/`: Reusable React components
- `src/server/`: tRPC API routes and server logic
- `src/utils/`: Shared utility functions
- `src/styles/`: Global styles and Tailwind config

Authentication: Handled by `src/server/auth.ts` (NextAuth.js)
Database: Prisma ORM, schema in `prisma/schema.prisma`
State Management: Jotai atoms in `src/store/`

## Testing

- TDD preferred: Write tests BEFORE implementation
- Unit tests: `__tests__/` directories, use Vitest
- Integration tests: `src/e2e/`, use Playwright
- Run `npm test` before every commit
- REQUIRED: All new features must have tests

Test utilities available in `src/test-utils/`

## Git Workflow

- Branch naming: `feature/description` or `fix/description`
- Commits: Conventional Commits format (via commit-guidelines skill)
- IMPORTANT: Always rebase, never merge
- PR template: Automatically populated, fill all sections
- Require 1 approval before merge

## Environment Setup

- Node.js: v18+ (use nvm: `nvm use`)
- Package manager: npm (NOT yarn or pnpm)
- Database: PostgreSQL 14+ (use Docker: `npm run db:up`)
- IMPORTANT: Run `npm install` after pulling main branch

Environment variables: Copy `.env.example` to `.env.local`

## Known Issues & Quirks

- Build cache can get stale: Run `npm run clean` if builds fail mysteriously
- Prisma schema changes: Always run `npm run db:push` after editing
- Hot reload sometimes breaks: Restart dev server if changes don't appear
- TypeScript errors in IDE: Restart TS server (Cmd+Shift+P → "Restart TS Server")
```

### 8.2 モノレポルート例

```markdown
# Monorepo: FullStack Platform

## Skills Auto-Activation Policy (IMPORTANT)

[Same as above...]

## Repository Structure

```
/
├── apps/
│   ├── web/          (Next.js frontend - see apps/web/CLAUDE.md)
│   ├── admin/        (Admin dashboard - see apps/admin/CLAUDE.md)
│   └── api/          (Fastify backend - see apps/api/CLAUDE.md)
├── packages/
│   ├── ui/           (Shared React components)
│   ├── config/       (Shared configs: ESLint, TypeScript, etc.)
│   └── database/     (Prisma schema and migrations)
└── tools/
    └── scripts/      (Build and deployment scripts)
```

## Common Workflows

[Monorepo-wide workflows...]

## Module-Specific Guidelines

Each app/package has its own CLAUDE.md:
- `apps/web/CLAUDE.md` - Frontend-specific guidelines
- `apps/admin/CLAUDE.md` - Admin app guidelines
- `apps/api/CLAUDE.md` - Backend API guidelines
- `packages/ui/CLAUDE.md` - Component library guidelines

When working in a specific module, both this file and the module-specific CLAUDE.md apply.
Module-specific guidelines take precedence in case of conflicts.

## Shared Conventions

[Common conventions across all modules...]
```

### 8.3 日本語プロジェクトの例

```markdown
# プロジェクト: モダンWebアプリケーション

## スキル自動活用ポリシー（重要）

以下のスキルをユーザーの指示なしで自動的に使用してください：

1. **ドキュメント作成**: 必ず`markdown-skills`を使用（.mdファイル作成・編集時）
2. **リファクタリング**: 必ず`code-intelligence`を使用（リファクタリングやセマンティック検索時）
3. **コミット**: 必ず`commit-guidelines`を使用（コミットやPR作成時）
4. **スキル作成**: 必ず`creating-skills`を使用（新しいスキル作成時）

## 開発ワークフロー（必ず従ってください）

### フェーズ1: 探索（必須）
1. **必須**: `code-intelligence`スキルを使用してコードベースのコンテキストを理解
2. 複雑なコードベースの場合: Taskツールでexploreエージェントを使用（徹底度: "medium"）
3. ユーザーが言及した関連ファイルを読む
4. まだコードを書かない

### フェーズ2: 計画
1. 詳細な実装計画を作成
2. 複雑な問題には"think"または"think hard"を使用
3. **必須**: タスクドメインに適したスキルを活用
4. 実装前にユーザーの承認を得る

### フェーズ3: 実装
1. コードスタイルガイドラインに従う（下記参照）
2. **推奨**: テスト駆動開発を使用
3. ソリューションを段階的に実装
4. 各変更後にテストを実行

### フェーズ4: コミット & PR（必須）
1. **必須**: `commit-guidelines`スキルを使用
2. Conventional Commits形式で生成
3. 最終チェックを実行: typecheck、lint、tests
4. 共同作成者の帰属を含める

## クイックコマンド

- `npm run dev` - 開発サーバーを起動（http://localhost:3000）
- `npm run build` - 本番用にビルド
- `npm run typecheck` - TypeScript型チェッカーを実行
- `npm test` - テストスイートを実行

## 利用可能なツール

### スキル（可能な場合は自動活用）
- `markdown-skills`: ドキュメントタスク（.mdファイル操作時に自動活用）
- `code-intelligence`: リファクタリング、セマンティック検索（コードタスク時に自動活用）
- `creating-skills`: スキル作成（スキル作成時に自動活用）
- `commit-guidelines`: Git操作（コミット/PR時に自動活用）

### カスタムコマンド
- `/project:fix-issue`: GitHubイシュー修正ワークフロー全体
- `/project:create-pr`: チェックとフォーマット付きでPR作成

### エージェント（Taskツール経由で使用）
- **Explore**: コードベース理解（デフォルトで"medium"の徹底度を使用）
- **Plan**: 複数ステップのタスク計画（複雑な実装に使用）

### MCPサーバー
- **Puppeteer**: UIテストとスクリーンショット（ビジュアルタスクに自動使用）
- **GitHub**: PR/イシュー管理（`gh` CLIが利用可能な場合は優先）

## コードスタイル（重要）

- ES modules使用（import/export）、CommonJS（require）は使用しない
- インポートを分割代入: `import { foo } from 'bar'`
- React: 関数コンポーネント + アロー関数のみ
- スタイリング: Tailwind CSSユーティリティクラス
- 状態: useState（ローカル）、Jotai（グローバル）
- API: 型安全なAPI呼び出しにtRPCを使用
- **重要**: コード変更後にtypecheckを実行

完全なドキュメント標準: `markdown-skills`を参照

## アーキテクチャ

主要ファイルとその責務:

- `src/app/`: Next.js 13+ App Routerページ
- `src/components/`: 再利用可能なReactコンポーネント
- `src/server/`: tRPC APIルートとサーバーロジック
- `src/utils/`: 共有ユーティリティ関数
- `src/styles/`: グローバルスタイルとTailwind設定

認証: `src/server/auth.ts`で処理（NextAuth.js）
データベース: Prisma ORM、スキーマは`prisma/schema.prisma`
状態管理: `src/store/`のJotai atoms

## テスト

- TDD推奨: 実装前にテストを書く
- ユニットテスト: `__tests__/`ディレクトリ、Vitestを使用
- 統合テスト: `src/e2e/`、Playwrightを使用
- すべてのコミット前に`npm test`を実行
- **必須**: すべての新機能にテストが必要

テストユーティリティは`src/test-utils/`で利用可能

## Gitワークフロー

- ブランチ命名: `feature/description`または`fix/description`
- コミット: Conventional Commits形式（commit-guidelinesスキル経由）
- **重要**: 常にrebase、決してmergeしない
- PRテンプレート: 自動入力、すべてのセクションを記入
- マージ前に1件の承認が必要

## 環境セットアップ

- Node.js: v18+（nvmを使用: `nvm use`）
- パッケージマネージャー: npm（yarnやpnpmは使用しない）
- データベース: PostgreSQL 14+（Dockerを使用: `npm run db:up`）
- **重要**: mainブランチをpull後に`npm install`を実行

環境変数: `.env.example`を`.env.local`にコピー

## 既知の問題と癖

- ビルドキャッシュが古くなる可能性: ビルドが不可解に失敗する場合は`npm run clean`を実行
- Prismaスキーマの変更: 編集後は必ず`npm run db:push`を実行
- ホットリロードが時々壊れる: 変更が表示されない場合は開発サーバーを再起動
- IDEでのTypeScriptエラー: TSサーバーを再起動（Cmd+Shift+P → "Restart TS Server"）
```

---

## 使用方法

### このスキルを使う

```
/skill claude-md-optimizer
```

### 新規CLAUDE.md作成

```
新しいプロジェクトのCLAUDE.mdを作成してください。
プロジェクトタイプ: [フロントエンド/バックエンド/フルスタック/モノレポ]
主な技術スタック: [言語/フレームワーク]
チームサイズ: [人数]
```

### 既存CLAUDE.md最適化

```
現在のCLAUDE.mdを分析して最適化提案をしてください。
特に以下に重点を置いてください:
- スキル自動活用の指示追加
- トークン効率化
- ワークフロー統合
```

### テンプレート取得

```
[プロジェクトタイプ]用のCLAUDE.mdテンプレートを生成してください。
```

詳細なテンプレート集: `resources/TEMPLATES.md`
実装パターン詳細: `resources/REFERENCE.md`

---

バージョン: 1.1.0
最終更新: 2025-11-15
変更内容: 日本語プロジェクト対応ルール追加
メンテナー: k-shir0

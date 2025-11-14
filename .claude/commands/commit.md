---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git log:*)
description: Conventional Commits準拠でコミットを作成（日本語説明）
model: claude-sonnet-4-5-20250929
---

# コミット作成タスク

commit-guidelinesスキルに従って、Conventional Commits形式（日本語説明）でコミットを作成してください。

## 現在の状態

**Git Status:**
!`git status`

**変更内容（未ステージング）:**
!`git diff HEAD`

**ステージング済み:**
!`git diff --staged`

**最近のコミット履歴（参考用）:**
!`git log --oneline -5`

## commit-guidelinesスキルの規則

このプロジェクトでは、以下の規則に**必ず**従ってください：

### 基本形式

```
<type>[optional scope]: <日本語の説明>

[optional body]

[optional footer(s)]
```

### 使用するタイプ（type）

| タイプ | 用途 |
|--------|------|
| `feat` | 新機能の追加 |
| `fix` | バグ修正 |
| `docs` | ドキュメントのみの変更 |
| `style` | コードの意味に影響しない変更（フォーマット等） |
| `refactor` | リファクタリング |
| `perf` | パフォーマンス改善 |
| `test` | テストの追加・修正 |
| `build` | ビルドシステムの変更 |
| `ci` | CI設定の変更 |
| `chore` | その他の変更 |
| `revert` | 以前のコミットを取り消す |

### スコープ（scope）- 任意

このリポジトリでの推奨スコープ：
- `docs` - ドキュメント関連
- `skills` - Claudeスキル関連
- `config` - 設定ファイル関連
- `templates` - テンプレート関連

### 説明文（description）のルール

- **必ず日本語**で記述
- 「〜を追加」「〜を修正」などの動詞形式
- 簡潔に（目安：50文字以内）
- 末尾に句点（。）は不要
- 何をしたかを明確に記述

### 参考資料

詳細な規則: `.claude/skills/commit-guidelines/resources/CONVENTIONS.md`
実例集: `.claude/skills/commit-guidelines/resources/EXAMPLES.md`

## タスク手順

1. **変更内容の分析**
   - 上記のgit diff出力を確認
   - 変更の性質（新機能、修正、ドキュメント等）を判断

2. **適切なtypeの選択**
   - 変更内容に最も適したtypeを選択
   - 複数の種類がある場合は、主要な変更に基づく

3. **コミットメッセージの作成**
   - typeは英語、説明は日本語
   - 簡潔で明確な説明
   - 必要に応じてscopeを追加
   - 複雑な変更の場合は本文も追加

4. **ファイルのステージング**
   - 必要なファイルを `git add` でステージング
   - 無関係なファイルは含めない

5. **コミットの実行**
   - 作成したメッセージでコミット
   - コミット後にgit statusで確認

## 良い例

```
feat(skills): コミットガイドラインスキルを追加
```

```
docs: READMEにインストール手順を追加
```

```
fix: ログイン時の認証エラーを修正
```

```
refactor: ユーザーサービスをクラスベースに書き換え

関数ベースの実装からクラスベースに変更。
テスタビリティと保守性を向上。
```

## 悪い例（避けること）

```
add: 追加  # 何を追加したのか不明、typeが不適切
update: ファイルを更新しました。  # typeが不適切、句点不要
feat: Add user authentication  # 説明が英語
fix: バグ修正  # どのバグか不明
```

## 重要な注意事項

- **必ずCommit-guidelinesスキルの規則に従う**
- **説明文は必ず日本語**
- **簡潔で明確なメッセージ**
- **1つのコミットに複数の無関係な変更を含めない**
- **コミット前に変更内容を確認**

それでは、上記の情報に基づいてコミットを作成してください。

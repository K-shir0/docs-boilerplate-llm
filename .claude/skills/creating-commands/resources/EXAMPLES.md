# コマンド実例集

良いコマンドと悪いコマンドの実例、ユースケース別のテンプレート、アンチパターンを紹介します。

## 目次

1. [基本的なコマンド例](#基本的なコマンド例)
2. [Git 操作系](#git-操作系)
3. [テスト・品質管理](#テスト品質管理)
4. [ドキュメント生成](#ドキュメント生成)
5. [コード生成](#コード生成)
6. [デプロイ・環境管理](#デプロイ環境管理)
7. [アンチパターンと改善例](#アンチパターンと改善例)

---

## 基本的なコマンド例

### 例1: シンプルなテンプレート展開

```markdown
---
description: バグレポート Issue テンプレートを作成
disable-model-invocation: true
---

# Bug Report: $ARGUMENTS

## Description
[バグの詳細な説明]

## Steps to Reproduce
1. [手順1]
2. [手順2]
3. [手順3]

## Expected Behavior
[期待される動作]

## Actual Behavior
[実際の動作]

## Environment
- OS: [例: macOS 14.0]
- Browser: [例: Chrome 120]
- Version: [例: v1.2.3]

## Screenshots
[スクリーンショットがあれば追加]

## Additional Context
[その他の補足情報]
```

**使用例:**
```
/bug-report ログインができない
```

**特徴:**
* `disable-model-invocation: true` で高速実行
* LLM 不要のシンプルなテンプレート
* `$ARGUMENTS` でタイトルをカスタマイズ

### 例2: 読み取り専用のレビューコマンド

```markdown
---
description: 最新のコミットをレビュー
allowed-tools: Bash(git log:*), Bash(git show:*), Bash(git diff:*), Read
model: claude-sonnet-4-5-20250929
---

# コミットレビュー

## 対象コミット

!`git log -1 --pretty=fuller`

## 変更内容

!`git show HEAD`

## レビュー観点

以下の観点でレビューしてください:

### 1. コード品質
* 命名規則は適切か
* 関数やクラスのサイズは適切か
* 重複コードはないか

### 2. バグの可能性
* エッジケースを考慮しているか
* エラーハンドリングは適切か
* null/undefined チェックはあるか

### 3. パフォーマンス
* 非効率なループやクエリはないか
* メモリリークの可能性はないか

### 4. セキュリティ
* 入力検証は適切か
* SQL インジェクションのリスクはないか
* 認証・認可は適切か

### 5. テストカバレッジ
* テストは追加されているか
* 十分なテストケースがあるか

## 出力形式

各観点について:
* ✅ 問題なし
* ⚠️  改善推奨
* ❌ 修正必須

具体的な指摘と改善案を提示してください。
```

**特徴:**
* 読み取り専用（`allowed-tools` で制限）
* 構造化されたレビュー観点
* 明確な出力形式

---

## Git 操作系

### 例3: インタラクティブなコミット作成

このプロジェクトの `.claude/commands/commit.md` が優れた例です:

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

1. **変更内容の分析**
   - 上記の git diff 出力を確認
   - 変更の性質（新機能、修正、ドキュメント等）を判断

2. **適切な type の選択**
   - 変更内容に最も適した type を選択
   - 複数の種類がある場合は、主要な変更に基づく

3. **コミットメッセージの作成**
   - type は英語、説明は日本語
   - 簡潔で明確な説明
   - 必要に応じて scope を追加
   - 複雑な変更の場合は本文も追加

4. **ファイルのステージング**
   - 必要なファイルを `git add` でステージング
   - 無関係なファイルは含めない

5. **コミットの実行**
   - 作成したメッセージでコミット
   - コミット後に git status で確認

## 良い例

\`\`\`
feat(skills): コミットガイドラインスキルを追加
\`\`\`

\`\`\`
docs: READMEにインストール手順を追加
\`\`\`

## 悪い例（避けること）

\`\`\`
add: 追加  # 何を追加したのか不明、type が不適切
update: ファイルを更新しました。  # type が不適切、句点不要
feat: Add user authentication  # 説明が英語
fix: バグ修正  # どのバグか不明
\`\`\`

## 重要な注意事項

- **必ず Commit-guidelines スキルの規則に従う**
- **説明文は必ず日本語**
- **簡潔で明確なメッセージ**
- **1つのコミットに複数の無関係な変更を含めない**
- **コミット前に変更内容を確認**

それでは、上記の情報に基づいてコミットを作成してください。

ARGUMENTS: $ARGUMENTS
```

**優れた点:**
* 動的な Git 情報取得
* スキルとの連携
* 明確な手順とチェックリスト
* 具体例の提示
* ツール制限によるセキュリティ

### 例4: PR 作成コマンド

```markdown
---
description: GitHub Pull Request を作成
allowed-tools: Bash(git *), Bash(gh pr create:*), Bash(gh pr list:*), Read
model: claude-sonnet-4-5-20250929
---

# Pull Request 作成

## 現在のブランチ

!`git branch --show-current`

## ベースブランチとの差分

!`git log origin/main..HEAD --oneline`
!`git diff origin/main...HEAD --stat`

## コミット履歴

!`git log origin/main..HEAD --pretty=format:"%h %s"`

## タスク

### 1. PR タイトルの生成

最新のコミットまたは変更内容から適切なタイトルを生成:
* 簡潔で明確
* 50文字以内推奨
* 日本語OK

### 2. PR 本文の生成

以下の形式で生成:

\`\`\`markdown
## 概要
[変更の概要を2-3文で]

## 変更内容
[コミット履歴から主要な変更を箇条書き]
- 変更1
- 変更2
- 変更3

## テスト
[テスト方法や確認事項]
- [ ] ローカルでビルド確認
- [ ] ユニットテスト実行
- [ ] 手動テスト実行

## 関連Issue
[あれば記載]
Closes #123

## スクリーンショット
[必要に応じて]
\`\`\`

### 3. PR 作成実行

\`gh pr create\` コマンドで PR を作成:
* タイトル: 生成したタイトル
* 本文: 生成した本文
* ベースブランチ: main（または適切なブランチ）

### 4. レビュアーのサジェスト

変更内容に基づいて適切なレビュアーを推奨（実際のアサインはユーザーが実行）。

## 引数

$ARGUMENTS が指定されている場合:
* PR タイトルとして使用
* または本文の追加情報として使用

## 実行例

\`\`\`bash
gh pr create \\
  --title "feat: ユーザー認証機能を追加" \\
  --body "$(cat <<'EOF'
## 概要
JWT ベースのユーザー認証機能を実装しました。

## 変更内容
- 認証ミドルウェアを追加
- JWT トークン生成・検証機能
- ログイン/ログアウト API エンドポイント

## テスト
- [x] ローカルでビルド確認
- [x] ユニットテスト実行
- [x] 手動テスト実行

Closes #45
EOF
)"
\`\`\`
```

**特徴:**
* Git と GitHub CLI の連携
* 動的な情報収集
* 構造化された PR 本文
* チェックリスト形式

### 例5: ブランチ同期コマンド

```markdown
---
description: ブランチを main と同期
allowed-tools: Bash(git *)
---

# ブランチ同期

## 現在の状態

**現在のブランチ:**
!`git branch --show-current`

**未コミットの変更:**
!`git status --short`

**ローカル main との差分:**
!`git log main..HEAD --oneline`

## 同期手順

### 1. 事前チェック

未コミットの変更がある場合:
→ エラー: 「未コミットの変更があります。まずコミットまたは stash してください。」
→ 処理を中止

### 2. main ブランチを最新化

\`\`\`bash
git checkout main
git pull origin main
\`\`\`

### 3. 元のブランチに戻る

\`\`\`bash
git checkout -  # 元のブランチに戻る
\`\`\`

### 4. main をマージ

\`\`\`bash
git merge main
\`\`\`

### 5. コンフリクト確認

コンフリクトがある場合:
→ コンフリクトファイルをリストアップ
→ ユーザーに解決を促す

コンフリクトがない場合:
→ 「同期完了」と表示

## 引数

$ARGUMENTS に "--rebase" が含まれる場合:
→ merge の代わりに rebase を使用

\`\`\`bash
git rebase main
\`\`\`
```

---

## テスト・品質管理

### 例6: テスト実行とレポート

```markdown
---
description: ユニットテストを実行してレポート生成
allowed-tools: Bash(npm test:*), Bash(npm run:*), Read, Write
---

# テスト実行

## テスト実行

!`npm test -- --coverage --json --outputFile=test-results.json`

## 結果の分析

test-results.json を読み取り、以下を分析:
* 成功/失敗したテスト数
* カバレッジ（全体、ファイル別）
* 失敗したテストの詳細

## レポート生成

以下の形式でレポートを `test-report.md` に生成:

\`\`\`markdown
# テストレポート

生成日時: [現在日時]

## サマリー

| 項目 | 値 |
|------|-----|
| 総テスト数 | X |
| 成功 | ✅ X |
| 失敗 | ❌ X |
| スキップ | ⏭️ X |
| カバレッジ | X% |

## カバレッジ詳細

| ファイル | ライン | ブランチ | 関数 |
|---------|--------|---------|------|
| file1.ts | 95% | 90% | 100% |
| file2.ts | 80% | 75% | 85% |

## 失敗したテスト

[失敗テストの詳細]

### Test: ユーザー登録
- **エラー:** [エラーメッセージ]
- **ファイル:** user.test.ts:45
- **期待値:** [期待値]
- **実際値:** [実際値]

## 推奨アクション

[カバレッジやテスト結果に基づく改善提案]
\`\`\`

## 出力

レポートを生成後:
1. test-report.md に保存
2. サマリーをコンソールに表示
3. 失敗がある場合は終了コード 1
```

### 例7: Lint とフォーマット

```markdown
---
description: Lint チェックと自動修正
allowed-tools: Bash(npm run lint:*), Bash(npx eslint:*), Read, Write
---

# Lint & Format

## 引数

$ARGUMENTS でファイルまたはディレクトリを指定（デフォルト: src/）

## 実行

### 1. Lint チェック

\`\`\`bash
npm run lint -- $ARGUMENTS
\`\`\`

### 2. エラー集計

Lint エラーを以下でカテゴリ分け:
* エラー（重大）
* ワーニング（改善推奨）
* フォーマット関連

### 3. 自動修正

修正可能なエラーを自動修正:

\`\`\`bash
npm run lint -- --fix $ARGUMENTS
\`\`\`

### 4. 結果レポート

修正前後の比較:
* 修正前のエラー数
* 修正後のエラー数
* 自動修正できなかったエラーのリスト

## 出力例

\`\`\`
🔍 Lint チェック実行中...

📊 結果:
  ❌ エラー: 12 → 3（9件自動修正）
  ⚠️  ワーニング: 25 → 8（17件自動修正）

🔧 自動修正できなかったエラー:
  src/utils/helper.ts:45
    - 'any' type は使用しないでください

  src/services/api.ts:120
    - Promise を返す関数は async を使用してください

✅ フォーマットは全て修正されました
\`\`\`
```

---

## ドキュメント生成

### 例8: API ドキュメント生成

```markdown
---
description: API エンドポイントのドキュメントを生成
allowed-tools: Read, Write, Grep
model: claude-sonnet-4-5-20250929
---

# API ドキュメント生成

## 対象

$ARGUMENTS で指定されたファイル（デフォルト: src/routes/）

## 処理手順

### 1. エンドポイントの抽出

対象ファイルから以下を抽出:
* HTTP メソッド（GET, POST, PUT, DELETE等）
* パス（/api/users/:id 等）
* ハンドラー関数
* ミドルウェア
* ドキュメントコメント

### 2. パラメータの解析

各エンドポイントについて:
* パスパラメータ
* クエリパラメータ
* リクエストボディ
* レスポンス形式

### 3. ドキュメント生成

`docs/api.md` に以下の形式で生成:

\`\`\`markdown
# API ドキュメント

生成日時: [日時]

## エンドポイント一覧

### GET /api/users

**説明:** 全ユーザーを取得

**パラメータ:**
| 名前 | 型 | 必須 | 説明 |
|------|-----|------|------|
| page | number | No | ページ番号（デフォルト: 1） |
| limit | number | No | 1ページあたりの件数（デフォルト: 20） |

**レスポンス:**
\`\`\`json
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    }
  ],
  "total": 100,
  "page": 1
}
\`\`\`

**エラーレスポンス:**
* 401: 認証エラー
* 500: サーバーエラー

---

[他のエンドポイントも同様に]
\`\`\`

### 4. 検証

生成されたドキュメントを検証:
* 全エンドポイントが含まれているか
* パラメータが正しいか
* 例が適切か
```

### 例9: Changelog 更新

```markdown
---
description: CHANGELOG.md を更新
allowed-tools: Bash(git log:*), Bash(git tag:*), Read, Write, Edit
---

# Changelog 更新

## 最新バージョン情報

**最新タグ:**
!`git describe --tags --abbrev=0`

**最新タグ以降のコミット:**
!`git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%h %s"`

## 処理

### 1. コミット分類

Conventional Commits 形式でコミットを分類:
* feat: 新機能
* fix: バグ修正
* docs: ドキュメント
* style: スタイル
* refactor: リファクタリング
* perf: パフォーマンス
* test: テスト
* chore: その他

### 2. 新バージョンの決定

引数 $ARGUMENTS でバージョンを指定、または:
* feat があれば MINOR アップ
* fix のみなら PATCH アップ
* BREAKING CHANGE があれば MAJOR アップ

### 3. Changelog エントリ生成

CHANGELOG.md の先頭に以下を挿入:

\`\`\`markdown
## [1.2.0] - 2025-01-15

### Added
- ユーザー認証機能を追加 (#45)
- ダッシュボード画面を追加 (#47)

### Fixed
- ログイン時のエラーを修正 (#46)
- メモリリークを修正 (#48)

### Changed
- API レスポンス形式を変更 (#49)

[1.2.0]: https://github.com/user/repo/compare/v1.1.0...v1.2.0
\`\`\`

### 4. 確認

更新内容をユーザーに表示し、確認を求める。
```

---

## コード生成

### 例10: React コンポーネント生成

```markdown
---
description: React コンポーネントを生成
allowed-tools: Read, Write, Skill
---

# React コンポーネント生成

## 引数

$ARGUMENTS: コンポーネント名（例: UserProfile）

## 生成ファイル

1. `src/components/$ARGUMENTS.tsx` - コンポーネント本体
2. `src/components/$ARGUMENTS.test.tsx` - テスト
3. `src/components/$ARGUMENTS.stories.tsx` - Storybook（オプション）

## コンポーネント本体

\`\`\`typescript
import React from 'react';

interface ${ARGUMENTS}Props {
  // プロパティを定義
}

/**
 * $ARGUMENTS コンポーネント
 *
 * @param props - コンポーネントのプロパティ
 */
export const $ARGUMENTS: React.FC<${ARGUMENTS}Props> = (props) => {
  return (
    <div className="$ARGUMENTS">
      {/* コンポーネントの実装 */}
    </div>
  );
};

$ARGUMENTS.displayName = '$ARGUMENTS';
\`\`\`

## テストファイル

\`\`\`typescript
import { render, screen } from '@testing-library/react';
import { $ARGUMENTS } from './$ARGUMENTS';

describe('$ARGUMENTS', () => {
  it('renders without crashing', () => {
    render(<$ARGUMENTS />);
    // アサーション
  });

  // 追加のテストケース
});
\`\`\`

## スタイル規約

coding-standards スキルに従う:
* TypeScript 使用
* Props は interface で定義
* JSDoc コメント必須
* 関数コンポーネント + アロー関数
```

### 例11: API エンドポイント生成

```markdown
---
description: REST API エンドポイントを生成
allowed-tools: Read, Write, Edit
---

# API エンドポイント生成

## 引数

$ARGUMENTS: リソース名（例: users）

## 生成ファイル

1. `src/routes/$ARGUMENTS.ts` - ルート定義
2. `src/controllers/${ARGUMENTS}Controller.ts` - コントローラー
3. `src/services/${ARGUMENTS}Service.ts` - ビジネスロジック
4. `src/models/$ARGUMENTS.ts` - データモデル
5. `src/routes/$ARGUMENTS.test.ts` - テスト

## 生成する CRUD エンドポイント

* `GET /api/$ARGUMENTS` - 一覧取得
* `GET /api/$ARGUMENTS/:id` - 個別取得
* `POST /api/$ARGUMENTS` - 新規作成
* `PUT /api/$ARGUMENTS/:id` - 更新
* `DELETE /api/$ARGUMENTS/:id` - 削除

## ルート定義

\`\`\`typescript
import express from 'express';
import { ${ARGUMENTS}Controller } from '../controllers/${ARGUMENTS}Controller';
import { authMiddleware } from '../middleware/auth';

const router = express.Router();
const controller = new ${ARGUMENTS}Controller();

router.get('/', authMiddleware, controller.getAll);
router.get('/:id', authMiddleware, controller.getById);
router.post('/', authMiddleware, controller.create);
router.put('/:id', authMiddleware, controller.update);
router.delete('/:id', authMiddleware, controller.delete);

export default router;
\`\`\`

## コントローラー

[CRUD 操作のコントローラーメソッドを生成]

## サービス

[ビジネスロジックを生成]

## モデル

[データモデル（TypeScript interface または ORM model）を生成]

## テスト

[各エンドポイントのテストを生成]
```

---

## デプロイ・環境管理

### 例12: 環境別デプロイ

```markdown
---
description: 環境を指定してデプロイ
allowed-tools: Bash(git *), Bash(npm *), Bash(docker *), Read
---

# デプロイコマンド

## 引数

$ARGUMENTS: 環境名（staging または production）

## 事前チェック

### 1. 環境検証

$ARGUMENTS が "staging" または "production" でない場合:
→ エラー: "Invalid environment. Use 'staging' or 'production'."
→ 処理を中止

### 2. ブランチ確認

**現在のブランチ:**
!`git branch --show-current`

production デプロイの場合:
→ main ブランチ以外ではエラー

staging デプロイの場合:
→ develop ブランチ推奨（警告のみ）

### 3. 未コミット確認

!`git status --short`

未コミットの変更がある場合:
→ エラー: "Uncommitted changes found. Please commit or stash."
→ 処理を中止

### 4. リモート同期確認

!`git fetch`
!`git status -sb`

リモートより遅れている場合:
→ エラー: "Local branch is behind remote. Please pull first."
→ 処理を中止

## デプロイ手順

### Staging

\`\`\`bash
# ビルド
npm run build:staging

# Docker イメージ作成
docker build -t myapp:staging .

# デプロイ
docker-compose -f docker-compose.staging.yml up -d

# ヘルスチェック
curl https://staging.example.com/health
\`\`\`

### Production

\`\`\`bash
# タグ作成
git tag -a v$(date +%Y%m%d-%H%M%S) -m "Production deploy"

# ビルド
npm run build:production

# Docker イメージ作成
docker build -t myapp:production .

# デプロイ
docker-compose -f docker-compose.production.yml up -d

# ヘルスチェック
curl https://example.com/health

# タグをプッシュ
git push --tags
\`\`\`

## デプロイ後確認

1. ヘルスチェックが成功したか
2. ログにエラーがないか
3. 主要な機能が動作するか

## ロールバック情報

問題が発生した場合のロールバック手順を表示:

\`\`\`bash
# 前のバージョンのイメージを使用
docker-compose -f docker-compose.$ARGUMENTS.yml down
docker-compose -f docker-compose.$ARGUMENTS.yml up -d --scale app=0
docker-compose -f docker-compose.$ARGUMENTS.yml up -d myapp:previous

# または Git タグでロールバック
git checkout [previous-tag]
# [デプロイ手順を再実行]
\`\`\`
```

---

## アンチパターンと改善例

### アンチパターン1: 過度に複雑なコマンド

❌ **悪い例:**

```markdown
---
description: 全ての作業を自動化
allowed-tools: Bash(*), Read, Write, Edit, Task
---

# 全自動コマンド

このコマンドは以下を全て実行します:
1. コードを lint
2. テストを実行
3. ビルド
4. コミット作成
5. PR 作成
6. レビュアーをアサイン
7. CI が通ったらマージ
8. デプロイ

[複雑な処理...]
```

✅ **良い例（分割）:**

```
/lint
/test
/build
/commit
/pr
```

または、エージェントを使用。

**理由:** 1つのコマンドは1つのタスクに集中すべきです。

---

### アンチパターン2: allowed-tools の未指定

❌ **悪い例:**

```markdown
---
description: ファイルをフォーマット
---

# フォーマット

ファイルをフォーマットしてください。
$ARGUMENTS
```

✅ **良い例:**

```markdown
---
description: ファイルをフォーマット
allowed-tools: Read, Write, Edit
---

# フォーマット

ファイルをフォーマットしてください。
$ARGUMENTS
```

**理由:** セキュリティと予測可能性の向上。

---

### アンチパターン3: 不明確な description

❌ **悪い例:**

```yaml
description: コミット
description: 実行
description: タスク
```

✅ **良い例:**

```yaml
description: Conventional Commits準拠でコミットを作成（日本語説明）
description: ユニットテストを実行してレポート生成
description: GitHub Pull Request を作成
```

**理由:** SlashCommand tool による自動実行の精度向上。

---

### アンチパターン4: 動的情報の欠如

❌ **悪い例:**

```markdown
---
description: コミットを作成
allowed-tools: Bash(git commit:*)
---

# コミット

コミットメッセージを作成してコミットしてください。
```

✅ **良い例:**

```markdown
---
description: コミットを作成
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git commit:*)
---

# コミット

## 現在の変更

!`git status`
!`git diff`

上記の変更内容に基づいてコミットメッセージを作成してください。
```

**理由:** Claude が現在の状態を把握できる。

---

### アンチパターン5: エラーハンドリングの欠如

❌ **悪い例:**

```markdown
---
description: デプロイ
allowed-tools: Bash(*)
---

# デプロイ

\`\`\`bash
git push origin main
npm run deploy
\`\`\`
```

✅ **良い例:**

```markdown
---
description: デプロイ
allowed-tools: Bash(git *), Bash(npm *)
---

# デプロイ

## 事前チェック

1. 未コミットの変更がないか確認
   !`git status --short`

   変更がある場合 → エラーで終了

2. main ブランチにいるか確認
   !`git branch --show-current`

   main 以外の場合 → エラーで終了

3. テストが通るか確認
   \`\`\`bash
   npm test
   \`\`\`

   失敗した場合 → エラーで終了

## デプロイ実行

全チェックが通過した場合のみデプロイ。
```

**理由:** 安全性と信頼性の向上。

---

## まとめ

良いコマンドの特徴:

1. **単一責任**: 1つのタスクに集中
2. **適切な制限**: `allowed-tools` でツールを制限
3. **動的情報**: `!`bash`` で最新情報を取得
4. **明確な構造**: セクションで論理的に整理
5. **エラーハンドリング**: 失敗ケースを考慮
6. **具体例**: 良い例と悪い例を提示
7. **スキル連携**: 既存スキルを参照して一貫性を保持

これらのパターンを参考に、プロジェクトに適したコマンドを作成してください。

---

バージョン: 1.0.0
最終更新: 2025-01-15

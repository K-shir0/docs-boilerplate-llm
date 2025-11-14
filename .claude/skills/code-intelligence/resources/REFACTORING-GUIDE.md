# リファクタリング支援詳細ガイド

このドキュメントは、code-intelligenceスキルにおけるリファクタリング操作の詳細な手順を説明します。

## 基本方針

リファクタリングは **安全性を最優先** し、段階的に実施します。

実行モード: **自動実行** (ユーザー設定に基づく)

### 安全性の原則

1. **事前確認**: 現在の状態を必ず診断
2. **影響範囲の把握**: 全参照箇所を追跡
3. **段階的変更**: 小さな単位で実施
4. **検証**: 各ステップで診断情報を確認
5. **記録**: 変更履歴をメモリに保存

## 安全性チェック詳細

リファクタリング実施前に必ず以下を確認します。

### 1. 診断情報の確認

**ツール**: `mcp__ide__getDiagnostics`

**目的**: 現在のエラーや警告を把握

**パラメータ**:
* `uri`: 対象ファイルURI (オプション)

**確認項目**:
* 既存のエラーがないか
* 警告の内容
* リファクタリング対象のシンボルに関連する問題

**実行例**:
```javascript
// 特定ファイルの診断情報取得
mcp__ide__getDiagnostics({
  uri: "file:///path/to/file.ts"
})

// プロジェクト全体の診断情報取得
mcp__ide__getDiagnostics({})
```

**期待される結果**:
```json
{
  "diagnostics": [
    {
      "severity": "error",
      "message": "Cannot find name 'foo'",
      "range": {
        "start": { "line": 10, "character": 5 },
        "end": { "line": 10, "character": 8 }
      }
    }
  ]
}
```

**判断基準**:
* エラーが0件: リファクタリング可能
* エラーが存在: 先にエラーを解決するか、影響を評価
* 警告のみ: リファクタリング可能（警告内容を確認）

### 2. 対象シンボルのスコープ分析

**実施内容**:
* シンボルの定義位置を確認
* 参照箇所を全て追跡 (`find_referencing_symbols`)
* 依存関係を把握
* 影響範囲を見積もり

**手順**:

1. **シンボル特定**: `find_symbol` でシンボルを見つける
```
find_symbol({
  name_path: "ClassName.methodName",
  relative_path: "src/services/service.ts",
  include_body: false  // まずは構造のみ
})
```

2. **参照追跡**: `find_referencing_symbols` で使用箇所を確認
```
find_referencing_symbols({
  name_path: "ClassName.methodName",
  relative_path: "src/services/service.ts"
})
```

3. **影響範囲の評価**:
   * 参照箇所が10件未満: 低リスク
   * 参照箇所が10-50件: 中リスク（注意が必要）
   * 参照箇所が50件以上: 高リスク（段階的に実施）

## リファクタリング操作詳細

### シンボルのリネーム

**ツール**: `rename_symbol`

**用途**: シンボル名をコードベース全体で変更

**パラメータ**:
* `name_path`: 対象シンボルの名前パス
* `relative_path`: シンボルを含むファイル
* `new_name`: 新しい名前

**注意事項**:
* メソッドオーバーロードがある言語 (Java など) では署名を含む
* 自動的に全参照箇所が更新される
* 実施後に診断情報を再確認

**実行例**:

**ケース1: 関数のリネーム**
```
rename_symbol({
  name_path: "getUserData",
  relative_path: "src/services/user-service.ts",
  new_name: "fetchUserData"
})
```

**ケース2: クラスメソッドのリネーム**
```
rename_symbol({
  name_path: "UserService.getData",
  relative_path: "src/services/user-service.ts",
  new_name: "fetchData"
})
```

**ケース3: クラス名のリネーム**
```
rename_symbol({
  name_path: "UserRepo",
  relative_path: "src/repositories/user-repo.ts",
  new_name: "UserRepository"
})
```

**JavaやC#などオーバーロード対応言語の場合**:
```
rename_symbol({
  name_path: "Calculator.add(int, int)",
  relative_path: "src/Calculator.java",
  new_name: "sum"
})
```

**成功後の確認**:
1. 診断情報を再取得: `mcp__ide__getDiagnostics({})`
2. エラーが増えていないことを確認
3. 変更をメモリに記録: `write_memory`

### シンボル本体の置換

**ツール**: `replace_symbol_body`

**用途**: シンボルの定義全体を置換

**パラメータ**:
* `name_path`: 対象シンボルの名前パス
* `relative_path`: シンボルを含むファイル
* `body`: 新しいシンボル本体 (署名含む)

**重要**:
* `body` には署名行を含む完全な定義を指定
* docstring / コメント / import は含まない
* 事前に `include_body=true` でシンボルを読み取っておく

**実行例**:

**ケース1: 関数本体の置換**
```
// 事前にシンボルを読み取る
find_symbol({
  name_path: "calculateTotal",
  relative_path: "src/utils/math.ts",
  include_body: true
})

// 置換実行
replace_symbol_body({
  name_path: "calculateTotal",
  relative_path: "src/utils/math.ts",
  body: `function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}`
})
```

**ケース2: クラスメソッドの置換**
```
replace_symbol_body({
  name_path: "UserService.fetchUser",
  relative_path: "src/services/user-service.ts",
  body: `async fetchUser(id: string): Promise<User> {
  const user = await this.repository.findById(id);
  if (!user) {
    throw new UserNotFoundError(id);
  }
  return user;
}`
})
```

**ケース3: クラス全体の置換**
```
replace_symbol_body({
  name_path: "UserConfig",
  relative_path: "src/config/user-config.ts",
  body: `class UserConfig {
  private static instance: UserConfig;

  private constructor(private readonly maxUsers: number = 100) {}

  static getInstance(): UserConfig {
    if (!UserConfig.instance) {
      UserConfig.instance = new UserConfig();
    }
    return UserConfig.instance;
  }

  getMaxUsers(): number {
    return this.maxUsers;
  }
}`
})
```

**注意点**:
* インデントは元のファイルに合わせる
* 型定義は正確に
* 置換後は必ず診断情報を確認

### コードの挿入

**ツール**: `insert_after_symbol` または `insert_before_symbol`

**用途**: シンボルの前後にコードを挿入

**パラメータ**:
* `name_path`: 基準となるシンボルの名前パス
* `relative_path`: シンボルを含むファイル
* `body`: 挿入するコード

**使用例**:

**ケース1: クラスへのメソッド追加**
```
// 最後のメソッドの後に挿入
insert_after_symbol({
  name_path: "UserService.deleteUser",  // 既存の最後のメソッド
  relative_path: "src/services/user-service.ts",
  body: `

  async updateUserEmail(userId: string, newEmail: string): Promise<void> {
    const user = await this.fetchUser(userId);
    user.email = newEmail;
    await this.repository.save(user);
  }`
})
```

**ケース2: import文の追加**
```
// 最初のトップレベルシンボルの前に挿入
insert_before_symbol({
  name_path: "UserService",  // 最初のクラス
  relative_path: "src/services/user-service.ts",
  body: `import { Logger } from '../utils/logger';

`
})
```

**ケース3: ファイル末尾への追加**
```
// 最後のトップレベルシンボルの後に挿入
insert_after_symbol({
  name_path: "UserService",  // 最後のクラス
  relative_path: "src/services/user-service.ts",
  body: `

export const userServiceInstance = new UserService();
`
})
```

**ケース4: インターフェースへのプロパティ追加**
```
insert_after_symbol({
  name_path: "User.email",  // 既存の最後のプロパティ
  relative_path: "src/types/user.ts",
  body: `
  phoneNumber?: string;`
})
```

**注意点**:
* 適切な空行を含める
* インデントレベルを合わせる
* 挿入位置を慎重に選ぶ

## 段階的変更の実施

大規模なリファクタリングは以下の手順で段階的に実施します。

### 詳細手順

#### ステップ1: 変更の分割

**方針**:
* 1つのシンボル変更
* 1つのファイル内の変更
* 関連性の高い変更のグループ化

**例**: 認証機能のリファクタリング

大きな変更:
```
認証システムの全面的な書き直し
- AuthService クラスのリネーム
- 認証メソッドの署名変更
- エラーハンドリングの統一
- テストコードの更新
```

分割後:
```
変更1: AuthService → AuthenticationService (リネーム)
変更2: login メソッドの署名変更
変更3: エラーハンドリングの統一
変更4: テストコードの更新
```

#### ステップ2: 各ステップで検証

**検証項目**:
1. 診断情報の確認
2. 参照の整合性チェック
3. 必要に応じてテスト実行

**検証手順**:
```
// 変更前
getDiagnostics({}) → エラー: 0件

// 変更実行
rename_symbol(...)

// 変更後
getDiagnostics({}) → エラー: 0件（変化なし）

// テスト実行（オプション）
npm test
```

#### ステップ3: 問題発生時は即座に停止

**問題検出**:
* 診断情報でエラーが増加
* テストが失敗
* 予期しない変更が発生

**対応手順**:
1. 変更内容をユーザーに報告
2. エラー詳細を提示
3. ロールバック手順を提示（git revert など）
4. 代替アプローチを検討

**報告例**:
```
リファクタリング中にエラーが発生しました。

変更内容:
- AuthService → AuthenticationService にリネーム

エラー:
- src/api/auth-controller.ts:15 - Cannot find name 'AuthService'
- src/middleware/auth.ts:23 - Cannot find name 'AuthService'

原因:
- 動的import文は rename_symbol で更新されない可能性

推奨対応:
1. git revert で変更を戻す
2. 動的import箇所を手動で確認
3. 手動で変更するか、別のアプローチを検討
```

## コードスタイルガイドライン

リファクタリングや新規コード追加時は既存のコーディング規則に従います。

### プロジェクト規則の遵守

**インデント**:
* プロジェクトの設定に従う（タブ or スペース）
* .editorconfig や prettier.config.js を確認
* 既存コードのパターンを維持

**命名規則**:
* camelCase: JavaScript/TypeScript の関数・変数
* PascalCase: クラス名・型名・インターフェース
* snake_case: Python の関数・変数
* UPPER_SNAKE_CASE: 定数

**コメントスタイル**:
* JSDoc: JavaScript/TypeScript
* docstring: Python
* Javadoc: Java
* XML documentation: C#

### 一貫性の保持

**ファイル構成**:
* import 文の順序（標準ライブラリ → 外部ライブラリ → 内部モジュール）
* export の配置（ファイル末尾 or インライン）
* セクション分け（コメントによる区切り）

**エラーハンドリング**:
* try-catch: 同期処理
* Promise.catch: 非同期処理
* Result 型: Rust, Scala など
* Either 型: 関数型言語

**型定義**:
* interface vs type (TypeScript)
  - interface: 拡張可能なオブジェクト型
  - type: Union, Intersection など
* any の使用ポリシー（禁止 or 制限付き許可）

### リファクタリング品質基準

実施したリファクタリングは以下を満たす必要があります:

1. **後方互換性**: 既存のAPIを壊さない
2. **テスト通過**: 既存テストが全て通る
3. **診断クリーン**: 新しいエラーや警告を導入しない
4. **可読性向上**: コードが理解しやすくなる
5. **保守性向上**: 将来の変更が容易になる

**品質チェックリスト**:
```
□ 診断情報: エラー0件、警告増加なし
□ テスト: 全て通過
□ コードレビュー: 可読性が向上している
□ ドキュメント: 必要に応じて更新
□ メモリ記録: リファクタリング履歴を保存
```

品質基準を満たさない場合は変更を差し戻し、代替アプローチを検討します。

## リファクタリング実施後のメモリ記録

リファクタリング完了時は **必ず** `write_memory` で記録します。

### 保存内容

* 変更内容の概要
* 変更理由（なぜこのリファクタリングを実施したか）
* 影響範囲（変更したファイルとシンボル）
* 検証結果（診断情報、テスト結果）
* 学んだ教訓や注意点

### メモリファイル名

* `refactoring_{feature}_{YYYYMMDD}.md`

**例**:
* `refactoring_auth_20250108.md`
* `refactoring_database_migration_20250115.md`

### 記録例

```markdown
# リファクタリング: 認証サービスのリネーム

## 日時
2025-01-15

## 変更内容
AuthService クラスを AuthenticationService にリネームし、命名規則を統一

## 変更理由
プロジェクトの命名規則では、サービスクラスは完全な単語を使用する方針のため

## 影響範囲
### 変更したファイル
* src/services/auth-service.ts:10 - クラス定義
* src/api/auth-controller.ts:5 - import文
* src/api/auth-controller.ts:23 - インスタンス化
* src/middleware/auth-middleware.ts:3 - import文
* src/middleware/auth-middleware.ts:15 - 使用箇所
合計3ファイル、5箇所

### 影響を受けたシンボル
* AuthService クラス → AuthenticationService

## 使用したツール
* find_symbol - クラス特定
* find_referencing_symbols - 参照追跡
* rename_symbol - リネーム実行
* getDiagnostics - 検証

## 検証結果
### 診断情報
* 変更前: エラー0件、警告2件
* 変更後: エラー0件、警告2件（変化なし）

### テスト結果
* 実行: 45件
* 成功: 45件
* 失敗: 0件

## 学んだ教訓
* rename_symbol は全参照を自動更新するため安全
* import文も自動的に更新される
* テストコード内の参照も更新されるため、テスト修正は不要だった

## 今後の参照ポイント
* 同様の命名規則統一を他のサービスクラスにも適用できる
```

成功・失敗に関わらず記録することで、今後の参考とします。

## トラブルシューティング

### よくあるエラー

**エラー1: シンボルが見つからない**
```
Error: Symbol 'ClassName.methodName' not found
```

**原因**:
* name_path の誤り
* relative_path の誤り
* シンボルが存在しない

**対処法**:
1. `get_symbols_overview` でシンボル一覧を確認
2. 正しい name_path を特定
3. relative_path がプロジェクトルートからの相対パスであることを確認

**エラー2: リネーム後にエラーが増加**
```
After rename: 5 new errors found
```

**原因**:
* 動的import文が更新されない
* 文字列リテラル内の参照
* コメント内の古い名前

**対処法**:
1. エラー箇所を特定
2. 手動で修正が必要な箇所を確認
3. 必要に応じてロールバック

**エラー3: 置換後に構文エラー**
```
SyntaxError: Unexpected token
```

**原因**:
* body のインデントミス
* 不完全なコード
* 型定義の誤り

**対処法**:
1. 元のシンボルを再度読み取る
2. インデントとフォーマットを確認
3. 構文を検証してから再実行

### 失敗時の対処法

**対処手順**:

1. **エラー内容の確認**
   * getDiagnostics で詳細を取得
   * エラーメッセージを分析

2. **影響範囲の特定**
   * どのファイルが影響を受けたか
   * どのシンボルに問題があるか

3. **ロールバック検討**
   * git status で変更を確認
   * git diff で差分を確認
   * git revert でロールバック

4. **代替アプローチの検討**
   * 手動編集が必要か
   * より小さな変更単位に分割できるか
   * 別のツールやアプローチが適切か

5. **ユーザーへの報告**
   * 問題の詳細を説明
   * 推奨する対応を提示
   * 次のステップを確認

## まとめ

リファクタリングは強力なツールですが、安全性を最優先してください:

1. **事前確認**: 診断情報と参照追跡
2. **段階的実施**: 小さな変更単位
3. **検証**: 各ステップで確認
4. **記録**: メモリに保存

これらの原則に従うことで、安全で効果的なリファクタリングを実現できます。

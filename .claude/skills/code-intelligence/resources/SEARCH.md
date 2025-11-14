# セマンティック検索完全リファレンス

このドキュメントは、code-intelligenceスキルにおけるセマンティック検索の詳細なリファレンスです。

## 基本戦略

### トークン効率化の原則

**"コンテキストウィンドウは公共財"** - creating-skillsガイドより

コードを理解する際は **段階的に情報を取得** し、トークン使用量を最小化します。

重要原則:

* ファイル全体の読み込みは **絶対に禁止**
* 必ずシンボル単位で読み取る
* `include_body=false` から始め、必要な時のみ `include_body=true`

### トークン削減の実例

**悪い例**:
```
Read tool でファイル全体を読む:
- src/services/user-service.ts (250行)
- トークン消費: 約250行分
```

**良い例**:
```
段階的アプローチ:
1. get_symbols_overview → 約15行
2. find_symbol (include_body=false) → 約8行
3. find_symbol (include_body=true, 必要な部分のみ) → 約20行

合計: 約43行（83%削減）
```

## 検索ワークフロー詳細

### ステップ1: ファイル概要の取得

**ツール**: `get_symbols_overview`

**目的**: トップレベルシンボルのリストを把握

**パラメータ**:
* `relative_path`: 対象ファイルのパス（プロジェクトルートからの相対パス）

**実行例**:
```
get_symbols_overview({
  relative_path: "src/services/user-service.ts"
})
```

**期待される結果**:
```
Symbols in src/services/user-service.ts:
- UserService (class)
- CreateUserDto (interface)
- UpdateUserDto (interface)
- getUserById (function)
- deleteUser (function)
```

**活用方法**:
* ファイルにどんなシンボルがあるかの全体像を把握
* 次のステップで詳細調査するシンボルを選択
* トークン消費を最小限に（通常10-20行程度）

### ステップ2: 特定シンボルの探索

**ツール**: `find_symbol`

**目的**: 目的のシンボルを詳細調査

**パラメータ**:
* `name_path`: シンボルの名前パス
  * トップレベル: `"ClassName"` or `"functionName"`
  * ネスト: `"ClassName.methodName"` or `"ClassName.NestedClass"`
* `relative_path`: 検索範囲（推奨: 特定のファイル）
* `include_body`: シンボル本体を含めるか
  * `false`: 署名・構造のみ（デフォルト、推奨）
  * `true`: 実装詳細を含む
* `depth`: 子シンボルの深さ
  * `0`: 指定シンボルのみ
  * `1`: 直接の子（メソッド、プロパティ）
  * `2`: 孫まで

**実行例**:

**例1: クラス構造のみ確認**
```
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts",
  include_body: false,
  depth: 1
})

結果:
class UserService {
  constructor(repository: UserRepository)
  fetchUser(id: string): Promise<User>
  createUser(data: CreateUserDto): Promise<User>
  updateUser(id: string, data: UpdateUserDto): Promise<User>
  deleteUser(id: string): Promise<void>
}
```

**例2: 特定メソッドの実装確認**
```
find_symbol({
  name_path: "UserService.fetchUser",
  relative_path: "src/services/user-service.ts",
  include_body: true
})

結果:
async fetchUser(id: string): Promise<User> {
  const user = await this.repository.findById(id);
  if (!user) {
    throw new UserNotFoundError(id);
  }
  return user;
}
```

**例3: プロジェクト全体から検索（relative_path未指定）**
```
find_symbol({
  name_path: "AuthService",
  include_body: false
})

結果:
Found in src/services/auth-service.ts:
class AuthService { ... }
```

**パラメータ選択ガイド**:

| 状況 | relative_path | include_body | depth |
|------|--------------|--------------|-------|
| ファイル内のクラス構造を知りたい | 指定 | false | 1 |
| 特定メソッドの実装を見たい | 指定 | true | 0 |
| プロジェクト全体からシンボルを探す | 未指定 | false | 0 |
| クラスの全メソッドとプロパティ | 指定 | false | 2 |

### ステップ3: 参照追跡

**ツール**: `find_referencing_symbols`

**目的**: シンボルの使用箇所を確認

**パラメータ**:
* `name_path`: 対象シンボルの名前パス
* `relative_path`: シンボルを含むファイル

**実行例**:
```
find_referencing_symbols({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})

結果:
References to UserService:
1. src/api/user-controller.ts:15
   - import { UserService } from '../services/user-service'

2. src/api/user-controller.ts:23
   - const userService = new UserService(repository)

3. src/api/admin-controller.ts:34
   - this.userService.fetchUser(userId)

4. tests/services/user-service.test.ts:10
   - describe('UserService', () => { ... })
```

**活用方法**:
* 依存関係の把握（どこから使われているか）
* リファクタリングの影響範囲の見積もり
* モジュール間の関係性の理解

## ツール選択の判断基準

### シナリオ別ツール選択

**シナリオ1: シンボル名が明確に分かっている**

使用ツール: `find_symbol`

```
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts",
  include_body: false
})
```

**シナリオ2: シンボル名が部分的にしか分からない**

使用ツール: `find_symbol` with `substring_matching=true`

```
find_symbol({
  name_path: "User",  // "User" を含むシンボルを全て検索
  substring_matching: true,
  include_body: false
})

結果:
- UserService
- UserRepository
- UserController
- CreateUserDto
- UpdateUserDto
```

**シナリオ3: シンボル名が全く分からない（キーワード検索）**

使用ツール: `search_for_pattern` (最終手段)

```
search_for_pattern({
  pattern: "authentication",
  relative_path: "src/"
})

結果:
src/services/auth-service.ts:15: // Authentication logic
src/middleware/auth.ts:23: function authenticateUser() { ... }
```

注意: `search_for_pattern` はトークン消費が多いため、補助的にのみ使用してください。

**シナリオ4: 特定の種別のシンボルのみ検索**

使用ツール: `find_symbol` with `include_kinds` / `exclude_kinds`

```
// クラスのみ検索
find_symbol({
  name_path: "User",
  substring_matching: true,
  include_kinds: ["class"]
})

結果:
- UserService (class)
- UserRepository (class)
- UserController (class)

// 関数のみ検索（クラスを除外）
find_symbol({
  name_path: "get",
  substring_matching: true,
  exclude_kinds: ["class"]
})

結果:
- getUserById (function)
- getUsers (function)
```

### ツール選択フローチャート

```
シンボルを探したい
    ↓
シンボル名は分かっている?
    ├─ YES → find_symbol (name_path指定)
    └─ NO
        ↓
    部分的に分かっている?
        ├─ YES → find_symbol (substring_matching=true)
        └─ NO
            ↓
        キーワードで検索したい?
            ├─ YES → search_for_pattern (最終手段)
            └─ NO → get_symbols_overview でファイルを探索
```

## パラメータ詳細解説

### include_body パラメータ

**`include_body=false` (デフォルト、推奨)**:
```
結果:
class UserService {
  fetchUser(id: string): Promise<User>
  createUser(data: CreateUserDto): Promise<User>
}

利点:
- トークン消費が少ない
- 構造を素早く把握
```

**`include_body=true` (詳細確認時のみ)**:
```
結果:
class UserService {
  async fetchUser(id: string): Promise<User> {
    const user = await this.repository.findById(id);
    if (!user) {
      throw new UserNotFoundError(id);
    }
    return user;
  }

  async createUser(data: CreateUserDto): Promise<User> {
    // 実装詳細...
  }
}

注意:
- トークン消費が多い
- 実装ロジックを理解する必要がある場合のみ使用
```

**推奨フロー**:
1. `include_body=false` で構造確認
2. 必要なシンボルのみ `include_body=true` で詳細確認

### depth パラメータ

**`depth=0`**:
```
指定シンボルのみ

find_symbol({ name_path: "UserService", depth: 0 })

結果:
class UserService { ... }
```

**`depth=1`**:
```
直接の子シンボルまで（メソッド、プロパティ）

find_symbol({ name_path: "UserService", depth: 1 })

結果:
class UserService {
  fetchUser(...)
  createUser(...)
  updateUser(...)
  deleteUser(...)
}
```

**`depth=2`**:
```
孫シンボルまで（ネストされたクラス、内部関数など）

find_symbol({ name_path: "UserService", depth: 2 })

結果:
class UserService {
  fetchUser(...) {
    // 内部の詳細構造も表示
  }
  ...
  private class InternalHelper { ... }
}
```

**推奨設定**:
* クラス構造の把握: `depth=1`
* 関数の詳細: `depth=0` + `include_body=true`

### relative_path パラメータ

**指定する場合（推奨）**:
```
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})

利点:
- 検索範囲が限定され、高速
- 曖昧性がない（同名シンボルが他にあっても問題ない）
```

**指定しない場合**:
```
find_symbol({
  name_path: "UserService"
})

動作:
- プロジェクト全体から検索
- 複数見つかる場合、全て返される

使用例:
- シンボルがどのファイルにあるか不明な場合
- プロジェクト全体から同名シンボルを列挙したい場合
```

## エッジケースの扱い方

### ケース1: 動的に生成されるシンボル

**問題**: ファクトリーパターンや動的importで生成されるシンボルは検索できない

**対処法**:
1. 生成元のファクトリーメソッドを検索
2. 実際の生成箇所を `search_for_pattern` で探す
3. 型定義から推測

**例**:
```
// 動的生成
const UserService = ServiceFactory.create('user');

// 検索方法
1. find_symbol({ name_path: "ServiceFactory.create" })
2. search_for_pattern({ pattern: "ServiceFactory.create\\('user'\\)" })
```

### ケース2: 同名シンボルが複数存在

**問題**: プロジェクト内に同名のクラスや関数が複数

**対処法**:
1. `relative_path` で範囲を限定
2. パッケージ名やモジュール名を含めて検索

**例**:
```
// 複数の UserService
- src/services/user-service.ts (メインの UserService)
- src/legacy/user-service.ts (古い UserService)

// 明確に指定
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})
```

### ケース3: ジェネリック型のシンボル

**問題**: `Repository<User>` のようなジェネリック型

**対処法**:
1. ベースクラス/インターフェースを検索
2. 型パラメータは無視

**例**:
```
// ジェネリッククラス
class Repository<T> { ... }
class UserRepository extends Repository<User> { ... }

// 検索
find_symbol({ name_path: "Repository" })  // ベースクラス
find_symbol({ name_path: "UserRepository" })  // 具体的な実装
```

### ケース4: プライベートシンボル

**問題**: private / internal なシンボルも検索したい

**対処法**:
* LSPは通常プライベートシンボルも返す
* アクセス修飾子で除外されることは稀

**例**:
```
class UserService {
  private validateUser(user: User): boolean { ... }
}

// プライベートメソッドも検索可能
find_symbol({
  name_path: "UserService.validateUser",
  include_body: true
})
```

## 実例とサンプルクエリ

### 実例1: モジュール全体の理解

**タスク**: 認証モジュールの構造を理解する

```
ステップ1: ファイル一覧
get_symbols_overview({
  relative_path: "src/auth/auth-service.ts"
})

結果:
- AuthService (class)
- TokenGenerator (class)
- hashPassword (function)
- verifyPassword (function)

ステップ2: メインクラスの構造
find_symbol({
  name_path: "AuthService",
  relative_path: "src/auth/auth-service.ts",
  include_body: false,
  depth: 1
})

結果:
class AuthService {
  constructor(userRepository, tokenGenerator)
  login(email, password): Promise<AuthResult>
  logout(token): Promise<void>
  validateToken(token): Promise<User>
}

ステップ3: ログインロジックの詳細（必要な場合）
find_symbol({
  name_path: "AuthService.login",
  relative_path: "src/auth/auth-service.ts",
  include_body: true
})

トークン使用量:
- ステップ1: 約10行
- ステップ2: 約15行
- ステップ3: 約30行
合計: 約55行（ファイル全体を読む場合の1/4以下）
```

### 実例2: 依存関係の追跡

**タスク**: UserServiceがどこで使われているか調べる

```
ステップ1: 参照箇所を取得
find_referencing_symbols({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})

結果:
- src/api/user-controller.ts
- src/api/admin-controller.ts
- src/services/profile-service.ts

ステップ2: 各使用箇所の詳細確認（必要に応じて）
find_symbol({
  name_path: "UserController",
  relative_path: "src/api/user-controller.ts",
  include_body: false,
  depth: 1
})

ステップ3: メモリに保存
write_memory({
  filename: "component_user_service_dependencies.md",
  content: "..."
})
```

### 実例3: 設計パターンの発見

**タスク**: Singletonパターンを探す

```
ステップ1: getInstance メソッドを検索
find_symbol({
  name_path: "getInstance",
  substring_matching: true
})

結果:
- DatabaseConnection.getInstance
- ConfigService.getInstance
- LoggerService.getInstance

ステップ2: 各クラスの詳細確認
find_symbol({
  name_path: "DatabaseConnection",
  include_body: false,
  depth: 1
})

結果:
class DatabaseConnection {
  private static instance: DatabaseConnection
  private constructor()
  static getInstance(): DatabaseConnection
  connect(): Promise<void>
  disconnect(): Promise<void>
}

判断: Singletonパターン（private constructor + getInstance）

ステップ3: パターンをメモリに記録
write_memory({
  filename: "pattern_singleton_database_connection.md",
  content: "..."
})
```

## パフォーマンス最適化のヒント

### ヒント1: キャッシュとしてのメモリ活用

同じシンボルを何度も検索しない:
```
悪い例:
- 調査のたびに find_symbol を実行
- 同じ情報を何度も取得

良い例:
1. 初回調査時に find_symbol
2. 結果を write_memory
3. 次回以降は read_memory
```

### ヒント2: 検索範囲の限定

プロジェクト全体ではなく、特定ディレクトリに限定:
```
遅い:
find_symbol({ name_path: "UserService" })

速い:
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/"  // ディレクトリで限定
})
```

### ヒント3: 段階的詳細化

一度に全てを取得しない:
```
非効率:
find_symbol({ name_path: "UserService", include_body: true, depth: 2 })

効率的:
1. find_symbol({ ..., include_body: false, depth: 1 })  # 構造のみ
2. 必要なメソッドのみ include_body: true で詳細取得
```

## トラブルシューティング

### 問題1: シンボルが見つからない

**原因**:
* name_path の誤り
* relative_path の誤り
* シンボルが実際に存在しない

**対処法**:
```
1. get_symbols_overview でファイル内のシンボル一覧を確認
2. 正しい name_path を特定
3. 相対パスがプロジェクトルートからのパスであることを確認
```

### 問題2: 検索結果が多すぎる

**原因**:
* substring_matching で広範囲に一致
* relative_path 未指定でプロジェクト全体を検索

**対処法**:
```
1. relative_path で範囲を限定
2. より具体的な name_path を使用
3. include_kinds / exclude_kinds でフィルタリング
```

### 問題3: 期待した詳細が得られない

**原因**:
* include_body=false で実装が見えない
* depth が不足

**対処法**:
```
1. include_body=true に変更
2. depth を増やす
3. 特定のメソッドに name_path を絞る
```

## まとめ

セマンティック検索を効果的に使うポイント:

1. **段階的アプローチ**: 概要 → 構造 → 詳細
2. **トークン効率**: include_body=false から始める
3. **範囲限定**: relative_path を指定
4. **メモリ活用**: 調査結果を保存して再利用
5. **適切なツール選択**: シナリオに応じて最適なツールを使用

これらの原則に従うことで、トークン使用量を最小化しながら効率的にコードベースを理解できます。

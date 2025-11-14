# コード理解ワークフロー詳細ガイド

このドキュメントは、code-intelligenceスキルにおけるコード理解の詳細なワークフローを説明します。

## プロジェクト構造の段階的探索

新しいコードベースを理解する際の推奨手順を詳しく説明します。

### ステップ1: メモリ確認

**目的**: 既存知識を活用し、重複調査を避ける

**手順**:

1. **利用可能なメモリを一覧**
```
list_memories()
```

2. **関連するメモリを特定**
   * プロジェクト名やモジュール名でフィルタ
   * 最近の調査履歴を確認
   * アーキテクチャ概要ファイルを探す

3. **メモリを読み取る**
```
read_memory({
  filename: "architecture_overview.md"
})
```

4. **既知の情報を元に探索範囲を絞る**
   * 既に調査済みのモジュールはスキップ
   * 未調査の領域を特定
   * 変更があった箇所を優先

**例**:
```
タスク: "ユーザー認証機能の理解"

メモリ確認:
- module_auth_structure.md が存在
- 最終更新: 2週間前
- 内容: AuthService の基本構造

判断:
- 基本構造は既知
- 最近の変更をチェック
- 認証フローの詳細を追加調査
```

### ステップ2: ディレクトリ構造の把握

**目的**: プロジェクト全体の構成を理解

**ツール**: `list_dir`

**パラメータ**:
* `relative_path`: "." (プロジェクトルート)
* `recursive`: true

**確認項目**:
* 主要ディレクトリの役割
* ソースコードの配置
* テストコードの配置
* 設定ファイルの場所

**実行例**:
```
list_dir({
  relative_path: ".",
  recursive: true
})
```

**結果の分析**:
```
プロジェクト構造:
/
├── src/
│   ├── api/          # REST APIエンドポイント
│   ├── services/     # ビジネスロジック
│   ├── models/       # データモデル
│   ├── middleware/   # Express middleware
│   └── utils/        # ユーティリティ関数
├── tests/
│   ├── unit/         # ユニットテスト
│   └── integration/  # 統合テスト
├── config/           # 設定ファイル
└── docs/             # ドキュメント

推測:
- 典型的なMVCアーキテクチャ
- API層 → Services層 → Models層
- テストは分離されている
```

### ステップ3: エントリポイントの特定

**目的**: アプリケーションの起動ポイントを見つける

**方法**:

1. **package.json の確認**
```
read_file({
  path: "package.json"
})
```

探すフィールド:
* `main`: メインエントリポイント
* `scripts.start`: 起動スクリプト
* `scripts.dev`: 開発環境の起動

2. **一般的なエントリポイントファイルを探す**
* `index.ts` / `main.ts` / `app.ts`
* `src/index.ts` / `src/main.ts`
* `server.ts` / `server.js`

3. **エントリポイントファイルの概要取得**
```
get_symbols_overview({
  relative_path: "src/index.ts"
})
```

**例**:
```
package.json:
{
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  }
}

エントリポイント: src/index.ts

get_symbols_overview の結果:
- initializeApp (function)
- createServer (function)
- startServer (function)
- app (const)

推測:
- initializeApp で初期化
- createServer でサーバー構築
- startServer で起動
```

### ステップ4: モジュール構造の理解

**目的**: 各主要モジュールの役割と構成を把握

**手順**:

各主要モジュールに対して以下を実施:

1. **トップレベルシンボルを取得**
```
get_symbols_overview({
  relative_path: "src/services/user-service.ts"
})
```

2. **主要なクラス・関数を詳細調査**
```
find_symbol({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts",
  include_body: false,  // まずは構造のみ
  depth: 1  # メソッド名まで取得
})
```

3. **依存関係を追跡**
```
find_referencing_symbols({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})
```

4. **モジュール構造をメモリに保存**
```
write_memory({
  filename: "module_user_service_structure.md",
  content: "..." // モジュール構造の詳細
})
```

**例**:
```markdown
# モジュール user-service の構造

## 調査日時
2025-01-15

## 対象ファイル
* src/services/user-service.ts

## シンボル構成
* UserService クラス (class)
  * constructor(repository: UserRepository)
  * fetchUser(id: string): Promise<User>
  * createUser(data: CreateUserDto): Promise<User>
  * updateUser(id: string, data: UpdateUserDto): Promise<User>
  * deleteUser(id: string): Promise<void>
  * validateUserData(data: any): boolean

## 依存関係
* UserRepository (src/repositories/user-repository.ts)
* CreateUserDto (src/dto/user.dto.ts)
* UpdateUserDto (src/dto/user.dto.ts)
* User (src/models/user.ts)

## 被依存（このクラスを使用している箇所）
* UserController (src/api/user-controller.ts)
* AdminService (src/services/admin-service.ts)

## 設計パターン
* Repository パターン（データアクセス層を分離）

## 参照ポイント
* CRUDの全操作を提供
* バリデーションロジックを含む
* 非同期処理（Promise）
```

### ステップ5: アーキテクチャの可視化

**目的**: 発見した情報を統合し、全体像を把握

**実施内容**:

1. **モジュール間の依存関係を整理**
```
依存グラフ:
UserController → UserService → UserRepository → Database
              ↘ UserDto
```

2. **レイヤー構造を特定**
```
レイヤー構造:
┌─────────────────────┐
│  Presentation層     │ (Controllers, Middleware)
├─────────────────────┤
│  Business Logic層   │ (Services)
├─────────────────────┤
│  Data Access層      │ (Repositories, Models)
├─────────────────────┤
│  Database層         │ (PostgreSQL)
└─────────────────────┘
```

3. **設計パターンを識別**
* MVC パターン
* Repository パターン
* DTO パターン
* Singleton パターン（設定管理）

4. **重要な設計判断を記録**
* 非同期処理の統一（Promise/async-await）
* エラーハンドリング戦略（try-catch + カスタムエラー）
* 型安全性（TypeScript strict mode）

5. **architecture_overview.mdに保存**
```
write_memory({
  filename: "architecture_overview.md",
  content: "..." // アーキテクチャ全体の説明
})
```

**例**:
```markdown
# プロジェクトアーキテクチャ概要

## プロジェクト名
User Management API

## アーキテクチャスタイル
Layered Architecture (3層)

## レイヤー構成
### Presentation層 (API)
- UserController
- AuthController
- AdminController
- Middleware (authentication, validation, error handling)

### Business Logic層 (Services)
- UserService
- AuthService
- AdminService

### Data Access層 (Repositories + Models)
- UserRepository
- SessionRepository
- User Model
- Session Model

## 主要な依存関係
API → Services → Repositories → Database

## 使用技術
- Language: TypeScript
- Framework: Express.js
- Database: PostgreSQL
- ORM: TypeORM
- Testing: Jest

## 設計パターン
- MVC パターン
- Repository パターン
- DTO パターン
- Singleton パターン (Database connection)

## 設計原則
- Dependency Injection
- Single Responsibility Principle
- 型安全性（strict TypeScript）

## エントリポイント
src/index.ts

## 重要な設定ファイル
- tsconfig.json (TypeScript設定)
- ormconfig.json (データベース設定)
- .env (環境変数)
```

## 依存関係の追跡

モジュール間の依存関係を詳細に理解する手順を説明します。

### 手順1: 対象モジュールのエクスポートを確認

**目的**: モジュールが外部に公開しているシンボルを特定

**方法**:
```
get_symbols_overview({
  relative_path: "src/services/auth-service.ts"
})
```

**確認項目**:
* export されているクラス・関数・定数
* default export か named export か
* 型定義のエクスポート

**例**:
```
エクスポートシンボル:
- AuthService (class) - default export
- AuthConfig (interface) - named export
- hashPassword (function) - named export
```

### 手順2: 各エクスポートの参照を追跡

**目的**: どこで使用されているかを把握

**方法**:
```
find_referencing_symbols({
  name_path: "AuthService",
  relative_path: "src/services/auth-service.ts"
})
```

**結果の分析**:
```
参照箇所:
1. src/api/auth-controller.ts
   - import { AuthService } from '../services/auth-service'
   - const authService = new AuthService()

2. src/middleware/auth-middleware.ts
   - import { AuthService } from '../services/auth-service'
   - AuthService.getInstance().verify(token)

3. tests/services/auth-service.test.ts
   - テストコード

判断:
- AuthController と AuthMiddleware で使用
- Singleton パターンを使用している可能性
```

### 手順3: import文のパターン検索

**目的**: 動的importや特殊なimportを発見

**方法**:
```
search_for_pattern({
  pattern: "import.*auth-service",
  relative_path: "src/"
})
```

**確認項目**:
* 静的import
* 動的import (`import()`)
* require() (CommonJS)
* re-export

### 手順4: 依存グラフの構築

**目的**: モジュール間の関係を可視化

**方法**:

1. **各モジュールの依存を記録**
```
AuthService の依存:
- UserRepository
- TokenGenerator
- ConfigService

UserRepository の依存:
- Database
- User Model

TokenGenerator の依存:
- jwt library
- ConfigService
```

2. **依存グラフを作成**
```
AuthService
  ↓
  ├→ UserRepository → Database
  ├→ TokenGenerator → jwt library
  └→ ConfigService → .env
```

3. **循環依存をチェック**
```
循環依存チェック:
AuthService → UserRepository → AuthService? → NO
AuthService → ConfigService → AuthService? → NO
結論: 循環依存なし
```

### 手順5: 結果をメモリに保存

**保存内容**:
```
write_memory({
  filename: "component_auth_service_dependencies.md",
  content: `
# AuthService の依存関係

## 直接依存
- UserRepository (データアクセス)
- TokenGenerator (JWT生成)
- ConfigService (設定読み込み)

## 間接依存
- Database (via UserRepository)
- User Model (via UserRepository)
- jwt library (via TokenGenerator)

## 被依存
- AuthController (API層)
- AuthMiddleware (認証チェック)

## 依存グラフ
AuthController → AuthService → UserRepository → Database
AuthMiddleware → AuthService → TokenGenerator → jwt library

## 循環依存
なし

## 注意点
- ConfigService はアプリケーション全体で共有
- UserRepository は他のサービスでも使用
  `
})
```

## 設計パターンの識別

コードベースから設計パターンを見つける方法を詳しく説明します。

### 検出すべきパターン詳細

#### Creational Patterns（生成パターン）

**Singleton パターン**
```typescript
// 検出方法
class AuthService {
  private static instance: AuthService;
  private constructor() {} // private constructor

  static getInstance(): AuthService { // getInstance メソッド
    if (!AuthService.instance) {
      AuthService.instance = new AuthService();
    }
    return AuthService.instance;
  }
}
```

検索キーワード:
* `getInstance`
* `private constructor`
* `static instance`

**Factory パターン**
```typescript
// 検出方法
class UserFactory {
  create(type: string): User { // create メソッド
    switch (type) {
      case 'admin': return new AdminUser();
      case 'regular': return new RegularUser();
    }
  }
}
```

検索キーワード:
* `Factory` (クラス名)
* `create` / `build` / `make` (メソッド名)

**Builder パターン**
```typescript
// 検出方法
class UserBuilder {
  private user: User = new User();

  setName(name: string): UserBuilder { // チェーン可能
    this.user.name = name;
    return this;
  }

  setEmail(email: string): UserBuilder {
    this.user.email = email;
    return this;
  }

  build(): User {
    return this.user;
  }
}
```

検索キーワード:
* `Builder` (クラス名)
* メソッドが `this` を返す（チェーン可能）
* `build()` メソッド

#### Structural Patterns（構造パターン）

**Adapter パターン**
```typescript
// 検出方法
class DatabaseAdapter {
  constructor(private legacyDb: LegacyDatabase) {}

  // 新しいインターフェースを提供
  async findById(id: string): Promise<User> {
    return this.legacyDb.getUserById(id); // 古いAPIを呼び出す
  }
}
```

検索キーワード:
* `Adapter` (クラス名)
* インターフェース変換

**Decorator パターン**
```typescript
// 検出方法
class LoggingUserService implements UserService {
  constructor(private userService: UserService) {} // 元のオブジェクトを保持

  async fetchUser(id: string): Promise<User> {
    console.log(`Fetching user ${id}`);
    return this.userService.fetchUser(id); // 元のメソッドを呼び出す
  }
}
```

検索キーワード:
* `Decorator` (クラス名)
* コンストラクタで同じインターフェースのオブジェクトを受け取る

**Proxy パターン**
```typescript
// 検出方法
class UserServiceProxy implements UserService {
  constructor(private realService: UserService) {}

  async fetchUser(id: string): Promise<User> {
    // アクセス制御やキャッシュなど
    if (this.cache.has(id)) {
      return this.cache.get(id);
    }
    return this.realService.fetchUser(id);
  }
}
```

検索キーワード:
* `Proxy` (クラス名)
* 元のオブジェクトへの参照を保持

#### Behavioral Patterns（振る舞いパターン）

**Strategy パターン**
```typescript
// 検出方法
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardStrategy implements PaymentStrategy {
  pay(amount: number): void { /* ... */ }
}

class PayPalStrategy implements PaymentStrategy {
  pay(amount: number): void { /* ... */ }
}
```

検索キーワード:
* `Strategy` (クラス名・インターフェース名)
* アルゴリズムを入れ替え可能なインターフェース

**Observer パターン**
```typescript
// 検出方法
class EventEmitter {
  private listeners: Map<string, Function[]> = new Map();

  subscribe(event: string, callback: Function): void { /* ... */ }
  notify(event: string, data: any): void { /* ... */ }
}
```

検索キーワード:
* `Observer` (クラス名)
* `subscribe` / `on` / `addEventListener`
* `notify` / `emit` / `trigger`

**Command パターン**
```typescript
// 検出方法
interface Command {
  execute(): void;
}

class CreateUserCommand implements Command {
  execute(): void { /* ... */ }
}
```

検索キーワード:
* `Command` (クラス名・インターフェース名)
* `execute()` メソッド

### 識別方法詳細

#### ステップ1: シンボル名からパターンを推測

**方法**:
```
get_symbols_overview({ relative_path: "src/" })
```

**探すパターン**:
* `*Factory`, `*Builder`, `*Adapter` などの接尾辞
* `create*`, `build*`, `execute` などのメソッド名
* `getInstance`, `subscribe`, `notify` などの特徴的なメソッド

#### ステップ2: 候補を詳細調査

**方法**:
```
find_symbol({
  name_path: "UserFactory",
  relative_path: "src/factories/user-factory.ts",
  include_body: true,  // メソッド構造を確認
  depth: 2
})
```

**確認項目**:
* クラス構造
* メソッド名とシグネチャ
* コンストラクタのパラメータ
* 返り値の型

#### ステップ3: 使用パターンを確認

**方法**:
```
find_referencing_symbols({
  name_path: "UserFactory",
  relative_path: "src/factories/user-factory.ts"
})
```

**確認項目**:
* 実際にどう使われているか
* Factory メソッドがどのように呼ばれているか
* パターンの意図が実現されているか

#### ステップ4: パターンをメモリに記録

**保存内容**:
```
write_memory({
  filename: "pattern_factory_user_creation.md",
  content: `
# Factory パターン: User作成

## パターン名
Factory Method Pattern

## 実装箇所
src/factories/user-factory.ts

## 説明
ユーザータイプに応じて適切なUserインスタンスを生成するFactoryパターン

## 利点
- ユーザータイプの追加が容易
- 生成ロジックの一元管理
- クライアントコードが具体的なクラスに依存しない

## 実装詳細
class UserFactory {
  create(type: 'admin' | 'regular' | 'guest'): User {
    switch (type) {
      case 'admin': return new AdminUser();
      case 'regular': return new RegularUser();
      case 'guest': return new GuestUser();
    }
  }
}

## 使用例
const factory = new UserFactory();
const adminUser = factory.create('admin');
const regularUser = factory.create('regular');

## 使用箇所
- UserController (ユーザー作成API)
- AuthService (認証後のユーザー生成)
  `
})
```

## 実践ワークフロー例

### 例1: 新規コードベース理解

**シナリオ**: 新しいプロジェクトに参加し、コードベース全体を理解する

**ワークフロー**:

1. **メモリ確認**
```
list_memories() → 空（初回）
```

2. **ディレクトリ構造把握**
```
list_dir({ relative_path: ".", recursive: true })

結果:
/src
  /api
  /services
  /models
  /utils
/tests
/config
```

3. **エントリポイント特定**
```
read_file({ path: "package.json" })
→ main: "src/index.ts"

get_symbols_overview({ relative_path: "src/index.ts" })
→ initializeApp, startServer
```

4. **主要モジュール調査**
```
// API層
get_symbols_overview({ relative_path: "src/api/user-controller.ts" })

// Service層
get_symbols_overview({ relative_path: "src/services/user-service.ts" })

// Model層
get_symbols_overview({ relative_path: "src/models/user.ts" })
```

5. **依存関係追跡**
```
find_referencing_symbols({
  name_path: "UserService",
  relative_path: "src/services/user-service.ts"
})
→ UserController, AdminController
```

6. **アーキテクチャ記録**
```
write_memory({
  filename: "architecture_overview.md",
  content: "..." // 全体構造
})
```

**所要時間**: 30-60分（中規模プロジェクト）

### 例2: 機能追加の準備

**シナリオ**: 既存プロジェクトに新機能（パスワードリセット）を追加する準備

**ワークフロー**:

1. **関連モジュールのメモリ確認**
```
list_memories()
→ module_auth_structure.md が存在

read_memory({ filename: "module_auth_structure.md" })
→ AuthService の基本構造を確認
```

2. **関連シンボルの詳細調査**
```
find_symbol({
  name_path: "AuthService",
  relative_path: "src/services/auth-service.ts",
  include_body: true,
  depth: 2
})
→ 既存の認証メソッドを確認
```

3. **類似機能の参照**
```
search_for_pattern({
  pattern: "password.*reset",
  relative_path: "src/"
})
→ 既存のパスワード関連機能を検索
```

4. **依存関係確認**
```
find_referencing_symbols({
  name_path: "AuthService",
  relative_path: "src/services/auth-service.ts"
})
→ 変更が影響する箇所を把握
```

5. **実装計画をメモリに記録**
```
write_memory({
  filename: "feature_password_reset_plan.md",
  content: `
# パスワードリセット機能追加計画

## 追加が必要なもの
- AuthService.requestPasswordReset() メソッド
- AuthService.resetPassword() メソッド
- PasswordResetToken モデル
- パスワードリセット用メール送信

## 影響範囲
- AuthController に新規エンドポイント追加
- メール送信サービスとの連携
  `
})
```

**所要時間**: 15-30分

### 例3: バグ調査

**シナリオ**: ユーザー登録時に時々エラーが発生する問題を調査

**ワークフロー**:

1. **エラー箇所の特定**
```
// エラーログから関連するシンボルを特定
find_symbol({
  name_path: "UserService.createUser",
  relative_path: "src/services/user-service.ts",
  include_body: true
})
```

2. **依存関係の追跡**
```
find_referencing_symbols({
  name_path: "UserService.createUser",
  relative_path: "src/services/user-service.ts"
})
→ どこから呼ばれているか確認
```

3. **呼び出しフローの分析**
```
UserController.register()
  ↓
UserService.createUser()
  ↓
UserRepository.save()
  ↓
Database

各ステップを詳細調査
```

4. **関連するシンボルの詳細確認**
```
find_symbol({
  name_path: "UserRepository.save",
  relative_path: "src/repositories/user-repository.ts",
  include_body: true
})
```

5. **調査結果をメモリに記録**
```
write_memory({
  filename: "bug_user_registration_investigation.md",
  content: `
# ユーザー登録エラー調査

## 問題
createUser 実行時に時々エラー

## 調査結果
UserRepository.save() でユニーク制約違反
→ メールアドレスの重複チェックが不十分

## 原因
並行リクエスト時に重複チェックと保存の間でrace condition発生

## 解決策
1. データベースレベルでユニーク制約
2. try-catchで重複エラーをハンドリング
3. 適切なエラーメッセージを返す
  `
})
```

**所要時間**: 20-40分

## まとめ

効果的なコード理解のポイント:

1. **段階的探索**: 概要から詳細へ
2. **メモリ活用**: 既存知識を再利用
3. **依存関係追跡**: モジュール間の関係を把握
4. **パターン識別**: 設計意図を理解
5. **記録**: 発見した知見を保存

これらのワークフローに従うことで、効率的にコードベースを理解できます。

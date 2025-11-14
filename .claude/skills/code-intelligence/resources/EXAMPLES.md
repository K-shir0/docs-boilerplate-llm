# Code Intelligence 実践例

このドキュメントは、code-intelligenceスキルの具体的な使用例を示します。

## セマンティック検索の例

### 例1: 認証機能の調査

**シナリオ**: 新しいプロジェクトで認証機能がどのように実装されているかを理解したい

**悪いアプローチ（トークン効率が悪い）**:
```
1. Read tool でファイル全体を読み込む
   - src/auth/index.ts (200行) を全て読む
   - src/auth/middleware.ts (150行) を全て読む
   - src/auth/validators.ts (100行) を全て読む

問題点:
- 不要な詳細まで読み込む（コメント、import文、実装詳細）
- トークンを450行分消費
- 必要な情報が埋もれる
```

**良いアプローチ（段階的・効率的）**:
```
ステップ1: メモリ確認
list_memories()
→ module_auth_structure.md が見つかる

read_memory({ filename: "module_auth_structure.md" })
→ 既存の構造を確認（トークン節約）

ステップ2: 概要取得
get_symbols_overview({
  relative_path: "src/auth/index.ts"
})

結果:
- AuthService (class)
- authMiddleware (function)
- TokenValidator (class)
→ 主要なシンボルを特定（約20行相当）

ステップ3: 必要なシンボルのみ詳細調査
find_symbol({
  name_path: "AuthService",
  relative_path: "src/auth/index.ts",
  include_body: false,  // まずは構造のみ
  depth: 1
})

結果:
- login(email, password)
- logout(token)
- validateToken(token)
→ メソッド一覧のみ取得（約10行相当）

ステップ4: 必要に応じて本体を確認
find_symbol({
  name_path: "AuthService.login",
  relative_path: "src/auth/index.ts",
  include_body: true  // ログインロジックのみ詳細確認
})

ステップ5: メモリに保存
write_memory({
  filename: "module_auth_structure.md",
  content: "..."
})

トークン使用量:
- 悪いアプローチ: 450行
- 良いアプローチ: 約50行（90%削減）
```

### 例2: データベースモデルの理解

**シナリオ**: Userモデルの構造とリレーションを理解したい

**ワークフロー**:
```
ステップ1: モデルファイルの概要取得
get_symbols_overview({
  relative_path: "src/models/user.ts"
})

結果:
- User (class)
- UserRole (enum)
- UserStatus (enum)

ステップ2: Userクラスの詳細確認
find_symbol({
  name_path: "User",
  relative_path: "src/models/user.ts",
  include_body: false,
  depth: 2  // プロパティとメソッドまで
})

結果:
class User {
  // プロパティ
  id: string
  email: string
  password: string
  role: UserRole
  status: UserStatus
  createdAt: Date
  updatedAt: Date

  // メソッド
  validatePassword(password: string): boolean
  toJSON(): object
}

ステップ3: リレーションの確認
find_referencing_symbols({
  name_path: "User",
  relative_path: "src/models/user.ts"
})

結果:
- UserRepository (データアクセス)
- AuthService (認証)
- UserController (API)
- Post モデル (1対多リレーション)

ステップ4: メモリに保存
write_memory({
  filename: "model_user_structure.md",
  content: `
# User モデル構造

## プロパティ
- id: string (主キー)
- email: string (ユニーク)
- password: string (ハッシュ化)
- role: UserRole (権限)
- status: UserStatus (アクティブ状態)
- createdAt: Date
- updatedAt: Date

## メソッド
- validatePassword() - パスワード検証
- toJSON() - JSON変換（パスワード除外）

## リレーション
- User → Post (1対多)

## 使用箇所
- UserRepository
- AuthService
- UserController
  `
})
```

## リファクタリングの例

### 例1: 変数名の統一

**シナリオ**: `getUserData` 関数を `fetchUserData` にリネームして命名規則を統一

**悪いアプローチ（危険）**:
```
1. いきなり rename_symbol を実行
   rename_symbol({
     name_path: "getUserData",
     relative_path: "src/services/user-service.ts",
     new_name: "fetchUserData"
   })

問題点:
- 事前確認なし
- 影響範囲が不明
- エラーが発生しても気づかない
```

**良いアプローチ（安全）**:
```
ステップ1: 現在の状態確認
mcp__ide__getDiagnostics({})

結果:
- エラー: 0件
- 警告: 2件（無関係）

ステップ2: 対象シンボルの特定
find_symbol({
  name_path: "getUserData",
  relative_path: "src/services/user-service.ts",
  include_body: false
})

結果:
function getUserData(userId: string): Promise<User>

ステップ3: 参照箇所の追跡
find_referencing_symbols({
  name_path: "getUserData",
  relative_path: "src/services/user-service.ts"
})

結果:
参照箇所: 4件
- src/api/user-controller.ts:23
- src/api/admin-controller.ts:67
- src/services/profile-service.ts:45
- tests/services/user-service.test.ts:12

影響範囲: 低リスク（4件のみ）

ステップ4: リネーム実行
rename_symbol({
  name_path: "getUserData",
  relative_path: "src/services/user-service.ts",
  new_name: "fetchUserData"
})

ステップ5: 検証
mcp__ide__getDiagnostics({})

結果:
- エラー: 0件
- 警告: 2件（変化なし）
→ 成功

ステップ6: メモリに記録
write_memory({
  filename: "refactoring_user_data_fetch_20250115.md",
  content: `
# リファクタリング: getUserData → fetchUserData

## 変更内容
関数名をgetUserDataからfetchUserDataに変更

## 理由
命名規則の統一（データ取得はfetchプレフィックス）

## 影響範囲
4ファイル、4箇所
- src/services/user-service.ts:15 (定義)
- src/api/user-controller.ts:23 (呼び出し)
- src/api/admin-controller.ts:67 (呼び出し)
- src/services/profile-service.ts:45 (呼び出し)

## 検証結果
エラー: 0件
テスト: 全て通過

## 学んだ教訓
rename_symbolは全参照を自動更新するため安全
  `
})
```

### 例2: 関数の抽出

**シナリオ**: 複雑なメソッドから検証ロジックを別関数に抽出

**ワークフロー**:
```
ステップ1: 現在のコードを確認
find_symbol({
  name_path: "UserService.createUser",
  relative_path: "src/services/user-service.ts",
  include_body: true
})

結果（現在のコード）:
async createUser(data: CreateUserDto): Promise<User> {
  // 検証ロジック（長い）
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  if (!data.password) throw new Error('Password required');
  if (data.password.length < 8) throw new Error('Password too short');

  // ユーザー作成ロジック
  const hashedPassword = await hashPassword(data.password);
  const user = await this.repository.save({
    email: data.email,
    password: hashedPassword
  });
  return user;
}

問題: 検証ロジックが長く、可読性が低い

ステップ2: 新しい検証関数を挿入
insert_before_symbol({
  name_path: "UserService.createUser",
  relative_path: "src/services/user-service.ts",
  body: `
  private validateUserData(data: CreateUserDto): void {
    if (!data.email) {
      throw new Error('Email required');
    }
    if (!data.email.includes('@')) {
      throw new Error('Invalid email');
    }
    if (!data.password) {
      throw new Error('Password required');
    }
    if (data.password.length < 8) {
      throw new Error('Password too short');
    }
  }

`
})

ステップ3: createUserメソッドを簡潔化
replace_symbol_body({
  name_path: "UserService.createUser",
  relative_path: "src/services/user-service.ts",
  body: `async createUser(data: CreateUserDto): Promise<User> {
    this.validateUserData(data);

    const hashedPassword = await hashPassword(data.password);
    const user = await this.repository.save({
      email: data.email,
      password: hashedPassword
    });
    return user;
  }`
})

ステップ4: 検証
mcp__ide__getDiagnostics({})
→ エラー: 0件

ステップ5: メモリに記録
write_memory({
  filename: "refactoring_extract_validation_20250115.md",
  content: "..."
})
```

## メモリ活用の例

### 例1: プロジェクト初回調査

**メモリファイル**: `architecture_overview.md`

```markdown
# E-Commerce API アーキテクチャ概要

## 調査日時
2025-01-15

## プロジェクト概要
Node.js + Express による REST API
ドメイン: E-Commerce (商品管理、注文、決済)

## アーキテクチャスタイル
Layered Architecture (3層)

## ディレクトリ構造
/src
  /api         - REST APIエンドポイント
  /services    - ビジネスロジック
  /repositories - データアクセス
  /models      - データモデル
  /middleware  - Express middleware
  /utils       - ユーティリティ
/tests
  /unit
  /integration

## 主要モジュール

### API層
- ProductController - 商品API
- OrderController - 注文API
- PaymentController - 決済API
- UserController - ユーザーAPI

### Service層
- ProductService - 商品ビジネスロジック
- OrderService - 注文処理
- PaymentService - 決済処理
- UserService - ユーザー管理

### Repository層
- ProductRepository - 商品データアクセス
- OrderRepository - 注文データアクセス
- PaymentRepository - 決済データアクセス
- UserRepository - ユーザーデータアクセス

## 依存関係フロー
Controller → Service → Repository → Database

## 使用技術
- Runtime: Node.js 18
- Framework: Express.js
- Language: TypeScript
- Database: PostgreSQL
- ORM: TypeORM
- Testing: Jest

## 設計パターン
- MVC パターン
- Repository パターン
- DTO パターン
- Singleton パターン (Database connection)
- Factory パターン (Payment processing)

## エントリポイント
src/index.ts
- initializeApp() - アプリケーション初期化
- connectDatabase() - データベース接続
- setupMiddleware() - ミドルウェア設定
- startServer() - サーバー起動

## 環境変数
- DATABASE_URL - データベース接続文字列
- JWT_SECRET - JWT署名キー
- STRIPE_API_KEY - Stripe API キー
- PORT - サーバーポート番号

## 次回調査時の参照ポイント
- 決済処理の詳細（PaymentService）
- 注文ステータス管理（OrderService）
- 在庫管理ロジック（ProductService）
```

### 例2: リファクタリング記録

**メモリファイル**: `refactoring_payment_service_20250115.md`

```markdown
# リファクタリング: Payment Service の改善

## 日時
2025-01-15

## 変更内容
PaymentService を Strategy パターンに基づいて再設計し、複数の決済プロバイダー（Stripe, PayPal）をサポート

## 変更理由
- 決済プロバイダーの追加が困難
- if-else による分岐が複雑化
- テストが困難

## 影響範囲

### 新規作成
- src/services/payment/payment-strategy.interface.ts (インターフェース)
- src/services/payment/stripe-strategy.ts (Stripe実装)
- src/services/payment/paypal-strategy.ts (PayPal実装)

### 変更
- src/services/payment-service.ts
  - Before: 単一クラスで全決済処理
  - After: Strategyパターンで各プロバイダーに委譲

### 影響を受けたファイル
- src/api/payment-controller.ts - インターフェース変更なし
- tests/services/payment-service.test.ts - テストケース追加

## 使用したツール
- insert_before_symbol - 新規インターフェース追加
- replace_symbol_body - PaymentService の実装変更
- getDiagnostics - 検証

## 変更前のコード
```typescript
class PaymentService {
  async processPayment(provider: string, amount: number) {
    if (provider === 'stripe') {
      // Stripe処理
    } else if (provider === 'paypal') {
      // PayPal処理
    }
    // 複雑な分岐...
  }
}
```

## 変更後のコード
```typescript
interface PaymentStrategy {
  charge(amount: number): Promise<PaymentResult>;
}

class StripeStrategy implements PaymentStrategy {
  async charge(amount: number): Promise<PaymentResult> {
    // Stripe処理
  }
}

class PayPalStrategy implements PaymentStrategy {
  async charge(amount: number): Promise<PaymentResult> {
    // PayPal処理
  }
}

class PaymentService {
  private strategies: Map<string, PaymentStrategy> = new Map([
    ['stripe', new StripeStrategy()],
    ['paypal', new PayPalStrategy()]
  ]);

  async processPayment(provider: string, amount: number) {
    const strategy = this.strategies.get(provider);
    if (!strategy) throw new Error('Unknown provider');
    return strategy.charge(amount);
  }
}
```

## 検証結果

### 診断情報
- 変更前: エラー0件、警告3件
- 変更後: エラー0件、警告3件（変化なし）

### テスト結果
- 実行: 67件（+15件）
- 成功: 67件
- 失敗: 0件

### コード品質
- 循環的複雑度: 12 → 5 (改善)
- 保守性インデックス: 65 → 82 (改善)

## 学んだ教訓
- Strategy パターンにより新規プロバイダーの追加が容易に
- 各決済ロジックが独立し、テストが簡単に
- インターフェース変更なしでリファクタリング可能

## 今後の改善点
- Factory パターンの追加（Strategy の生成を分離）
- エラーハンドリングの統一
- リトライロジックの実装

## 参照
- Pattern: Strategy Pattern
- 関連メモリ: pattern_strategy_payment.md
```

## エンドツーエンドシナリオ

### シナリオ1: 新機能の追加準備

**タスク**: パスワードリセット機能を追加する

**完全ワークフロー**:

```
=== Phase 1: 調査 ===

ステップ1: メモリ確認
list_memories()
→ module_auth_structure.md が存在

read_memory({ filename: "module_auth_structure.md" })
→ AuthService の既存構造を確認

ステップ2: 関連機能の調査
find_symbol({
  name_path: "AuthService",
  relative_path: "src/services/auth-service.ts",
  include_body: false,
  depth: 2
})

結果:
class AuthService {
  login(email, password)
  logout(token)
  validateToken(token)
  changePassword(userId, oldPassword, newPassword)
  // パスワードリセットメソッドは未実装
}

ステップ3: 類似機能の参照
find_symbol({
  name_path: "AuthService.changePassword",
  relative_path: "src/services/auth-service.ts",
  include_body: true
})
→ パスワード変更ロジックを参考に

ステップ4: 依存関係の確認
find_referencing_symbols({
  name_path: "AuthService",
  relative_path: "src/services/auth-service.ts"
})
→ AuthController で使用されている

=== Phase 2: 設計 ===

ステップ5: 実装計画をメモリに記録
write_memory({
  filename: "feature_password_reset_plan.md",
  content: `
# パスワードリセット機能実装計画

## 必要なメソッド
1. requestPasswordReset(email: string)
   - パスワードリセットトークンを生成
   - リセット用メールを送信

2. resetPassword(token: string, newPassword: string)
   - トークンを検証
   - パスワードを更新

## 必要なモデル
- PasswordResetToken
  - token: string
  - userId: string
  - expiresAt: Date

## 影響範囲
- AuthService (新規メソッド追加)
- AuthController (新規エンドポイント追加)
- EmailService (メール送信機能追加)

## 実装順序
1. PasswordResetToken モデル作成
2. AuthService.requestPasswordReset 実装
3. AuthService.resetPassword 実装
4. AuthController にエンドポイント追加
5. テスト作成
  `
})

=== Phase 3: 実装 ===

ステップ6: メソッドを追加
insert_after_symbol({
  name_path: "AuthService.changePassword",
  relative_path: "src/services/auth-service.ts",
  body: `

  async requestPasswordReset(email: string): Promise<void> {
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      // セキュリティ: ユーザーが存在しない場合も成功を返す
      return;
    }

    const token = generateSecureToken();
    await this.resetTokenRepository.save({
      token,
      userId: user.id,
      expiresAt: new Date(Date.now() + 3600000) // 1時間有効
    });

    await this.emailService.sendPasswordResetEmail(user.email, token);
  }

  async resetPassword(token: string, newPassword: string): Promise<void> {
    const resetToken = await this.resetTokenRepository.findByToken(token);

    if (!resetToken || resetToken.expiresAt < new Date()) {
      throw new Error('Invalid or expired token');
    }

    const hashedPassword = await hashPassword(newPassword);
    await this.userRepository.updatePassword(resetToken.userId, hashedPassword);
    await this.resetTokenRepository.delete(resetToken.id);
  }`
})

ステップ7: 検証
mcp__ide__getDiagnostics({})
→ エラー: 0件

=== Phase 4: 記録 ===

ステップ8: 実装記録をメモリに保存
write_memory({
  filename: "feature_password_reset_implementation.md",
  content: `
# パスワードリセット機能実装

## 実装日
2025-01-15

## 実装内容
AuthService に2つのメソッドを追加:
- requestPasswordReset(email)
- resetPassword(token, newPassword)

## 変更ファイル
- src/services/auth-service.ts (+45行)
- src/models/password-reset-token.ts (新規)
- src/api/auth-controller.ts (+25行)

## テスト
- 実行: 82件 (+12件)
- 成功: 82件
- カバレッジ: 95%

## 完了
実装完了、デプロイ準備完了
  `
})
```

### シナリオ2: レガシーコードの理解

**タスク**: 5年前に書かれた複雑な在庫管理ロジックを理解する

**完全ワークフロー**:

```
=== Phase 1: 初期調査 ===

ステップ1: エントリポイントの特定
search_for_pattern({
  pattern: "inventory",
  relative_path: "src/"
})

結果:
- src/services/inventory-service.ts
- src/models/inventory.ts
- src/api/inventory-controller.ts

ステップ2: メイン処理の特定
get_symbols_overview({
  relative_path: "src/services/inventory-service.ts"
})

結果:
- InventoryService (class)
- updateStock (function) ← 複雑そう
- checkAvailability (function)
- reserveItems (function)

=== Phase 2: 段階的理解 ===

ステップ3: 複雑な関数の構造確認
find_symbol({
  name_path: "updateStock",
  relative_path: "src/services/inventory-service.ts",
  include_body: false
})

結果:
function updateStock(productId, quantity, type)
→ 長いメソッド（100行以上）

ステップ4: 依存関係の追跡
find_referencing_symbols({
  name_path: "updateStock",
  relative_path: "src/services/inventory-service.ts"
})

結果:
使用箇所:
- OrderService (注文時)
- ReturnService (返品時)
- AdminController (手動調整)

ステップ5: 本体の詳細確認
find_symbol({
  name_path: "updateStock",
  relative_path: "src/services/inventory-service.ts",
  include_body: true
})

→ ロジックを読解

=== Phase 3: 分析と記録 ===

ステップ6: 理解した内容をメモリに記録
write_memory({
  filename: "legacy_inventory_understanding.md",
  content: `
# 在庫管理ロジック理解

## 調査日時
2025-01-15

## 対象
src/services/inventory-service.ts の updateStock 関数

## ロジックフロー

1. トランザクション開始
2. 現在の在庫を取得（行ロック）
3. 在庫タイプに応じて処理
   - 'purchase': 在庫増加
   - 'sale': 在庫減少
   - 'adjustment': 手動調整
4. 在庫アラートチェック
   - 閾値を下回る場合、アラート送信
5. 在庫履歴に記録
6. トランザクションコミット

## 複雑さの原因
- 複数の在庫タイプ処理
- トランザクション管理
- アラート機能
- 履歴記録
→ 単一関数に多くの責務

## 発見した問題
1. 関数が長すぎる（100行以上）
2. 単一責任原則違反
3. テストが困難
4. エラーハンドリングが不明瞭

## 改善提案
1. 在庫タイプ別にStrategy パターン
2. アラート機能を別サービスに抽出
3. 履歴記録をObserver パターンで実装

## 使用箇所
- OrderService.createOrder()
- ReturnService.processReturn()
- AdminController.adjustInventory()

## 注意点
- 本番環境で稼働中のため、慎重に変更
- 大量のトランザクションが発生
- パフォーマンスクリティカル
  `
})

=== Phase 4: 設計パターンの識別 ===

ステップ7: 関連する設計パターンを記録
write_memory({
  filename: "pattern_opportunities_inventory.md",
  content: `
# 在庫管理での設計パターン適用機会

## 現状
単一の大きな関数で全てを処理

## 適用可能なパターン

### 1. Strategy Pattern（在庫タイプ処理）
interface InventoryOperation {
  execute(productId, quantity): Promise<void>
}

class PurchaseOperation implements InventoryOperation
class SaleOperation implements InventoryOperation
class AdjustmentOperation implements InventoryOperation

### 2. Observer Pattern（アラート）
class InventoryObserver {
  onStockChange(event: StockChangeEvent): void
}

class AlertObserver implements InventoryObserver
class HistoryObserver implements InventoryObserver

### 3. Repository Pattern（既存）
InventoryRepository - データアクセス分離済み

## 優先度
1. Strategy Pattern - 高（複雑度削減）
2. Observer Pattern - 中（拡張性向上）

## リファクタリング計画
別途作成予定
  `
})
```

## まとめ

これらの実例から学べるポイント:

1. **段階的アプローチ**: 概要から詳細へ
2. **トークン効率**: 必要な情報のみ取得
3. **メモリ活用**: 知見を記録し再利用
4. **安全性**: 事前確認と検証
5. **記録**: 変更履歴を残す

効果的なcode-intelligence スキルの活用により、コードベースの理解とリファクタリングを安全かつ効率的に実施できます。

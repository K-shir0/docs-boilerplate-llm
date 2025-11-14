# Serenaメモリ活用詳細ガイド

このドキュメントは、code-intelligenceスキルにおけるSerenaメモリの詳細な活用方法を説明します。

## メモリファイル命名規則

### 基本形式

メモリファイル名は以下の形式で作成します:

* `architecture_overview.md`: プロジェクト全体構造
* `module_{name}_structure.md`: 各モジュールのシンボル構成
* `component_{name}_dependencies.md`: コンポーネント間依存関係
* `refactoring_{feature}_{date}.md`: リファクタリング履歴
* `pattern_{pattern_name}.md`: 発見された設計パターン
* `symbol_{name}_references.md`: 重要シンボルの参照関係

日付形式: YYYYMMDD（例: `refactoring_auth_20250108.md`）

### 命名例

プロジェクトタイプ別の具体例:

**Webアプリケーション**
* `module_auth_structure.md` - 認証モジュールの構造
* `module_api_structure.md` - APIモジュールの構造
* `component_user_service_dependencies.md` - ユーザーサービスの依存関係
* `pattern_singleton_database_connection.md` - データベース接続のSingletonパターン

**ライブラリプロジェクト**
* `architecture_overview.md` - ライブラリ全体のアーキテクチャ
* `module_core_structure.md` - コアモジュールの構造
* `symbol_parseJson_references.md` - parseJson関数の参照関係

**マイクロサービス**
* `component_order_service_dependencies.md` - 注文サービスの依存関係
* `component_payment_service_dependencies.md` - 決済サービスの依存関係
* `refactoring_api_gateway_20250115.md` - APIゲートウェイのリファクタリング履歴

## 保存タイミング詳細

### シンボル探索完了時

**タイミング**:
* `find_symbol` で重要なシンボル構造を発見した時
* `get_symbols_overview` でモジュール構造を理解した時

**保存すべき情報**:
* 発見したシンボルの一覧（クラス、関数、変数など）
* シンボルの階層構造
* エクスポートされているシンボル
* 内部専用のシンボル

**例**:
```markdown
調査対象: src/utils/validation.ts
発見したシンボル:
- validateEmail (function) - exported
- validatePassword (function) - exported
- ValidationError (class) - exported
- isValidFormat (function) - internal helper
```

### 参照追跡完了時

**タイミング**:
* `find_referencing_symbols` で依存関係を把握した時
* モジュール間の関連性が明らかになった時

**保存すべき情報**:
* 参照元のファイルとシンボル
* 参照のタイプ（import, call, extend など）
* 依存関係の方向性
* 循環依存の有無

**例**:
```markdown
AuthServiceクラスの参照:
- src/api/auth-controller.ts (import, call)
- src/middleware/auth-middleware.ts (import, call)
- src/services/user-service.ts (import, call)
依存: AuthService → DatabaseService → ConfigService
循環依存: なし
```

### リファクタリング実施後

**タイミング**:
* 変更が完了し、診断情報を確認した後
* 成功・失敗に関わらず記録

**保存すべき情報**:
* 変更内容の概要
* 変更理由（なぜこのリファクタリングを実施したか）
* 影響範囲（変更したファイルとシンボル）
* 検証結果（診断情報、テスト結果）
* 学んだ教訓や注意点

**例**:
```markdown
リファクタリング: getUserData → fetchUserData
理由: 命名規則の統一（データ取得は fetch プレフィックス）
影響範囲:
- src/services/user-service.ts:45 (定義)
- src/api/user-controller.ts:23 (呼び出し)
- src/api/admin-controller.ts:67 (呼び出し)
- 合計3ファイル、4箇所を変更
検証結果:
- 診断情報: エラー0件、警告0件
- 既存テスト: 全て通過
教訓: rename_symbol は全参照を自動更新するため安全
```

### 設計パターン発見時

**タイミング**:
* Factory, Singleton, Strategy などのパターンを識別した時
* アーキテクチャ上の重要な設計判断を発見した時

**保存すべき情報**:
* パターン名と説明
* 実装箇所（ファイルとシンボル）
* パターンの役割と利点
* 使用例

**例**:
```markdown
パターン: Singleton (AuthService)
実装: src/auth/auth-service.ts
説明: AuthServiceクラスはアプリケーション全体で1つのインスタンスのみ
利点: 認証状態の一元管理、リソース効率
使用例:
- getInstance() でインスタンス取得
- private constructor で直接インスタンス化を防止
```

### 初回コードベース探索完了時

**タイミング**:
* プロジェクト全体の構造を初めて理解した時
* 主要なディレクトリ構成とモジュール分割を把握した時

**保存すべき情報**:
* ディレクトリ構造
* 主要モジュールの役割
* エントリポイント
* 重要なファイルと設定

**例**:
```markdown
プロジェクト構造: E-Commerceアプリケーション
エントリポイント: src/index.ts
主要モジュール:
- src/api/ - REST APIエンドポイント
- src/services/ - ビジネスロジック
- src/models/ - データモデル
- src/middleware/ - Express middleware
- src/utils/ - ユーティリティ関数
依存関係フロー: API → Services → Models → Database
```

## メモリ内容の構成

### 推奨フォーマット

メモリファイルには以下の情報を含めます:

* 調査日時
* 調査対象（ファイルパス、シンボル名）
* 発見した構造と関係性
* 重要な設計判断や実装パターン
* 今後の作業で参照すべきポイント

### テンプレート: モジュール構造

```markdown
# モジュール {module_name} の構造

## 調査日時
YYYY-MM-DD

## 対象ファイル
* path/to/file1.ts
* path/to/file2.ts
* path/to/file3.ts

## シンボル構成
* {ClassName} クラス (class)
  * {methodName}() メソッド
  * {propertyName} プロパティ
* {functionName} 関数 (function)
* {constantName} 定数 (const)

## 依存関係
* {moduleA} モジュールに依存
* {moduleB} モジュールから設定を読み込み
* {moduleC} モジュールを使用

## 設計パターン
* {PatternName} パターン ({使用箇所})
* {PatternName2} パターン ({使用箇所})

## 参照ポイント
* {重要な情報1}
* {重要な情報2}
```

### テンプレート: コンポーネント依存関係

```markdown
# コンポーネント {component_name} の依存関係

## 調査日時
YYYY-MM-DD

## 対象コンポーネント
path/to/component.ts

## 直接依存
* {dependency1} - {理由}
* {dependency2} - {理由}

## 間接依存
* {dependency3} (via {dependency1})
* {dependency4} (via {dependency2})

## 被依存（このコンポーネントを使用している箇所）
* {consumer1} - {使用方法}
* {consumer2} - {使用方法}

## 依存グラフ
{consumer1} → {component_name} → {dependency1}
{consumer2} → {component_name} → {dependency2}

## 注意点
* {重要な注意事項}
```

### テンプレート: リファクタリング履歴

```markdown
# リファクタリング: {feature_name}

## 日時
YYYY-MM-DD

## 変更内容
{変更の概要を2-3文で}

## 変更理由
{なぜこのリファクタリングを実施したか}

## 影響範囲
### 変更したファイル
* {file1}:{line} - {変更内容}
* {file2}:{line} - {変更内容}

### 影響を受けたシンボル
* {symbol1} - {変更内容}
* {symbol2} - {変更内容}

## 使用したツール
* {tool1} - {目的}
* {tool2} - {目的}

## 検証結果
### 診断情報
* エラー: {件数}
* 警告: {件数}

### テスト結果
* 実行: {件数}
* 成功: {件数}
* 失敗: {件数}

## 学んだ教訓
* {教訓1}
* {教訓2}

## 今後の参照ポイント
* {重要な情報}
```

## ベストプラクティス

### メモリの整理

**定期的なレビュー**:
* 古いメモリファイルを確認
* 最新の情報に更新
* 不要になったメモリを削除

**カテゴリ別の整理**:
* モジュール構造メモリをまとめる
* リファクタリング履歴を時系列で整理
* 設計パターンをパターン種別で分類

### メモリの更新

**更新すべきタイミング**:
* コードベースが大きく変更された時
* モジュール構造が変わった時
* リファクタリングで依存関係が変化した時

**更新方法**:
* 既存メモリファイルを読み取る
* 変更箇所を特定
* 新しい情報で上書き
* 更新日時を記録

### メモリの検索

**効果的な検索戦略**:
* `list_memories` で利用可能なメモリを一覧
* ファイル名から関連するメモリを推測
* タスクに関連するキーワードで絞り込み

**検索例**:
```
タスク: "認証機能のリファクタリング"
検索キーワード: "auth", "authentication", "refactoring"
候補メモリ:
- module_auth_structure.md
- refactoring_auth_20250108.md
- component_auth_service_dependencies.md
```

## よくある質問

### Q: メモリファイル名が長すぎる場合は？

A: 略語を使用するか、最も重要な情報のみを含めます。

例:
* `component_user_authentication_service_dependencies.md`
  → `component_user_auth_deps.md`

### Q: 複数モジュールにまたがる情報はどうする？

A: 最も関連性の高いモジュール名を使用するか、`cross_module_` プレフィックスを使います。

例:
* `cross_module_auth_payment_integration.md`

### Q: 一時的な調査結果も保存すべき？

A: 将来参照する可能性がある場合は保存します。一時的な調査でも、後で役立つことがあります。

### Q: メモリファイルが多くなりすぎたら？

A: 関連するメモリをまとめて統合するか、プロジェクトの主要な変更時に古いメモリをアーカイブします。

## まとめ

Serenaメモリを効果的に活用することで:
* 同じコードの再調査を回避
* プロジェクト知識の蓄積による精度向上
* チーム全体での知見共有
* トークン使用量の大幅削減

メモリは **必ず** 保存し、継続的に活用してください。

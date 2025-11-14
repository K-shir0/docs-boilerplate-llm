---
name: code-intelligence
description: "Provides semantic code search, LSP-based code understanding, and safe refactoring support. Automatically activates for: (1) Code navigation requests, (2) Refactoring tasks, (3) Code review and understanding tasks."
version: 1.0.0
category: development
keywords: [code-search, refactoring, LSP, semantic-search, code-intelligence]
---

# Code Intelligence

## 概要

このスキルは Serena MCP と IDE MCP を活用した包括的なコードインテリジェンス機能を提供します。プログラミング言語に依存しない汎用的なアプローチで、セマンティック検索、LSP ベースの正確なコード理解、安全なリファクタリング支援を実現します。

主要機能:

* セマンティック検索: プロジェクト全体を横断した関連コードの探索
* 定義ジャンプと参照追跡: LSP ベースの正確なシンボル解析
* 安全なリファクタリング: 段階的変更の提案と実施
* 知識の永続化: Serena メモリ機能による調査結果の蓄積

サポート言語: TypeScript, JavaScript, Python, Go, Java など LSP 対応言語全般

## ワークフロー判断

### 自動起動の判断基準

このスキルは以下のユーザー要求時に自動的に起動します:

* コードナビゲーション要求
  * 「この関数の定義を探して」
  * 「この変数がどこで使われているか調べて」
  * 「このクラスの実装を確認して」
  * 「モジュール間の依存関係を教えて」
* リファクタリング要求
  * 「関数を抽出して」
  * 「このクラスを別ファイルに移動して」
  * 「変数名を変更して」
  * 「コードを整理して」
* コードレビューと理解
  * 「このコードベースの構造を教えて」
  * 「この機能の実装を調査して」
  * 「設計パターンを確認して」
  * 「アーキテクチャを理解したい」

### ワークフロー開始時の必須動作

1. SKILL.md 全体読み取り
   * このスキル自身の SKILL.md ファイルを必ず最初に読み取る
   * 他の関連スキルの SKILL.md も必要に応じて読み取る
   * ガイドラインに従ってツール選択と実行順序を決定
2. Serena メモリの確認
   * `list_memories` で利用可能なメモリを確認
   * タスクに関連するメモリがあれば `read_memory` で読み取る
   * 既存知識を活用してトークン使用量を削減

詳細な初期化手順と関連スキルの読み取りルールは、現在のSKILL.mdの内容に従ってください。

## SKILL.md 読み取りルール

### 必須読み取りタイミング

* スキル起動時: 必ず最初にこのスキルの SKILL.md 全体を読み取る
* 他スキル連携時: 連携する他スキルの SKILL.md も読み取る
* ガイドライン確認時: 実装方針や判断基準が不明な時に再読み取り

### 読み取り対象

* 現在起動中のスキルの SKILL.md: **必須**
* タスクに関連する他スキルの SKILL.md: 必要に応じて
  * 例: Markdown ドキュメント作成を伴う場合は markdown-skills の SKILL.md

### 活用方法

SKILL.md から以下の情報を取得して活用します:

* ツール選択基準: どのツールをどの順序で使用するか
* 実装方針: セマンティック検索やリファクタリングの具体的手順
* 制約事項: 禁止事項や注意点（ファイル全体読み込み禁止など）
* ベストプラクティス: トークン効率や安全性の確保方法

## Serena メモリ活用の基本

### 基本原則

Serena ツールで調べた情報は **必ず** `write_memory` で保存します。これにより:

* 同じコードの再調査を回避
* プロジェクト知識の蓄積による精度向上
* チーム全体での知見共有
* トークン使用量の大幅削減

### メモリファイル命名規則

メモリファイル名は以下の形式で作成します:

* `architecture_overview.md`: プロジェクト全体構造
* `module_{name}_structure.md`: 各モジュールのシンボル構成
* `component_{name}_dependencies.md`: コンポーネント間依存関係
* `refactoring_{feature}_{date}.md`: リファクタリング履歴
* `pattern_{pattern_name}.md`: 発見された設計パターン
* `symbol_{name}_references.md`: 重要シンボルの参照関係

日付形式: YYYYMMDD（例: `refactoring_auth_20250108.md`）

### 保存タイミング

以下のタイミングで必ず memorize します:

* シンボル探索完了時: `find_symbol` で重要なシンボル構造を発見した時
* 参照追跡完了時: `find_referencing_symbols` で依存関係を把握した時
* リファクタリング実施後: 変更内容と理由を記録
* 設計パターン発見時: Factory, Singleton, Strategy などのパターンを識別した時
* 初回コードベース探索完了時: プロジェクト全体の構造を初めて理解した時

### 詳細ガイド

メモリファイルの詳細な構成、テンプレート、ベストプラクティスは `resources/MEMORY-GUIDE.md` を参照してください。

## セマンティック検索の基本

### 基本戦略

コードを理解する際は **段階的に情報を取得** し、トークン使用量を最小化します。

重要原則:

* ファイル全体の読み込みは **絶対に禁止**
* 必ずシンボル単位で読み取る
* `include_body=false` から始め、必要な時のみ `include_body=true`

### 検索ワークフロー

段階的な探索アプローチ:

1. **ファイル概要の取得**: `get_symbols_overview` でトップレベルシンボルのリストを把握
2. **特定シンボルの探索**: `find_symbol` で目的のシンボルを詳細調査
   * `relative_path` で検索範囲を限定（推奨）
   * `depth` パラメータで子シンボル取得を制御
   * 初回は `include_body=false`、詳細確認時のみ `include_body=true`
3. **参照追跡**: `find_referencing_symbols` でシンボルの使用箇所を確認
   * 依存関係の把握
   * 影響範囲の見積もり

### ツール選択の判断基準

* シンボル名が明確: `find_symbol` を使用
* シンボル名が不明確: `substring_matching=true` で部分一致検索
* シンボル名が全く不明: `search_for_pattern` を補助的に使用（最終手段）
* 特定の種別のみ: `include_kinds` / `exclude_kinds` でフィルタリング

### 詳細ガイド

セマンティック検索の完全なワークフロー、パラメータ詳細、エッジケースの扱いは `resources/SEARCH.md` を参照してください。

## リファクタリングの基本

### 基本方針

リファクタリングは **安全性を最優先** し、段階的に実施します。

実行モード: **自動実行** (ユーザー設定に基づく)

### 安全性チェック

リファクタリング実施前に必ず確認:

1. **診断情報の確認**: `mcp__ide__getDiagnostics` で現在のエラーや警告を把握
2. **対象シンボルのスコープ分析**: `find_referencing_symbols` で全参照箇所を追跡し、影響範囲を見積もり

### 主要リファクタリング操作

* `rename_symbol`: シンボル名をコードベース全体で変更
* `replace_symbol_body`: シンボルの定義全体を置換
* `insert_after_symbol` / `insert_before_symbol`: シンボルの前後にコードを挿入

### 段階的変更の原則

大規模なリファクタリングは以下の手順で段階的に実施:

1. 小さな変更単位に分割
2. 各ステップで検証（診断情報、参照の整合性）
3. 問題発生時は即座に停止し、ユーザーに報告

### 詳細ガイド

リファクタリング操作の詳細手順、コードスタイルガイドライン、トラブルシューティングは `resources/REFACTORING-GUIDE.md` を参照してください。

## コード理解の基本ワークフロー

新しいコードベースを理解する際の推奨5ステップ:

### ステップ1: メモリ確認

`list_memories` で既存知識を確認し、関連するメモリがあれば `read_memory` で読み取ります。

### ステップ2: ディレクトリ構造の把握

`list_dir` でプロジェクトルートから再帰的にディレクトリ構造を確認します。

### ステップ3: エントリポイントの特定

package.json の "main" フィールドや、index.ts / main.ts などを探索してエントリポイントを特定します。

### ステップ4: モジュール構造の理解

各主要モジュールに対して:
1. `get_symbols_overview` でトップレベルシンボルを取得
2. 主要なクラス・関数を `find_symbol` で詳細調査
3. `find_referencing_symbols` で依存関係を追跡
4. `write_memory` でモジュール構造を保存

### ステップ5: アーキテクチャの可視化

発見した情報を統合し、モジュール間の依存関係、レイヤー構造、設計パターンを整理して `architecture_overview.md` に保存します。

### 詳細ガイド

プロジェクト構造の段階的探索、依存関係追跡、設計パターンの識別方法、実践ワークフロー例は `resources/WORKFLOW-GUIDE.md` を参照してください。

## クイックリファレンス

### 主要ツール一覧

| 操作 | ツール | 主要パラメータ | 使用例 |
|------|--------|--------------|--------|
| ファイル概要取得 | `get_symbols_overview` | relative_path | モジュールのトップレベルシンボル確認 |
| シンボル検索 | `find_symbol` | name_path, include_body, depth | クラス・関数の詳細調査 |
| 参照追跡 | `find_referencing_symbols` | name_path | 依存関係・影響範囲の把握 |
| パターン検索 | `search_for_pattern` | pattern, relative_path | 特定文字列の検索（補助的） |
| シンボルリネーム | `rename_symbol` | name_path, new_name | 変数・関数名の一括変更 |
| シンボル置換 | `replace_symbol_body` | name_path, body | メソッド実装の書き換え |
| コード挿入 | `insert_after_symbol` / `insert_before_symbol` | name_path, body | メソッド追加、import追加 |
| 診断情報取得 | `mcp__ide__getDiagnostics` | uri (optional) | エラー・警告の確認 |
| メモリ一覧 | `list_memories` | - | 既存知識の確認 |
| メモリ読取 | `read_memory` | filename | 過去の調査結果の参照 |
| メモリ書込 | `write_memory` | filename, content | 調査結果の保存 |

### よくあるシナリオ

**シナリオ1: 関数定義を探す**
```
1. get_symbols_overview({ relative_path: "src/services/" })
2. find_symbol({ name_path: "functionName", include_body: false })
3. find_symbol({ name_path: "functionName", include_body: true }) (必要に応じて)
```

**シナリオ2: 依存関係を調べる**
```
1. find_symbol({ name_path: "ClassName", include_body: false })
2. find_referencing_symbols({ name_path: "ClassName" })
3. write_memory({ filename: "component_ClassName_dependencies.md" })
```

**シナリオ3: 安全にリファクタリング**
```
1. mcp__ide__getDiagnostics({}) (事前確認)
2. find_referencing_symbols({ name_path: "oldName" }) (影響範囲確認)
3. rename_symbol({ name_path: "oldName", new_name: "newName" })
4. mcp__ide__getDiagnostics({}) (事後確認)
5. write_memory({ filename: "refactoring_YYYYMMDD.md" })
```

## よくある質問

### Q: いつSerenaメモリを使うべきか？

A: Serena ツールで調べた情報は **必ず** メモリに保存してください。特に以下の場合:
* シンボル探索完了時
* 参照追跡完了時
* リファクタリング実施後
* 設計パターン発見時
* 初回コードベース探索完了時

メモリを使うことで、同じコードの再調査を避け、トークン使用量を大幅に削減できます。

### Q: ファイル全体を読み込んではいけない理由は？

A: トークン効率が極めて悪くなるためです。

悪い例:
* Read tool でファイル全体（200行）を読む → 200行のトークンを消費

良い例:
* get_symbols_overview → 約10-20行
* find_symbol (include_body=false) → 約5-10行
* 必要な部分のみ include_body=true → 約10-20行

合計: 約25-50行（75-90%削減）

ファイル全体を読むのは、シンボル単位のアクセスが不可能な場合の **最終手段** です。

### Q: リファクタリングが失敗した場合は？

A: 以下の手順で対処してください:

1. エラー内容を確認: `mcp__ide__getDiagnostics({})`
2. 影響範囲を特定: どのファイル・シンボルに問題があるか
3. ユーザーに報告: 問題の詳細と推奨対応を提示
4. ロールバック検討: git revert や手動修正
5. 代替アプローチ: より小さな変更単位や別のツールの使用

失敗したリファクタリングも **必ず** メモリに記録し、今後の参考にしてください。

### Q: 複数モジュールの依存関係を理解するには？

A: 段階的に調査してください:

1. 各モジュールのエクスポートを確認: `get_symbols_overview`
2. 各エクスポートの参照を追跡: `find_referencing_symbols`
3. 依存グラフを構築: モジュール A → モジュール B の関係を整理
4. 循環依存をチェック: 依存ループがないか確認
5. メモリに保存: `component_{name}_dependencies.md`

詳細は `resources/WORKFLOW-GUIDE.md` の「依存関係の追跡」セクションを参照してください。

## 参考資料

### このスキルのリソース

* `resources/MEMORY-GUIDE.md` - Serenaメモリ活用の詳細ガイド
* `resources/SEARCH.md` - セマンティック検索の完全リファレンス
* `resources/REFACTORING-GUIDE.md` - リファクタリング操作の詳細手順
* `resources/WORKFLOW-GUIDE.md` - コード理解の実践ガイド
* `resources/EXAMPLES.md` - 実践例とユースケース

### 関連スキル

* `creating-skills` - スキル作成のベストプラクティス
* `markdown-skills` - ドキュメント作成ガイドライン

### 外部リソース

* Serena MCP Documentation
* IDE MCP Documentation
* LSP (Language Server Protocol) Specification

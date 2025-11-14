# Markdown 実例集

このドキュメントでは、実際のシナリオに基づいた Before/After の例を提供し、記述規則の適用方法を示します。

## 目次

1. [技術仕様書の例](#技術仕様書の例)
2. [API ドキュメントの例](#api-ドキュメントの例)
3. [README の例](#readme-の例)
4. [設計書の例](#設計書の例)
5. [トラブルシューティングガイドの例](#トラブルシューティングガイドの例)
6. [チュートリアルの例](#チュートリアルの例)

## 技術仕様書の例

### Before: 問題のある記述

```markdown
# システム仕様書

#### システムの概要
本システムはユーザー管理機能を提供します。
**重要**: データベースにはPostgreSQLを使用します。

#### 機能一覧
- ユーザー登録
  - メールアドレスの検証
  - パスワードの暗号化
- ログイン機能
  - セッション管理
  - リメンバーミー機能

#### 技術スタック
**フロントエンド**: React
**バックエンド**: Node.js
**データベース**: PostgreSQL

コマンドの実行:
```
npm install
npm start
```
```

**問題点:**
1. H4 見出しを使用（H2-H3 に制限すべき）
2. 強調表示（**）を過度に使用
3. ハイフン（-）でリストを記述
4. スペースでインデント
5. 英語と日本語の間にスペースなし
6. コードブロックに言語指定なし

### After: 改善された記述

```markdown
# システム仕様書

## システムの概要

本システムはユーザー管理機能を提供します。データベースには PostgreSQL を使用します。

## 機能一覧

* ユーザー登録
	* メールアドレスの検証
	* パスワードの暗号化
* ログイン機能
	* セッション管理
	* リメンバーミー機能

## 技術スタック

* フロントエンド: React
* バックエンド: Node.js
* データベース: PostgreSQL

## セットアップ

コマンドの実行:

```bash
npm install
npm start
```
```

**改善点:**
1. H2-H3 のみ使用
2. 強調表示を削除し、構造で表現
3. アスタリスク（*）でリスト記述
4. タブでインデント
5. 英語と日本語の間に半角スペース
6. bash 言語指定を追加

## API ドキュメントの例

### Before: 問題のある記述

```markdown
# API Reference

## Endpoints

##### GET /api/users
ユーザー一覧を取得します。

**Parameters:**
- page: ページ番号
- limit: 取得件数

**Response:**
```
{
  "users": [],
  "total": 0
}
```

##### POST /api/users
*新しいユーザーを作成します。*

**Required fields:**
- email
- password
```

**問題点:**
1. H5 見出しを使用
2. 強調表示と斜体を使用
3. ハイフンでリスト記述
4. JSON に言語指定なし

### After: 改善された記述

```markdown
# API Reference

## Endpoints

### GET /api/users

ユーザー一覧を取得します。

パラメータ:

* `page`: ページ番号
* `limit`: 取得件数

レスポンス:

```json
{
  "users": [],
  "total": 0
}
```

### POST /api/users

新しいユーザーを作成します。

必須フィールド:

* `email`: メールアドレス
* `password`: パスワード

リクエスト例:

```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

レスポンス:

```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2025-11-15T00:00:00Z"
}
```
```

**改善点:**
1. H3 見出しに変更
2. 強調表示と斜体を削除
3. アスタリスクでリスト記述
4. JSON に言語指定を追加
5. より詳細な例を追加

## README の例

### Before: 問題のある記述

```markdown
# MyProject

**これは素晴らしいプロジェクトです！**

## Installation
以下のコマンドでインストール:
```
git clone https://github.com/user/myproject.git
cd myproject
npm install
```

## Usage
プロジェクトを起動するには:
```
npm start
```

## Features
- *高速*な処理
- **安全**な認証
- 使いやすいUI

## License
MITライセンス
```

**問題点:**
1. 過度な強調表示
2. 感嘆符による感情的表現
3. コードブロックに言語指定なし
4. ハイフンでリスト記述
5. 英語と日本語の間にスペースなし

### After: 改善された記述

```markdown
# MyProject

効率的なユーザー管理システムを提供するプロジェクトです。

## インストール

以下のコマンドでインストール:

```bash
git clone https://github.com/user/myproject.git
cd myproject
npm install
```

## 使い方

プロジェクトを起動:

```bash
npm start
```

ブラウザで `http://localhost:3000` にアクセスしてください。

## 主な機能

* 高速な処理
	* 最適化されたクエリ
	* キャッシング機能
* 安全な認証
	* JWT トークン
	* パスワードハッシュ化
* 使いやすい UI
	* レスポンシブデザイン
	* 直感的な操作

## ライセンス

MIT License - 詳細は [LICENSE](./LICENSE) を参照してください。
```

**改善点:**
1. 強調表示を削除
2. 客観的な説明に変更
3. bash 言語指定を追加
4. アスタリスクでリスト記述
5. 英語と日本語の間に半角スペース
6. 階層構造を追加して詳細を表現

## 設計書の例

### Before: 問題のある記述

```markdown
# システム設計書

## 1. 概要
このシステムはECサイトです。

## 2. アーキテクチャ
#### 2.1 フロントエンド
Reactを使用します。
##### 2.1.1 状態管理
Reduxを使用します。
##### 2.1.2 ルーティング
React Routerを使用します。

#### 2.2 バックエンド
Node.jsとExpressを使用します。
```

**問題点:**
1. 見出しに連番を使用
2. H4、H5 の深い見出しレベル
3. 英語と日本語の間にスペースなし
4. 簡潔すぎる説明

### After: 改善された記述

```markdown
# システム設計書

## 概要

このシステムは EC サイトの構築を目的とし、以下の機能を提供します:

* 商品閲覧と検索
* ショッピングカート管理
* 決済処理
* 注文履歴管理

## アーキテクチャ

### フロントエンド

* フレームワーク: React 18.x
	* TypeScript による型安全性
	* 関数コンポーネントと Hooks を使用
* 状態管理: Redux Toolkit
	* グローバルステート管理
	* Redux Thunk による非同期処理
* ルーティング: React Router v6
	* SPA の画面遷移
	* 認証ガード実装

### バックエンド

* ランタイム: Node.js 20.x
* フレームワーク: Express 4.x
	* RESTful API 設計
	* ミドルウェアによる認証・認可
* データベース: PostgreSQL 15.x
	* リレーショナルデータモデル
	* トランザクション処理

## データベース設計

### テーブル構成

* users テーブル
	* id: プライマリキー
	* email: ユーザーメールアドレス
	* password_hash: ハッシュ化パスワード
	* created_at: 作成日時
* products テーブル
	* id: プライマリキー
	* name: 商品名
	* price: 価格
	* stock: 在庫数
* orders テーブル
	* id: プライマリキー
	* user_id: 外部キー (users.id)
	* total_amount: 合計金額
	* status: 注文状態
	* created_at: 注文日時
```

**改善点:**
1. 見出しから連番を削除
2. H3 とリストのネストで階層構造を表現
3. 英語と日本語の間に半角スペース
4. 具体的で詳細な説明を追加
5. 読みやすい構造に整理

## トラブルシューティングガイドの例

### Before: 問題のある記述

```markdown
# トラブルシューティング

### エラーが出た時

**エラーメッセージ:** "Connection refused"

**解決方法:**
- サーバーが起動しているか確認
- ポート番号が正しいか確認

**エラーメッセージ:** "Out of memory"

**解決方法:**
- メモリを増やす
```

**問題点:**
1. H3 から開始（H2 がない）
2. 強調表示を使用
3. ハイフンでリスト記述
4. 簡潔すぎる説明

### After: 改善された記述

```markdown
# トラブルシューティングガイド

## 接続エラー

### 症状

アプリケーション起動時に以下のエラーが表示される:

```
Error: Connection refused
ECONNREFUSED 127.0.0.1:5432
```

### 原因

* データベースサーバーが起動していない
* ポート番号の設定が誤っている
* ファイアウォールがブロックしている

### 解決方法

1. データベースサーバーの起動確認

	```bash
	# PostgreSQL の状態確認
	systemctl status postgresql
	```

2. ポート番号の確認

	`.env` ファイルで設定を確認:

	```
	DB_PORT=5432
	```

3. ファイアウォール設定の確認

	```bash
	# ポートが開いているか確認
	sudo ufw status
	```

## メモリエラー

### 症状

実行中に以下のエラーが発生:

```
FATAL ERROR: Ineffective mark-compacts near heap limit
Allocation failed - JavaScript heap out of memory
```

### 原因

* Node.js のデフォルトメモリ制限（約 1.5GB）を超過
* メモリリークの可能性
* 大量データの一括処理

### 解決方法

1. メモリ制限の引き上げ

	`package.json` のスクリプトを修正:

	```json
	{
		"scripts": {
			"start": "node --max-old-space-size=4096 index.js"
		}
	}
	```

2. メモリ使用状況の監視

	```javascript
	const used = process.memoryUsage();
	console.log('Memory usage:', {
		rss: `${Math.round(used.rss / 1024 / 1024)} MB`,
		heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
		heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`
	});
	```

3. ストリーム処理への変更

	大量データは分割して処理:

	```javascript
	const stream = fs.createReadStream('large-file.csv');
	stream.on('data', (chunk) => {
		// チャンク単位で処理
	});
	```
```

**改善点:**
1. H2 で主要エラー分類、H3 で詳細セクション
2. 強調表示を削除
3. アスタリスクでリスト記述
4. 具体的なコード例を追加
5. 段階的な解決手順を明示
6. 実践的な情報を提供

## チュートリアルの例

### Before: 問題のある記述

```markdown
# クイックスタート

## ステップ1
まずインストールします。
```
npm install
```

## ステップ2
次にサーバーを起動。
```
npm start
```

**これで完了！**
```

**問題点:**
1. 見出しに連番を使用
2. コードブロックに言語指定なし
3. 感嘆符による感情的表現
4. 説明が不十分

### After: 改善された記述

```markdown
# クイックスタートガイド

このガイドでは、5分でアプリケーションを起動する方法を説明します。

## 前提条件

以下がインストールされていることを確認してください:

* Node.js 20.x 以降
* npm 10.x 以降
* Git

バージョン確認:

```bash
node --version
npm --version
```

## インストール

1. リポジトリをクローン

	```bash
	git clone https://github.com/user/myproject.git
	cd myproject
	```

2. 依存関係をインストール

	```bash
	npm install
	```

	インストールには約 1-2 分かかります。

## 設定

1. 環境変数ファイルを作成

	```bash
	cp .env.example .env
	```

2. `.env` ファイルを編集し、必要な値を設定

	```
	PORT=3000
	DATABASE_URL=postgresql://localhost:5432/mydb
	```

## 起動

開発サーバーを起動:

```bash
npm start
```

正常に起動すると、以下のメッセージが表示されます:

```
Server is running on http://localhost:3000
```

ブラウザで `http://localhost:3000` にアクセスしてください。

## 次のステップ

基本的なセットアップが完了しました。次は以下を試してください:

* [認証の設定](./docs/authentication.md)
* [API の使い方](./docs/api-usage.md)
* [デプロイ方法](./docs/deployment.md)

## トラブルシューティング

問題が発生した場合は [トラブルシューティングガイド](./docs/troubleshooting.md) を参照してください。
```

**改善点:**
1. 見出しから連番を削除（内容で順序を表現）
2. bash 言語指定を追加
3. 感嘆符を削除し、客観的な記述に
4. 前提条件、期待される出力、次のステップを追加
5. 具体的で実践的な手順を提供
6. トラブルシューティングへのリンクを追加

## まとめ

これらの例から学べる重要なポイント:

1. **構造化**: 見出しレベルを H2-H3 に制限し、リストで階層を表現
2. **一貫性**: アスタリスクとタブで統一されたリスト
3. **明確性**: 言語指定付きコードブロック
4. **スペーシング**: 英語と日本語の間に半角スペース
5. **客観性**: 強調表示や感情的表現を避け、構造で表現
6. **詳細性**: 具体的な例と説明を提供

より詳細な規則は [REFERENCE.md](./REFERENCE.md) を参照してください。

---

**Version:** 1.0.0
**Last Updated:** 2025-11-15

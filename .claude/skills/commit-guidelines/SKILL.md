# Commit Guidelines Skill

このスキルは、リポジトリのコミットメッセージ規則を提供し、Conventional Commits準拠の日本語コミットメッセージ作成をサポートします。

## 目的

- 一貫性のあるコミットメッセージの作成
- Conventional Commits形式（日本語版）の準拠
- 変更履歴の可読性向上
- 自動化ツール（changelog生成など）との互換性

## 基本ルール

### コミットメッセージの形式

```
<type>[optional scope]: <日本語の説明>

[optional body]

[optional footer(s)]
```

### タイプの定義

- `feat:` - 新機能の追加
- `fix:` - バグ修正
- `docs:` - ドキュメントのみの変更
- `style:` - コードの意味に影響しない変更（空白、フォーマット、セミコロンなど）
- `refactor:` - バグ修正や機能追加を含まないコードの変更
- `perf:` - パフォーマンスを向上させるコードの変更
- `test:` - テストの追加や既存テストの修正
- `chore:` - ビルドプロセスや補助ツールの変更

### Breaking Changes

破壊的変更がある場合は、`!`を追加するか、フッターに`BREAKING CHANGE:`を記載します：

```
feat!: 新しいAPI仕様に移行
```

## Claude Codeでの使用

### 自動適用

Claude Codeがコミットを作成する際、このスキルに基づいて自動的に適切な形式でコミットメッセージを生成します。

### トリガー条件

以下の場合に自動的にこのスキルが適用されます：
- ユーザーがコミットの作成を依頼したとき
- Claude Codeがコード変更後にコミットするとき
- PRの作成時にコミット履歴を確認するとき

## 詳細情報

- 規則の詳細: [CONVENTIONS.md](./resources/CONVENTIONS.md)
- 実例とベストプラクティス: [EXAMPLES.md](./resources/EXAMPLES.md)

## リポジトリ固有のルール

このプロジェクトでは以下の追加ルールを適用します：

1. **言語**: コミットメッセージの説明文は日本語で記述
2. **タイプ**: 英語のConventional Commitsタイプを使用
3. **スコープ**: 任意だが、大規模な変更の場合は推奨
4. **詳細度**: 単一行で簡潔に、必要に応じて本文で詳細を追加

## 参考資料

- [Conventional Commits公式](https://www.conventionalcommits.org/)
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit)

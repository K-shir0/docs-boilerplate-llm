# コマンドのトラブルシューティング

コマンド作成・実行時のよくある問題と解決方法を網羅したガイドです。

## 目次

1. [コマンドが認識されない](#コマンドが認識されない)
2. [Frontmatter エラー](#frontmatter-エラー)
3. [$ARGUMENTS が展開されない](#arguments-が展開されない)
4. [Bash コマンドが実行されない](#bash-コマンドが実行されない)
5. [allowed-tools の問題](#allowed-tools-の問題)
6. [パフォーマンス問題](#パフォーマンス問題)
7. [SlashCommand Tool の問題](#slashcommand-tool-の問題)
8. [ネームスペースの問題](#ネームスペースの問題)

---

## コマンドが認識されない

### 症状

`/command-name` を実行しても「コマンドが見つかりません」というエラーが表示される。

### 原因と解決策

#### 原因1: ファイルが正しい場所にない

**確認方法:**

```bash
ls -la .claude/commands/
```

**解決策:**

コマンドファイルを `.claude/commands/` または `~/.claude/commands/` に配置してください。

```bash
# プロジェクトコマンド
mkdir -p .claude/commands
mv command-name.md .claude/commands/

# パーソナルコマンド
mkdir -p ~/.claude/commands
mv command-name.md ~/.claude/commands/
```

#### 原因2: ファイル名が正しくない

**悪い例:**
```
command_name.md    # アンダースコア
commandName.md     # キャメルケース
command name.md    # スペース
```

**良い例:**
```
command-name.md    # ケバブケース（ハイフン）
```

**解決策:**

```bash
mv command_name.md command-name.md
```

#### 原因3: 拡張子が `.md` でない

**確認方法:**

```bash
ls -la .claude/commands/ | grep -v ".md$"
```

**解決策:**

拡張子を `.md` に変更:

```bash
mv command-name.txt command-name.md
```

#### 原因4: `description` フィールドがない

**悪い例:**

```yaml
---
allowed-tools: Read, Write
---
```

**良い例:**

```yaml
---
description: コマンドの簡潔な説明
allowed-tools: Read, Write
---
```

**解決策:**

Frontmatter に `description` を追加してください。

#### 原因5: ファイルパーミッションの問題

**確認方法:**

```bash
ls -l .claude/commands/command-name.md
```

**解決策:**

読み取り権限を付与:

```bash
chmod 644 .claude/commands/command-name.md
```

---

## Frontmatter エラー

### 症状

コマンド実行時に YAML パースエラーが発生する。

### 原因と解決策

#### 原因1: YAML 構文エラー

**悪い例:**

```yaml
---
description: Text with: colon
allowed-tools: Read, Write,  # カンマの後にスペース2つ
---
```

**良い例:**

```yaml
---
description: "Text with: colon"  # コロンを含む場合はクォート
allowed-tools: Read, Write
---
```

**確認方法:**

オンライン YAML バリデーターで検証:
```bash
# Python がある場合
python3 -c "import yaml; yaml.safe_load(open('.claude/commands/command-name.md').read().split('---')[1])"
```

#### 原因2: インデントの問題

**悪い例:**

```yaml
---
  description: Indented  # インデント不要
allowed-tools: Read
---
```

**良い例:**

```yaml
---
description: Not indented
allowed-tools: Read
---
```

#### 原因3: クォートのエスケープ漏れ

**悪い例:**

```yaml
---
description: Text with "quotes"
---
```

**良い例:**

```yaml
---
description: "Text with \"quotes\""
# または
description: 'Text with "quotes"'
---
```

#### 原因4: 複数行の扱い

**悪い例:**

```yaml
---
description: This is a very long description
that spans multiple lines
---
```

**良い例:**

```yaml
---
description: "This is a very long description that spans multiple lines"
# または
description: >
  This is a very long description
  that spans multiple lines
---
```

---

## $ARGUMENTS が展開されない

### 症状

`$ARGUMENTS` がそのまま表示され、引数が埋め込まれない。

### 原因と解決策

#### 原因1: 引数を指定していない

**問題:**

```
/command-name
```

**解決策:**

引数を指定:

```
/command-name some arguments here
```

#### 原因2: 間違ったプレースホルダー構文

**悪い例:**

```markdown
${ARGUMENTS}
$ARGS
%(ARGUMENTS)
```

**良い例:**

```markdown
$ARGUMENTS
```

#### 原因3: disable-model-invocation が false

`disable-model-invocation: true` の場合、単純な文字列置換が行われます。`false`（デフォルト）の場合、LLM がプロンプトを解釈します。

**テスト方法:**

```markdown
---
description: Test argument expansion
disable-model-invocation: true
---

引数: $ARGUMENTS
```

これで引数が正しく展開されるか確認してください。

---

## Bash コマンドが実行されない

### 症状

`!`command`` が展開されず、そのまま表示される。

### 原因と解決策

#### 原因1: バッククォートが正しくない

**悪い例:**

```markdown
!'git status'   # シングルクォート
!`git status'   # 閉じクォートが異なる
!'git status`   # 開始クォートが異なる
```

**良い例:**

```markdown
!`git status`   # バッククォート（`）
```

#### 原因2: allowed-tools に Bash がない

**確認:**

Frontmatter に `allowed-tools` があり、`Bash` が含まれているか確認:

```yaml
---
allowed-tools: Read, Write  # Bash がない！
---
```

**解決策:**

```yaml
---
allowed-tools: Bash(git status:*), Read, Write
---
```

#### 原因3: コマンドのパーミッション

コマンド自体が実行権限を持っていない場合。

**確認方法:**

```bash
which git
ls -l /usr/bin/git
```

**解決策:**

Claude Code は通常のシェルとして実行されるため、ユーザーが実行できるコマンドは実行可能です。もしコマンドが見つからない場合、PATH に含まれているか確認してください。

#### 原因4: コマンドが存在しない

**エラーメッセージ:**

```
bash: command-name: command not found
```

**解決策:**

コマンドをインストールするか、正しいパスを指定:

```markdown
# 悪い例
!`nonexistent-command`

# 良い例（フルパス）
!`/usr/local/bin/my-command`

# または環境確認
!`which my-command`
```

---

## allowed-tools の問題

### 症状

コマンド実行時に「Tool X is not allowed」というエラーが表示される。

### 原因と解決策

#### 原因1: ツールが allowed-tools に含まれていない

**エラー例:**

```
Error: Tool 'Write' is not allowed
```

**解決策:**

Frontmatter に該当ツールを追加:

```yaml
---
allowed-tools: Read, Write  # Write を追加
---
```

#### 原因2: パターンマッチの誤り

**悪い例:**

```yaml
allowed-tools: Bash(git commit)  # :* がない
```

**良い例:**

```yaml
allowed-tools: Bash(git commit:*)
```

#### 原因3: ワイルドカードの誤用

**動作しない例:**

```yaml
allowed-tools: Bash(git*)  # git と * の間にスペースが必要
```

**正しい例:**

```yaml
allowed-tools: Bash(git *)
```

#### 原因4: カンマ区切りの問題

**悪い例:**

```yaml
allowed-tools: Read Write  # カンマがない
allowed-tools: Read,Write   # スペースがない（推奨しない）
```

**良い例:**

```yaml
allowed-tools: Read, Write
```

---

## パフォーマンス問題

### 症状

コマンドの実行が遅い、タイムアウトする。

### 原因と解決策

#### 原因1: 大量の Bash コマンド出力

**問題:**

```markdown
!`git log --all`  # 何千行も出力される可能性
```

**解決策:**

出力を制限:

```markdown
!`git log --all --oneline -20`  # 最新20件のみ
!`git log --all --oneline | head -n 20`  # 最新20件のみ
```

#### 原因2: 不適切なモデル選択

**問題:**

シンプルなテンプレート展開に Opus を使用している。

**解決策:**

```yaml
# シンプルなタスク
model: claude-haiku-*

# または LLM 不要
disable-model-invocation: true
```

#### 原因3: 冗長なプロンプト

**問題:**

プロンプトが何千行もある。

**解決策:**

* 必要最小限の情報に絞る
* 詳細情報は外部ファイル参照
* スキルを活用

**悪い例:**

```markdown
[コーディング規約全文を貼り付け]
[テストガイドライン全文を貼り付け]
[API仕様全文を貼り付け]
```

**良い例:**

```markdown
以下のスキルに従ってください:
* coding-standards
* testing-guidelines
* api-specifications
```

---

## SlashCommand Tool の問題

### 症状

SlashCommand tool でコマンドを実行できない。

### 原因と解決策

#### 原因1: description がない

**解決策:**

```yaml
---
description: コマンドの説明を追加
---
```

#### 原因2: ビルトインコマンドを実行しようとしている

SlashCommand tool はカスタムコマンドのみサポートします。

**動作しない:**

```
/compact
/init
/help
```

**動作する:**

```
/commit
/pr
/my-custom-command
```

#### 原因3: SlashCommand tool が allowed-tools にない

**解決策:**

```yaml
---
allowed-tools: SlashCommand, Bash(*)
---
```

---

## ネームスペースの問題

### 症状

ネームスペース付きコマンドが認識されない。

### 原因と解決策

#### 原因1: 呼び出し構文が正しくない

**悪い例:**

```
/git/commit     # スラッシュ
/git-commit     # ハイフン
/git.commit     # ドット
```

**良い例:**

```
/git:commit     # コロン
```

#### 原因2: ディレクトリ構造が正しくない

**悪い例:**

```
.claude/commands/git-commit.md  # フラット
.claude/commands/git_commit.md  # アンダースコア
```

**良い例:**

```
.claude/commands/git/commit.md  # ディレクトリ
```

#### 原因3: 深すぎるネスト

**動作しない（現在サポートされていない）:**

```
.claude/commands/git/branch/create.md  # 2階層
```

**動作する:**

```
.claude/commands/git/branch-create.md  # 1階層
```

---

## デバッグ方法

### 基本的なデバッグ手順

#### 1. ファイル構造の確認

```bash
tree .claude/commands/
# または
find .claude/commands/ -name "*.md"
```

#### 2. Frontmatter の検証

```bash
head -n 10 .claude/commands/command-name.md
```

YAML が正しいか確認:

```bash
# Python がある場合
python3 << 'EOF'
import yaml
with open('.claude/commands/command-name.md') as f:
    content = f.read()
    parts = content.split('---')
    if len(parts) >= 3:
        try:
            yaml.safe_load(parts[1])
            print("✅ YAML is valid")
        except Exception as e:
            print(f"❌ YAML error: {e}")
    else:
        print("❌ No frontmatter found")
EOF
```

#### 3. 最小限のコマンドでテスト

シンプルなテストコマンドを作成:

```markdown
---
description: Test command
disable-model-invocation: true
---

# Test

Command works!
Arguments: $ARGUMENTS
```

これが動作すれば、Claude Code の設定は正しいです。

#### 4. Bash コマンドのテスト

```markdown
---
description: Test bash
allowed-tools: Bash(echo:*)
---

# Bash Test

!`echo "Hello from bash"`
```

#### 5. 段階的な機能追加

最小限のコマンドから始めて、徐々に機能を追加:

1. テンプレート展開のみ
2. $ARGUMENTS 追加
3. !`bash` 追加
4. allowed-tools 追加
5. 複雑なロジック追加

各ステップで動作確認することで、問題箇所を特定しやすくなります。

---

## よくある質問

### Q: コマンドを削除したのにまだ表示される

**A:** キャッシュの問題かもしれません。以下を試してください:

1. Claude Code を再起動
2. ファイルが本当に削除されているか確認: `ls .claude/commands/`
3. パーソナルコマンドもチェック: `ls ~/.claude/commands/`

### Q: 同じ名前のコマンドが2つある場合

**A:** プロジェクトコマンド（`.claude/commands/`）がパーソナルコマンド（`~/.claude/commands/`）より優先されます。

競合を避けるため、異なる名前を使用してください。

### Q: コマンドの更新が反映されない

**A:**

1. ファイルを保存したか確認
2. Claude Code を再起動
3. コマンドを再実行

### Q: エラーメッセージが表示されない

**A:** コマンド内でエラーハンドリングを追加してください:

```markdown
## エラーチェック

!`git status`

上記コマンドが失敗した場合:
→ エラーメッセージを表示
→ 処理を中止
```

---

## まとめ

トラブルシューティングの基本ステップ:

1. **ファイル配置を確認**: `.claude/commands/` に `.md` ファイルがあるか
2. **Frontmatter を検証**: YAML 構文が正しいか、`description` があるか
3. **最小限のコマンドでテスト**: シンプルなコマンドが動作するか確認
4. **段階的に機能追加**: 一度に全部ではなく、徐々に機能を追加
5. **エラーメッセージを確認**: 具体的なエラー内容を読む

それでも解決しない場合:
* [Claude Code ドキュメント](https://docs.claude.com/en/docs/claude-code)を参照
* GitHub Issue で報告: https://github.com/anthropics/claude-code/issues

---

バージョン: 1.0.0
最終更新: 2025-01-15

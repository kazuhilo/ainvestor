---
description: docs/adr/accepted/ の ADR を廃止して deprecated/ へ移動する（代替なし。あるなら `/adr:supersede`）。
model: sonnet
---

# adr:deprecate

古くなった・不要になった・前提が崩れた等で **代替なしで** 使用をやめる場合に使う。
**別の ADR で置き換える場合は `/adr:supersede`** を使うこと（混同しない）。

## ライフサイクル

本コマンドは `Accepted → Deprecated` の遷移のみを扱う。全体仕様は [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) を参照。Deprecated は終端状態。

## deprecate と supersede の使い分け

| 状況 | 使うコマンド |
|---|---|
| 代替 ADR は **無い**。単に使わなくなった | `/adr:deprecate`（本コマンド） |
| 別の ADR が決定を **置き換える** | `/adr:supersede` |

判断に迷ったら、ヒアリングの中で「代替 ADR は既にありますか？」を確認し、ある場合は `/adr:supersede` の使用を促す。

## 実行手順

### 1. ADR ディレクトリ解決

`git rev-parse --show-toplevel` で `<ROOT>` を取得。
失敗時はユーザーに ADR を含むプロジェクトルートの絶対パスを問い合わせる。
ADR ルート: `<ROOT>/docs/adr/`

### 2. 対象ファイル特定

**accepted フォルダ** から特定する。

#### (A) `$ARGUMENTS` が空
`<ROOT>/docs/adr/accepted/*.md` を Glob で列挙:
- 0 件 → 「accepted の ADR がありません」と報告して終了
- 1 件 → そのまま採用
- 2〜4 件 → **AskUserQuestion** で選択肢提示
- 5 件以上 → 一覧をテキストで提示し、ファイル名（または部分一致キーワード）を入力するよう依頼

#### (B) `$ARGUMENTS` あり
1. `<ROOT>/docs/adr/accepted/$ARGUMENTS` がそのまま存在すれば採用
2. 拡張子 `.md` を補って `$ARGUMENTS.md` でも探す
3. 上記で見つからなければ Glob `<ROOT>/docs/adr/accepted/*$ARGUMENTS*.md` で部分一致検索
   - 0 件: 「該当する ADR が見つかりません。accepted 一覧は以下です:」と一覧提示、再入力依頼
   - 1 件: 採用
   - 2〜4 件: **AskUserQuestion** で選択
   - 5 件以上: 一覧テキスト提示、再入力依頼

**accepted 以外のフォルダのファイルは対象外**（lifecycle.md の禁止遷移）。検出したら「この ADR は <現状態> です。deprecate できるのは Accepted のみ」と報告して終了。

### 3. 廃止理由ヒアリング

ユーザーに素のテキスト質問:

> 「{subject}」を廃止する理由を教えてください。代替となる別の ADR があれば、その時点で `adr:supersede` の使用を提案します。

回答に「代替 ADR」「後継」「置き換え」等の語が含まれていたら:

> 代替 ADR がある場合は `/adr:deprecate` ではなく `/adr:supersede` を使うのが正しい遷移です。本当に deprecate（代替なし廃止）で進めますか？

の確認質問を AskUserQuestion で実施。「supersede へ切り替え」を選んだら本コマンドは終了し、`/adr:supersede` を案内する。

#### 短すぎる場合
回答が 10 文字未満の場合は **1 回だけ** 追加質問:

> もう少し詳しく教えてください。なぜ不要になったのか、いつから適用しないのか等。

それでも短い場合はそのまま採用（強制しない）。

### 4. ファイル内容更新

特定ファイルを **Read** で読み、**Edit** で以下を更新:

#### 4-1. Status 書き換え
```
- Status: Accepted
```
→
```
- Status: Deprecated
```

既に Deprecated の場合は変更不要（冪等）。Accepted 以外（Proposed/Rejected/Superseded）からの遷移は禁止。

#### 4-2. Decision-Date 更新
`- Status: Deprecated` の **直下** の `- Decision-Date:` 行の値を **今日の YYYY-MM-DD に上書き**する。
無ければ Status 行直下に追加する。

#### 4-3. Deprecation Reason セクション追記
ファイル末尾に以下を追記:

```markdown

## Deprecation Reason

{ヒアリングで得た理由}
```

既に `## Deprecation Reason` セクションがある場合は **本文を上書き**（重複セクションを作らない）。

### 5. ファイル移動

```
<ROOT>/docs/adr/accepted/<filename>
  → <ROOT>/docs/adr/deprecated/<filename>
```

git 追跡判定: `git ls-files --error-unmatch <path>` の exit code が 0 なら `git mv`、それ以外は通常の `mv`。

### 6. 完了報告

3〜5 行で以下を伝える:
- 廃止した ADR の subject（タイトル行から抽出）
- 移動先パス（`docs/adr/deprecated/<filename>`）
- 廃止理由のサマリ（1 文）

CLAUDE.md の規約に従い自動コミットはしない。

## やってはいけないこと

- 代替 ADR があるのに adr:deprecate を使う（必ず adr:supersede に誘導）
- 廃止理由を聞かずに（または推測で）勝手に追記する
- accepted 以外のフォルダのファイルを対象にする
- Deprecation Reason セクションを重複追加する
- Decision-Date を上書きせず複数行を追加する
- 自動的に git add / git commit する

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様
- [`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) — ADR 雛形（参照のみ）
- `adr:new` — ADR の新規作成
- `adr:accept` — proposed → accepted への遷移
- `adr:supersede` — accepted → superseded への遷移（代替 ADR を指定）

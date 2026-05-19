---
description: docs/adr/accepted/ の ADR を別の ADR で置き換え、旧側を superseded/ へ移動する。
model: sonnet
---

# adr:supersede

**旧 ADR** を別の **新 ADR**（accepted）で置き換え、旧 ADR を `docs/adr/superseded/` へ移動して旧↔新の相互リンクを張る。

代替 ADR が **存在しない** 場合は `/adr:deprecate` を使う。

## ライフサイクル

本コマンドは `Accepted → Superseded` の遷移を扱い、新 ADR 側は Accepted のまま変更しない（本文末尾に `## Supersedes` セクションだけ追記）。全体仕様は [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) を参照。Superseded は終端状態。

## 実行手順

### 1. ADR ディレクトリ解決

`git rev-parse --show-toplevel` で `<ROOT>` を取得。
失敗時はユーザーに ADR を含むプロジェクトルートの絶対パスを問い合わせる。
ADR ルート: `<ROOT>/docs/adr/`

### 2. 旧 ADR の特定

**accepted フォルダ** から特定する。

#### (A) `$ARGUMENTS` が空
`<ROOT>/docs/adr/accepted/*.md` を Glob で列挙:
- 0 件 → 「accepted の ADR がありません」と報告して終了
- 1 件 → 候補 1 つしかないが、確認のため AskUserQuestion で「この ADR で間違いないか」確認
- 2〜4 件 → **AskUserQuestion** で「supersede する **旧** ADR を選択してください」と提示
- 5 件以上 → 一覧をテキストで提示し、ファイル名（または部分一致キーワード）を入力するよう依頼

#### (B) `$ARGUMENTS` あり
1. `<ROOT>/docs/adr/accepted/$ARGUMENTS` がそのまま存在すれば採用
2. 拡張子 `.md` を補って `$ARGUMENTS.md` でも探す
3. 上記で見つからなければ Glob `<ROOT>/docs/adr/accepted/*$ARGUMENTS*.md` で部分一致検索（上記と同様の 0/1/複数件処理）

**accepted 以外のフォルダのファイルは対象外**（lifecycle.md の禁止遷移）。検出したら「この ADR は <現状態> です。supersede できるのは Accepted のみ」と報告して終了。

特定した旧 ADR ファイル名を `<old-filename>`、タイトル行から抽出した subject を `<old-title>` とする。

### 3. 新 ADR の特定

ユーザーに素のテキスト質問:

> 「{old-title}」を置き換える **新しい ADR** のファイル名（または部分一致キーワード）を教えてください。

新 ADR は `docs/adr/accepted/` から探す（**accepted のみ**）。

#### 検索手順
1. `<ROOT>/docs/adr/accepted/<入力値>` がそのまま存在すれば採用
2. 拡張子 `.md` を補って試す
3. Glob `<ROOT>/docs/adr/accepted/*<入力値>*.md` で部分一致検索
   - 0 件: 以下を順に試す:
     - `<ROOT>/docs/adr/proposed/*<入力値>*.md` で proposed に存在するかチェック。存在すれば「新 ADR はまだ proposed です。先に `/adr:accept` で採用してください」と案内して終了
     - 該当なしなら「該当する新 ADR が見つかりません。accepted 一覧は以下です:」と提示、再入力依頼
   - 1 件: 採用
   - 2〜4 件: **AskUserQuestion** で選択
   - 5 件以上: 一覧テキスト提示、再入力依頼

#### 旧 ADR と同一の防止
特定した新 ADR ファイル名が `<old-filename>` と同じなら「同じ ADR は指定できません」と報告して再入力依頼。

特定した新 ADR を `<new-filename>`、subject を `<new-title>` とする。

### 4. 旧 ADR の更新

旧 ADR ファイル（`<ROOT>/docs/adr/accepted/<old-filename>`）を **Read** で読み、**Edit** で以下を更新:

#### 4-1. Status 書き換え
```
- Status: Accepted
```
→
```
- Status: Superseded by [<new-filename>](../accepted/<new-filename>)
```

既に Superseded の場合（重複適用）は変更不要だが、後続リンク先が変わるなら値を更新する。

#### 4-2. Decision-Date 更新
`- Status:` 行の **直下** の `- Decision-Date:` 行の値を **今日の YYYY-MM-DD に上書き**する。
無ければ Status 行直下に追加する。

#### 4-3. Superseded By セクション追記
ファイル末尾に以下を追記:

```markdown

## Superseded By

[<new-title>](../accepted/<new-filename>)
```

既に `## Superseded By` セクションがある場合は **本文を上書き**（重複セクションを作らない）。

### 5. 新 ADR の更新

新 ADR ファイル（`<ROOT>/docs/adr/accepted/<new-filename>`）を **Read** で読み、**Edit** で以下を更新:

#### 5-1. Status / Decision-Date は変更しない
新 ADR は Accepted のまま。`Status` も `Decision-Date` も触らない。

#### 5-2. Supersedes セクション追記
ファイル末尾に以下を追記:

```markdown

## Supersedes

[<old-title>](../superseded/<old-filename>)
```

既に `## Supersedes` セクションがある場合は、**既存のリンクを残したまま新規行を追加**（複数の旧 ADR を 1 つの新 ADR が置き換えるケースがあるため）。重複する旧 ADR エントリは追加しない。

例（複数 supersede）:
```markdown
## Supersedes

[旧 ADR その 1](../superseded/20260101-foo.md)
[旧 ADR その 2](../superseded/20260201-bar.md)
```

### 6. ファイル移動

旧 ADR のみ移動する（新 ADR は accepted に残す）:

```
<ROOT>/docs/adr/accepted/<old-filename>
  → <ROOT>/docs/adr/superseded/<old-filename>
```

git 追跡判定: `git ls-files --error-unmatch <path>` の exit code が 0 なら `git mv`、それ以外は通常の `mv`。

### 7. 完了報告

4〜6 行で以下を伝える:
- 旧 ADR の subject と移動先（`docs/adr/superseded/<old-filename>`）
- 新 ADR の subject と現在地（`docs/adr/accepted/<new-filename>`）
- 相互リンクを張ったことの明示（旧側: `## Superseded By`、新側: `## Supersedes`）

CLAUDE.md の規約に従い自動コミットはしない。

## やってはいけないこと

- 新 ADR が proposed のまま supersede を実行する（先に `/adr:accept` させる）
- 新 ADR の Status を勝手に変更する（Accepted のまま、`## Supersedes` セクションだけ追記）
- 旧 ADR と新 ADR に同じファイルを指定する
- accepted 以外のフォルダの旧 ADR を対象にする
- 相互リンクを片方向だけ張る（必ず両方向）
- リンクパス（`../accepted/...`, `../superseded/...`）の相対パスを誤る（旧 ADR は移動先 superseded/ から見た相対、新 ADR は accepted/ から見た相対）
- 自動的に git add / git commit する

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様
- [`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) — ADR 雛形（参照のみ）
- `adr:new` — ADR の新規作成
- `adr:accept` — proposed → accepted への遷移（supersede の前提）
- `adr:deprecate` — accepted → deprecated への遷移（代替なし廃止）

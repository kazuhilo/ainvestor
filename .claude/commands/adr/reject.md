---
description: docs/adr/proposed/ の ADR を却下して rejected/ へ移動する。
model: sonnet
---

# adr:reject

ファイル移動・Status / Decision-Date 更新に加え、却下理由をヒアリングして本文末尾に追記する。

## ライフサイクル

本コマンドは `Proposed → Rejected` の遷移のみを扱う。全体仕様は [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) を参照。Rejected は終端状態。

## 実行手順

### 1. ADR ディレクトリ解決

`git rev-parse --show-toplevel` で `<ROOT>` を取得。
失敗時はユーザーに ADR を含むプロジェクトルートの絶対パスを問い合わせる。
ADR ルート: `<ROOT>/docs/adr/`

### 2. 対象ファイル特定

#### (A) `$ARGUMENTS` が空
`<ROOT>/docs/adr/proposed/*.md` を Glob で列挙:
- 0 件 → 「proposed の ADR がありません」と報告して終了
- 1 件 → そのまま採用
- 2〜4 件 → **AskUserQuestion** で選択肢提示
- 5 件以上 → 一覧をテキストで提示し、ファイル名（または部分一致キーワード）を入力するよう依頼

#### (B) `$ARGUMENTS` あり
1. `<ROOT>/docs/adr/proposed/$ARGUMENTS` がそのまま存在すれば採用
2. 拡張子 `.md` を補って `$ARGUMENTS.md` でも探す
3. 上記で見つからなければ Glob `<ROOT>/docs/adr/proposed/*$ARGUMENTS*.md` で部分一致検索
   - 0 件: 「該当する ADR が見つかりません」と一覧提示、再入力依頼
   - 1 件: 採用
   - 2〜4 件: **AskUserQuestion** で選択
   - 5 件以上: 一覧テキスト提示、再入力依頼

### 3. 却下理由ヒアリング

ユーザーに素のテキスト質問:

> 「{subject}」を却下する理由を教えてください。

#### 短すぎる場合
回答が 10 文字未満の場合は **1 回だけ** 追加質問:

> もう少し詳しく教えてください。具体的な懸念点・代替方針があれば。

それでも短い場合はそのまま採用（強制しない）。

### 4. ファイル内容更新

特定ファイルを **Read** で読み、**Edit** で以下を更新:

#### 4-1. Status 書き換え
```
- Status: Proposed
```
→
```
- Status: Rejected
```

既に Rejected の場合は変更不要（冪等）。**Proposed 以外（Accepted/Deprecated/Superseded）からの遷移は禁止**。検出したらユーザーに「この ADR は既に <状態> です」と報告して終了。

#### 4-2. Decision-Date 追記
`- Status: Rejected` の **直下** に以下を追記:
```
- Decision-Date: {今日の YYYY-MM-DD}
```

既存行があれば値を今日に更新（重複追加しない）。

#### 4-3. Rejection Reason セクション追記
ファイル末尾に以下を追記:

```markdown

## Rejection Reason

{ヒアリングで得た理由}
```

既に `## Rejection Reason` セクションがある場合は **本文を上書き**（重複セクションを作らない）。

### 5. ファイル移動

```
<ROOT>/docs/adr/proposed/<filename>
  → <ROOT>/docs/adr/rejected/<filename>
```

git 追跡判定: `git ls-files --error-unmatch <path>` の exit code が 0 なら `git mv`、それ以外は通常の `mv`。

### 6. 完了報告

3〜5 行で以下を伝える:
- 却下した ADR の subject（タイトル行から抽出）
- 移動先パス（`docs/adr/rejected/<filename>`）
- 却下理由のサマリ（1 文）

CLAUDE.md の規約に従い自動コミットはしない。

## やってはいけないこと

- 却下理由を聞かずに（または推測で）勝手に追記する
- 引数を曖昧なまま推測して却下する（候補が複数あれば必ず選択させる）
- Rejection Reason セクションを重複追加する
- Decision-Date を上書きせず複数行を追加する
- proposed 以外のフォルダのファイルを却下対象にする
- 自動的に git add / git commit する

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様
- [`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) — ADR 雛形（参照のみ）
- `adr:new` — ADR の新規作成
- `adr:accept` — proposed → accepted への遷移
- `adr:deprecate` — accepted → deprecated への遷移
- `adr:supersede` — accepted → superseded への遷移
- `adr:discard` — proposed の軽い破棄（うっかり作った／立ち消え等。本コマンドとの使い分けは `discard.md` 参照）

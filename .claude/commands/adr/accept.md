---
description: docs/adr/proposed/ の ADR を採用して accepted/ へ移動する。
model: sonnet
---

# adr:accept

ファイル移動と同時に Status / Decision-Date を Accepted に更新する。

## ライフサイクル

本コマンドは `Proposed → Accepted` の遷移のみを扱う。全体仕様は [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) を参照。

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
   - 0 件: 「該当する ADR が見つかりません。proposed 一覧は以下です:」と一覧提示、再入力依頼
   - 1 件: 採用
   - 2〜4 件: **AskUserQuestion** で選択
   - 5 件以上: 一覧をテキスト提示、再入力依頼

### 3. ファイル内容更新

特定したファイルを **Read** で読み、**Edit** で以下 2 点を更新:

#### 3-1. Status 書き換え
```
- Status: Proposed
```
→
```
- Status: Accepted
```

既に Accepted の場合は変更不要（冪等）。

**Proposed 以外（Rejected/Deprecated/Superseded）からの遷移は禁止**（lifecycle.md の禁止遷移）。検出したらユーザーに「この ADR は既に <状態> です。やり直したい場合は新規 ADR を作ってください」と報告して終了。

#### 3-2. Decision-Date 追記
`- Status: Accepted` の **直下** に以下を追記:
```
- Decision-Date: {今日の YYYY-MM-DD}
```

既に `- Decision-Date:` 行が存在する場合は **重複追加せず値を今日に更新**する。

### 4. ファイル移動

```
<ROOT>/docs/adr/proposed/<filename>
  → <ROOT>/docs/adr/accepted/<filename>
```

git 追跡判定: `git ls-files --error-unmatch <path>` の exit code が 0 なら `git mv`、それ以外は通常の `mv`。

### 5. 完了報告

3〜5 行で以下を伝える:
- 採用した ADR の subject（タイトル行から抽出）
- 移動先パス（`docs/adr/accepted/<filename>`）
- 更新した Status・Decision-Date
- 次のアクション案内: 「廃止時は `/adr:deprecate <filename>`、後継 ADR で置き換える時は `/adr:supersede <filename>`」

CLAUDE.md の規約に従い自動コミットはしない。

## やってはいけないこと

- 引数を曖昧なまま勝手に推測して採用する（候補が複数あれば必ず選択させる）
- proposed 以外のフォルダのファイルを採用対象にする（lifecycle.md の禁止遷移）
- 自動的に git add / git commit する
- Decision-Date を上書きせず複数行を追加する（重複は冪等性を壊す）

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様
- [`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) — ADR 雛形（参照のみ）
- `adr:new` — ADR の新規作成
- `adr:reject` — proposed → rejected への遷移
- `adr:deprecate` — accepted → deprecated への遷移
- `adr:supersede` — accepted → superseded への遷移

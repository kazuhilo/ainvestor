---
description: 新しい ADR を docs/adr/proposed/ に MADR ライト形式で作成する。
model: opus
---

# adr:new

ヒアリングしながら MADR ライト形式で ADR を記録する。**重要設計判断・技術選定の意思決定**を後から辿れる形で残すために使う。

## ライフサイクル

ADR 全体のライフサイクルは [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) を参照。本コマンドは **Proposed 状態のファイル作成のみ**を行う。

## 実行手順

### 1. 前提取得

#### 1-1. ADR ディレクトリの解決
`git rev-parse --show-toplevel` で `<ROOT>` を取得。
取得失敗時はユーザーに「ADR の保存先プロジェクトルートの絶対パスを指定してください」と問い合わせ。

以下 5 ディレクトリを `mkdir -p` で作成（既存ならスキップ）:
- `<ROOT>/docs/adr/proposed/`
- `<ROOT>/docs/adr/accepted/`
- `<ROOT>/docs/adr/rejected/`
- `<ROOT>/docs/adr/deprecated/`
- `<ROOT>/docs/adr/superseded/`

#### 1-2. subject の確認
引数 `$ARGUMENTS` があれば subject 原文として採用。なければユーザーに「ADR の subject（題名）は？」と素のテキスト質問。
取得した subject は **そのままタイトル本文に使う**（slug 化前の原文）。

### 2. ヒアリング

#### 2-1. 自由記述で背景収集
1 ターン目でユーザーに依頼:

> 「{subject}」の ADR について、背景・経緯・検討した案・想定影響など、思いついた範囲で自由に教えてください。順序や形式は問いません。

#### 2-2. 5 項目への振り分けと不足項目の追加質問

ユーザー回答を MADR ライトの 5 項目（[`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) 参照）に振り分け、**埋まらなかった項目だけ** 追加質問する:

| 項目 | 質問例 |
|---|---|
| Context | この決定が必要になった背景・現状の課題は？ |
| Decision | 結論としてどう決めますか？（1〜2 文で） |
| Consequences (Positive) | 採用することで得られるメリット・改善は？ |
| Consequences (Negative) | 採用によるデメリット・コスト・リスクは？ |
| Alternatives Considered | 他に検討した案／不採用にした理由は？（無ければ「なし」と答えてもらう） |
| Deciders | 決定者は？（個人名・チーム名・空欄可） |

追加質問は素のテキスト質問で行う（自由記述になりやすいため）。
「Alternatives は無い」「Deciders は未定」のような明確な二択であれば AskUserQuestion で OK。

#### 2-3. ヒアリング打ち切り
3 ターン以上ヒアリングを重ねても埋まらない項目は `TBD` で記録して先に進む。
ユーザーが「もう書いて」と指示したら即座に生成へ進む。

### 3. ファイル名生成

#### 3-1. 日付
実行日の `YYYYMMDD`（CLAUDE.md の `currentDate` 優先、無ければ `date +%Y%m%d`）。

#### 3-2. subject の slug 化（日本語そのまま許容）
- 全角/半角スペースは **除去**（`-` や `_` で結合しない。日本語タイトルは続けて書くのが自然なため）
- 以下記号類は除去: `/ \ : * ? " < > | . , （ ） ( ) [ ] 「 」 『 』 、 。 ！ ？ ・ # @ & ; ：`
- 連続する `-` は 1 個に圧縮、先頭末尾の `-` を除去（subject 原文に元から含まれるハイフンが連続した場合の整形のみ）
- ひらがな・カタカナ・漢字・英数字・ハイフンはそのまま残す

例:
- `ADR を採用する` → `ADRを採用する` (スペース除去)
- `Migrate to BOB ORM` → `MigratetoBOBORM`（英語タイトルもスペース除去するため読みづらくなる。英語主体ならハイフン区切りの subject を最初から指定すること）
- `feature-flag を導入する` → `feature-flagを導入する` (原文のハイフンは残す)

#### 3-3. ファイル名
`<YYYYMMDD>-<slug>.md`

同名ファイルが `<ROOT>/docs/adr/{proposed,accepted,rejected,deprecated,superseded}/` のいずれかに存在する場合は末尾に `-2`, `-3` ... を付ける。

### 4. ファイル作成

[`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) のテンプレートを読み込み、プレースホルダ `{...}` をヒアリング結果で置換して `<ROOT>/docs/adr/proposed/<filename>` に **Write** ツールで書き出す。

テンプレートは将来変更される可能性があるため **このコマンドファイル内にコピーせず、必ず参照先を読み込む**。

### 5. 完了報告

ユーザーに 3〜5 行で報告:
- 作成したファイルの相対パス（`docs/adr/proposed/<filename>`）
- subject
- 主な決定内容（1 文）
- 次のアクション案内: 「採用時は `/adr:accept <filename>`、却下時は `/adr:reject <filename>`」

git リポジトリの場合も、CLAUDE.md の規約に従い自動コミットはしない。

## やってはいけないこと

- ヒアリングをスキップしてテンプレートを `TBD` だけで埋める（最低 1 ターンは自由記述を聞く）
- subject の英訳・要約をユーザー無断で行う（日本語 subject は原文を採用、slug 化のみ）
- テンプレートをコマンドファイル内にハードコード（必ず `command-references/adr/` から参照）
- 新規作成時に Proposed 以外の状態にする（必ず Proposed から始める。[`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) 参照）
- 自動的に git add / git commit する

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様
- [`../../command-references/adr/madr-light-template.md`](../../command-references/adr/madr-light-template.md) — ADR 雛形
- `adr:accept` — proposed → accepted への遷移
- `adr:reject` — proposed → rejected への遷移
- `adr:deprecate` — accepted → deprecated への遷移
- `adr:supersede` — accepted → superseded への遷移（後継 ADR 指定つき）
- `adr:discard` — proposed の取り消し（Trash 移動、軽い破棄ケース用）

---
description: docs/adr/proposed/ の ADR を Trash に破棄する（軽いケース。議論を経た却下は `/adr:reject`）。
model: sonnet
---

# adr:discard

ADR を **記録を残さず破棄** する。ファイルは Trash に移動され、ADR ツリーから完全に消える（後から「やっぱり残しておけば」となる類の決定には使わない）。

## 軽い破棄ケース

以下のようなケースで使う:

- うっかり作ってしまった（テスト用、操作ミス）
- 話が立ち消えた、議論にもならず流れた
- 書く対象を間違えた（別の決定と混同していた）
- 書くべきでなかった（社外秘、機密、不適切な内容）
- 書いてみたけど、よく考えたら言わずもがなだった（当たり前すぎて記録不要）

これらは **議論を経た却下 (Rejected)** とは性質が違うため、rejected/ には残さず Trash に流す。

## discard と reject の使い分け

| 状況 | 使うコマンド | 結果 |
|---|---|---|
| 議論を経て「採用しない」と意思決定した（記録に残す価値あり） | `/adr:reject` | `docs/adr/rejected/` に残る |
| そもそも提案として成立しなかった／間違い／言わずもがな | `/adr:discard`（本コマンド） | Trash に移動、ADR ツリーから消える |

判断に迷ったら **/adr:reject** を選ぶのが安全（履歴が残る方が後悔が少ない）。

## ライフサイクル

本コマンドは **正規の状態遷移ではない** 物理削除に近い操作。[`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) の状態遷移図には載らないが、proposed 状態の **取り消し** として補助的に存在する。

**対象は proposed のみ**。accepted/rejected/deprecated/superseded のファイルは対象外（既に意思決定済みなので、後から「無かったことに」はしない）。

## 実行手順

### 1. ADR ディレクトリ解決

`git rev-parse --show-toplevel` で `<ROOT>` を取得。
失敗時はユーザーに ADR を含むプロジェクトルートの絶対パスを問い合わせる。
ADR ルート: `<ROOT>/docs/adr/`

### 2. 対象ファイル特定

**proposed フォルダのみ** から特定する。

#### (A) `$ARGUMENTS` が空
`<ROOT>/docs/adr/proposed/*.md` を Glob で列挙:
- 0 件 → 「proposed の ADR がありません」と報告して終了
- 1 件 → そのまま採用（ただし step 3 の確認は省略しない）
- 2〜4 件 → **AskUserQuestion** で選択肢提示
- 5 件以上 → 一覧をテキストで提示し、ファイル名（または部分一致キーワード）を入力するよう依頼

#### (B) `$ARGUMENTS` あり
1. `<ROOT>/docs/adr/proposed/$ARGUMENTS` がそのまま存在すれば採用
2. 拡張子 `.md` を補って `$ARGUMENTS.md` でも探す
3. 上記で見つからなければ Glob `<ROOT>/docs/adr/proposed/*$ARGUMENTS*.md` で部分一致検索
   - 0 件: 「該当する ADR が見つかりません。proposed 一覧は以下です:」と一覧提示、再入力依頼
   - 1 件: 採用
   - 2〜4 件: **AskUserQuestion** で選択
   - 5 件以上: 一覧テキスト提示、再入力依頼

**proposed 以外のフォルダ（accepted/rejected/deprecated/superseded）のファイルは対象外**。検出したらユーザーに「この ADR は既に <状態> です。discard できるのは Proposed のみ。意思決定済みの ADR は履歴として残してください」と報告して終了。

### 3. 確認

破棄は不可逆（Trash からの復元は手動操作が必要）なので、必ず **AskUserQuestion** で確認:

> 「{subject}」を破棄して Trash に移動します。記録は残りません。続行しますか？

選択肢:
- 「破棄する（Trash 移動）」
- 「やめる（何もしない）」
- 「却下扱いに変更（/adr:reject を使う）」

「やめる」「却下扱い」を選んだら本コマンドは終了し、必要なら `/adr:reject` を案内する。

理由はヒアリングしない（破棄ケースなので深掘りしない）。ただし、ユーザーが自発的に理由を述べた場合は完了報告に含める。

### 4. Trash 移動

```
<ROOT>/docs/adr/proposed/<filename>
  → ~/.Trash/<YYYYMMDD-HHMMSS>-<filename>
```

タイムスタンプを接頭辞に付けて同名衝突を回避する（複数 ADR を discard することがあるため）。

```bash
mv "<ROOT>/docs/adr/proposed/<filename>" "$HOME/.Trash/$(date +%Y%m%d-%H%M%S)-<filename>"
```

`rm` は使わない（CLAUDE.md の「ファイル削除のルール」遵守）。
git 追跡済みであっても `git rm` は使わない。Trash 移動後、git にはファイル削除として反映されるが、コミットはユーザー判断に委ねる（自動コミット禁止）。

### 5. 完了報告

2〜4 行で簡潔に:
- 破棄した ADR の subject（タイトル行から抽出）
- Trash 移動先パス
- 復元したい場合は Trash から元の場所に戻せる旨を一言

CLAUDE.md の規約に従い自動コミットはしない。git 追跡されていた場合は「git status にファイル削除として表示されているので、必要なら自分でコミットしてください」と案内。

## やってはいけないこと

- accepted/rejected/deprecated/superseded のファイルを discard する（意思決定済みなので履歴を残す）
- 確認なしに Trash 移動する（不可逆なので必ず AskUserQuestion で確認）
- `rm` / `git rm` で物理削除する（CLAUDE.md の Trash 規約遵守）
- 自動的に git add / git commit する
- 「議論の末に不要と判断した」ケースで本コマンドを使う（それは `/adr:reject` の領域）

## 関連

- [`../../command-references/adr/lifecycle.md`](../../command-references/adr/lifecycle.md) — ADR 状態遷移の共通仕様（本コマンドは正規の遷移ではない）
- `adr:new` — ADR の新規作成
- `adr:reject` — 議論を経た正式な却下（履歴を残す）

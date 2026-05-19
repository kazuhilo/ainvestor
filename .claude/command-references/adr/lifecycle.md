# ADR ライフサイクル

adr:new / adr:accept / adr:reject / adr:deprecate / adr:supersede の 5 skill が扱う ADR 状態遷移の共通仕様。各 skill から参照する。

## 状態遷移図

```
            ┌──────────┐
   /adr:new │ Proposed │ ╌╌/adr:discard (補助)╌╌▶ 🗑 Trash（履歴なし破棄）
            └────┬─────┘
                 │
       ┌─────────┴────────┐
       │                  │
 /adr:accept        /adr:reject
       │                  │
       ▼                  ▼
  ┌──────────┐       ┌──────────┐
  │ Accepted │       │ Rejected │（終端）
  └────┬─────┘       └──────────┘
       │
   ┌───┴────────────┐
   │                │
/adr:deprecate   /adr:supersede
   │                │
   ▼                ▼
┌────────────┐  ┌────────────┐
│ Deprecated │  │ Superseded │（終端、Superseded by <adr:new>）
└────────────┘  └────────────┘
   （終端）

凡例:
  ───▶  正規の状態遷移（履歴を残す）
  ╌╌▶  補助操作（正規の状態遷移ではない、履歴も残さない）
```

## 各状態の意味

| 状態 | 説明 | 配置フォルダ |
|---|---|---|
| Proposed | 提案中。議論・承認待ち | `docs/adr/proposed/` |
| Accepted | 採用済み。現役の決定 | `docs/adr/accepted/` |
| Rejected | 却下された決定 | `docs/adr/rejected/` |
| Deprecated | **代替なしで**使用をやめた決定（古くなった / 不要になった） | `docs/adr/deprecated/` |
| Superseded | **別の ADR が置き換えた**決定（後継 ADR を必ず参照） | `docs/adr/superseded/` |

## 遷移可能なエッジ

| from → to | skill | 備考 |
|---|---|---|
| (新規) → Proposed | `/adr:new` | docs/adr/proposed/ にファイル新規作成 |
| Proposed → Accepted | `/adr:accept` | docs/adr/proposed/ → accepted/ |
| Proposed → Rejected | `/adr:reject` | 却下理由を本文末尾に追記 |
| Accepted → Deprecated | `/adr:deprecate` | 廃止理由を本文末尾に追記、代替 ADR は **存在しない** |
| Accepted → Superseded | `/adr:supersede` | 後継 ADR（Accepted）を必ず指定、相互リンクを張る |

**禁止遷移**:
- Rejected / Deprecated / Superseded からの再遷移は無し（これらは終端状態）
- Proposed → Deprecated / Superseded への直接遷移は無し（必ず Accepted を経由）
- Accepted → Proposed への巻き戻しは無し（やり直したい場合は新規 ADR を作る）

## 補助操作: discard（正規の状態遷移ではない）

Proposed の ADR を **記録を残さず破棄** する補助操作として `/adr:discard` がある。

- 対象: **Proposed のみ**（accepted/rejected/deprecated/superseded は対象外）
- 操作: ファイルを Trash に移動（ADR ツリーから完全に消える、履歴も残らない）
- 用途: うっかり作った／話が立ち消えた／間違えた／書くべきでなかった／言わずもがな等の **軽い破棄ケース**
- 議論を経た正式な却下は `/adr:reject` を使う（discard ではなく rejected/ に履歴を残す）

`adr:discard` は本ライフサイクル図には載らない（状態遷移ではなく取り消し）が、proposed の取り消し手段として補助的に存在する。

## Status ヘッダ表記

ADR ヘッダの `- Status:` 行は以下のいずれか:

```
- Status: Proposed
- Status: Accepted
- Status: Rejected
- Status: Deprecated
- Status: Superseded by [<successor-filename>](../accepted/<successor-filename>)
```

`Superseded` のみ後続リンクを併記する（Markdown リンク表記）。

## Decision-Date

Proposed 以外の状態には `- Decision-Date: YYYY-MM-DD` をヘッダに追加する（Status 行の直下）。
複数回の状態遷移があっても、Decision-Date は **直近の決定日に上書き**する（履歴は git log で追う前提）。

## 本文末尾追記セクション

skill によっては本文末尾にセクションを追加する:

| skill | 追記セクション | 内容 |
|---|---|---|
| `/adr:reject` | `## Rejection Reason` | 却下理由 |
| `/adr:deprecate` | `## Deprecation Reason` | 廃止理由（代替なし） |
| `/adr:supersede` (旧 ADR 側) | `## Superseded By` | `[<新 ADR タイトル>](../accepted/<新 ADR ファイル名>)` |
| `/adr:supersede` (新 ADR 側) | `## Supersedes` | `[<旧 ADR タイトル>](../superseded/<旧 ADR ファイル名>)` |

同名セクションが既存なら **重複追加せず本文を上書き**する（冪等性）。

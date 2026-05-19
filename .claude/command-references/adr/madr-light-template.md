# MADR ライト テンプレート

adr:new で生成する ADR の雛形。プレースホルダ `{...}` をヒアリング結果で置換して `docs/adr/proposed/<YYYYMMDD>-<slug>.md` に書き出す。

ADR ライフサイクル全体は [`lifecycle.md`](lifecycle.md) を参照。

---

```markdown
# {subject 原文}

- Status: Proposed
- Date: {YYYY-MM-DD}
- Deciders: {ヒアリング結果 / 未指定なら -}

## Context

{現状の課題・背景・この決定が必要になった経緯}

## Decision

{結論として採用する方針を 1〜数文で}

## Consequences

### Positive

- {採用によって得られるメリット・改善}
- ...

### Negative

- {採用によるデメリット・コスト・リスク}
- ...

## Alternatives Considered

- {案 1}: {不採用にした理由}
- {案 2}: {不採用にした理由}
- （他に検討した案が無ければ「なし」と記載）
```

## セクション要件

| セクション | 必須 | 内容 |
|---|---|---|
| Title | 必須 | subject 原文をそのまま `# ` の後ろに |
| Status | 必須 | 新規時は `Proposed` 固定（[`lifecycle.md`](lifecycle.md) の遷移表に従う） |
| Date | 必須 | 作成日（実行日の `YYYY-MM-DD`） |
| Deciders | 任意 | 個人名・チーム名・空欄可（`-`） |
| Context | 必須 | ヒアリングで埋まらなければ `TBD` |
| Decision | 必須 | ヒアリングで埋まらなければ `TBD` |
| Consequences (Positive) | 必須 | 1 項目以上、無ければ `TBD` |
| Consequences (Negative) | 必須 | 1 項目以上、無ければ `TBD` |
| Alternatives Considered | 必須 | 無いなら「なし」と明記 |

## 状態遷移時の本文変更

各 skill が以下を変更する（詳細は [`lifecycle.md`](lifecycle.md)）:

| skill | Status 変更 | 追記セクション |
|---|---|---|
| `/adr:accept` | `Accepted` | なし |
| `/adr:reject` | `Rejected` | `## Rejection Reason` |
| `/adr:deprecate` | `Deprecated` | `## Deprecation Reason` |
| `/adr:supersede`（旧側） | `Superseded by [<新>](..)` | `## Superseded By` |
| `/adr:supersede`（新側） | （Accepted のまま） | `## Supersedes` |

いずれも `- Decision-Date: YYYY-MM-DD` を Status 行直下に追加・更新する（Proposed 以外）。

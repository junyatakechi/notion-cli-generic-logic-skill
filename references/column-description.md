# 列の説明と改名

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 対象の列と、書きたい説明・新しい名前 | 対象の属性 | ○ | 文章 | 判断できない |

## 答え

1. **列の説明は、DDLでは書けない。REST の `PATCH /data_sources/{id}` を使う。**
2. **その列の型オブジェクトを同送する。** 書き換えるつもりが無くても、`get_data_source` で現在値を読んでそのまま返す。
3. **説明はプレーンな文字列で渡し、280字以内に収める。** 長い設計メモは入らないので、要点だけ書いてスキル側へ逃がす。
4. **括弧付きの名前への改名は、この `PATCH` の `name` で行う。** プロパティIDは変わらないので、全行の値は保持される。
5. **改名でも、残したい属性（`description` など）は毎回すべて同送する。** `PATCH` は列定義の差分更新ではなく、指定した列の再定義として効く。
6. **リレーション列の説明は、Notionの画面で書く。** どちらの経路でも書けない。

```python
from notion_client import notion_request, get_data_source
body = {"properties": {"訪問件数": {
    "number": {"format": "number"},          # ← 型オブジェクトの同送が必須
    "description": "その週に…",              # ← 文字列。rich text配列だと400
}}}
notion_request("PATCH", f"/data_sources/{ds_id}", body)

# 改名
{"properties": {"平均滞在時間(h)": {"name": "平均滞在時間(h/回)",
                                   "number": {"format": "number"},
                                   "description": "…"}}}   # ← 省くと説明が消える
```

## 道具

`get_data_source`（現在の型オブジェクトの読み出し）・`notion_request("PATCH", f"/data_sources/{id}", ...)`。

## 理由

1. `notion-update-data-source` のDDLに列の説明を書く構文は無い。`ALTER COLUMN ... SET` に `description` を渡すと、列ではなくデータソース自体の説明になる。2026-09-03に3列へ書き込んで確認した。
2. 手で書くと `number_format` などを取りこぼし、気づかないまま表示形式が変わる。

| 症状 | 原因 |
|---|---|
| `body.properties.X.number should be defined` | その列の型オブジェクトを同送していない |
| `description should be a string` | rich text配列で渡した |
| `description.length should be ≤ 280` | 280字の上限を超えた |

4. DDLの `RENAME COLUMN` は、括弧付きの名前へは黙って効かない（[スキーマの変更](schema-change.md)）。
5. `name` だけ渡すと、その列の `description` が空になった（2026-09-03に実際に消して書き直した）。
6. `description` を付ける `PATCH` は、rich_text・title・number・selectでは効くが、**リレーション列では黙って無視される**（型オブジェクトを同送しても成功レスポンスが返り、読み直すと `None` のまま）。`description` だけを送ると、型オブジェクトが無いので400になる。2026-09-07に、2つのリレーション列で確認した。

## 却下した案

| 案 | 理由 |
|---|---|
| DDLの `ALTER COLUMN ... SET` で説明を書く | 理由1。データソース自体の説明になる |
| 型オブジェクトを手で書いて同送する | 理由2 |
| `name` だけを送って改名する | 理由5 |

## 当てはまる範囲と限界

- 列の意味をデータ側に残す設計を採っているなら、リレーション列だけは手作業になることを見込んでおく。

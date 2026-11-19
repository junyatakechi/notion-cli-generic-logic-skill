# 新しいデータベース

## 扱うこと

DBを作るとき、何を付け、何を画面で直すか

## 答え方と理由

### 0. 判断全体

なし

### 1. 頼まれていなくても、3列を必ず付ける

頼まれていなくても、次の3列を必ず付ける。 列名はこの英語表記で固定する。

| 列名 | 型 | RESTでの書き方 |
|---|---|---|
| `ID` | unique_id | `{"unique_id": {}}`（prefixは付けない） |
| `Created time` | created_time | `{"created_time": {}}` |
| `Last edited time` | last_edited_time | `{"last_edited_time": {}}` |

1. （根拠）どのDBでも同じ名前で辿れるようにするための決めごと。既存のDBとそろう。（決定日：記録なし）

### 2. 3列は、RESTでもMCPでも作れる

3列とも、REST の `POST /databases` でそのまま作れる。MCPの `notion-create-database` なら、DDLに `"ID" UNIQUE_ID, "Created time" CREATED_TIME, "Last edited time" LAST_EDITED_TIME` を入れる。既存のDBへ後から足すなら、`ADD COLUMN` か `PATCH /data_sources/{id}` で足す。

1. （原因）公式ドキュメントはこの3つを "read-only" と書くが、それは「ページの値を書き換えられない」という意味で、列を作ること自体は通る。（発生日：2026-09-18）
2. （根拠）ドキュメントの文言だけ見て諦めない。（決定日：記録なし）

### 3. date型の書式は、作ったあとに画面で変える

date型の列は、Date format を「Year/Month/Day」、Time format を「24 hour」にする。ただしAPIからは設定できないので、作ったあとに画面で変える。 そのことを、作った直後にユーザーへ伝える。画面で直したかは、APIからは確かめられない。

1. （原因）書けそうな経路を全部試して全滅を確認した。黙って無視されるのではなく400で弾かれる。`GET /data_sources/{id}` が返すdate列も常に `{"date": {}}` で読み取りもできないので、設定済みかどうかをAPIから確かめる術が無い。（発生日：2026-09-18）

   | 試したこと | 結果 |
   |---|---|
   | REST `PATCH /data_sources/{id}` に `date.date_format` / `date.time_format` | 400 `date_format should be not present` |
   | 同 `date.dateFormat` / `date.timeFormat` | 400 同上 |
   | 同 `date.format` | 400 同上 |
   | MCP DDL `ALTER COLUMN "Date" SET DATE FORMAT 'YYYY/MM/DD'` | 400 `Expected ADD, DROP, RENAME, or ALTER keyword, got "FORMAT"` |

## 判断定数

なし

## 未決

なし

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| DB名と、作りたい列 | 対象の値 | ○ | 文章 | 判断できない |

## 計算

なし

## 道具

`notion_request("POST", "/databases", ...)`。

答え方1・2：

```python
notion_request("POST", "/databases", {
    "parent": {"type": "page_id", "page_id": PARENT_PAGE_ID},
    "title": [{"type": "text", "text": {"content": "<DB名>"}}],
    "initial_data_source": {"properties": {
        "Name": {"title": {}},
        "<日付列>": {"date": {}},
        "ID": {"unique_id": {}},
        "Created time": {"created_time": {}},
        "Last edited time": {"last_edited_time": {}},
    }},
})
```

# 新しいデータベース

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| DB名と、作りたい列 | 対象の属性 | ○ | 文章 | 判断できない |

## 答え

1. **頼まれていなくても、次の3列を必ず付ける。** 列名はこの英語表記で固定する。

   | 列名 | 型 | RESTでの書き方 |
   |---|---|---|
   | `ID` | unique_id | `{"unique_id": {}}`（prefixは付けない） |
   | `Created time` | created_time | `{"created_time": {}}` |
   | `Last edited time` | last_edited_time | `{"last_edited_time": {}}` |

2. 3列とも、REST の `POST /databases` でそのまま作れる。MCPの `notion-create-database` なら、DDLに `"ID" UNIQUE_ID, "Created time" CREATED_TIME, "Last edited time" LAST_EDITED_TIME` を入れる。既存のDBへ後から足すなら、`ADD COLUMN` か `PATCH /data_sources/{id}` で足す。
3. **date型の列は、Date format を「Year/Month/Day」、Time format を「24 hour」にする。ただしAPIからは設定できないので、作ったあとに画面で変える。** そのことを、作った直後にユーザーへ伝える。

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

## 道具

`notion_request("POST", "/databases", ...)`。

## 理由

1. （根拠）どのDBでも同じ名前で辿れるようにするための決めごと。既存のDBとそろう。
2. （原因・根拠）公式ドキュメントはこの3つを "read-only" と書くが、それは「ページの値を書き換えられない」という意味で、列を作ること自体は通る（2026-09-18に実測）。ドキュメントの文言だけ見て諦めない。
3. （原因）2026-09-18に、書けそうな経路を全部試して全滅を確認した。黙って無視されるのではなく400で弾かれる。`GET /data_sources/{id}` が返すdate列も常に `{"date": {}}` で読み取りもできないので、**設定済みかどうかをAPIから確かめる術が無い。**

| 試したこと | 結果 |
|---|---|
| REST `PATCH /data_sources/{id}` に `date.date_format` / `date.time_format` | **400** `date_format should be not present` |
| 同 `date.dateFormat` / `date.timeFormat` | **400** 同上 |
| 同 `date.format` | **400** 同上 |
| MCP DDL `ALTER COLUMN "Date" SET DATE FORMAT 'YYYY/MM/DD'` | **400** `Expected ADD, DROP, RENAME, or ALTER keyword, got "FORMAT"` |

## 却下した案

| 案 | 理由 |
|---|---|
| "read-only" の記述を見て、3列をAPIで作るのを諦める | 理由2 |

## 当てはまる範囲と限界

- 書式を画面で直したかは、APIからは確かめられない。

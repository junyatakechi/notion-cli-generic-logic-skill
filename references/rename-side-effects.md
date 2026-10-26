# 改名の副作用

## 扱うこと

データソースやリレーションを改名するとき、何が黙って変わるか

## 答え方と理由

### 0. 判断全体

なし

### 1. データソースの改名で、アイコンが戻る

データソースを改名すると、アップロード画像のアイコンが既定のアイコンに戻る。
- 改名の前に、アイコンの値を控える。
- 改名後に確かめ、消えていたら控えた値で戻す。
- 実際の表示は、親ページで確かめる。

1. （原因）`notion-update-data-source` に `title` を渡して改名したところ、アップロード画像のアイコンが灰色の既定アイコンにリセットされた。`properties` を省くと `requires a "properties" parameter` で400になる。`notion-update-data-source` のレスポンスとデータソースの `fetch` 結果は、復元後も `/icons/emoji-neutral_gray.svg` を返し続ける。（発生日：2026-09-13）

### 2. リンクドDBの別名は、APIでは変えられない

親ページ上のリンクドDBに付けた別名は、APIでは変えられない。 DBの改名に追随させるには、画面で手で直す。

1. （原因）`update_content` は別名への置換を受け付けて成功レスポンス（page_id）を返すが、実際には何も変わらない。2度試して2度とも無変更だった。（発生日：2026-09-13）

### 3. dual relation の相手側の列名は、`RENAME COLUMN` で直す

dual relation の相手側の列名は、相手側のデータソースへ DDL の `RENAME COLUMN` を流して直す。 他の経路では直らない。相手側の既定名（`Related to A (B)` など）は括弧を含むが、括弧なしの名前へ改名する分には効く。新しい名前に括弧があると捨てられる（→ [スキーマの変更](schema-change.md)）。

1. （原因）`ADD COLUMN "X" RELATION('<ds>', DUAL)` で作ると、相手側に `Related to <DB名> (<列名>)` という名前の列ができる。試した結果は次のとおり。（発生日：2026-09-07）

   | 経路 | 結果 |
   |---|---|
   | 相手側DSへ `PATCH {"<旧名>": {"name": "<新名>", ...}}` | 400 `<新名> is not a valid property schema` |
   | 自DSの `dual_property.synced_property_name` を書き換えて `PATCH` | 無反応（成功レスポンス・値は変わらない） |
   | 相手側DSへ DDL `RENAME COLUMN "Related to A (B)" TO "A"` | 通る |

## 未決

なし

## 判断定数

なし

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 改名する対象（データソース・リンクドDBの別名・dual relationの相手側の列） | 対象の属性 | ○ | 文章 | 判断できない |

## 計算

なし

## 道具

公式MCPの `notion-update-data-source`（改名・DDL）・`notion-fetch`・`notion-update-page`。

答え方1：

- 控えるのは、`notion-fetch` で取ったアイコンの値（`attachment:<uuid>:<filename>` 形式）。
- 戻すときは、`notion-update-page` に `page_id`＝データベースIDと `icon`＝控えた文字列を渡す。`command: "update_properties"` には `properties: {}` を必ず添える。
- 表示を確かめるときは、親ページを `fetch` して `<database ... icon="...">` を見る。

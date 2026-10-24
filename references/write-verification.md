# 書き込んだあとの確かめ方と直し方

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 書き込んだもの | 対象の属性 | ○ | ページの作成か、本文の追記か。本文なら、書いた表などの数 | 判断できない |

## 答え

1. **ページを作成したら、ページを取り直して、親とプロパティを確かめる。**
2. **本文を書いたら、子ブロックの種類を数えて、書いたとおりに入ったかを別に確かめる。** 例えば、`table` の数が本文に書いた表の数と合うか。
3. **崩れていたら、ページを作り直さず、同じページのブロックを差し替える。**
4. 表の中身が正しいかは、この判断では答えない。呼び出し側が確かめる。

## 値

なし

## 計算

なし

## 道具

- 答え1：`notion_client.get_page(page_id)` で取り直す。
- 答え2：`notion_client.get_block_children(page_id, recursive=False)` で子ブロックを取り、種類を数える。

```python
from collections import Counter
import notion_client as nc
kids = nc.get_block_children(page_id, recursive=False)
print(Counter(k["type"] for k in kids))
```

- 答え3：`notion_client.notion_request("DELETE", f"/blocks/{block_id}")` で子ブロックを消してから、`append_block_children` で入れ直す。`notion_request` のパスは先頭のスラッシュが要る。
- 道具の呼び方は、[APIの呼び方](client-usage.md)に従う。

## 理由

1. （原因・知見）Notion APIは、親の指定漏れをエラーにせず、ワークスペース直下に作る。2026-07-13に、公式MCPの `notion-create-pages` で `parent` を渡し忘れて起きた。
2. （知見）ページの作成の応答やプロパティの確認では、本文の崩れ（表が段落になる、など）は分からない。Markdownからの変換も、崩れをエラーにしない（[Markdownからブロックへ変換するときの書き方](markdown-to-blocks.md)）。
3. （根拠）ページを作り直すとURLが変わり、既に共有した参照が切れる。
4. 記録なし

## 未決

なし

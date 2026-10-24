# Markdownからブロックへ変換するときの書き方

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 書く本文 | 対象の属性 | ○ | Markdown | 判断できない |

## 答え

1. **`notion_blocks.markdown_to_blocks` に渡す本文の表は、パイプ形式のMarkdownで書く。** `| … |` のヘッダー行と `|---|---|` の区切り行を持つ形。HTMLの `<table>` は使わない。
2. **セルの中では `|` をそのまま使えない。**
3. **強調は `**太字**` を使う。** `<b>` などのHTMLのタグは効かない。
4. 公式MCPで本文を書くときの落とし穴は、この判断では答えない（→ [MCPで本文を書くときのリンク化](mcp-autolink.md)）。

## 値

なし

## 計算

なし

## 道具

`scripts/notion_blocks.py` の `markdown_to_blocks(markdown_text)`。返ったブロックを `notion_client.append_block_children` か `create_page` の `children` に渡す。書いたあとの確かめ方は、[書き込んだあとの確かめ方と直し方](write-verification.md)に従う。

## 理由

1. （知見）`markdown_to_blocks` は、`|` で始まる行と区切り行の組だけを表として解釈する。HTMLの `<table>` は、エラーにならず段落として流し込まれる。
2. （知見）セルは `|` で区切って読み、エスケープを解釈しない。
3. （知見）インラインで解釈する記法は `**太字**` だけで、ほかは素通しする。
4. 記録なし

## 未決

なし

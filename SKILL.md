---
name: notion-cli-generic-logic-skill
description: "文脈: Notion。種類: 一般的な業務知識。使う場面: 名前で指名されたときだけ使い、会話の内容からは選ばない。"
metadata:
  version: "0.3.25"
  repository: "https://github.com/junyatakechi/notion-cli-generic-logic-skill"
  design: "人の判断を切り出すAgent Skillsの設計_v3.6.0"
  design-url: "https://app.notion.com/p/3eaf2fd6925e8102b86fee3b6b3dd5a3"
---

# 索引

| 判断名 | 扱うこと | ファイル | 依存する判断 | 計算 |
|---|---|---|---|---|
| スキーマの変更 | 列を追加・改名・型変更するとき、値を壊さないためにどう進めるか | [schema-change.md](references/schema-change.md) | — | — |
| selectの選択肢 | 選択肢を入れ替える・改名する・色を変えるとき、どう進めるか | [select-options.md](references/select-options.md) | — | — |
| 列の説明と改名 | 列の説明を書く・列を改名する（括弧付きの名前を含む）とき、どの経路で何を送るか | [column-description.md](references/column-description.md) | — | — |
| 新しいデータベース | DBを作るとき、何を付け、何を画面で直すか | [new-database.md](references/new-database.md) | — | — |
| 改名の副作用 | データソースやリレーションを改名するとき、何が黙って変わるか | [rename-side-effects.md](references/rename-side-effects.md) | — | — |
| APIでできないこと | 数式・ロールアップ・グループ化などを作るとき、APIで作れるか、画面で作るしかないか | [api-limits.md](references/api-limits.md) | — | — |
| MCPで本文を書くときのリンク化 | 公式MCPでページ本文を書くとき、頼んでいないリンクができるのをどう防ぐか | [mcp-autolink.md](references/mcp-autolink.md) | — | — |
| MCPで本文を書くときの崩れ | 公式MCPで本文を書く・書き替えるとき、表のセル・強調・置き換え・差し込む位置で崩れないよう、どう書くか | [mcp-write-pitfalls.md](references/mcp-write-pitfalls.md) | — | — |
| コメントの扱い | コメントに関わる作業をするとき・コメントの付いた表の行や列を増減させるとき、コメントをどう取り、誰に、どの経路で返信し、スレッドをどう守るか | [comments.md](references/comments.md) | — | — |
| 語彙DBのキャッシュ | 語彙DBを処理の中で引くとき、どう読むか | [vocabulary-cache.md](references/vocabulary-cache.md) | — | — |
| APIの呼び方 | Notionを読み書きするコードを書くとき、どの入口から呼び、何を呼び出し側で書かないか | [client-usage.md](references/client-usage.md) | — | — |
| Markdownからブロックへの変換 | `markdown_to_blocks` に本文を渡すとき、表と強調をどう書くか | [markdown-to-blocks.md](references/markdown-to-blocks.md) | — | — |
| 書き込んだあとの確かめ方 | ページを作った・本文を書いたあと、何を確かめ、崩れていたらどう直すか | [write-verification.md](references/write-verification.md) | — | — |
| 設定の不足 | 道具が `NotionAuthError`・404・403 を返したとき、公式MCPのツールが見つからないとき、どの設定の不足によるものか | [missing-setup.md](references/missing-setup.md) | — | — |

# 手続きに使う道具

道具は `scripts/` にある。関数の一覧と引数は、各モジュールの冒頭の docstring が正。コードを書く前に読む。

| モジュール | 役割 |
|---|---|
| `notion_client.py` | API通信。認証・レート制限・ページネーション・再試行・エラー |
| `notion_blocks.py` | Markdown→Notionブロック / Notionブロック→テキストの変換 |
| `notion_props.py` | プロパティ値の組み立て（`title_prop`・`date_prop` など） |
| `notion_tables.py` | ページ本文の表を、直前の見出しをキーにして読み出す |

```python
import os, sys
sys.path.insert(0, os.path.normpath(os.path.join(os.path.dirname(os.path.abspath(__file__)), "..", "..", "notion-cli-generic-logic-skill", "scripts")))
from notion_client import NotionError, query_data_source, create_page

pages = query_data_source(ds_id, {"filter": {...}})
create_page({"parent": {"data_source_id": ds_id}, "properties": {...}})
```

## 例外

| 例外 | 意味 |
|---|---|
| `NotionError` | 基底。スクリプトはこれを捕まえる |
| `NotionAuthError` | `NOTION_TOKEN` が環境変数にも起動フォルダの `.env` にも無い |
| `NotionAPIError` | APIがエラーを返した。`.status` にHTTPステータス |

## シェルから起動するスクリプト

プロジェクトのフォルダから `<skill-dir>/scripts/...` の形で起動し、スキルのフォルダへ cd しない。`NOTION_TOKEN` は環境変数、無ければ起動フォルダの `.env` から読む。結果はJSONで標準出力に出し、HTTPエラーのときは標準エラーに内容を出して0以外で終わる。

| スクリプト | 中身 |
|---|---|
| `<skill-dir>/scripts/get_page.sh <page_id>` | ページを取得する |
| `uv run <skill-dir>/scripts/get_block_children.py <block_id>` | ブロックの子を再帰で取得する。ページ本文全体なら `page_id` を渡す |
| `uv run <skill-dir>/scripts/query_data_source.py <data_source_id> [body.json]` | データソースをクエリする（ページネーション解決済み） |
| `<skill-dir>/scripts/create_page.sh <body.json>` | ページを作成する |
| `<skill-dir>/scripts/update_page.sh <page_id> <body.json>` | ページのプロパティを更新する・アーカイブする |
| `<skill-dir>/scripts/update_block.sh <block_id> <body.json>` | ブロックを更新する（本文の既存の行を書き換える） |

# このスキルの設計上の決まりごと

1. このスキルは判断を答えるだけで、判断の対象もデータも持たない。対象の値は、材料の写しとして受け取る。データを取りに行かず、保存もせず、IDも受け取らない。
2. 答える範囲は、聞きたい事を索引の「扱うこと」と照らして決める。索引の「依存する判断」にある判断を、先に出す。
3. 判断ファイルの「求める材料」がそろわなければ答えない。計算がある判断は、`uv run <skill-dir>/scripts/judge.py <判断> <材料.json>` で計算する。<判断> は `references/` のファイル名。依存する判断の材料も、一緒に渡す。
4. 答えは、指定された形にそろえ、理由と組にして返す。
5. 中核判断は、判断ファイルで答えられなければ、一般論で答えず「判断が無い」と返す。「未決」に当たったら、未決であることと、決めるまでの暫定の扱いを含める。一般的な業務知識は、判断ファイルに無いことを一般知識で補ってよい。
6. 読み込まれたら、索引の「扱うこと」に当たる作業で、その判断ファイルに従う。どの道具をどう呼ぶかは、判断ファイルの「道具」が示す。手続きは代わりに行わず、呼び出し側が道具を呼ぶ。操作する対象のIDは、材料ではなく、道具の引数で渡す。

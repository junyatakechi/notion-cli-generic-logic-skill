---
name: notion-cli-generic-logic-skill
description: "文脈: Notion。種類: 一般的な業務領域の判断。使う場面: 名前で指名されたときだけ使い、会話の内容からは選ばない。"
metadata:
  version: "0.3.16"
  repository: "https://github.com/junyatakechi/notion-cli-generic-logic-skill"
  design: "人の判断を切り出すAIスキルの設計_v1.0.0"
  design-url: "https://app.notion.com/p/jay-t/AI-3eaf2fd6925e8102b86fee3b6b3dd5a3"
  design-rule: "このスキルを作る・直すときは、design の設計文書を読み、その型と定型文に従う。"
---

# 索引

| 判断名 | ファイル | 扱うこと | 依存する判断 | 計算 |
|---|---|---|---|---|
| スキーマの変更 | [schema-change.md](references/schema-change.md) | 列を追加・改名・型変更するとき、値を壊さないためにどう進めるか | — | — |
| selectの選択肢 | [select-options.md](references/select-options.md) | 選択肢を入れ替える・改名する・色を変えるとき、どう進めるか | — | — |
| 列の説明と改名 | [column-description.md](references/column-description.md) | 列の説明を書く・列を改名する（括弧付きの名前を含む）とき、どの経路で何を送るか | — | — |
| 新しいデータベース | [new-database.md](references/new-database.md) | DBを作るとき、何を付け、何を画面で直すか | — | — |
| 改名の副作用 | [rename-side-effects.md](references/rename-side-effects.md) | データソースやリレーションを改名するとき、何が黙って変わるか | — | — |
| APIでできないこと | [api-limits.md](references/api-limits.md) | 数式・ロールアップ・グループ化などを作るとき、APIで作れるか、画面で作るしかないか | — | — |
| MCPで本文を書くときのリンク化 | [mcp-autolink.md](references/mcp-autolink.md) | 公式MCPでページ本文を書くとき、頼んでいないリンクができるのをどう防ぐか | — | — |
| コメントの扱い | [comments.md](references/comments.md) | コメントに関わる作業をするとき・コメントの付いた表の行や列を増減させるとき、コメントをどう取り、誰に、どの経路で返信し、スレッドをどう守るか | — | — |
| 語彙DBのキャッシュ | [vocabulary-cache.md](references/vocabulary-cache.md) | 語彙DBを処理の中で引くとき、どう読むか | — | — |
| APIの呼び方 | [client-usage.md](references/client-usage.md) | Notionを読み書きするコードを書くとき、どの入口から呼び、何を呼び出し側で書かないか | — | — |
| Markdownからブロックへの変換 | [markdown-to-blocks.md](references/markdown-to-blocks.md) | `markdown_to_blocks` に本文を渡すとき、表と強調をどう書くか | — | — |
| 書き込んだあとの確かめ方 | [write-verification.md](references/write-verification.md) | ページを作った・本文を書いたあと、何を確かめ、崩れていたらどう直すか | — | — |
| 設定の不足 | [missing-setup.md](references/missing-setup.md) | 道具が `NotionAuthError`・404・403 を返したとき、公式MCPのツールが見つからないとき、どの設定の不足によるものか | — | — |

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

答えるとき
1. このスキルは判断を答えるだけで、判断の対象もデータも持たない。対象の値の写しを材料として受け取って答えを返す。データを取りに行かず、保存もせず、IDも受け取らない。
2. 聞きたい事・判断先・欲しい答えの形を受け取ったら、判断先の判断ファイルの「求める材料」を返す。聞きたい事は、索引の「扱うこと」と照らして答える範囲を決める。
3. 答えは、指定された形に合わせて返す。聞きたい事が判断の一部だけなら、その部分だけを返す。
4. 答えと理由を組にして返す。理由が無いと、呼び出した側も人も答えを更新できないため。
5. 必須の材料が無ければ、答えずに足りない材料を返す。必須でない材料が無ければ、判断ファイルの「無いとき」のとおりに答え、何が無かったために何を言えなかったかを添える。
6. 判断ファイルで答えられない聞きたい事には、一般論で答えず「判断が無い」と返す。一般論はAIの一般知識で、作成者の判断ではないため。
7. 判断ファイルの「未決」に当たったら、未決であることと、決めるまでの暫定の扱いを答えに含める。
8. 索引の「依存する判断」にある判断は、先に出す。その答えを材料にするため。

判断に使う道具を使うとき
1. 計算は `uv run <skill-dir>/scripts/judge.py <判断> <材料.json>` で行う。<判断> は `references/` のファイル名（拡張子なし）。
2. 材料は、判断ファイルの「求める材料」の形にそろえたJSONで渡す。別の判断の答えを材料にする判断は、その判断の材料も一緒に渡せば、計算の中で出す。

手続きに使う道具の使い方を答えるとき
1. 手続きに関わる判断では、答えに道具の使い方（どの道具を、どう呼び、何を守るか）を含める。
2. 手続きは代わりに行わない。道具は呼び出し側が「手続きに使う道具」から自分で呼び、結果か例外を受け取る。
3. 呼び出し側は道具を写さず、自前の薄いラッパーも持たない。
4. 操作する対象のID（ページ・データソース・ブロックなど）は、判断の材料ではない。道具を呼ぶときに、呼び出し側が引数で渡す。
5. 道具は失敗を例外で返す。続けるか止めるかは、呼び出し側が選ぶ。

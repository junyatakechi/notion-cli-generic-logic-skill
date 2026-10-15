---
name: notion-cli-generic-logic
description: "文脈: Notion。種類: 一般的な業務領域の判断。使う場面: この文脈の判断を直接聞かれたときに使う。この文脈の作業の前に読み、索引の「いつ聞くか」に当たる判断を聞く。"
metadata:
  version: "0.2.1"
  repository: "https://github.com/junyatakechi/notion-cli-generic-logic"
---

# Notionの一般的な判断

## このスキルの役割

**Notionを扱うときの、一般的な判断を持つ generic-logic スキル。** Notionを使う人なら誰でも同じ答えになるが、AIの一般知識にはまだ無い判断を持つ。どれも、実際に値を壊したり、成功に見えて何も変わっていなかったりして見つけたもの。

| | このスキル |
|---|---|
| 依存してよい環境 | 誰もが共有する環境だけ（Notion REST API・公式のNotion MCP） |
| 持たないもの | 判断の対象（ページ・DB・列）と、導入先の環境。トークンはスキルの外の環境ファイルに置き、DBのIDやどのDBに何を書くかは呼び出し側が持つ |
| 持つ道具 | 手続きに使う道具だけ（APIとの通信と、値の変換）。判断に使う道具は持たない |
| 共有するとき | 混ぜて1つに育てる。新しく踏んだ落とし穴は、判断として足していく |

## 聞かれ方

他のスキルから聞かれるときは、ロジックスキル共通のやり取りで進める。

| 順 | 向き | 渡すもの |
|---|---|---|
| 1 | 呼び出し側 → このスキル | 聞きたい事・判断先（判断ファイルのパス）・欲しい答えの形 |
| 2 | このスキル → 呼び出し側 | 判断に要る材料の要求（各判断の「求める材料」） |
| 3 | 呼び出し側 → このスキル | 集めた材料 |
| 4 | このスキル → 呼び出し側 | 指定された形の答えと、その理由。手続きに関わる判断では、道具の使い方（どの道具を、どう呼び、何を守るか）も含める |

- 答えと理由を、組にして返す。
- 必須の材料が無ければ、答えずに足りない材料を返す。必須でない材料が無ければ、何が無かったために何を言えなかったかを添えて答える。
- 判断ファイルで答えられない聞きたい事には、一般論で答えず「判断が無い」と返す。新しい落とし穴を踏んだら、判断として足す。
- 判断のファイルに「未決」とある箇所に当たったら、未決であることを答えに含める。
- **このスキルの答えには、道具の使い方を含めることが多い。** 索引の「いつ聞くか」に当たったら、呼び出し側は聞きたい事・判断先・欲しい答えの形を伝えて聞き、返ってきた使い方に従って道具を自分で呼ぶ。
- **このスキルは判断して答えるだけで、手続きは代わりに行わない。** 手続きに使う道具は、呼び出し側が `scripts/` から自分で呼び、手続きの結果か例外を受け取る。
- コメントの取得・返信は、ユーザーから直接頼まれることもある（[comments.md](references/comments.md)）。

## 道具

道具は `scripts/` にある。どれも手続きに使う道具で、呼び出し側が手続きの中で使う。`notion_client.py` と `notion_tables.py` の `read_page_tables` はスキルの外を読み書きし、`notion_blocks.py`・`notion_props.py` は値の変換だけをする。**Pythonから使うなら常にimport、シェル・他言語からならスクリプトを起動する**（理由は [client-usage.md](references/client-usage.md)）。

- **呼び出し側は道具を写さず、自前の薄いラッパーも持たない。** 操作ごとの入口は `notion_client.py` に1つだけ置く。
- **操作する対象のID（ページ・データソース・ブロック）は、判断の材料ではない。** 道具を呼ぶときに、呼び出し側が引数で渡す。
- **道具で強制できる判断は、道具の中に実装してある。** 再試行する対象・件数の上限の検査などで、理由は判断ファイルに書いてある。強制できない判断（スキーマの変更を1つずつ流す、など）は、このスキルが答えとして伝え、呼び出し側が守る。
- 失敗は例外で返す。続けるか止めるかは、呼び出し側が選ぶ。

```python
import os, sys
sys.path.insert(0, os.path.normpath(os.path.join(os.path.dirname(os.path.abspath(__file__)), "..", "..", "notion-cli-generic-logic", "scripts")))
from notion_client import NotionError, query_data_source, create_page

pages = query_data_source(ds_id, {"filter": {...}})
create_page({"parent": {"data_source_id": ds_id}, "properties": {...}})
```

| モジュール | 役割 |
|---|---|
| `notion_client.py` | API通信。認証・レート制限・ページネーション・再試行・エラー |
| `notion_blocks.py` | Markdown→Notionブロック / Notionブロック→テキストの変換 |
| `notion_props.py` | プロパティ値の組み立て（`title_prop`・`date_prop` など） |
| `notion_tables.py` | ページ本文の表を、直前の見出しをキーにして読み出す |

### `notion_client.py` の操作別の関数

| 関数 | エンドポイント |
|---|---|
| `get_page(page_id)` | `GET /pages/{id}` |
| `create_page(body)` | `POST /pages` |
| `update_page(page_id, body)` | `PATCH /pages/{id}` |
| `query_data_source(ds_id, body=None)` | `POST /data_sources/{id}/query`（ページネーション解決済み） |
| `get_data_source(ds_id)` | `GET /data_sources/{id}`（スキーマ取得） |
| `select_options(ds_id, prop_name)` | select/multi_selectの選択肢名のリスト |
| `get_block_children(block_id, recursive=True)` | `GET /blocks/{id}/children`（再帰で `children` を埋め込む） |
| `append_block_children(block_id, children)` | `PATCH /blocks/{id}/children`（100件以内） |
| `update_block(block_id, body)` | `PATCH /blocks/{id}` |
| `get_comments(block_id, recursive=False)` | `GET /comments?block_id=`（未解決のみ。`recursive=True` で本文の全ブロックを辿る） |
| `create_comment(body)` | `POST /comments`（スレッドへの返信・ページ/ブロックへのコメント） |

上に無いエンドポイントは、低レベルの `notion_request(method, path, body)` / `notion_post_paginate` / `notion_get_paginate` で叩く。

| 例外 | 意味 |
|---|---|
| `NotionError` | 基底。スクリプトはこれを捕まえる |
| `NotionAuthError` | トークンファイルが無い / `NOTION_TOKEN` 未設定 |
| `NotionAPIError` | APIがエラーを返した。`.status` にHTTPステータス |

### シェルから起動するスクリプト

いずれも `~/.claude-skills-env/notion-cli-generic-logic.env` から `NOTION_TOKEN` を読み、結果をJSONで標準出力に出す。HTTPエラーのときは標準エラーに内容を出し、0以外で終了する。別のトークンを一時的に使いたいときは、環境変数 `NOTION_ENV_FILE` にファイルの絶対パスを渡す。

| スクリプト | 中身 |
|---|---|
| `scripts/get_page.sh <page_id>` | ページを取得する |
| `uv run scripts/get_block_children.py <block_id>` | ブロックの子を再帰で取得する。ページ本文全体なら `page_id` を渡す |
| `uv run scripts/query_data_source.py <data_source_id> [body.json]` | データソースをクエリする（ページネーション解決済み） |
| `scripts/create_page.sh <body.json>` | ページを作成する |
| `scripts/update_page.sh <page_id> <body.json>` | ページのプロパティを更新する・アーカイブする |
| `scripts/update_block.sh <block_id> <body.json>` | ブロックを更新する（本文の既存の行を書き換える） |

## 索引

| 判断 | ファイル | 扱うこと | 依存する判断 | いつ聞くか |
|---|---|---|---|---|
| スキーマの変更 | [schema-change.md](references/schema-change.md) | 列の追加・改名・型変更で、値を壊さないためにどう進めるか | — | 列を変える前 |
| selectの選択肢 | [select-options.md](references/select-options.md) | 選択肢を入れ替える・改名する・色を変えるとき、どう進めるか | — | 選択肢を変える前 |
| 列の説明と改名 | [column-description.md](references/column-description.md) | 列の説明を書く・列を改名するとき、どの経路で何を送るか | — | 列の説明を書く前・括弧付きの名前へ改名する前 |
| 新しいデータベース | [new-database.md](references/new-database.md) | DBを作るとき、何を付け、何を画面で直すか | — | DBを作る前 |
| 改名の副作用 | [rename-side-effects.md](references/rename-side-effects.md) | データソースやリレーションの改名で、何が黙って変わるか | — | データソース・リレーションを改名する前 |
| APIでできないこと | [api-limits.md](references/api-limits.md) | APIで作れるものと、画面で作るしかないものはどれか | — | 数式・ロールアップ・グループ化を作る前 |
| MCPで本文を書くときのリンク化 | [mcp-autolink.md](references/mcp-autolink.md) | 頼んでいないリンクができるのをどう防ぐか | — | 公式MCPでページ本文を書く前 |
| コメントの扱い | [comments.md](references/comments.md) | コメントをどう取り、誰に、どの経路で返信するか | — | コメントに関わる作業の前・コメントの付いた表を書き換える前・公式MCPで読んだ本文に `discussion-urls` が見えたとき |
| 語彙DBのキャッシュ | [vocabulary-cache.md](references/vocabulary-cache.md) | 語彙DBを処理の中で引くとき、どう読むか | — | 語彙DBを引くコードを書く前 |
| APIの呼び方 | [client-usage.md](references/client-usage.md) | どの入口から呼び、何を呼び出し側で書かないか | — | Notionを読み書きするコードを書く前 |
| Markdownからブロックへの変換 | [markdown-to-blocks.md](references/markdown-to-blocks.md) | `markdown_to_blocks` に渡す本文の表と強調をどう書くか | — | `markdown_to_blocks` に本文を渡すコードを書く前 |
| 書き込んだあとの確かめ方 | [write-verification.md](references/write-verification.md) | 作成・追記のあとに何を確かめ、崩れていたらどう直すか | — | ページを作る・本文を書くコードを書く前・崩れを直す前 |

## 依存の順

判断どうしの依存（ある判断の答えを、別の判断の材料にすること）は無い。どの判断から聞いてもよい。関連する判断へのリンクは、各ファイルの本文にある。

## 判断ファイルの形

判断を足すとき・直すときは、この形を守る。

| 項目 | 書くこと |
|---|---|
| 求める材料 | 呼び出し側に求めるもの。列は「材料・種類・必須・形・無いとき」。判断の対象は、Notionでする作業。種類は `対象の属性`（作業の中身）/ `対象の記録` / `別の判断` / `導入先の環境`（判断が環境に依るときだけ）。操作する対象のIDは入れない |
| 答え | 判断そのもの |
| 値 | 作成者が決めた値だけを、`| 名前 | 値 | 単位 |` の表で書く。無ければ省く |
| 道具 | 道具の使い方。どの道具を、どう呼び、何を守るか（`scripts/` の道具か、公式MCPのツールか） |
| 理由 | なぜそう判断するか。実際に踏んだ日付と、起きたことを残す |
| 却下した案 | 選ばなかった案と、その理由。記録が無ければ「記録なし」と書く |
| 当てはまる範囲と限界 | どこまで使えるか |

- **導入先の環境は書かない。** トークン・DBのID・どのDBに何を書くかは、呼び出し側か環境ファイルが持つ。
- **環境の仕様に合わせた値は、道具のコードの定数が正。** レート制限・再試行・APIのバージョンのような値は、人が決めるものではないので判断ファイルに値の表として置かない。判断の本文から「`notion_client.py` の定数が正」と指す。
- **道具は値の表を読まない。** 多くのスキルが読み込むたびにimportするので、表の書き損じ1つで、使うすべてのスキルが止まるため。作成者が決めた値が道具に要るときは、コードに定数として置き、決めた理由を判断ファイルに書く。

## 設定が無いとき → README の「セットアップ」

トークンは導入先の環境なので、スキルの外に置く。Integrationの作成・ページへの接続・トークンの保存は人の手作業で、手順は README の「セットアップ」だけが持つ。次の合図が出たら先へ進まず、README の「セットアップ」をユーザーに案内する。

| 合図 | 足りないもの |
|---|---|
| `NotionAuthError`（スクリプトでは `Notion token file not found` / `NOTION_TOKEN is not set`） | `~/.claude-skills-env/notion-cli-generic-logic.env`、またはその中の `NOTION_TOKEN` |
| あるはずのページ・データソースで `NotionAPIError`（`.status` が404） | そのページへのIntegrationの接続 |
| コメントの取得・返信で403、または中身の無い結果 | Integrationのコメント権限（[comments.md](references/comments.md)） |

## 未決

無し。

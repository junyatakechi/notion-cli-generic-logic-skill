# MCPで本文を書くときのリンク化

## 答え方

1. **ファイル名・パス・ドメインは、段落でも見出しでも表のセルでも、必ずコード表記（`` ` `` で囲む）で書く。** 拡張子が国別ドメインと同じものは特に注意する（`.md`・`.ai` は実測。`.py`＝パラグアイ・`.sh`＝セントヘレナ・`.io` なども同じ形だが未実測）。
2. **リンクにしたい実在のドメインは `[claude.ai](https://claude.ai)` と明示して書き、前後の日本語とは半角スペースで区切る。**
3. **書いたら `notion-fetch` で読み直し、`](http://` を探す。** 意図しないリンクがあれば、`update_content` の `old_str` に**fetchで見えるリンク込みの文字列**をそのまま渡し、コード表記へ置き換える。同じ文字列が複数あるなら `replace_all_matches: true`。
4. **`.` をエスケープしてリンク化を防ごうとしない。**
5. RESTの経路（`notion_blocks.py` → `append_block_children` など）では、この判断は当てはまらない見込み（未決 U1）。ただし `notion_blocks.py` は `` ` `` をコード表記に変換しない（`**太字**` 以外は素通し）ので、REST経由で書くと `` ` `` がそのまま文字として残る。

## 理由

1. （原因）`notion-update-page`（`update_content` など）や `notion-create-pages` で本文をMarkdownとして書くと、ドメイン名に見える語が、頼んでいないリンクに変わる。エラーも警告も出ず、レスポンスは成功のまま（2026-09-23に、あるページへ節を書き足したときに発生）。`.md`（モルドバ）・`.ai`（アンギラ）は実在する国別ドメインなので、`CLAUDE.md`・`SKILL.md`・`claude.ai` がWebアドレスとして扱われる。日本語は語の間にスペースが無いので、直前の日本語までリンクに飲み込まれる。

| 書いたもの | 保存されたもの |
|---|---|
| 組織向けCLAUDE.md | `[組織向けCLAUDE.md](http://組織向けCLAUDE.md)` |
| 6〜8のCLAUDE.md同士 | `6〜[8のCLAUDE.md](http://8のCLAUDE.md)同士` |
| auto memory（MEMORY.md） | `auto memory（[MEMORY.md](http://MEMORY.md)）` |
| `` `CLAUDE.md` ``（コード表記） | **そのまま。リンクにならない**（同じ書き込みの中での対照） |

2. （原因）明示したリンクは、書いたとおりに保存された（2026-09-23に確認）。
3. （原因）既存のページにも、`またはclaude.ai` が丸ごと `http://またはclaude.ai` へのリンクになった例があった。
4. （知見）Notion-flavored Markdownの仕様（`notion://docs/enhanced-markdown-spec`）にこの挙動の記載は無く、エスケープ対象の文字にも `.` は無い。防げると確かめたのはコード表記だけ。
5. （知見）RESTのrich textは `text.link` を明示したものだけがリンクになり、`notion_blocks.py` はリンクを付けない。

## 未決

- U1 RESTの経路で、頼んでいないリンクができないかは実測していない。決めるまでは、起きない見込みとして扱う。

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 書く本文 | 対象の属性 | ○ | Markdown | 判断できない |

## 値

なし

## 計算

なし

## 道具

公式MCPの `notion-update-page`・`notion-create-pages`・`notion-fetch`。

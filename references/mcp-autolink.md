---
sources:
  S1:
    name: "Notion-flavored Markdown の仕様（`notion://docs/enhanced-markdown-spec`）"
    checked: "記録なし"
---

# MCPで本文を書くときのリンク化

## 扱うこと

公式MCPでページ本文を書くとき、頼んでいないリンクができるのをどう防ぐか

## 答え方と理由

### 0. 判断全体

なし

### 1. ファイル名・パス・ドメインはコード表記で書く

ファイル名・パス・ドメインは、段落でも見出しでも表のセルでも、必ずコード表記（`` ` `` で囲む）で書く。 拡張子が国別ドメインと同じものは特に注意する（`.md`・`.ai` は実測。`.py`＝パラグアイ・`.sh`＝セントヘレナ・`.io` なども同じ形だが未実測）。

1. （原因）`notion-update-page`（`update_content` など）や `notion-create-pages` で本文をMarkdownとして書くと、ドメイン名に見える語が、頼んでいないリンクに変わる。エラーも警告も出ず、レスポンスは成功のまま。あるページへ節を書き足したときに起きた。`.md`（モルドバ）・`.ai`（アンギラ）は実在する国別ドメインなので、`CLAUDE.md`・`SKILL.md`・`claude.ai` がWebアドレスとして扱われる。日本語は語の間にスペースが無いので、直前の日本語までリンクに飲み込まれる。（発生日：2026-09-23）

| 書いたもの | 保存されたもの |
|---|---|
| 組織向けCLAUDE.md | `[組織向けCLAUDE.md](http://組織向けCLAUDE.md)` |
| 6〜8のCLAUDE.md同士 | `6〜[8のCLAUDE.md](http://8のCLAUDE.md)同士` |
| auto memory（MEMORY.md） | `auto memory（[MEMORY.md](http://MEMORY.md)）` |
| `` `CLAUDE.md` ``（コード表記） | そのまま。リンクにならない（同じ書き込みの中での対照） |

### 2. リンクにしたいドメインは明示して書く

リンクにしたい実在のドメインは `[claude.ai](https://claude.ai)` と明示して書き、前後の日本語とは半角スペースで区切る。

1. （原因）明示したリンクは、書いたとおりに保存された。（発生日：2026-09-23）

### 3. 書いたら読み直して直す

書いたら `notion-fetch` で読み直し、`](http://` を探す。 意図しないリンクがあれば、`update_content` の `old_str` にfetchで見えるリンク込みの文字列をそのまま渡し、コード表記へ置き換える。同じ文字列が複数あるなら `replace_all_matches: true`。

1. （原因）既存のページにも、`またはclaude.ai` が丸ごと `http://またはclaude.ai` へのリンクになった例があった。（発生日：記録なし）

### 4. `.` をエスケープしない

`.` をエスケープしてリンク化を防ごうとしない。

1. （知見）Notion-flavored Markdownの仕様（`notion://docs/enhanced-markdown-spec`）にこの挙動の記載は無く、エスケープ対象の文字にも `.` は無い。防げると確かめたのはコード表記だけ。（出典：S1）

### 5. RESTの経路では当てはまらない見込み

RESTの経路（`notion_blocks.py` → `append_block_children` など）では、この判断は当てはまらない見込み（未決 U1）。ただし `notion_blocks.py` は `` ` `` をコード表記に変換しない（`**太字**` 以外は素通し）ので、REST経由で書くと `` ` `` がそのまま文字として残る。

1. （知見）RESTのrich textは `text.link` を明示したものだけがリンクになり、`notion_blocks.py` はリンクを付けない。（出典：なし）

## 判断定数

なし

## 未決

- U1：RESTの経路で、頼んでいないリンクができないかは実測していない。暫定の扱い：起きない見込みとして扱う。

## 求める材料

| 材料 | 種類 | 必須 | 形 | 無いとき |
|---|---|---|---|---|
| 書く本文 | 対象の値 | ○ | Markdown | 判断できない |

## 計算

なし

## 道具

公式MCPの `notion-update-page`・`notion-create-pages`・`notion-fetch`。

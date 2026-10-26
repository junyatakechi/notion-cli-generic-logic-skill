# notion-cli-generic-logic-skill

Notion を Claude から読み書きするときの、一般的な判断を持つロジックスキル。
Notion が黙って値を壊す落とし穴とその避け方、コメントの扱いを答え、Notion REST API を呼ぶ道具を部品として渡す。

## 必要なもの

- [Claude Code](https://docs.claude.com/en/docs/claude-code/overview)
- Python 3.11以上と [uv](https://docs.astral.sh/uv/)
- `sh` と `curl`（WindowsではGit Bash）
- Notion の Internal Integration
- Claude に接続した公式の [Notion MCP](https://developers.notion.com/docs/mcp)（コメントに答えてもらうときだけ）

## インストール

```bash
git clone https://github.com/junyatakechi/notion-cli-generic-logic-skill.git ~/.claude/skills/notion-cli-generic-logic-skill
```

特定のプロジェクトだけで使うなら、`<プロジェクト>/.claude/skills/` の中でクローンする。

## セットアップ

### Pythonとuvを入れる

Windowsでは、[winget](https://learn.microsoft.com/windows/package-manager/winget/)で入れる。

```bash
winget install --id Python.Python.3.14 --exact
```

```bash
winget install --id astral-sh.uv --exact
```

macOS・Linuxでは、[uvの公式インストーラー](https://docs.astral.sh/uv/getting-started/installation/)でuvを入れ、uvでPythonを入れる。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash
uv python install 3.14
```

入れたらターミナルを開き直し、`uv --version` でバージョンが表示されるのを確かめる。

### Notionの準備

1. [Notionのインテグレーション画面](https://www.notion.so/profile/integrations)で Internal Integration を作り、トークン（`ntn_` で始まる。2024-09-25より前に発行したものは `secret_` で始まる）を控える。
2. 読み書きしたいページやデータベースを開き、右上の「接続」からこのIntegrationを追加する。追加していないページはAPIから見えず、404になる。
3. `.env.example` のキーを、プロジェクトのフォルダ直下（Claude Code を起動するフォルダ）の `.env` に足し、控えたトークンを書く。キーの中身は `.env.example` のコメントにある。`.env` はプロジェクトのスキルで共有する。

### コメントに答えてもらうときの準備

コメントを拾う・返信するときだけ要る。

1. [Notionのインテグレーション画面](https://www.notion.so/profile/integrations)で、Integrationの「Read comments」と「Insert comments」を有効にする。既定はオフで、オフのままだと403になるか、中身の無い結果が返る。
2. 公式の [Notion MCP](https://developers.notion.com/docs/mcp) をClaudeに接続する。

## 使い方

主に、アプリケーションスキルから呼ばれる。
直接頼むときも、`/notion-cli-generic-logic-skill` で名前を指名する。指名しないと、このスキルは読み込まれない。

- 「/notion-cli-generic-logic-skill このDBの列名を変えて <URL>」
- 「/notion-cli-generic-logic-skill このページのコメントを拾って <URL>」
- 「/notion-cli-generic-logic-skill このページのコメントに返信して <URL>」

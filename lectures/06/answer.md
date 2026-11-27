# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：
  ```text
  [tool call]   glossary_search_glossary({'query': '3レベル'})
  ```
- STEP 3 の [tool call]：
  ```text
  [tool call]   fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
  ```
- STEP 4 の [tool call]：
  ```text
  [tool call]   aws_aws___search_documentation({'search_phrase': 'Amazon S3 versioning overview'})
  ```
- 気づき・疑問：設定ファイルに接続先を追加するだけで、エージェントが使えるツールが増えた。公開されているサーバーでも、自分の環境で起動するものと、外部のURLに接続するものがあると分かった。ツールが増えたとき、モデルがどの説明を見て使い分けるのかが気になった。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：発展（基礎・実装にも取り組んだ）
- やったこと：
  - 基礎：`app/mcp_server.py` に `server.tool()(tools.get_lecture_summary)` を追加した。エージェント側のコードや設定を変えずに、ツール一覧と質問時の呼び出しを確認した。
  - 実装：第3回で作成した `make_quiz` を `server.tool()(tools.make_quiz)` で公開した。既存の「対象の回の用語がない場合にメッセージを返す処理」を使い、第3回と第99回を質問して比較した。
  - 発展：Claude Codeの公式ドキュメントを調べ、講義の `app/mcp_servers.json` と、設定場所・コマンド起動・URL接続・有効範囲を比較した。今回は設定方法の調査までで、Claude Codeへの登録・接続は実行していない。

### 発展：Claude Codeとの設定方法の比較

調査日：2026年11月27日

参照した公式ドキュメント：[Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp)

| 比較項目 | 講義のエージェント | Claude Code |
|---|---|---|
| 設定を書く場所 | `app/mcp_servers.json` に書き、`app/mcp_agent.py` が読み込む。 | プロジェクトで共有する設定はルートの `.mcp.json`。個人用のlocal・user設定は `~/.claude.json` に保存される。`claude mcp add` コマンドでも登録できる。 |
| コマンドで起動する書き方 | `mcpServers` の各サーバーに `command` と `args` を指定する。 | `.mcp.json` でも `mcpServers` 内に `command` と `args` を指定できる。コマンドでは `claude mcp add --transport stdio --scope project glossary -- uv run python -m app.mcp_server` のように、`--` の後ろに起動コマンドを書く。 |
| URLで接続する書き方 | `"aws": {"url": "https://knowledge-mcp.global.api.aws"}` と書く。 | JSONでは `"type": "http"` と `url` を指定する。登録コマンドの例は `claude mcp add --transport http --scope project aws https://knowledge-mcp.global.api.aws`。 |
| 有効になる範囲 | この設定ファイルを読む講義エージェントで使われる。 | local（既定）は現在のプロジェクトで自分だけ、projectは現在のプロジェクトで設定を共有、userは自分の全プロジェクトで使う。`--scope` で選択できる。 |

上のコマンドは公式の構文に講義の接続先を当てはめた記述例で、実行結果ではない。stdioの例を使う場合は、実行する環境にuvと講義のコードが必要になる。

- 結果：
  - 基礎：`glossary_get_lecture_summary` が一覧に増え、`[tool call]   glossary_get_lecture_summary({'lecture': 2})` と、第2回の要点の返却を確認した。テストは21件成功した。
  - 実装：`glossary_make_quiz` が一覧に増えた。第3回では `[tool call]   glossary_make_quiz({'lecture': 3})` が呼ばれ、問題と答えが返った。第99回では `[tool call]   glossary_make_quiz({'lecture': 99})` が呼ばれ、「第99回の用語が見つからないため、確認問題を作れません。」が返った。モデルも作れない旨を回答し、テストは21件成功した。
  - 発展：講義とClaude Codeで、手元のコマンドを起動する方法と、外部のURLに接続する方法が共通していると分かった。一方、設定の保存場所や有効範囲、HTTP接続時の `type` 指定には違いがあり、講義の設定をそのままコピーすればよいわけではないと分かった。
- 気づき・疑問：
  - 普通のPython関数を少ない追加コードでMCPのツールとして公開できる点が便利だと感じた。関数の説明文や型ヒントも、モデルに使い方を伝える役割があると分かった。
  - Claude Codeでは、個人用の設定とプロジェクトで共有する設定を分けられるので、リポジトリに設定を残したい場合と、自分だけで使いたい場合に使い分けられそうだと思った。
  - 第99回の最終回答には、ツールが返していない「第1回〜第15回」という範囲も付け足されていた。ツールの結果が正しくても、モデルの回答に根拠のない補足が入る場合があり、`tool return` と最終回答の両方を確認する必要があると感じた。

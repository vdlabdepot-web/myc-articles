---
title: Claude Code が「決めたこと」を忘れない：完全ローカル・API キー不要のエージェント記憶レイヤー myc
tags:
  - ClaudeCode
  - AI
  - Bun
  - MCP
  - OSS
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

Claude Code や Codex を 1 時間以上使うと、ほぼ必ず起きること。最初に「A 案で行く、B 案は理由 C で却下」と決めたのに、コンテキストが圧縮された後、エージェントが平然と B 案を提案してくる。理由はもう彼のコンテキストに存在しないからです。

GitHub で見つけた **myc**（mycelium＝菌糸から）は、まさにこの問題に特化した OSS です。クラウドの記憶 API ではなく、プロジェクトの隣に置く SQLite 1 ファイルのローカルなタスク＋記憶レイヤー。

### できること

- **タスクキュー**：依存関係・ブロッカー・アトミックな claim。複数エージェントが同じタスクを取り合わない。
- **決定ログ**：「なぜ A で B ではないか」をノードとして保存し、ハイブリッド検索（BM25 + ベクトル）。
- **PreCompact フック**：Claude Code がコンテキストを圧縮する直前にセッション全体をディスクへ保存（秘密情報はマスク）し、生き残るコンテキストに「救援パケット」を書き戻す。
- **候補であって事実ではない**：会話から自動抽出した「決定」は人が確認するまで検索結果に出ない。嘘をつく記憶は、記憶がないより危険という設計思想。
- **コードにアンカーされた記憶**：知識はコードの特定の範囲に紐づき、リファクタで移動しても追従。コードが消えれば `[code gone ×0.2]` と表示して降格し、古い情報を事実として渡さない。
- **内蔵コードインデックス**：tree-sitter、TypeScript / JavaScript / Python。

### 速度は制約

10 万ノード、p99：コンテキストパケット 0.6 ms（予算 30 ms）、ハイブリッド検索 8.2 ms、コールドスタート 21 ms。`bun run scripts/bench-latency.ts` で再現でき、サイトのビルド時に数字が計測結果と食い違うとビルドが落ちる仕組み。テスト 3 700 本超。

### 正直な制約

- **Bun 専用**（`bun:sqlite` + `sqlite-vec`）。Node / Deno では起動しない。
- macOS / Linux。Windows は WSL 経由。
- 現状はシングルユーザーのローカルツール。サーバー・チーム機能・ACL はロードマップ上のみ。

### インストール

```bash
bun install -g @aistastudio/myc
cd your-project
myc init
myc wire   # Claude Code / Codex / opencode / Kimi を 1 コマンドで接続
```

リポジトリ：https://github.com/aistastudio/myc
サイト（機能・ロードマップ・再現可能な計測、英/露）：https://aistastudio.github.io/myc/

公開されたばかりで毎日リリースが出ています。フィードバック歓迎とのこと。
